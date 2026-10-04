---
name: common-project-structure-using-scriptcollection-onboarding
description: Describes how to migrate ("onboard") an existing repository so that it implements the "common project structure" and uses ScriptCollection as the automation behind it. Use this skill when a repository does not yet have the common project structure (no "<repository>/.ScriptCollection/ProductInformation.xml", no codeunits, no "<codeunit>/Other/..."-scripts) and should be converted to it. For working with a repository which already implements the structure, use the skills "common-project-structure" and "common-project-structure-using-scriptcollection" instead.
metadata:
  purpose: "Migration of repositories to the common project structure."
  tags: migration, onboarding, conventions, automation, scriptcollection
  version: 1.0.0
---

# Onboarding a repository to the common-project-structure-using-scriptcollection

## Goal

This skill describes how to take an **existing** repository and restructure it so that:

1. it conforms to the **common project structure** (the conventions themselves), and
2. its automation is implemented **using ScriptCollection** (the thin python-scripts delegate to `ScriptCollection.TFCPS`).

This process is called *onboarding* (or *migration*) here.

The two topics this skill builds on are described in separate skills, and both of them should be read before starting:

- `common-project-structure` — what the structure is (codeunits, which scripts must exist, the defined source-files like `codeunit.xml` and `ProductInformation.xml`, how a codeunit may depend on another).
- `common-project-structure-using-scriptcollection` — the concrete automation: the `sc...`-commands (most importantly `scbuildcodeunits`), the `Taskfile.yml`, which files are generated automatically.

This skill is the *procedure* which uses the knowledge of those two.

## The single migration-target-criterion

The onboarding is **finished** exactly when the following command exits with **exit-code 0**:

```text
scbuildcodeunits -c -r <repo>
```

where `<repo>` is the absolute path to the repository.

- `-c` runs the whole pipeline inside the standardized container, so a green run does not depend on the local machine.
- `-r <repo>` selects the repository to build.

Nothing else defines "done": as long as this command exits non-zero, the migration is not complete; once it exits 0, the repository implements the structure correctly (all codeunits build, lint, test, generate their reference, and the pipeline is idempotent).

Therefore the whole onboarding is an iteration: change the structure, run `scbuildcodeunits -c -r <repo>`, read the error, fix the cause, repeat — until the exit-code is 0.

While iterating, `-v 4` (higher verbosity) makes a failing run easier to diagnose, and `-f`/`--fastlane` (skips the testcases) can be used for a faster intermediate check — but the final, deciding run that proves the migration is finished must be the full `scbuildcodeunits -c -r <repo>` (without `-f`) and must exit 0.

## Look at the examples first

Do **not** assemble the required files from memory. There is an official repository with one minimal example-codeunit per technology and project-type:

- Locally (if available): `E:\Data\Projects\CommonProjectStructureExamples`
- Online: https://github.com/anionDev/CommonProjectStructureExamples (note: user `anionDev`, not the organization `anionDevelopment`)

For every codeunit to be created, pick the example whose technology and type matches (for example `DotNetLibraryCodeUnit`, `DotNetConsoleCodeUnit`, `DotNetWebAPICodeUnit`, `DotNetWebAPIClientCodeUnit`, `AngularCodeUnit`, `PythonCodeUnit`, `MavenCodeUnit`, `GoCodeUnit`, `RustCodeUnit`, `DartLibraryCodeUnit`, `CPPLibrary`, `CExecutable`, `CombinedBackendFrontendCodeUnit`, or the image-only `WebAPIContainerCodeUnit` / `WebAppContainerCodeUnit`), copy its structure and adapt names and content.

The examples are minimal on purpose: copy their **shape**, not their content.

## Target structure

The following is the concrete target. All paths are relative to `<repository>`.

### Repository-root overhead

