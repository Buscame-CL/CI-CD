# CI/CD

## Purpose

Most of the components can be reusable in your own projects:
- Project configuration
- CI workflow to build, lint, test, check test coverage and run mutation tests
- CD workflow to test on all platforms and publish on [crates.io](https://crates.io)

## Github Actions workflows

### Jobs

The `CI` workflow is triggered for any commit or pull request on the `main` branch and runs the following jobs:
- Test the crate
    - Build the project for all targets and with default features to ensure everything compiles
    - Run unit, integration and doc tests with default features
- Run test coverage and mutation tests
    - Run [source-based coverage](https://marco-c.github.io/2020/11/24/rust-source-based-code-coverage.html) with [grcov](https://github.com/mozilla/grcov)
    - Upload HTML coverage report in workflow artifacts
    - Upload HTML coverage report on Codecov
    - Fail if the coverage threshold is not reached
    - Run mutation tests with [mutagen](https://github.com/llogiq/mutagen) (note that the job automatically adds `#[mutate]` annotations)
    - Fail if the mutation threshold is not reached
- Run several code checks
    - Run `clippy`
    - Run `rustfmt` to check code is correctly formatted
    - Run `cargo-deny` to check dependencies
    - Check encoding of all files is UTF-8
    - Check all line endings are LF
    - Check there is no TODO left in Rust code

This workflow is only run on Ubuntu virtual environment.

The `CD` workflow is triggered manually and runs the following jobs:
- Check version to release in `Cargo.toml` files
- Test on Ubuntu
- Test on Windows
- Test on MacOS
- Make a publication dry run on [crates.io](https://crates.io)
- Publish on [crates.io](https://crates.io)
- Create the release and prepare the next one in the repository
    - Update the latest version and its release date in the `CHANGELOG.md` file and push the changes
    - Tag the created commit with the name of the released version
    - Create a GitHub release from the created tag and the content of the `CHANGELOG.md` file
    - Create a new section for the next release in the `CHANGELOG.md` file and push the changes

### Secrets

The `CD` workflow requires the following secrets:
- `CRATES_IO_TOKEN`: token to publish the crate(s) on [crates.io](https://crates.io)
- `GIT_TOKEN`: GitHub personal access token to allow pushing changes as administrator (if the `main` branch is protected, "Include administrators" must be unchecked in settings)

These secrets must be stored in a [repository environment](https://docs.github.com/en/actions/reference/environments) called `Deployment`.

### Settings

Settings of the `CI` workflow can be modified in the file `.github/workflows/ci.yml`, in the `env` section:
- `RUST_VERSION_STABLE`: version of the stable Rust compiler to use
- `RUST_VERSION_NIGHTLY`: version of the nightly Rust compiler to use when required
- `MUTAGEN_COMMIT`: commit of the mutagen version to install ([from mutagen repository](https://github.com/llogiq/mutagen))
- `COV_THRESHOLD`: minimum threshold the coverage must reached to succeed the job
- `MUTAGEN_THRESHOLD`: minimum threshold the mutation tests must reached to succeed the job
- `CRATE_PATHS`: package names separated by `;` in publication order if the repository is a Cargo workspace, else `.`

Settings of the `CD` workflow can be modified in the file `.github/workflows/cd.yml`, in the `env` section:
- `RUST_VERSION_STABLE`: version of the stable Rust compiler to use
- `CRATE_PATHS`: package names separated by `;` in publication order if the repository is a Cargo workspace, else `.`

### Run locally

[act](https://github.com/nektos/act) can be used to run the workflows on your own machine.<br>
Once installed, run `act -P ubuntu-18.04=nektos/act-environments-ubuntu:18.04` in the repository folder to run all supported jobs.

### Workspaces

This repository contains a single crate, but the workflows should also work with a Cargo workspace.

