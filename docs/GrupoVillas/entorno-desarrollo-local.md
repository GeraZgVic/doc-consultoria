# Entorno de Desarrollo Local — Plataforma Corte Láser GrupoVillas
**Versión:** 1.1  
**Fecha:** Mayo 2026  
**Desarrollador:** Ing. Victor Gerardo Zúñiga Gamboa  
**Documento:** Uso interno de desarrollo / Confidencial

---

## Sistema Base

| Herramienta | Versión confirmada | Notas |
|---|---|---|
| Windows | 10 / 11 | Host principal |
| WSL2 | — | Entorno Linux sobre Windows |
| Distro WSL | Debian | Base del entorno de desarrollo |
| Docker Desktop | — | Con integración WSL2 activa |

---

## Herramientas Principales

| Herramienta | Versión confirmada | Instalación |
|---|---|---|
| PHP | 8.4.20 | `sudo apt install php8.4` |
| Composer | 2.9.7 | [getcomposer.org](https://getcomposer.org) |
| Node.js | 22.22.2 | Recomendado vía `nvm` |
| npm | 10.9.7 | Incluido con Node |

---

## Extensiones PHP Requeridas

Todas deben estar instaladas en el PHP del WSL. Instalar con:

```bash
sudo apt install php8.4-mysql php8.4-mbstring php8.4-xml php8.4-curl php8.4-zip php8.4-bcmath
```

| Extensión | Motivo |
|---|---|
| `pdo_mysql` | Conexión a MySQL — requerida por Laravel + Eloquent |
| `mbstring` | Manipulación de strings multibyte — requerida por Laravel 12 |
| `xml` | Procesamiento XML — requerida por Laravel y DomPDF |
| `curl` | Peticiones HTTP — requerida por Laravel y Composer |
| `zip` | Manejo de archivos zip — requerida por Composer |
| `bcmath` | Aritmética de precisión — requerida por Laravel |

### Verificar extensiones instaladas

```bash
php -m | grep -E "pdo|mbstring|xml|curl|zip|bcmath"
```

---

## Base de Datos — Contenedor Docker

Se usa un contenedor MySQL 8 en lugar de instalación local.

### Levantar el contenedor

```bash
docker run -d \
  --name grupovillas-mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=grupovillas \
  -e MYSQL_USER=grupovillas \
  -e MYSQL_PASSWORD=secret \
  -p 3306:3306 \
  mysql:8
```

### Verificar que está corriendo

```bash
docker ps
```

### Detener / reiniciar

```bash
docker stop grupovillas-mysql
docker start grupovillas-mysql
```

> **Nota:** El contenedor no se elimina al detenerlo. Los datos persisten mientras no se ejecute `docker rm`.

---

## Configuración `.env` para desarrollo local

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=grupovillas
DB_USERNAME=grupovillas
DB_PASSWORD=secret
```

---

## Requisitos del Servidor de Producción (cPanel)

### Ajustes PHP — Solicitar a Carlos vía WHM

| Parámetro | Valor actual | Valor requerido | Estado |
|---|---|---|---|
| PHP | 8.2.31 | 8.2 o superior | ✅ |
| OPcache | Activo | Activo | ✅ |
| `memory_limit` | 128M | 256M | ⏳ Pendiente Carlos |
| `upload_max_filesize` | 2M | 5M | ⏳ Pendiente Carlos |
| `post_max_size` | 8M | 16M | ⏳ Pendiente Carlos |
| `max_execution_time` | 0 | 0 (sin límite) | ✅ |

### Extensiones PHP — Habilitar/instalar desde WHM

Solicitar a Carlos que habilite las siguientes extensiones para la cuenta del proyecto:

- `pdo_mysql`
- `mbstring`
- `xml`
- `curl`
- `zip`
- `bcmath`

---

## Checklist de Configuración Inicial

- [x] WSL2 instalado y funcionando
- [x] Docker Desktop con integración WSL2 activa
- [x] PHP 8.4.20 instalado en WSL
- [x] Extensiones PHP instaladas (`pdo_mysql`, `mbstring`, `xml`, `curl`, `zip`, `bcmath`)
- [x] Composer 2.9.7 instalado
- [x] Node.js 22.22.2 y npm 10.9.7 instalados
- [x] Contenedor MySQL levantado (`grupovillas-mysql`)
- [x] Proyecto Laravel 12 creado (`composer create-project laravel/laravel:^12.0 grupovillas-laser`)
- [x] `.env` configurado con credenciales MySQL
- [x] `php artisan migrate` ejecutado sin errores
- [x] Livewire 4 instalado
- [x] Mary UI instalado
- [x] DomPDF ^3.0 instalado
- [x] Spatie Permission instalado
- [x] Breeze con stack Livewire instalado
- [ ] npm install ejecutado
- [x] Inicializar repositorio Git y subir a GitHub

---

## Dependencias del Proyecto

### Orden de instalación y comandos correctos

> **Importante:** Composer no acepta constraints tipo `^6.x` — usar siempre `^6.0`. El `.x` causa error de parsing.

```bash
# 1. Livewire
composer require livewire/livewire:^4.0

# 2. Mary UI
composer require robsontenorio/mary

# 3. DomPDF — usar ^3.0, la versión ^2.x no es compatible con Laravel 12
composer require barryvdh/laravel-dompdf:^3.0

# 4. Spatie Permission
composer require spatie/laravel-permission:^6.0

# 5. Breeze con stack Livewire (autenticación)
composer require laravel/breeze --dev
php artisan breeze:install livewire
```

### Versiones instaladas confirmadas

| Paquete | Versión |
|---|---|
| `livewire/livewire` | ^4.0 |
| `robsontenorio/mary` | ^2.8 |
| `barryvdh/laravel-dompdf` | ^3.0 |
| `spatie/laravel-permission` | ^6.0 |
| `laravel/breeze` | ^2.4 |

### npm

```bash
npm install
npm install -D @tailwindcss/forms@^0.5.x
```

---

## Repositorio Git

### Repositorio actual (temporal — cuenta personal)

```bash
git remote add origin git@github.com:GeraZgVic/GrupoVillas.git
```

> Rama principal: `main`
> Primer commit: `chore: initial project setup`

### Comandos de inicialización (referencia)

```bash
git init
git add .
git commit -m "chore: initial project setup"
git branch -m master main        # Git crea 'master' por defecto, renombrar a 'main'
git remote add origin <url>
git push -u origin main
```

### Cambiar al repositorio definitivo (cuando esté disponible)

```bash
git remote set-url origin git@github.com:repo-definitivo/GrupoVillas.git
git push -u origin main
```

> El historial de commits se preserva completamente al cambiar el remoto.

---

*Documento de uso interno — Desarrollo Plataforma Corte Láser GrupoVillas — v1.1 Mayo 2026*
