# pod-ollama

The `ollama` candy of the OpenCharly candy library, as a standalone repo
(kind-prefixed naming). It ships a GPU-agnostic Ollama LLM inference server on
port `11434`, with persistent model storage.

## What it provides

Installs the Ollama binary and runs `ollama serve` as a custom supervisord service
that listens on `0.0.0.0:11434` (no systemd). Two install paths, one resulting
layout (`/usr/bin/ollama` plus the backend libraries under `/usr/lib/ollama`): on
a distro that PACKAGES ollama the package is installed and the package manager
owns the version; everywhere else the upstream release tarball is extracted to
`/usr` at the version `OLLAMA_VERSION` selects.

The candy declares NO GPU dependency — a GPU backend is an IMAGE-level composition
choice (compose `ollama-cuda` or `ollama-rocm`). What that buys differs by install
path: on a PACKAGED distro the base package is genuinely CPU-only at 66 MiB and a
backend is strictly opt-in (which keeps this layer small); on the TARBALL path the
single upstream archive already contains the CUDA backend, so a tarball consumer
pays for it whether or not any box composes `ollama-cuda`. The companion
`plugin-ollama` candy adds the `charly ollama` management CLI.

| Property | Value |
|---|---|
| Service | `ollama` (`ollama serve`, `restart: always`, `scope: system`) |
| Port | `11434` |
| Requires | `layer-supervisord` |
| Volume | `models` at `~/.ollama` |
| Env | `OLLAMA_HOST=0.0.0.0`, `OLLAMA_MODELS=~/.ollama/models` |
| `env_provide` | `OLLAMA_HOST=http://{{.ContainerName}}:11434` |
| Var | `OLLAMA_VERSION=v0.32.14` (tarball path only) |
| Alias | `ollama` |

`OLLAMA_VERSION` is PINNED to a release tag and must stay one: the download cache
is content-addressed by the sha256 of the URL, so a `latest` URL would hash to one
key forever and serve the first tarball it ever fetched.

## How to use it

```bash
charly box build ollama
charly config ollama
charly start ollama
charly shell ollama -c "ollama pull llama3"
charly shell ollama -c "ollama run llama3 'Hello'"
```

11434 is the port INSIDE the container, not necessarily on your host — a deploy
can publish it elsewhere. Read it from `charly status ollama`:

```bash
charly status ollama
#   Ports: 34117:11434/tcp   <- HOST:CONTAINER, so the host port is 34117
charly ollama list --server http://127.0.0.1:34117
```

Endpoint resolution: `--server` flag > `OLLAMA_HOST` env > `http://127.0.0.1:11434`.
The candy's own `check:` steps assert the binary, the `/api/tags` `200`,
`ollama --version`, `ollama list`, and the compiled-in `charly ollama`
management CLI (`help` / `version` / `list`).

## Layout

- `charly.yml` — the `ollama:` candy entity plus its `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-Calver history.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-ollama:ollama` — the server, the GPU-agnostic
  declaration, and the two install paths.
- `/charly-ollama:ollama-cli` — the compiled-in `charly ollama` management CLI.
- `/charly-ollama:ollama-layer` — the ollama candy surface.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
