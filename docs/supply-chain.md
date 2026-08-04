# Supply chain

Every package in [`ghcr.io/go-pkgx/packages`](registry.md) ships with the evidence
needed to audit and trust it. Three artifacts are attached to each package as **OCI
referrers**, so they travel with the image and are discoverable from its digest:

- a **CycloneDX SBOM** — the package's components and versions
  ([`go-attest/sbom`](https://github.com/go-attest/sbom));
- an in-toto **SLSA provenance** statement — how and from what the package was built;
- a **cosign-style + minisign signature** over the package
  ([`go-attest/sign`](https://github.com/go-attest/sign)).

## The pinned key

Signatures verify against a single pinned public key:

```
RWQ+rmH+fXy2iYr+gReQAOQtYWtH0A7UlxcAa2hpr+txNBwGqtpFsR6L
```

The factory signs with the matching private key (held only as a CI secret); every
consumer checks against this exact public key.

## Verify on install

The [`go-pkgx/bottle`](https://github.com/go-pkgx/bottle) backend that both `pkgx`
and `pkgm` import performs verification at install time:

- `bottle.VerifySignature` validates a package's signature against the pinned key.
- Setting **`PKGX_VERIFY=1`** makes verification **fail-closed**: a package with no
  signature, or one that does not verify, is **refused** rather than installed.

```sh
# refuses to install anything that isn't validly signed by the pinned key
PKGX_DIST=oci://ghcr.io/go-pkgx/packages PKGX_VERIFY=1 pkgm install lz4.org
```

This closes the loop end to end: the factory signs and attests at publish time, and
the consumer refuses to run anything it cannot verify.
