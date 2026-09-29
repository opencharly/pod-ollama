# AGENTS.md — pod-ollama

Standalone candy repo for the `ollama` candy — a GPU-agnostic Ollama LLM
inference server on port `11434` with persistent model storage. The entire candy
lives in `charly.yml` at the repo root.

Canonical files:

- `charly.yml` — the `ollama:` candy entity (description, `require`, `var`,
  `distro`, `env`, `env_provide`, `port`, `volume`, `alias`, `service`, `plan`)
  and its `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-Calver history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-ollama:ollama` — the owning skill: the server, the GPU-agnostic
  declaration, the two install paths, and the box-level backend composition. Load
  before editing, building, deploying, or troubleshooting this candy.
- `/charly-ollama:ollama-cli` — the compiled-in `charly ollama` management CLI
  the candy's checks exercise.
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (this candy is
  composed into a box; tree-position nesting, volumes, ports).
- `/charly-check:check` — the check/R10 framework: the `check:` step verbs and
  `charly check run <bed>`.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs, `env_provide`, service declarations).
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The live R10 witness is a composing box's `check` bed; the candy's own `check:`
  steps assert the `/usr/bin/ollama` binary, the running service's `/api/tags`
  `200`, `ollama --version`, `ollama list` exit 0, and the compiled-in
  `charly ollama` CLI (`help` / `version` / `list`) against the published port.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.

## Modify this repo

- Edit the `ollama:` candy entity in `charly.yml`; the `skill:` entity in the
  same file is the owning skill's source — a candy change and its skill change
  land together.
- `OLLAMA_VERSION` MUST stay a concrete release tag, never `latest`: the
  `download:` verb content-addresses its cache by the sha256 of the URL, so a
  `latest` URL would serve the first tarball ever fetched forever.
- The install step is capability-gated (`unless_exists: /usr/bin/ollama`), not
  distro-gated; a distro that gains an ollama package is picked up with no edit.
- GPU backends are IMAGE-level composition choices (`ollama-cuda` / `ollama-rocm`),
  never a `require:` on this candy.
- The `skill:` entity is the source for `/charly-ollama:ollama`; never edit the
  generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
