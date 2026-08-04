# Package registry

[`ghcr.io/go-pkgx/packages`](https://github.com/go-pkgx/packages) is a pure-Go
package factory: it builds [pkgx pantry](https://github.com/pkgxdev/pantry)
recipes and publishes **signed, attested packages** for **every platform** through
**two channels**.

## Two channels, every platform

Every platform — **linux/x86-64**, **linux/aarch64**, **darwin/aarch64**,
**darwin/x86-64** and **windows/x86-64** — publishes to **both**:

- the **signed OCI registry** `ghcr.io/go-pkgx/packages` (each package carrying an
  SBOM, provenance and signature as OCI referrers), and
- a **GitHub Pages pkgx-dist mirror** at
  [`https://go-pkgx.github.io/packages`](https://go-pkgx.github.io/packages),
  carrying all platforms side by side (`<project>/<os>/<arch>/…`).

## The factory

The [`go-pkgx/packages`](https://github.com/go-pkgx/packages) repo is the factory.
GitHub Actions workflows build each recipe with
[`bk`](https://github.com/go-pkgx/bk) — the CGO-free re-implementation of brewkit
— and publish the result to both channels.

- **Factories:** `build.yml` (linux), `darwin.yml` (macOS), `windows.yml` (Go) +
  `windows-rust.yml` (Rust) + `windows-proof.yml` (e2e), each publishing to the OCI
  registry and uploading a pkgx dist tree; a single `pages.yml` aggregator unions
  the latest run of every build into the combined Pages mirror.
- **Schedule:** each factory runs on a schedule (plus manual `workflow_dispatch`).
- **Auth:** the workflow's **native `GITHUB_TOKEN`** (`permissions.packages: write`)
  — no long-lived PAT to manage or rotate.
- **Ordering:** requested projects are expanded to their **topologically-ordered
  runtime-dependency closure**, so every dependency is built before its dependents.
- **Idempotent:** any `(project, version, platform)` already in ghcr is **skipped**,
  so shared dependencies build once and the catalog grows progressively.

Per-recipe failures are logged, never fatal; the recipe list is grown outward from
dependency-free leaves toward the full pantry.

## What is published

Packages are ordinary OCI artifacts — each with a signature, an SBOM, and a
provenance statement attached as [referrers](supply-chain.md). Because the
factories keep adding recipes, treat the published set as a moving target — it
grows continuously. Example packages published at the time of writing (a snapshot,
not the full catalog):

| project | version |
| --- | --- |
| `zlib.net` | 1.3.2 |
| `tukaani.org/xz` | 5.8.3 |
| `lz4.org` | 1.10.0 |
| `gnu.org/tar` | 1.35 |
| `sourceware.org/bzip2` | 1.0.8 |

## Consuming

Point the go-pkgx tools at the registry and verify against the pinned key:

```sh
PKGX_DIST=oci://ghcr.io/go-pkgx/packages PKGX_VERIFY=1 pkgm install lz4.org
```

`PKGX_VERIFY=1` is fail-closed: an unsigned or badly-signed package is refused, not
installed. See [supply chain](supply-chain.md) for the verification model.

Or consume the same packages from the GitHub Pages pkgx-dist mirror:

```sh
PKGX_DIST=https://go-pkgx.github.io/packages pkgm install lz4.org
```

Packages are OCI artifacts, so any OCI client can also pull them directly:

```sh
docker pull ghcr.io/go-pkgx/packages/lz4.org:1.10.0
```
