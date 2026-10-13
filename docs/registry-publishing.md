# Runtime and library registries

The reusable workflow currently builds release artifacts for Go and Rust command-line applications. Go module indexing is implemented as an opt-in step. Publishing to crates.io, PyPI, and npm remains a documented extension point; this file records the work each publisher needs before those registry jobs are added.

## Go modules and pkg.go.dev

Go modules are published from versioned Git tags rather than uploaded to a separate package registry. Use a public module path that matches the repository layout, run tests, and push a stable `vX.Y.Z` tag. The Go caller example uses `v*.*.*` and enables `index-go-module: true`; after the GitHub release is complete, the reusable workflow requests the module from `proxy.golang.org`, which prompts indexing for Go tooling and `pkg.go.dev`, as described in the [official Go publishing guide](https://go.dev/doc/modules/publishing).

For a module in a repository root, set the `module` line in `go.mod` to its public import path, such as `github.com/OWNER/REPOSITORY`. The example's `example.com/go-cli` value is a placeholder and must be replaced. Module subdirectories require path-prefixed tags and are not supported by the current indexing step. Treat a published version as immutable; release a new version instead of moving a version tag.

## Rust libraries on crates.io

The current Rust example validates and packages a CLI binary, but it does not publish Cargo crates. A future publisher should follow the [Cargo publishing guide](https://doc.rust-lang.org/cargo/reference/publishing.html) and:

- Confirm crate name availability and complete `Cargo.toml` metadata, including description, license, repository, and readme.
- Run `cargo test --locked` and `cargo publish --dry-run` before any upload.
- Add a distinct `publish-crate` job with `id-token: write`, an explicit GitHub environment, and the official [`rust-lang/crates-io-auth-action`](https://github.com/rust-lang/crates-io-auth-action) to obtain a short-lived crates.io token. Use `CARGO_REGISTRY_TOKEN` only for the publish step.
- Require the crate's first version to be published manually before configuring crates.io Trusted Publishing; then bind the crate to the repository and exact workflow file.

This should be a separate opt-in from Rust CLI artifact builds. A Cargo package version and a Git tag need a checked mapping, and an already published crate version cannot be overwritten.

## Python packages on PyPI

There is no Python publisher in the reusable workflow yet. A future job should build both a source distribution and wheel from `pyproject.toml`, inspect them with `twine check`, run tests against the built artifacts, then publish through [`pypa/gh-action-pypi-publish`](https://github.com/pypa/gh-action-pypi-publish) using [PyPI Trusted Publishing](https://docs.pypi.org/trusted-publishers/) (`id-token: write` and a narrowly scoped GitHub environment). Project owners must configure the repository, workflow filename, and optional environment on PyPI. Do not add a long-lived PyPI token when Trusted Publishing is available.

`.deb`, `.rpm`, and other Linux packages are optional for Python projects and are not part of the default workflow.

## TypeScript packages on npm

There is no npm publisher in the reusable workflow yet. A future job should verify package metadata and the publish file list, run `npm ci`, build and test, then publish from a protected release workflow using [npm Trusted Publishing](https://docs.npmjs.com/trusted-publishers/). That requires `id-token: write`, a trusted-publisher entry matching the GitHub repository and workflow filename, and a GitHub-hosted runner. Current npm guidance requires npm CLI 11.5.1 or newer and Node.js 22.14.0 or newer; the `repository` field in `package.json` must match the GitHub repository. Pin the Node/npm versions and validate the workflow before enabling publishing.

`.deb`, `.rpm`, and other Linux packages are optional for TypeScript projects and are not part of the default workflow.
