# Visual-Regression-Baseline Specification

## Purpose

Records how the baseline-images of a product's visual-regression-tests are structured, independent of the technology the user-interface and the tests are built with.
They are not derivable from the source-code itself:

- They state that every visual-regression-scenario is covered in both the light- and the dark-theme, so that an unintended visual change is caught in both themes and not only in whichever one happened to be rendered.
- They state that a baseline-image's filename begins with the theme-mode it was rendered in, so that the two baseline-images of a scenario are told apart at a glance and it is obvious which theme a baseline belongs to.

This specification is product-independent and meant to be reused.
A product which has visual-regression-tests adopts it as-is.
A product MAY add further, product-specific rules about the rest of the filename on top of it, but those are stated in that product's own specification, not here.

## Requirements

### Requirement: Every visual-regression-test covers both the light- and the dark-theme

Every visual-regression-scenario SHALL be generated once in the light-theme and once in the dark-theme, so that an unintended visual change is caught in both themes.
The two runs produce two baseline-images of the same scenario, one per theme.

#### Scenario: A visual-regression-scenario is added

- **WHEN** a visual-regression-scenario is added
- **THEN** it is generated and asserted in both the light-theme and the dark-theme, producing one baseline-image per theme

#### Scenario: A scenario covers only one theme

- **WHEN** a visual-regression-scenario produces a baseline-image for only one of the two themes
- **THEN** this is a defect, resolved by also generating and asserting it in the other theme

### Requirement: A baseline-image's filename begins with its theme-mode and contains only dash as word-separator

The filename of every visual-regression-baseline-image SHALL begin with the theme-mode it was rendered in, written as the prefix `light-` or `dark-`, followed by the rest of the name which describes what the image shows.
The resulting form is `<theme-mode>-<name>`, for example `light-settings-page` or `dark-login-form`.

#### Scenario: A baseline-image is created

- **WHEN** a baseline-image is created
- **THEN** its filename begins with `light-` or `dark-` according to the theme it was rendered in, followed by the rest of the name

#### Scenario: A baseline-image's filename is missing the theme-mode prefix

- **WHEN** a baseline-image's filename does not begin with a `light-` or `dark-` theme-mode prefix
- **THEN** this is a defect, resolved by renaming the baseline-image and the name the test refers to it by to the form `<theme-mode>-<name>`
