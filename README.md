# Langfuse image builds

CI that builds the self-hosted Langfuse **web** image from upstream source with
a custom base path baked in at build time, and publishes it to GHCR.

- Image: `ghcr.io/bermuglu/langfuse-web` (public — anonymous pulls)
- Base path: repository variable `NEXT_PUBLIC_BASE_PATH` (currently `/langfuse`)

## How it works

- **Check upstream releases** (daily 06:30 UTC, or manually) finds the newest
  `v3.*` and `v4.*` release tags of `langfuse/langfuse` and builds any that are
  missing from the registry — or whose `langfuse.base-path` label does not match
  the current variable. The registry is the only state; this repo never changes
  after setup.
- **Build web image** (manual, or called by the check) builds one upstream tag
  or commit SHA with the stock `web/Dockerfile` (linux/amd64) and pushes it as
  `ghcr.io/bermuglu/langfuse-web:<tag>` (or `sha-<short>` for a bare commit SHA).

## Usage

- Deployments pull a pinned tag, no credentials required:
  `docker pull ghcr.io/bermuglu/langfuse-web:v4.86.0`
- Build a specific version or commit: Actions → *Build web image* → Run
  workflow, enter the tag (e.g. `v4.85.0`) or a full commit SHA.
- Change the base path: edit the `NEXT_PUBLIC_BASE_PATH` repository variable.
  The next check rebuilds both tip tags with the new value automatically.

Only the web image needs a custom build, because the base path is inlined into
static assets at build time. The worker and all other self-hosted components
keep using the official prebuilt images.
