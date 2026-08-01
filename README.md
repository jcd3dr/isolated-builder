# isolated-builder

Builder aislado y efimero para validar, construir, levantar, probar (smoke test) y destruir stacks docker-compose, corriendo en runners gratuitos de GitHub Actions (no consume recursos del VPS).

## Como funciona

Repo publico -> minutos de Actions ilimitados y gratuitos, aunque el codigo construido (`target_repo`) sea privado.

Workflow: `.github/workflows/isolated-builder.yml` (pendiente de subir por permisos del token).
