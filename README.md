<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/icon-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="assets/icon-light.svg">
    <img src="assets/icon.svg" alt="Stash logo" width="128">
  </picture>
</p>

<h1 align="center">aac-stash</h1>

<p align="center">Self-hosted deployment definition for Stash, a self-hosted media organizer.</p>

<p align="center">
  <a href="https://github.com/stashapp/stash"><img alt="Upstream" src="https://img.shields.io/badge/upstream-stashapp%2Fstash-blue?logo=github"></a>
  <img alt="Image" src="https://img.shields.io/badge/image-stashapp%2Fstash%3Av0.31.1-2496ED?logo=docker&logoColor=white">
  <a href="https://github.com/stashapp/stash/blob/develop/LICENSE"><img alt="License" src="https://img.shields.io/badge/license-AGPL--3.0-green"></a>
</p>

## About

Stash is a self-hosted web application written in Go. It organizes and serves a local
collection of video and image files: it scans your media, gathers metadata for scenes and
performers from configurable sources, and lets you tag, search and browse the collection
through a web interface. Stash can generate previews and sprites for videos, which are
stored in the `generated` and `cache` volumes.

Key features (as provided by upstream):

- Scanning and organization of video and image libraries
- Metadata scraping from internet sources and community-built plugins
- Tagging, search and statistics for performers, tags and studios
- Support for a wide range of video and image formats
- Generated previews and sprites for faster browsing

## Upstream

- Project: <https://github.com/stashapp/stash>
- Documentation: <https://docs.stashapp.cc>
- Container image: `stashapp/stash:v0.31.1` (Docker Hub)
- Database dependency: `postgres:18` (deployed as `aac-stash-db`)
- License: AGPL-3.0 (upstream project)

## Quick start (Docker Compose)

The Compose file is rendered from `service.yml` by the App-as-Code template engine. For a
manual run, the external network used for routing must exist first:

```bash
docker network create services-secured

# Provide the secrets listed under "Configuration" in .env (see below)
docker compose up -d
```

This deployment requests the NVIDIA container runtime (`runtime: nvidia` with GPU
reservations) in `service.yml`. A host without an NVIDIA GPU and container toolkit must
adjust or remove that section before deploying.

Secrets are not stored in this repository. Provide them through the deployment pipeline
or create the `.env` file yourself (see below). Do not commit them.

## Configuration

Environment variables (defaults from `service.yml`; `CHANGE_ME` must be set per deployment):

| Variable | Default | Description |
|---|---|---|
| `STASH_GENERATED` | `/generated` | Directory for generated content (previews, sprites) |
| `STASH_CACHE` | `/cache` | Cache directory |
| `DB_PASSWORD` | `CHANGE_ME` (secret) | Database password used by the deployment |
| `POSTGRES_DB` | `stash` (secret) | Database name of the `postgres` dependency |
| `POSTGRES_USER` | `stash` (secret) | Database user of the `postgres` dependency |
| `POSTGRES_PASSWORD` | `CHANGE_ME` (secret) | Database password of the `postgres` dependency |

Ports:

| Port | Protocol | Purpose |
|---|---|---|
| `9999` | TCP | Web interface (`web`) |

Volumes:

| Mount | Description |
|---|---|
| `config` -> `/root/.stash` | Stash application configuration |
| `data` -> `/data` | Main data directory |
| `metadata` -> `/metadata` | Metadata storage |
| `cache` -> `/cache` | Cache directory |
| `blobs` -> `/blobs` | Blob storage |
| `generated` -> `/generated` | Generated content (previews, sprites) |
| `local_time` -> `/etc/localtime` (read-only) | Host time zone (bind mount) |
| `nsfw_media` -> `/nsfw` | Media library (bind mount of a host directory) |
| `db` -> `/var/lib/postgresql` | PostgreSQL data (database service) |

## Deploy with App-as-Code

1. Edit `service.yml`. It is the single source of truth for image, ports, environment,
   volumes, dependencies and per-stage overrides. Do not edit the rendered output.
2. Pushing to `main` (or a configured stage branch) runs the service pipeline, which renders
   the manifests with `aac-template-engine` and triggers the deployment.
3. Releases of this definition are published under <https://github.com/app-as-code/aac-stash>.

## Credits and license

Stash is developed by the Stash project and licensed under AGPL-3.0. This repository only
contains the deployment definition for it and is not a fork of the application. The logo
is taken from the selfh.st icon collection; see [`assets/ICON_SOURCE.md`](assets/ICON_SOURCE.md)
for icon sources and licenses.
