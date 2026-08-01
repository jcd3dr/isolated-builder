# AGENTS.md

This repo hosts a single reusable GitHub Actions workflow (`.github/workflows/isolated-builder.yml`) that any agent can dispatch to validate/build/test code from any of jcd3dr's repos (public or private) in an ephemeral, disposable GitHub-hosted runner.

**Read `README.md` first.** It documents what this system is, why it's public, the two operating modes, and how to invoke it. This file is only for an agent that is going to modify this repo's own code — not for an agent that just wants to use the builder against some other repo.

## Before changing the workflow

- The existing `workflow_dispatch` inputs (`target_repo`, `ref`, `env_vars`, `node_version`, `python_version`, `setup_commands`, `test_commands`, `compose_file`, `smoke_url`) are a public API — other agents/automations call this workflow by these exact input names. Treat renaming or removing an existing input as a breaking change; don't do it without checking with the owner first.
- Adding a new optional input with a safe empty/no-op default is always fine.
- Keep the docker-compose mode and the generic (`setup_commands`/`test_commands`) mode independently functional — they're designed to be usable together or separately, gated by whether `compose_file` / `test_commands` are non-empty.

## How to verify a change actually works

There is no local test suite — this repo IS the test harness, so a YAML edit that merely looks correct is not evidence it works.

1. Commit directly to `main` — this repo has no branch/PR workflow, it's solo-maintained infra.
2. Dispatch a real run (`workflow_dispatch`) against a known-good target — `examples/docker-compose.yml` in this repo for the docker mode, or any repo you already know passes — and confirm every step actually succeeds (check job/step status, not just that the trigger call returned 204).
3. Do not report a change as done without having triggered and confirmed a live run. This repo has already failed silently in ways that looked fine at a glance (missing `Workflows` permission on the token gave a bare 404; a missing `CROSS_REPO_PAT` secret gave `remote: Repository not found`) — both only surfaced by actually running it.

## Conventions

- No branch protection, no PR review on this repo — commit directly to `main`.
- `CROSS_REPO_PAT` is a repo secret: never print it, never request its value, never put it in a commit or log line.
- This repo is intentionally public so Actions minutes stay free/unlimited even when validating private target repos — don't propose making it private as a "cleanup."
