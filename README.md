# Release Workflow Template

A GitHub template repository for shipping Go and Rust command-line or terminal applications. It standardizes tag-driven GitHub Releases, Linux amd64 archives and system packages, native macOS Intel and Apple Silicon builds, checksums, and package naming. Go releases can also prompt the public Go module proxy to index the module for `pkg.go.dev`.

The examples use GoReleaser and nFPM for Go and Rust. GoReleaser's Rust builder builds the Rust example on its native Linux target; the reusable workflow builds macOS packages on native GitHub-hosted macOS runners. This keeps both languages on the same release and Linux package path.

## Start a project

1. Use this repository as a GitHub template or copy one example directory into a new repository.
2. Keep the example `.goreleaser.yaml`, edit its project name and package metadata, and place it at the root of the new project.
3. Copy the example caller workflow into `.github/workflows/release.yml`. It calls this repository's reusable workflow at the `0.1.2` tag. Before releasing Go, replace the placeholder module path in `go.mod` with the public repository's root module path.
4. Set the GitHub Actions workflow token to allow release creation (`contents: write`). No separate secret is needed for GitHub Releases.
5. For Go, push a Go-style tag such as `v0.1.0`; enable `index-go-module: true` to request indexing through `proxy.golang.org`. For Rust binaries, the included example keeps matching its Cargo version with a no-prefix tag such as `0.1.0`. The workflow publishes a GitHub Release.

## Included artifacts

- Linux amd64: `.deb`, `.rpm`, Arch Linux `.pkg.tar.zst`, and `.tar.gz` binary archive.
- macOS: architecture-specific `.pkg` for Intel (`amd64`) and Apple Silicon (`arm64`), built on native runners.
- SHA-256 checksum files for each package/archive.

macOS packages are unsigned and not notarized by default. Direct distribution with standard Gatekeeper behavior requires a Developer ID certificate and Apple notarization credentials; add these as repository secrets and adapt the workflow if signing is needed.

## Examples

- [`examples/go-cli`](examples/go-cli) contains a small Go CLI, GoReleaser config, and caller workflow.
- [`examples/rust-cli`](examples/rust-cli) contains a small Rust CLI, GoReleaser config, and caller workflow.
- [`.github/workflows/release.yml`](.github/workflows/release.yml) is the reusable workflow both examples call.
- [`docs/registry-publishing.md`](docs/registry-publishing.md) describes Go module discovery and future crates.io, PyPI, and npm publishing support.

The reusable workflow expects the consumer project to have a Go module or Cargo package, an appropriate `.goreleaser.yaml`, and a single executable. Go module indexing is opt-in and currently requires the Go module at the repository root and a public GitHub repository. Rust crate publication, PyPI, and npm are guidance only; they are not published by these workflows. Python and TypeScript system packages are optional future work. GUI applications need app-bundle-specific packaging and signing settings, so use these examples as the release foundation rather than as a finished desktop installer configuration.

## Verify locally

```sh
cd examples/go-cli && go test ./...
cd ../..
cargo test --locked --manifest-path examples/rust-cli/Cargo.toml
goreleaser check --config examples/go-cli/.goreleaser.yaml
goreleaser check --config examples/rust-cli/.goreleaser.yaml
```
