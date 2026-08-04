# go-pkgx

A **dependency-free, pure-Go** (`CGO_ENABLED=0`) package manager for
[pkgx](https://pkgx.sh) packages — a single static binary that installs **and
runs** packages on a `FROM scratch` image, with no runtime dependencies of its
own.

The reference `pkgm` is a Deno/TypeScript script that shells out to `pkgx`,
`deno`, `curl`, `openssl`, `zlib` and `xz` — roughly **515 MB** of runtime
closure just to install a package. go-pkgx replaces all of it with one static
~9 MB binary.

## Install

**Linux / macOS** — one line:

```sh
curl -fsSL https://go-pkgx.github.io/install.sh | sh
```

**Windows** — one line (PowerShell):

```powershell
irm https://go-pkgx.github.io/install.ps1 | iex
```

The installer downloads the static binary for your os/arch from the latest
[release](https://github.com/go-pkgx/pkgm/releases/latest), verifies it against
the release `SHA256SUMS`, and puts `pkgm` on your `PATH` (`$HOME/.local/bin`, or
`%LOCALAPPDATA%\Programs\go-pkgx` on Windows; set `PKGM_INSTALL` to override on
Unix).

**Go users**:

```sh
go install github.com/go-pkgx/pkgm@latest
```

Once installed, `pkgm install lz4.org` verifies each bottle against the signed
registry by default.

## Usage

```
pkgm install|i    <pkg>[@version] ...   install to /usr/local (root) or ~/.local
pkgm uninstall|rm <pkg> ...             remove an installation
pkgm shim|stub    <pkg> ...             create a shim in <prefix>/bin
pkgm list|ls                            list what's installed
pkgm outdated                           list outdated installations
pkgm update|up|upgrade                  update installations to latest
pkgm pin          <pkg>@version ...     install pinned to an exact version
pkgm run|x        <pkg> [-- args...]    run a pkg (works FROM scratch)

flags: -h/--help  -v/--version  -p/--pin  -P/--prefix DIR  -s/--from-scratch
env:   PKGX_DIR   package store (default: ~/.pkgx)
       PKGM_PREFIX default install prefix (ideal for FROM scratch)
```

The `install`/`uninstall`/`shim`/`list`/`outdated`/`update`/`pin` command
surface and the `~/.local` vs `/usr/local` prefix logic mirror the reference
`pkgm`, so it is a drop-in replacement.

## FROM scratch

A scratch image whose only file is the `pkgm` binary can install and run real
packages, with no system libc:

```dockerfile
FROM scratch
COPY pkgm /pkgm
ENV PKGX_DIR=/pkgx
ENTRYPOINT ["/pkgm"]
```

```sh
$ docker run --rm pkgm-scratch run gnu.org/bash -- --version
GNU bash, version 5.3.0(1)-release (aarch64-unknown-linux-gnu)
```

See [FROM scratch](from-scratch.md) for how the closure is resolved, the
conformance matrix, and the pantry-wide audit.
