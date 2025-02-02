# Personal Container Images

## Distroless Python: [`ghcr.io/cueaz/python`](//github.com/cueaz/library/pkgs/container/python)

Compared to [`gcr.io/distroless/python3`](//github.com/GoogleContainerTools/distroless/tree/main/python3):

-   Uses the latest Python release
-   Smaller image size

As of 2025-02-03:

| Image                     | Python Version | Image Size |             Platform              |
| ------------------------- | :------------: | :--------: | :-------------------------------: |
| gcr.io/distroless/python3 | 3.11 (debian)  |   ~65MB    | amd64, arm64, arm, s390x, ppc64le |
| ghcr.io/cueaz/python      | 3.13 (latest)  |   ~46MB    |           amd64, arm64            |

## MLflow + Litestream: [`ghcr.io/cueaz/mlflow`](//github.com/cueaz/library/pkgs/container/mlflow)

-   No external database required
-   Requires only S3-compatible storage
-   Exposed internal port is 8000

### Environment Setup Example

```bash
docker run -p 8000:8000 \
    -e MLFLOW_S3_ENDPOINT_URL=https://s3.endpoint:port \
    -e MLFLOW_ARTIFACTS_DESTINATION=s3://bucket \
    -e AWS_ACCESS_KEY_ID=user \
    -e AWS_SECRET_ACCESS_KEY=password \
    ghcr.io/cueaz/mlflow
```
