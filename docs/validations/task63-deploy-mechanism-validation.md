# Task 63 — Define and Validate Deploy Mechanism per Environment

**Elaborado por:** Ing. Victor Gerardo Zuñiga Gamboa  
**Fecha:** 30 de abril de 2026  
**Versión:** v1.0  
**Estado:** Resolved  

---

## Objetivo

Definir y validar el mecanismo de deploy para el ambiente Staging, de forma que cada merge a `develop` dispare automáticamente una actualización en el servidor, sin intervención manual.

---

## Decisión técnica

Se optó por **GitHub Actions** como mecanismo de deploy automático, en lugar de un webhook o script manual. Las razones:

- Es la solución estándar en proyectos reales
- Es trazable — el historial de deploys queda registrado en GitHub
- Es escalable — el mismo esquema se usará para Production en su momento
- Permite agregar pasos adicionales en el futuro (migraciones, cache, etc.)

---

## Mecanismo de deploy por ambiente

| Ambiente | Mecanismo | Trigger |
|---|---|---|
| Staging | GitHub Actions (SSH) | Push/merge a `develop` |
| Production | Por definir (Task futura) | Push/merge a `main` |
| Development | Manual (`git pull`) | A demanda |

---

## Configuración realizada

### Paso 1 — Agregar llave pública a authorized_keys en el servidor

Para que GitHub Actions pueda autenticarse en el servidor vía SSH, se agregó la llave pública existente al archivo `authorized_keys`:

```bash
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Verificación:

```bash
ls -la ~/.ssh/
```

**Salida:**

```
-rw-------  1 ubizdevops ubizdevops   95 Apr 30 21:22 authorized_keys
-rw-------  1 ubizdevops ubizdevops  399 Apr 20 22:29 id_ed25519
-rw-r--r--  1 ubizdevops ubizdevops   95 Apr 20 22:29 id_ed25519.pub
-rw-------  1 ubizdevops ubizdevops  828 Apr 20 22:32 known_hosts
```

---

### Paso 2 — Configurar secrets en GitHub

Ir a:
> Repositorio → Settings → Secrets and variables → Actions → New repository secret

Se agregaron los siguientes secrets:

| Name | Descripción |
|---|---|
| `STAGING_SSH_HOST` | IP del servidor |
| `STAGING_SSH_USER` | Usuario SSH (`ubizdevops`) |
| `STAGING_SSH_KEY` | Llave privada (`~/.ssh/id_ed25519`) completa incluyendo headers |

> **Nota:** Los secrets son encriptados por GitHub y nunca se exponen en los logs del workflow.

---

### Paso 3 — Crear el workflow

Se creó una rama de trabajo siguiendo el flujo estándar:

```bash
git checkout develop
git pull origin develop
git checkout -b feature/github-actions-staging-deploy
mkdir -p .github/workflows
nano .github/workflows/deploy-staging.yml
```

Contenido del archivo `.github/workflows/deploy-staging.yml`:

```yaml
name: Deploy to Staging

on:
  push:
    branches:
      - develop

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.STAGING_SSH_HOST }}
          username: ${{ secrets.STAGING_SSH_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          script: |
            cd ~/laravel_app_staging
            git pull origin develop
```

Commit y push:

```bash
git add .github/workflows/deploy-staging.yml
git commit -m "ci: add GitHub Actions workflow for staging auto-deploy"
git push -u origin feature/github-actions-staging-deploy
```

---

### Paso 4 — Pull Request y merge hacia develop

Configuración del PR:
- **Base branch:** `develop`
- **Compare:** `feature/github-actions-staging-deploy`

Merge exitoso. El workflow se disparó automáticamente al detectar el push a `develop`.

---

## Resultado de la validación

### Ejecución del workflow

**Deploy to Staging #1** — succeeded in 9s

| Paso | Resultado | Tiempo |
|---|---|---|
| Set up job | ✅ | 1s |
| Build appleboy/ssh-action@v1.0.3 | ✅ | 3s |
| Deploy via SSH | ✅ | 2s |
| Complete job | ✅ | 0s |

### Log del deploy en el servidor

```
======CMD======
cd ~/laravel_app_staging
git pull origin develop
======END======
err: From github.com:GeraZgVic/repo-de-prueba-ultrabiz
err:  * branch            develop    -> FETCH_HEAD
err:    b4bc5f3..0ee85ed  develop    -> origin/develop
out: Updating b4bc5f3..0ee85ed
out: Fast-forward
out:  .github/workflows/deploy-staging.yml | 21 +++++++++++++++++++++
out:  hotfix-test.txt                      |  1 +
out:  2 files changed, 22 insertions(+)
out:  create mode 100644 .github/workflows/deploy-staging.yml
out:  create mode 100644 hotfix-test.txt
==============================================
✅ Successfully executed commands to all host.
==============================================
```

> **Nota:** Las líneas con prefijo `err:` corresponden a salida estándar de Git hacia stderr — no son errores reales. El deploy fue exitoso.

---

## Flujo completo validado

```
feature/* → PR → develop → GitHub Actions → deploy automático en Staging
```

- ✅ Workflow se dispara automáticamente en merge a `develop`
- ✅ Conexión SSH al servidor establecida correctamente
- ✅ `git pull origin develop` ejecutado en `~/laravel_app_staging`
- ✅ Cambios reflejados en el servidor sin intervención manual

---

## Notas para replicar en Production

Cuando se configure el deploy automático para Production, el proceso es idéntico sustituyendo:

| Variable | Staging | Production |
|---|---|---|
| Workflow | `deploy-staging.yml` | `deploy-production.yml` |
| Trigger branch | `develop` | `main` |
| Secret host | `STAGING_SSH_HOST` | `PRODUCTION_SSH_HOST` |
| Secret user | `STAGING_SSH_USER` | `PRODUCTION_SSH_USER` |
| Secret key | `STAGING_SSH_KEY` | `PRODUCTION_SSH_KEY` |
| Script path | `~/laravel_app_staging` | `~/laravel_app` |

---

*Última actualización: 30 de abril de 2026*
