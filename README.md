# 5g-nf-platform

Container image factory for 5G Network Functions.

Aggregates NFs from open-source upstream projects, applies research patches to extend and integrate them, and publishes versioned images to GHCR. Each NF is built as an independent image from a pinned upstream tag.

This repository is part of the [5g-k3s-kubedge-testbed](https://github.com/Jacobbista/5g-k3s-kubedge-testbed) initiative and acts as its image supply chain.

---

## Versioning

Image tags follow the format `<upstream-version>-p<patch-count>`.

- `upstream-version` mirrors the upstream project release the image is built from.
- `p<N>` counts the local patches applied on top. `p0` means unmodified upstream.

This makes it explicit which upstream release an image tracks and how many local modifications it carries. Canonical tags for all NFs are always in [`versions.json`](versions.json).

---

## Open5GS NFs

The current core is built on [Open5GS](https://open5gs.org) v2.7.5. All standard 5G SA NFs are available.

| NF | Image | Patches |
|----|-------|---------|
| SMF | `ghcr.io/jacobbista/5g-nf-platform/smf:2.7.5-p1` | session-info endpoint |
| AMF | `ghcr.io/jacobbista/5g-nf-platform/amf:2.7.5-p0` | — |
| UPF | `ghcr.io/jacobbista/5g-nf-platform/upf:2.7.5-p0` | — |
| UDM | `ghcr.io/jacobbista/5g-nf-platform/udm:2.7.5-p0` | — |
| AUSF | `ghcr.io/jacobbista/5g-nf-platform/ausf:2.7.5-p0` | — |
| UDR | `ghcr.io/jacobbista/5g-nf-platform/udr:2.7.5-p0` | — |
| NRF | `ghcr.io/jacobbista/5g-nf-platform/nrf:2.7.5-p0` | — |
| PCF | `ghcr.io/jacobbista/5g-nf-platform/pcf:2.7.5-p0` | — |
| BSF | `ghcr.io/jacobbista/5g-nf-platform/bsf:2.7.5-p0` | — |
| NSSF | `ghcr.io/jacobbista/5g-nf-platform/nssf:2.7.5-p0` | — |

### Patches

#### SMF — [`nfs/smf/patches/`](nfs/smf/patches/README.md)

| Patch | Description |
|-------|-------------|
| `0001-session-info-endpoint.patch` | Adds `GET /session-info` on the metrics port (`:9090`). Returns active PDU sessions as JSON — IMSI, DNN, UE IP, S-NSSAI, UP connection state. |

See [nfs/smf/patches/README.md](nfs/smf/patches/README.md) for full API spec and implementation notes.

---

## Usage

```sh
# pull
docker pull ghcr.io/jacobbista/5g-nf-platform/smf:2.7.5-p1

# build locally
docker build \
  --build-arg UPSTREAM_TAG=v2.7.5 \
  --build-arg NF_NAME=smf \
  -t smf-local \
  nfs/smf/
```

---

## Adding a patch

1. Edit sources in `upstream/<project>/` (gitignored local clone).
2. Export: `git diff HEAD > nfs/<nf>/patches/<N>-<description>.patch`
3. Push — CI counts patches automatically and updates the image tag.