| Path                                         | Hand-authored / generated | Purpose                                                                                         |
| -------------------------------------------- | ------------------------- | ----------------------------------------------------------------------------------------------- |
| `.ScriptCollection/ProductInformation.xml`   | hand-authored             | Product-level metadata (see below). Its existence is what marks a repo as using this structure. |
| `.ScriptCollection/OCIImages/ImageDefinition.csv` | hand-authored        | Pinned OCI-images the pipeline/test-services use (only if container-images are needed).          |
| `.ScriptCollection/.gitignore`               | hand-authored             | Ignores `/Cache/`.                                                                               |
| `<repository>.code-workspace`                | hand-authored             | The source of truth for the tasks (`tasks`-section with `aliases`); references the repo as one folder. |
| `Taskfile.yml`                               | **generated**             | Generated from the `.code-workspace`-file during the build. Never edit by hand.                  |
| `ReadMe.md`                                  | hand-authored             | Product overview + codeunit-table.                                                               |
| `License.txt`                                | hand-authored             | License.                                                                                         |
| `Contributing.md` / `ContributorLicenseAgreement.txt` | hand-authored    | Contribution-rules + CLA (optional, but present in the examples).                                |
| `.gitignore`, `.gitattributes`, `.pylintrc`, `.betterleaks.toml` | hand-authored | Repo-level config.                                                                     |
| `Other/`                                     | mixed                     | Repository-level resources (changelog, scripts, local test-services, …); see below.             |
| `Other/Scripts/PrepareBuildCodeunits.py`     | hand-authored (thin)      | Repo-level pre-build hook (delegates to `ScriptCollection.TFCPS.TFCPS_Generic`).                 |
| `Other/Resources/Changelog/v<version>.md`    | hand-authored             | One changelog-file per version.                                                                  |
| `Other/Reference/RepositoryStructure.md`     | hand-authored             | Documents what a codeunit is and the script-run-order.                                           |
| one folder per codeunit (`<codeunit>/`)      | mixed                     | See "Each codeunit".                                                                             |

AI-tooling folders (`.agents/`, `.claude/`, `.github/`, `.vscode/`, …) are present in the examples; create them only if the repository actually uses the corresponding tooling. The skills under `.agents/skills` are the single source of truth and are synchronized into the other agents' folders automatically by the build (see `common-project-structure-using-scriptcollection`).

`.ScriptCollection/ProductInformation.xml` has this shape:

```xml
<?xml version='1.0' encoding='utf-8'?>
<productinformation xmlns="https://projects.aniondev.de/PublicProjects/Common/ProjectTemplates/-/tree/main/Conventions/RepositoryStructure/CommonProjectStructure">
  <producttitle>YourProductName</producttitle>
  <remoteaddress>https://github.com/anionDev/YourProductName</remoteaddress>
  <requiredenvironmentvariables />
</productinformation>
```

It must be valid against `productinformation.xsd` (referenced by URL in the schema, not stored in the repository).

### Each codeunit

A codeunit lives in `<repository>/<codeunit>/` and is recognizable by `<codeunit>/<codeunit>.codeunit.xml`. Its layout:

```text
<codeunit>/
├── <codeunit>.codeunit.xml          # the codeunit-manifest (valid against codeunit.xsd)
├── ReadMe.md                        # codeunit-level readme
├── <codeunit>/                      # the productive source-code
├── <codeunit>Tests/                 # the testcases (mandatory if the codeunit has testable source-code)
└── Other/
    ├── CommonTasks.py               # preparation-tasks (version, dependent codeunits, …)
    ├── UpdateDependencies.py        # updates the codeunit's dependencies (only if it has updatable ones)
    ├── Build/
    │   └── Build.py                 # builds the productive artifact
    ├── QualityCheck/
    │   ├── Linting.py               # runs the linting
    │   └── RunTestcases.py          # runs the testcases + coverage-check (only if testable source-code)
    ├── Reference/
    │   ├── GenerateReference.py     # generates the html-reference
    │   └── ReferenceContent/        # Hints.md, HowToBuild.md, articles, images, …
    ├── Resources/                   # Constants, badges, … (mostly generated)
    └── Artifacts/                   # build-output, entirely generated and git-ignored
```

The exact set of technology-specific extra files (for .NET: `.sln`, `.csproj`, `runsettings.xml`, `Other/Build/*.nuspec`, `Other/Reference/docfx.json`, …) differs per technology — take them from the matching example.

### The scripts are thin wrappers

The python-scripts of a codeunit must be at exactly those paths and names, must be python, and must only **delegate** to `ScriptCollection.TFCPS`. They contain no business-logic. For a .NET-codeunit they look like this (one method-call each):

```python
from ScriptCollection.TFCPS.DotNet.TFCPS_CodeUnitSpecific_DotNet import TFCPS_CodeUnitSpecific_DotNet_Functions,TFCPS_CodeUnitSpecific_DotNet_CLI


def build():
    tf:TFCPS_CodeUnitSpecific_DotNet_Functions=TFCPS_CodeUnitSpecific_DotNet_CLI.parse(__file__)
    tf.build(False)


if __name__ == "__main__":
    build()
```

For another technology, the `.DotNet.` segment and the class-names are replaced by that technology's variant. Always copy the concrete scripts from the matching example instead of writing them yourself — this guarantees the correct method-calls and arguments.

`<codeunit>/<codeunit>.codeunit.xml` must be valid against `codeunit.xsd` and declares `name`, `version`, owner, `developerteam`, the `properties` (for example `codeunithastestablesourcecode`, `codeunithasupdatabledependencies`, `testsettings/@minimalcodecoverageinpercent`, `pipelinedemands`) and `dependentcodeunits`. Use the same `codeunitspecificationversion` as the current examples. Copy it from the matching example and adapt it.

