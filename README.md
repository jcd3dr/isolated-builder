# isolated-builder

Builder aislado y efimero: valida/construye/testea codigo de cualquier repo de `jcd3dr` (publico o privado), corriendo en runners gratuitos de GitHub Actions. No consume el VPS ni el sandbox habitual — existe para lo que esos entornos no pueden cubrir (por ejemplo, el sandbox de desarrollo no tiene Docker instalado).

**Nota para cualquier agente (Claude, ChatGPT, u otro) que llegue a este repo sin contexto previo:** este README es la fuente de verdad completa del sistema. No hace falta que el usuario te lo explique de nuevo — leelo entero antes de operar.

## Por que existe

Este repo es publico a proposito: los minutos de GitHub Actions son ilimitados y gratuitos en repos publicos, aunque el codigo que se valida (`target_repo`) sea privado. La unica pieza sensible es el secret `CROSS_REPO_PAT`, que GitHub nunca expone en texto plano ni siquiera siendo el repo publico (ver seccion Seguridad).

## Que es la "maquina efimera"

Cada corrida del workflow crea una VM Linux nueva (`ubuntu-latest`, no GPU) que GitHub destruye automaticamente al terminar el job. No queda nada vivo despues — ni contenedor, ni proceso, ni URL accesible. Es un runner de CI, no un entorno de staging: sirve para probar "esto arranca/pasa desde cero", no para dejar algo corriendo que se pueda abrir en el navegador.

## Dos modos, se pueden combinar

### Modo generico (cualquier tipo de validacion)

Para lo que no sea especificamente Docker: tests, linters, builds sin contenedor, scripts.

| Input | Ejemplo |
|---|---|
| `node_version` | `20` (opcional, instala Node antes del setup) |
| `python_version` | `3.12` (opcional) |
| `setup_commands` | `npm ci` |
| `test_commands` | `npm test` / `pytest -v` / `npm run lint` |

### Modo docker-compose

Para validar que un stack levanta de verdad: valida config, construye imagen, levanta contenedor, hace smoke test contra una URL, y destruye todo.

| Input | Default | Descripcion |
|---|---|---|
| `compose_file` | (vacio = se salta este modo) | Ruta al compose dentro del repo |
| `smoke_url` | `http://localhost:3000/health` | Endpoint a validar tras levantar el stack |

### Comun a ambos modos

| Input | Default | Descripcion |
|---|---|---|
| `target_repo` | (vacio = este repo) | `owner/repo` a validar |
| `ref` | `main` | branch/tag/commit |
| `env_vars` | (vacio) | Variables para `.env`, una por linea `KEY=VALUE` (para compose que requiere vars como `OWNER_PASSCODE`) |

## Setup para validar repos privados de jcd3dr

1. PAT fine-grained en github.com/settings/personal-access-tokens: `Contents: Read-only`, `Repository access: All repositories` (asi cubre repos nuevos sin tocar el token despues).
2. Guardarlo como secret `CROSS_REPO_PAT` en este repo (`Settings > Secrets and variables > Actions`).
3. Solo el owner del repo puede disparar `workflow_dispatch`, asi que el secret no queda expuesto por ser el repo publico.

## Seguridad — por que el repo publico no expone el token

- El **valor** del secret nunca se muestra en texto plano a nadie, ni en la UI, ni en logs (GitHub lo reemplaza por `***` automaticamente).
- Los secrets **no se copian a forks** — un fork de un tercero no tiene acceso a `CROSS_REPO_PAT`.
- Disparar `workflow_dispatch` requiere permiso de **write/colaborador** en el repo — un visitante anonimo no puede ejecutarlo, solo ver el codigo del workflow.
- Riesgo real a vigilar: no agregar colaboradores externos con permiso de write, y revisar cualquier PR externo antes de mergearlo (recien ahi se ejecutaria codigo no confiable con el secret disponible).

## Ejemplo / self-test (modo docker)

`examples/docker-compose.yml` levanta `traefik/whoami` en el puerto 3000:

```
compose_file = examples/docker-compose.yml
smoke_url    = http://localhost:3000/
```

## Disparo remoto (MCP / API)

Pensado para ser invocado por un agente via `github-mcp` (o la API de GitHub Actions directamente): `workflow_dispatch` con los inputs de arriba, despues polling de `list_workflow_jobs` / `get_job_logs` hasta que el job termine. Sin intervencion manual del usuario en el medio.
