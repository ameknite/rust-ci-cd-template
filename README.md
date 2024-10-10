# Rust CI/CD template

[![License](https://img.shields.io/badge/license-CC0--1.0%2FMIT%2FApache--2.0-blue.svg)](https://github.com/ameknite/rust-ci-cd-template?tab=readme-ov-file#license)
[![Crates.io](https://img.shields.io/crates/v/rust-ci-cd-template.svg)](https://crates.io/crates/rust-ci-cd-template)
[![Minimum Supported Rust Version](https://img.shields.io/badge/MSRV-1.81.0+-red.svg)](./Cargo.toml#L8)
[![CI](https://github.com/ameknite/rust-ci-cd-template/actions/workflows/ci.yaml/badge.svg)](https://github.com/ameknite/rust-ci-cd-template/actions?workflow=CI)

Template to maintain a rust project running CI/CD with Github Workflows.

<!-- todo!(Change this file) -->

## CI Workflow

[ci.yaml](./.github/workflows/ci.yaml)

![CI](https://github.com/ameknite/rust-ci-cd-template/assets/104745335/b190e0c7-53dc-4d8a-bc86-88ced3bf559c)

This workflow will run every time you push to main or make a pull request.
You can also run the workflow manually in github.

It will run:

- cargo build: Compiling for ubuntu-latest.
- cargo [fmt](https://github.com/rust-lang/rustfmt):
  - Rust Formatter.
  - Check for the file [rustfmt.toml](./rustfmt.toml).
- cargo [clippy](https://github.com/rust-lang/rust-clippy):
  - Lints catch common mistakes and improve your Rust code.
  - Check for the sections `[lints.rust]` and `[linsts.clippy]`
    in the [Cargo.toml file](./Cargo.toml).
- cargo doc: Build the documentation.
- cargo doc --test: Run tests in the documentation, only if the crate is a lib.
- [cargo-nextest](https://nexte.st/): Execute unit and integration tests.
- [cargo miri](https://github.com/rust-lang/miri) test: Help to detect certain classes of [undefined behavior](https://doc.rust-lang.org/reference/behavior-considered-undefined.html).
- [typos](https://crates.io/crates/typos):
  - Source code spell checker.
  - Check for the [typos.toml](./typos.toml) file.
  - [docs](https://github.com/crate-ci/typos/blob/master/docs/reference.md)
- [taplo fmt](https://taplo.tamasfe.dev/): Toml formatter.
- [markdownlint-cli2](https://github.com/DavidAnson/markdownlint-cli2):
  - Markdown linter and formatter.
  - Check for the file [.markdownlint.yaml](./.markdownlint.yaml)
- [yamlfmt](https://github.com/google/yamlfmt/): Yaml formatter.
- [cargo deny](https://github.com/EmbarkStudios/cargo-deny):
  - Check for licenses, bans, advisories and sources.
  - Check for the [deny.toml](./deny.toml) file.
  - [template](https://github.com/EmbarkStudios/cargo-deny/blob/main/deny.template.toml)
  - [docs](https://embarkstudios.github.io/cargo-deny/)
- [cargo audit](https://crates.io/crates/cargo-audit):
  - Check for security vulnerabilities.
  - Uses the [advisory-db](https://github.com/RustSec/advisory-db/)
- [cargo msrv](https://github.com/foresterre/cargo-msrv):
  Keep the rustc version on check.

## CD Workflow

![CD-1](https://github.com/ameknite/rust-ci-cd-template/assets/104745335/908c7e28-fa79-4669-95a1-a293a2de404a)
![CD-2](https://github.com/ameknite/rust-ci-cd-template/assets/104745335/35c1f888-03ee-4090-989e-bde9dc838e76)

[build_and_release.yaml](./.github/workflows/build_and_release.yaml)

This workflow will only run manually through GitHub.

It will:

- Validate your input version with regex (e.g. v3.1.4-alpha+159).
- Run the ci workflow.
- Run [cargo semver-checks](https://crates.io/crates/cargo-semver-checks):
  - Lint your crate API changes for semver violations.
  - This will only run if you crate is a library.
  - Uses your previous tag as a baseline.
- Run [add-notice](https://crates.io/crates/add-notice):
  - Add header notices to your rust files.
  - Check for the [NOTICEe file](./NOTICE).
  - Auto-commit.
- Run [cargo-bump](https://crates.io/crates/cargo-bump):
  - Bump the current version of the Cargo.toml file.
  - Uses the input version if provided.
  - Auto-commit.
- Run [cargo-attributions](https://crates.io/crates/cargo-attribution):
  - Comply with licenses distribution.
  - Add licenses, notices and metadata of your dependencies.
  - Crate an [attribution folder](./attribution/)
  - Auto-commit.
- Create a new tag:
  - Uses the input version if provided.
  - It's created only if the tag not exists.
- Create a new github release:
  - Uses [create-gh-release-action](https://github.com/taiki-e/create-gh-release-action/tree/v1/)
  - Includes your changelog if matches the same version.
  - Uses the created tag as a reference.
- Build and Release in Github:
  - Using [upload-rust-binary-action](https://www.github.com/taiki-e/upload-rust-binary-action/tree/v1/)
  - And [setup-cross-toolchain-action](https://github.com/taiki-e/setup-cross-toolchain-action/tree/v1/)
  - Only builds if your crate contains a binary crate.
  - Uses the created tag as a reference.
  - Bundle the attribution folder with licenses and notices.
  - Create checksum with sha256.
  - It builds for linux: gnu (x86_64|aarch64|riscv64gc) and musl (x86_64|aarch64).
  - It builds for windows: msvc (x86_64|aarch64) and gnu (x86_64).
  - It builds for mac-os (x86_64|aarch64).
- Publish the create to crates.io:
  - Only if you have the secrets.CARGO_REGISTRY_TOKEN variable added to repository secrets.
  - Path: Settings -> Security -> Secrets and variables -> Actions -> Repository secrets.

### How to use the Build and Release workflow

You will run this directly in your Github repo -> Section `Actions` -> `[Main] Build & Release` workflow. -> `Run workflow` button.

![Build and Release Workflow](https://github.com/ameknite/rust-ci-cd-template/assets/104745335/2c6235cb-def6-4cb6-ada8-d3d32523d097)

Here you can do three things to run the workflow:

  1. Write the new version you want to release:
    - The version is in the form of:
      - Version parts: `${MAJOR}.${MINOR}.${PATCH}-${PRE-RELEASE}+${BUILD}`
      - Example: 3.1.4-alpha+159
  2. Select the release_type you want:
      ![release-type](https://github.com/ameknite/rust-ci-cd-template/assets/104745335/5a503614-2418-44fa-ac84-f0eb051196dc)
      - `patch`, `minor` and `major`.
      - It will use your current crate version as a baseline.
      - Keep in `none` if you only want the writing version.
  3. Keep all with the defaults (without version and release_type = none)
      - It will use your current crate version as a baseline.
      - Run if you already update your crate version.
      - Useful for testing when you already run the workflow.

The two booleans allow you to publish your crate to creates.io and overwrite your tag to add modifications to a release, respectively.
  
## Dependabot

This template includes a setup for dependabot and a workflow for auto-merge.

- It will update packages for cargo and github-actions weekly.

- For rust:
  - It will create pull request in groups.
  - One group for patch updates, one for minor updates and one for major updates.
  - It will auto-merge the pull request if passes ci.
  
- For github-actions:
  - It will create a pull request in groups.
  - Only one group for the updates.
  - It will auto-merge the pull request if passes ci.

## Other workflows

This template uses [workflow call](https://docs.github.com/en/actions/using-workflows/reusing-workflows) and [workflow dispatch](https://docs.github.com/en/actions/using-workflows/manually-running-a-workflow) to reuse workflows and be able to run them individually. So you can run various jobs of the ci/cd workflows independently.

Some workflows to consider:

- [weekly-workflow](./.github/workflows/weekly.yaml)
  - Run the ci workflow.
  - Update Dependencies, Attributions and Notices.
- [monthly-workflow](./.github/workflows/monthly.yaml)
  - Run the build_and_release workflow.
  - Do a patch release and publish to crates.io
- [update_msrv.yaml](./.github/workflows/update_msrv.yaml)
  - It will run cargo-msrv to update your `rust-version` in Cargo.toml
  - If you uses the MSRV in your README.md it will also updated it.
- [cargo_semver_checks](./.github/workflows/cargo_semver_checks.yaml)
  - Test semver violations before releasing.
- [cache.yaml](./.github/workflows/cache.yaml)
  - To clean the caches generated for the workflows.
  - Run if you have problems with the cache.
- [add_notice.yaml](./.github/workflows/add_notice.yaml)
  - To replace and update your notices.
  - To apply notices to other kind of files.
  - To use different comment styles.
- [publish.yaml](./.github/workflows/publish.yaml)
  - To publish your crate to crates.io.

## What to change if you want  to use this template

- Change [this file](.).
- Change the metadata in [Cargo.toml](./Cargo.toml)
- Change the licenses files. Right now uses [LICENSE-CC0-1.0](./LICENSE-CC0-1.0), [LICENSE-MIT](./LICENSE-MIT)
and [LICENSE-Apache-2.0](./LICENSE-Apache-2.0).
- Change the [deny.toml](./deny.toml) file.
- Change the [NOTICE](./NOTICE) file:
  - Remove the notice in [main.rs](./deny.toml).
  - Or you can run add-notice --remove before you update the NOTICE file.
- Change the [FUNDING.yaml](./.github/FUNDING.yaml) file.

## Similar projects that might help you

- [cargo-dist](https://github.com/axodotdev/cargo-dist?tab=readme-ov-file).
- [cargo-release](https://github.com/crate-ci/cargo-release).
- [release-plz](https://github.com/MarcoIeni/release-plz).
- [cargo-smart-release](https://github.com/Byron/cargo-smart-release).

## LICENSE

SPDX-License-Identifier: CC0-1.0 OR MIT OR Apache-2.0

Licensed under a triple license at your option:

- [CC0-1.0](LICENSE-CC0-1.0) license.
- [MIT](LICENSE-MIT) license.
- [Apache-2.0](LICENSE-Apache-2.0) license.