## Migration procedure

Work top-down and verify continuously with the migration-target-criterion.

1. **Read the prerequisites.** Read the skills `common-project-structure` and `common-project-structure-using-scriptcollection`, and open the matching example(s) in `CommonProjectStructureExamples`. Make sure ScriptCollection is installed (`pip install ScriptCollection`) and that `scbuildcodeunits --help` works.

2. **Identify the codeunits.** Decide how the existing code maps onto codeunits. The usual case is one codeunit; split into several only if there are genuinely independently-buildable parts. For each codeunit decide its technology/type and the matching example.

3. **Create the repository-level overhead.** Create `.ScriptCollection/ProductInformation.xml` (adapt `producttitle`, `remoteaddress`), the `<repository>.code-workspace` (with at least the `Base: Build all codeunits` task, alias `bb`), the repo-level `Other/` (at least `Other/Resources/Changelog/`, `Other/Reference/RepositoryStructure.md`, and `Other/Scripts/PrepareBuildCodeunits.py` if the examples' one applies), and the config-files (`.gitignore`, `.pylintrc`, `.betterleaks.toml`, …). Do **not** create `Taskfile.yml` by hand — it is generated.

4. **Move each codeunit into place.** For every codeunit:
   - Create the folder `<repository>/<codeunit>/`.
   - Move the productive source-code into `<repository>/<codeunit>/<codeunit>/` and the tests into `<repository>/<codeunit>/<codeunit>Tests/`. Use `git mv` so the history is kept.
   - Copy the `Other/`-subtree and the `<codeunit>.codeunit.xml` from the matching example, then adapt all names (folder-names, namespaces, project-names, the `name`/`version`/owner/description in `codeunit.xml`).
   - Fix the paths that the technology-specific files assume (for .NET: `OutputPath`, `DocumentationFile`, `ErrorLog`, strong-name-key path, the `.nuspec`-file-references, `docfx.json`-source-path, the `InternalsVisibleTo` of the test-project, …). The example is the reference for what these should point to.
   - Declare dependencies between codeunits in `dependentcodeunits` of the `codeunit.xml` and consume them only through `<codeunit>/Other/Resources/DependentCodeUnits` (see `common-project-structure`).

5. **Run the migration-target-criterion and iterate.** Run:

   ```text
   scbuildcodeunits -c -r <repo> -v 4
   ```

   Read the first error, find its root-cause (use the `root-cause`-skill if available), fix the **cause** (usually a wrong path, a missing file, a not-declared dependency, a failing test or a linting-issue), and run again. Repeat until the command exits 0. Then run it once more without extra switches:

   ```text
   scbuildcodeunits -c -r <repo>
   ```

   When this exits with 0, the onboarding is finished.

6. **Create the changelog-entry.** Determine the version with `scshowprojectversion` (after the changes) and create `Other/Resources/Changelog/v<version>.md`.

## Hints and pitfalls

- **The criterion is the container-run.** A build that only works with a locally-run `scbuildcodeunits` (without `-c`) but not with `-c` is not finished — the standardized container is what makes the result reproducible.
- **Do not put logic into the scripts.** The codeunit-scripts only delegate to ScriptCollection. If a behaviour is missing or wrong, the fix usually belongs into the ScriptCollection-repository, not into these scripts. Do not reimplement pipeline-logic in the repository.
- **Do not edit generated files.** `Taskfile.yml`, the generated diagrams/references, the synchronized agent-skill-folders and the generated constant-value-files are derived; a manual change is lost on the next build. Change the source instead (for tasks: the `.code-workspace`-file).
- **Pin versions explicitly.** Dependencies and tool-/image-versions must be pinned to exact versions (see the examples: exact-version brackets in `.csproj`, pinned tags in `ImageDefinition.csv`). Do not use "latest".
- **Keep the history.** Use `git mv` when relocating existing files so blame/history survive the migration.
- **When in doubt, diff against the example.** If a codeunit behaves differently than expected, compare it file-by-file with the matching example — a difference is usually the reason.

## When something is missing in ScriptCollection

If the migration cannot be finished because an automation is missing or wrong in ScriptCollection itself (not because of a mistake in the repository being migrated), the fix belongs into the ScriptCollection-repository (https://github.com/anionDevelopment/ScriptCollection, mirror https://github.com/anionDev/ScriptCollection), not as a workaround into the repository being onboarded. See the "Find out what a function really does" section of the `common-project-structure-using-scriptcollection`-skill for where to read its source-code.
