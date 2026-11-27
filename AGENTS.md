# AGENTS.md

Minimal OCI base images built with `apko` and published to Azure Container Registry (`nucleus61bfb23a.azurecr.io/bohr-uk/alpine`, overridable via `vars.ACR_NAME`). Currently defines a single `x86_64` Alpine image (`apko/jlink.yml`, `nonroot` `10000:10000`) for Java `jlink` runtimes.

## Local Build & Inspection

`apko`, `trivy`, and `crane` are not preinstalled in the dev container; build via Docker:

```bash
docker run --rm -v "${PWD}:/work" -w /work cgr.dev/chainguard/apko:latest \
  build ./apko/jlink.yml bohr-uk/alpine:jlink image.tar
docker load -i image.tar
```

- `apko build` suffixes single-arch local tar tags with the architecture (e.g. `bohr-uk/alpine:jlink-amd64`) and writes `sbom-x86_64.spdx.json` to the working directory.
- Inspect SBOM packages: `jq -r '.packages[] | "\(.name) \(.versionInfo)"' sbom-x86_64.spdx.json`
- To match CI output (e.g. for local Trivy scans), pass an OCI layout directory (`./oci-dir/`) instead of `image.tar`.
- Build outputs (`*.tar`, `*.tmp`, `sbom*`, `oci-dir/`, `.cache/`) are gitignored; do not commit them.

## CI & Publishing Gotchas (`.github/workflows/deploy.yaml`)

- **Hardcoded image config:** The workflow does not auto-discover `apko/*.yml`. Adding a new image requires updating `.github/workflows/deploy.yaml` (and `README.md`).
- **Alpine version regex coupling:** CI derives tag versions directly from `apko/jlink.yml` via `grep -oP 'alpine/v\K[0-9]+\.[0-9]+' ./apko/jlink.yml | head -n 1`. Keep a `.../alpine/vX.Y/...` repository URL in `apko/jlink.yml` or CI will fail.
- **Pinned tool checksums:** `APKO_VERSION`/`APKO_SHA256` and `CRANE_VERSION`/`CRANE_SHA256` are SHA-256 verified against Linux `amd64`/`x86_64` release tarballs; update both version and checksum together.
- **Vulnerability gate:** PRs and pushes build `./oci-dir/` and fail on fixed `HIGH`/`CRITICAL` OS CVEs via Trivy (`ignore-unfixed: true`, `vuln-type: os`).
- **Publishing flow:** Only runs on `main` (push, Monday `06:00 UTC` schedule, or `workflow_dispatch` with `push_images: true`). The `build` job archives `./oci-dir` into `oci-layout.tar` and passes its SHA-256 to an isolated `publish` job, which verifies the archive, logs into ACR via Azure OIDC, pushes `<alpine_version>-jlink-<short_sha>` with `crane push --index`, and aliases `<alpine_version>-jlink`, `<alpine_major>-jlink`, and `jlink` via `crane tag`.
- **Path filters:** `pull_request` and `push` only ignore `.gitignore` and `README.md`; edits to other files (including `AGENTS.md`) trigger the build/scan job.

## PR Conventions

- Follow `.github/PULL_REQUEST_TEMPLATE.md` (`# NUC-###` heading and `## Description`).
