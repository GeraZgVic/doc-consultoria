# Task 61 — Configure Staging Environment in cPanel (Minimal)

**Elaborado por:** Ing. Victor Gerardo Zuñiga Gamboa  
**Fecha:** 30 de abril de 2026  
**Versión:** v1.0  
**Estado:** Resolved  

---

## Objetivo

Configurar el ambiente Staging en el servidor cPanel, creando el subdominio correspondiente, clonando el repositorio en su propia ruta, configurando el symlink y dejando la aplicación sirviendo desde la rama `develop`.

---

## Paso 1 — Crear el subdominio en cPanel

Desde el panel de administración de cPanel:

1. Ir a **Domains → Create A New Domain**
2. Configurar con los siguientes valores:

| Campo | Valor |
|---|---|
| Type | Registered Domain |
| Domain | `staging.ubiz-devops.xyz` |
| Share document root | ❌ Desactivado |
| Document Root | `staging.ubiz-devops.xyz` |

3. Click en **Submit**

**Resultado:** cPanel crea automáticamente la carpeta:

```
/home/ubizdevops/staging.ubiz-devops.xyz
```

> **Nota:** Al crear el subdominio aparece un aviso indicando que HTTPS/AutoSSL no está activo. Se ignora en esta etapa de validación mínima.

---

## Paso 2 — Clonar el repositorio

El proyecto se clona **fuera** del Document Root del subdominio, siguiendo el mismo modelo usado en Production (Modelo 2 — symlink).

Desde el directorio home:

```bash
cd ~
git clone git@github.com:GeraZgVic/repo-de-prueba-ultrabiz.git laravel_app_staging
```

**Salida:**

```
Cloning into 'laravel_app_staging'...
remote: Enumerating objects: 77, done.
remote: Counting objects: 100% (77/77), done.
remote: Compressing objects: 100% (57/57), done.
remote: Total 77 (delta 1), reused 77 (delta 1), pack-reused 0
Receiving objects: 100% (77/77), 81.78 KiB | 1.49 MiB/s, done.
Resolving deltas: 100% (1/1), done.
```

---

## Paso 3 — Cambiar a la rama develop

Staging debe servir siempre desde la rama `develop`. La rama ya existe en el remoto, por lo que se crea localmente siguiendo la remota:

```bash
cd laravel_app_staging
git checkout -b develop origin/develop
```

Verificar:

```bash
git branch
```

**Salida esperada:**

```
* develop
  main
```

---

## Paso 4 — Instalar dependencias PHP

```bash
composer install --no-dev --optimize-autoloader
```

---

## Paso 5 — Configurar el symlink

El Document Root del subdominio se reemplaza con un symlink que apunta a `laravel_app_staging/public`, siguiendo el mismo modelo que Production.

### Regla de oro

> **El symlink siempre reemplaza la carpeta del subdominio, nunca vive dentro de ella.**

```bash
cd ~
mv ~/staging.ubiz-devops.xyz ~/staging.ubiz-devops.xyz_backup
ln -s ~/laravel_app_staging/public ~/staging.ubiz-devops.xyz
```

### Verificación

```bash
ls -ld ~/staging.ubiz-devops.xyz
```

**Salida esperada:**

```
lrwxrwxrwx 1 ubizdevops ubizdevops 43 Apr 30 18:29 /home/ubizdevops/staging.ubiz-devops.xyz -> /home/ubizdevops/laravel_app_staging/public
```

### Cómo revertir si algo sale mal

```bash
rm ~/staging.ubiz-devops.xyz
mv ~/staging.ubiz-devops.xyz_backup ~/staging.ubiz-devops.xyz
```

---

## Paso 6 — Crear y configurar el archivo `.env`

```bash
cd ~/laravel_app_staging
touch .env
nano .env
```

Contenido mínimo requerido:

```env
APP_NAME=Laravel
APP_ENV=staging
APP_DEBUG=true
APP_URL=https://staging.ubiz-devops.xyz
```

Generar la clave de aplicación:

```bash
php artisan key:generate
```

**Salida:**

```
INFO  Application key set successfully.
```

---

## Resultado final

| Aspecto | Resultado |
|---|---|
| Subdominio creado | ✅ `staging.ubiz-devops.xyz` |
| Repositorio clonado | ✅ `~/laravel_app_staging/` |
| Rama activa | ✅ `develop` |
| Dependencias instaladas | ✅ |
| Symlink configurado | ✅ `~/staging.ubiz-devops.xyz → ~/laravel_app_staging/public` |
| `.env` configurado | ✅ `APP_ENV=staging` |
| Aplicación sirviendo | ✅ Validado vía navegador |

---

## Estructura final en el servidor

```
~/laravel_app/                    ← Production (rama main)
~/laravel_app_staging/            ← Staging (rama develop)
~/public_html                     → ~/laravel_app/public
~/staging.ubiz-devops.xyz         → ~/laravel_app_staging/public
```

---

## Notas para replicar en otro ambiente

Para replicar este proceso en un nuevo ambiente (por ejemplo Development), seguir los mismos pasos sustituyendo:

| Variable | Staging | Development |
|---|---|---|
| Subdominio | `staging.ubiz-devops.xyz` | `dev.ubiz-devops.xyz` |
| Carpeta del proyecto | `laravel_app_staging` | `laravel_app_dev` |
| Rama | `develop` | variable (`feature/*`, `fix/*`, `hotfix/*`) |
| `APP_ENV` | `staging` | `local` |

---

*Última actualización: 30 de abril de 2026*
