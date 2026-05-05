# leilfs-container

Container image set for [LeilFS](https://github.com/leil-io/saunafs):
master, metalogger, chunkserver, cgiserver, client. Built from a shared
`leilfs-base` Ubuntu image that pulls LeilFS from the upstream APT
repository. Drives both a `docker compose` demo cluster and the
images consumed by the `leilfs-operator` on Kubernetes.

> ⚠ For testing and educational use only. Not production-ready.

## Repository layout

```
docker-compose.yml          # demo cluster (master + chunkservers + …)
leilfs-base/Dockerfile     # shared base: APT key + LeilFS repo
leilfs-master/             # master image + start script
leilfs-metalogger/         # metalogger image + start script
leilfs-chunkserver/        # chunkserver image + start script
leilfs-cgiserver/          # CGI web UI image
leilfs-client/             # FUSE client image (saunafs-mount)
volumes/                    # local bind mounts for the demo cluster
```

Each subdirectory is the build context for one image. The base image
must be built first; child images reference it via `BASE_IMAGE` build
arg.

## Pre-built images

Published on every push to `main` and on every `v*.*.*` tag:

```
ghcr.io/leil-io/leilfs-container/leilfs-base
ghcr.io/leil-io/leilfs-container/leilfs-master
ghcr.io/leil-io/leilfs-container/leilfs-metalogger
ghcr.io/leil-io/leilfs-container/leilfs-chunkserver
ghcr.io/leil-io/leilfs-container/leilfs-cgiserver
ghcr.io/leil-io/leilfs-container/leilfs-client
```

Tag = git tag.

## Build

`docker compose` is the entrypoint for both build and the demo cluster.
Build-only services live under the `build` profile so they are not
started by `docker compose up`.

```sh
# Build everything (base first, then the rest)
docker compose --profile build build

# Pin a specific LeilFS APT package version
SAUNAFS_VERSION=5.9.0 docker compose --profile build build

# Build into a custom registry namespace
LEILFS_REGISTRY=ghcr.io/myorg/leilfs-container \
LEILFS_IMAGE_TAG=dev \
  docker compose --profile build build
```

Three environment variables drive image naming and packaging:

| Variable | Purpose | Default |
|---|---|---|
| `LEILFS_REGISTRY` | Registry + owner prefix. Unset = local build (images named `leilfs-*` without prefix). | _unset_ |
| `LEILFS_IMAGE_TAG` | Image tag. | `latest` |
| `SAUNAFS_VERSION` | LeilFS APT package version to install in the image. | _latest available_ |

## Push

```sh
LEILFS_REGISTRY=ghcr.io/leil-io/leilfs-container docker compose push
```

You must be logged in (`docker login ghcr.io`) with a token that has
`write:packages`.

## Run the demo cluster

```sh
docker compose up
```

Brings up master + chunkserver(s) + metalogger + cgiserver, with the
master port mapped to `29421`. Persistent state is written to
`./volumes/`. Tear down with `docker compose down`.

## Image conventions

- Base: `ubuntu:24.04`, LeilFS APT key imported into
  `/usr/share/keyrings/saunafs-archive-keyring.gpg`, then LeilFS repo
  added.
- Each child image has a `leilfs-<role>.start.sh` that handles config
  templating from environment variables and execs the LeilFS daemon
  in the foreground.
- `LABEL org.opencontainers.image.source` is set so GHCR links the
  image to this repo.

## Contributing

Commit messages in this repo are free-form (no Conventional Commits
contract). Keep them short and descriptive. Open issues / PRs against
the upstream `leil-io/leilfs-container`.
