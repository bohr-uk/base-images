# Base Images

Minimal OCI base images built with [apko](https://github.com/chainguard-dev/apko) and published to the Nucleus Azure Container Registry (`nucleus61bfb23a.azurecr.io`).

## Images

| Config           | Image Repository                            | Published Tags                                       | Description                                                                                                                                                           |
| ---------------- | ------------------------------------------- | ---------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `apko/jlink.yml` | `nucleus61bfb23a.azurecr.io/bohr-uk/alpine` | `jlink`, `3-jlink`, `3.24-jlink`, `3.24-jlink-<sha>` | Minimal `x86_64` Alpine Linux base image (`musl`, `zlib`, `alpine-release`) running as `nonroot` (`10000:10000`), tailored for Java `jlink` runtimes (`nucleus-api`). |

## CI / Publishing

The GitHub Actions workflow (`.github/workflows/deploy.yaml`):

- **Branch Pushes & Pull Requests:** Builds `apko/jlink.yml` into an OCI layout directory (`./oci-dir/`) and scans OS packages for `HIGH` / `CRITICAL` vulnerabilities with Trivy.
- **Pushes to `main` & Weekly Schedule (`Mondays 06:00 UTC`):** Transfers the exact Trivy-scanned OCI layout artifact (SHA-256 verified) to an isolated `publish` job, authenticates to Azure Container Registry via OIDC, pushes the immutable image with `crane push --index`, and aliases the rolling tags (`3.24-jlink`, `3-jlink`, `jlink`) via `crane tag`.
- **Manual Dispatch (`workflow_dispatch`):** Builds and scans on any branch; optionally publishes the scanned image to ACR when run from `main` with `push_images: true`.

## Local Development

Build and load the `jlink` image into local Docker:

```bash
docker run --rm \
  -v "${PWD}:/work" \
  -w /work \
  cgr.dev/chainguard/apko:latest \
  build ./apko/jlink.yml bohr-uk/alpine:jlink image.tar

docker load -i image.tar
```

> Note: apko suffixes single-arch local tar tags with the architecture (e.g. bohr-uk/alpine:jlink-amd64)

Inspect generated SPDX SBOM packages:

```bash
jq -r '.packages[] | "\(.name) \(.versionInfo)"' sbom-x86_64.spdx.json
```

## Dependency Versioning

Version numbers and sha256 checksums (where needed) are available at the following locations for ongoing maintenance:

| Dependency | Release Page                                            | Update In                                                             |
| ---------- | ------------------------------------------------------- | --------------------------------------------------------------------- |
| `Alpine`   | https://alpinelinux.org/releases                        | `apko/jlink.yml`                                                      |
| `apko`     | https://github.com/chainguard-dev/apko/releases         | `APKO_VERSION` and `APKO_SHA256` in `.github/workflows/deploy.yaml`   |
| `crane`    | https://github.com/google/go-containerregistry/releases | `CRANE_VERSION` and `CRANE_SHA256` in `.github/workflows/deploy.yaml` |
