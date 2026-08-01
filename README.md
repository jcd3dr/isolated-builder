# isolated-builder

Builder aislado y efimero para validar, construir, levantar, probar (smoke test) y destruir stacks docker-compose, corriendo 100% en runners gratuitos de GitHub Actions (no consume recursos del VPS).

## Como funciona

Este repo es publico, por lo que los minutos de Actions son ilimitados y gratuitos, aunque el codigo que se construye (`target_repo`) sea privado.

El workflow `.github/workflows/isolated-builder.yml` se dispara manualmente (`workflow_dispatch`) con estos inputs:

| Input | Default | Descripcion |
|---|---|---|
| `target_repo` | (vacio = este repo) | `owner/repo` a construir |
| `ref` | `main` | branch/tag/commit |
| `compose_file` | `docker-compose.yml` | ruta al compose dentro del repo |
| `smoke_url` | `http://localhost:3000/health` | endpoint a validar tras levantar el stack |

Pasos que ejecuta, siempre en una VM efimera (`ubuntu-latest`) que GitHub destruye automaticamente al terminar el job:

1. Checkout del repo objetivo (usa `CROSS_REPO_PAT` si es un repo externo/privado, o el `GITHUB_TOKEN` por defecto si es este mismo repo)
2. `docker compose config` (validacion)
3. `docker compose build`
4. `docker compose up -d`
5. Poll + smoke test contra `smoke_url`
6. `docker compose down -v --remove-orphans` (siempre, incluso si el smoke test falla)

## Setup para construir repos privados externos

1. Genera un PAT fine-grained con scope `contents:read` limitado a los repos que quieras poder construir.
2. Agregalo como secret del repo: `Settings > Secrets and variables > Actions > New repository secret` con nombre `CROSS_REPO_PAT`.
3. Solo el owner puede disparar `workflow_dispatch`, asi que el secret no queda expuesto por ser el repo publico.

## Ejemplo / self-test

`examples/docker-compose.yml` levanta `traefik/whoami` en el puerto 3000, para probar el pipeline completo sin depender de otro repo:

```
compose_file = examples/docker-compose.yml
smoke_url    = http://localhost:3000/
```

## Disparo remoto (MCP)

Pensado para ser invocado por un agente (Claude u otro) via `github-mcp` / API de GitHub Actions (`workflow_dispatch` + polling de logs), sin intervencion manual.
