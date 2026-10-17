---
name: adame-knowledge
description: Gives information about how to use adame and how to use a adame-repository.
metadata:
  purpose: server-maintenance
  tags: server-management
  version: 1.0.0
---

# Skill adame-knowledge

## What Adame is

Adame ("Automatic Docker Application Management Engine") manages one containerized application on Linux.
Each managed application lives in its own git-repository ("Adame-repository") which contains configuration, volumes and logs.
Adame commits state-changes into this repository, so git-history is the history of the application-environment.

Source-code and reference: <https://github.com/anionDev/Adame> (reference: `Adame/Other/Reference/ReferenceContent`).

## Usage

Requirements:

- Adame is installed as python-package `Adame`, CLI-command is `adame`.
- Adame must run with elevated privileges (for example `sudo adame ...`) because it calls `docker`, `chmod`, `chown` and optionally `snort`.
- Commands are not intended for unattended scripts when gpg-signing is enabled (pinentry-dialog requires user-interaction).
- All commands except `create` are idempotent.
- Global options: `--verbose` (`-v`), `--diagnostic` (`-d`).

### Create new Adame-repository

```sh
sudo adame create --name MyApplication --folder /srv/MyApplication --image httpd:2.4.62 --owner MyName
```

- `--name` (required): Name of application. Spaces are replaced by `-`.
- `--folder`: Target-folder. Must not exist or must be empty.
- `--image`: Docker-image used in generated `docker-compose.yml`. Use explicitly pinned version.
- `--owner` (required): Owner of repository.
- `--gpgkey_of_owner` (optional): GPG-key used for signing commits.

Result: new git-repository with initial commit (structure see below).
After creation usually `Configuration/docker-compose.yml` must be adapted (ports, volumes, environment) and committed.

### Start application ("up")

Adame has no `up`-command. Use one of:

```sh
sudo adame startadvanced --configurationfile /srv/MyApplication/Configuration/Adame.configuration
sudo adame start --configurationfile /srv/MyApplication/Configuration/Adame.configuration
```

- `startadvanced` (recommended): Runs `stopadvanced`, re-applies configuration, then starts IDS (if enabled) and container. Use this after changing configuration or image-version.
- `start`: Only ensures IDS (if enabled) and container are running, changes nothing else.
- Internally Adame runs `prescript` (if configured) and then `docker compose --project-name <name> up --detach --build --force-recreate ...` in folder `Configuration`. If `Configuration/Parameters.env` exists it is passed as `--env-file`.

### Stop application ("down")

Adame has no `down`-command. Use one of:

```sh
sudo adame stopadvanced --configurationfile /srv/MyApplication/Configuration/Adame.configuration
sudo adame stop --configurationfile /srv/MyApplication/Configuration/Adame.configuration
```

- `stopadvanced` (recommended): Stops container and IDS, commits current state of repository ("Saved changes"), exports logs.
- `stop`: Only stops container and IDS, does not commit.
- Internally Adame runs `docker compose --project-name <name> down` in folder `Configuration` and then `postscript` (if configured).

### Other commands

All take `--configurationfile <path to Adame.configuration>`:

- `applyconfiguration`: Regenerates `Networktraffic.Generated.rules` and commits.
- `checkintegrity`: Checks signatures of commits of last 7 days (only warnings, exitcode is always 0).
- `exportlogs`: Moves logs to `logtargetfolder` and/or exports them to SIEM via `rsync`.
- `diagnosis`: Shows diagnostic information (`--configurationfile` optional).
- `checkout --branch <branch>`: Checks out given branch of Adame-repository.

## Structure of Adame-repository

| Path | Content |
| --- | --- |
| `ReadMe.md`, `License.txt`, `.gitignore` | Generated once on `create`, freely editable, never regenerated. |
| `Configuration/Adame.configuration` | Main configuration (INI, section `[general]`): `name`, `owner`, `repositoryid`, `networkinterface`, `maximalexpectedstartduration` (seconds to wait after start), `prescript`, `postscript`, optional `logtargetfolder`. Path of this file is argument for all commands except `create`. |
| `Configuration/docker-compose.yml` | Docker-compose-definition of application. Freely editable. Paths are relative to `Configuration`. |
| `Configuration/Parameters.env` | Optional env-file for docker compose. |
| `Configuration/Volumes/` | Conventional location for volumes mounted into container (for example `./Volumes/Data`). Content is committed into repository. |
| `Configuration/RunningInformation.txt` | Last known state of container- and IDS-process, written by Adame. Do not edit. |
| `Configuration/FileMetadata.csv` | File-metadata of volumes, written by Adame. Do not edit. |
| `Configuration/.gitconfig` | Git-configuration used by Adame (author, signing). |
| `Configuration/Security/Security.configuration` | INI: `[general]` with `gpgkeyofowner`, `remoteaddress`/`remotename`/`remotebranch` (backup-remote), `enableids`, `idsname`, `siemaddress`/`siemfolder`/`siemuser`; `[snort]` with `globalconfigurationfile`. |
| `Configuration/Security/Networktraffic.Custom.rules` | Custom IDS-rules, editable. |
| `Configuration/Security/Networktraffic.Generated.rules` | Generated IDS-rules. Do not edit, overwritten by Adame. |
| `Configuration/Security/ApplicationProvidedSecurityInformation.xml` | Optional security-information about application. |
| `Logs/Application/` | Application-logs (mount it into container, for example `./../Logs/Application:/...`). Git-ignored. |
| `Logs/IDS/` | IDS-logs. Git-ignored. |
| `Logs/Overhead/` | Logs of Adame itself. Git-ignored. Look here first when a command fails. |

## Hints for working with Adame-repository

- Change configuration by editing files in `Configuration`, then run `startadvanced`.
- Do not edit files marked as generated above.
- Use explicitly pinned image-versions in `docker-compose.yml` (no `latest`).
- Known open issues of Adame (for example missing restore of file-metadata, IDS-rules commented out, `start` reporting success even on failure) are documented in `Adame/Other/Reference/ReferenceContent/Hints.md` in Adame-source-repository. Fix such issues in Adame-repository itself, not by workarounds in Adame-managed repository.
