# Base Images

Minimal OCI base images built with [apko](https://github.com/chainguard-dev/apko) and published to the Nucleus Azure Container Registry (`nucleus61bfb23a.azurecr.io`).

## Images

| Config | Image Repository | Published Tags | Description |
|---|---|---|---|
| `apko/jlink.yml` | `nucleus61bfb23a.azurecr.io/bohr-uk/alpine` | `jlink`, `3-jlink`, `3.24-jlink`, `3.24-jlink-<sha>` | Minimal `x86_64` Alpine Linux base image (`musl`, `zlib`, `alpine-release`) running as `nonroot` (`10000:10000`), tailored for Java `jlink` runtimes (`nucleus-api`). |

## CI / Publishing

The GitHub Actions workflow (`.github/workflows/deploy.yaml`):
- **Pull Requests (`main`):** Builds `apko/jlink.yml` into `image.tar` and scans OS packages for `HIGH` / `CRITICAL` vulnerabilities with Trivy.
- **Pushes to `main` & Weekly Schedule (`Mondays 06:00 UTC`):** Builds and scans the image first, then authenticates to Azure Container Registry via OIDC and publishes all tags using `apko publish`.
- **Manual Dispatch (`workflow_dispatch`):** Builds and scans on any branch; optionally publishes to ACR when run from `main` with `push_images: true`.

## Local Development

Build and load the `jlink` image into local Docker:

```bash
docker run --rm \
  -v "${PWD}:/work" \
  -w /work \
  cgr.dev/chainguard/apko:latest \
  build ./apko/jlink.yml bohr-uk/alpine:jlink image.tar

docker load -i image.tar
# Note: apko suffixes single-arch local tar tags with the architecture (e.g. bohr-uk/alpine:jlink-amd64)
```

Inspect generated SPDX SBOM packages:

```bash
jq -r '.packages[] | "\(.name) \(.versionInfo)"' sbom-x86_64.spdx.json
```
