[ubizdevops@103-138-188-103 laravel_app]$ cat despliegue-laravel-cpanel.md
# Despliegue de Laravel en servidor cPanel

> Documentación paso a paso para levantar una aplicación Laravel en un servidor cPanel compartido.

---

## Requisitos del servidor

| Componente | Versión detectada |
|---|---|
| PHP (CLI) | 8.1.34 |
| Zend Engine | v4.1.34 |
| ionCube PHP Loader | v12.0.5 |
| Zend OPcache | v8.1.34 |
| Composer | 2.9.7 |
| Sistema operativo | Linux (cPanel) |

---

## Paso 1 — Verificar la versión de PHP

```bash
php -v
```

**Salida esperada:**

```
PHP 8.1.34 (cli) (built: Jan 9 2026 00:00:00) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.1.34, Copyright (c) Zend Technologies
    with the ionCube PHP Loader v12.0.5, Copyright (c) 2002-2022, by ionCube Ltd.
    with Zend OPcache v8.1.34, Copyright (c), by Zend Technologies
```

---

## Paso 2 — Instalar Composer

Composer es el gestor de dependencias de PHP requerido por Laravel.

### 2.1 Descargar el instalador

Desde el directorio home del usuario:

```bash
cd ~
curl -sS https://getcomposer.org/installer -o composer-setup.php
```

### 2.2 Ejecutar el instalador

> **Nota:** En servidores cPanel, la directiva `allow_url_fopen` puede estar deshabilitada para el CLI. Se pasa como parámetro en línea para evitar modificar el `php.ini` global.

```bash
php -d allow_url_fopen=On composer-setup.php
```

**Salida esperada:**

```
All settings correct for using Composer
Downloading...

Composer (version 2.9.7) successfully installed to: /home/ubizdevops/composer.phar
Use it: php composer.phar
```

### 2.3 Verificar la instalación

```bash
php composer.phar -v
```

Debe mostrar el logo ASCII de Composer junto con la versión y los comandos disponibles.

---

## Paso 3 — Hacer Composer accesible globalmente

Para poder invocar `composer` desde cualquier directorio sin necesidad de escribir `php composer.phar`, se mueve el binario a `~/bin` y se agrega esa ruta al `PATH`.

### 3.1 Confirmar el shell en uso

```bash
echo $SHELL
```

**Salida esperada:**

```
/bin/bash
```

### 3.2 Mover el binario y hacerlo ejecutable

```bash
mkdir -p ~/bin
mv ~/composer.phar ~/bin/composer
chmod +x ~/bin/composer
```

Esto coloca el binario en `/home/<usuario>/bin/composer` y le otorga permisos de ejecución.

### 3.3 Agregar `~/bin` al PATH

Editar el archivo `~/.bashrc`:

```bash
nano ~/.bashrc
```

Agregar al final del archivo (si no está ya presente):

```bash
export PATH=$HOME/bin:$PATH
```

El archivo `~/.bashrc` resultante debe tener una estructura similar a esta:

```bash
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# User specific environment
if ! [[ "$PATH" =~ "$HOME/.local/bin:$HOME/bin:" ]]
then
    PATH="$HOME/.local/bin:$HOME/bin:$PATH"
fi
export PATH

# ...otras configuraciones...

export PATH=$HOME/bin:$PATH
```

> **Nota:** En cPanel, el archivo `~/.bash_profile` carga automáticamente `~/.bashrc`, por lo que basta con modificar este último.

### 3.4 Aplicar los cambios

```bash
source ~/.bashrc
```

### 3.5 Verificar que Composer es accesible globalmente

```bash
composer -V
```

**Salida esperada:**

```
Composer version 2.9.7 2026-04-14 13:31:52
PHP version 8.1.34 (/opt/cpanel/ea-php81/root/usr/bin/php)
Run the "diagnose" command to get more detailed diagnostics output.
```

A partir de este punto, el comando `composer` estará disponible en cualquier directorio de la sesión.

---

## Paso 4 — Verificar Git

Git viene preinstalado en el servidor. Se verifica la versión disponible antes de usarlo para gestionar el proyecto Laravel.

```bash
git --version
```

**Salida esperada:**

```
git version 2.48.2
```

No se requiere instalación adicional. Git estará disponible para inicializar el repositorio del proyecto y vincularlo a un repositorio remoto en los pasos siguientes.

---

## Paso 5 — Análisis de arquitectura: dónde colocar el proyecto

Antes de crear el proyecto Laravel, es necesario resolver un conflicto estructural propio de los hostings cPanel: **el servidor web apunta a `public_html`, pero Laravel espera que el servidor apunte a `proyecto/public`**.

Esta sección documenta el análisis realizado para elegir el modelo de despliegue más limpio.

### El conflicto

| Expectativa de Laravel | Realidad en cPanel |
|---|---|
| El servidor apunta a `/proyecto/public` | El servidor apunta fijo a `/public_html` |

### Modelos evaluados

#### ❌ Modelo 1 — Copia de `/public` dentro de `public_html` (clásico, no recomendado)

```
~/laravel_app/       ← proyecto completo
~/public_html/       ← copia manual de /public
```

**Problemas:**
- Se duplican archivos entre `laravel_app/public` y `public_html`
- Hay que modificar `index.php` para que apunte al proyecto padre
- Cada deploy requiere sincronizar manualmente la carpeta `public`
- Propenso a errores y difícil de mantener con Git

#### ✅ Modelo 2 — Symlink de `public_html` hacia `laravel_app/public` (modelo confirmado)

```
~/public_html  →  ~/laravel_app/public   (symlink)
```

**Ventajas:**
- No se duplican archivos
- Laravel conserva su estructura natural
- No es necesario copiar manualmente `public` en cada despliegue
- Evita modificaciones manuales recurrentes sobre `index.php`
- Se integra mejor con un flujo de despliegue basado en Git

**Estado de validación:**
- ✅ Symlink funcional a nivel de filesystem
- ✅ Symlink funcional dentro de `public_html`
- ✅ Symlink servido correctamente por Apache vía HTTP

> **Conclusión:** En este servidor cPanel, el Modelo 2 es viable y queda confirmado como estrategia de despliegue aplicable al dominio principal.

#### ✅ Modelo 3 — Document root apuntando directamente a `laravel_app/public` (ideal)

```
dominio  →  ~/laravel_app/public   (document root directo)
```

**Ventajas:**
- Laravel funciona exactamente como fue diseñado
- Sin duplicación de archivos
- Sin modificaciones a `index.php`
- Git deploy limpio y predecible

**Condición:** Depende de si cPanel permite cambiar el document root (disponible en dominios adicionales y subdominios, generalmente no en el dominio principal).

### Validación del estado actual del servidor

Se ejecutaron los siguientes comandos para entender la estructura real del hosting:

```bash
pwd
ls -ld ~/public_html ~/www
readlink ~/public_html
readlink ~/www
```

**Salida:**

```
/home/ubizdevops
drwxr-x--- 4 ubizdevops nobody     4096 Apr 15 22:57 /home/ubizdevops/public_html
lrwxrwxrwx 1 ubizdevops ubizdevops   11 Apr 14 02:51 /home/ubizdevops/www -> public_html
public_html
```

```bash
namei -l ~
namei -l ~/public_html
```

**Salida:**

```
f: /home/ubizdevops
dr-xr-xr-x root       root       /
drwxr-xr-x root       root       home
drwx--x--x ubizdevops ubizdevops ubizdevops

f: /home/ubizdevops/public_html
dr-xr-xr-x root       root       /
drwxr-xr-x root       root       home
drwx--x--x ubizdevops ubizdevops ubizdevops
drwxr-x--- ubizdevops nobody     public_html
```

### Conclusiones de la validación

| Hallazgo | Detalle |
|---|---|
| `public_html` es un directorio real | No es symlink; es la raíz web fija del dominio principal |
| `www` apunta a `public_html` | Es un alias estándar de cPanel |
| Esquema activo | Clásico cPanel — sin redirección de document root |

> **Decisión tomada:** Aunque el Modelo 3 sigue siendo ideal a nivel teórico, en este servidor ya se confirmó que el Modelo 2 (symlink de `public_html` hacia `laravel_app/public`) funciona correctamente tanto a nivel de filesystem como vía HTTP. Por lo tanto, el despliegue continuará con el Modelo 2 por ser el más adecuado y práctico para este caso.

---

## Paso 6 — Validar viabilidad de symlinks para despliegue

Antes de evaluar el Modelo 2, se validó si el sistema de archivos de la cuenta permite crear y leer enlaces simbólicos en general.

### Prueba realizada

```bash
mkdir -p ~/test_symlink_target
echo "ok" > ~/test_symlink_target/test.txt
ln -s ~/test_symlink_target ~/test_symlink_link
ls -l ~/test_symlink_link
cat ~/test_symlink_link/test.txt
```

**Salida:**

```
lrwxrwxrwx 1 ubizdevops ubizdevops 36 Apr 20 17:13 /home/ubizdevops/test_symlink_link -> /home/ubizdevops/test_symlink_target
ok
```

### Conclusiones

| Aspecto | Resultado |
|---|---|
| Creación de symlinks en el home | ✅ Permitida |
| Lectura de archivos a través del symlink | ✅ Funciona correctamente |
| Bloqueo de symlinks a nivel de usuario | ✅ No existe bloqueo |

> **Importante:** Esta prueba confirma que el sistema de archivos no bloquea los symlinks a nivel de cuenta. Sin embargo, **no garantiza** que Apache vaya a servir contenido web a través de ellos. Eso depende de si la directiva `FollowSymLinks` está habilitada en la configuración de Apache/cPanel.

---

### Validación adicional — Symlinks dentro de `public_html`

Se realizó una prueba para verificar si es posible crear y leer enlaces simbólicos directamente dentro del directorio público del dominio.

### Prueba realizada

```bash
mkdir -p ~/test_public_target
echo "public test" > ~/test_public_target/probe.txt

ln -s ~/test_public_target/probe.txt ~/public_html/probe_symlink.txt

ls -l ~/public_html/probe_symlink.txt
cat ~/public_html/probe_symlink.txt
```

**Salida:**

```
lrwxrwxrwx 1 ubizdevops ubizdevops 45 Apr 20 17:22 /home/ubizdevops/public_html/probe_symlink.txt -> /home/ubizdevops/test_public_target/probe.txt
public test
```

### Conclusiones

| Aspecto | Resultado |
|---|---|
| Creación de symlinks dentro de `public_html` | ✅ Permitida |
| Lectura de archivos enlazados desde terminal | ✅ Funciona correctamente |
| Restricciones a nivel de filesystem en `public_html` | ❌ No detectadas |

**Interpretación técnica:** El directorio `public_html` permite contener enlaces simbólicos que apuntan a rutas fuera de sí mismo. No existe bloqueo inmediato por parte del sistema de archivos. Esto confirma que, a nivel de cuenta y filesystem, el Modelo 2 sigue siendo viable.

> **Importante:** Aún no se ha validado si Apache/cPanel sirve correctamente estos symlinks vía HTTP. Eso dependerá de la directiva `FollowSymLinks`.

---

### Validación final — Symlink servido correctamente vía HTTP

Se realizó la validación definitiva accediendo desde el navegador al archivo enlazado dentro de `public_html`:

```
https://ubiz-devops.xyz/probe_symlink.txt
```

**Resultado observado:**

```
public test
```

### Conclusiones finales de la prueba web

| Aspecto | Resultado |
|---|---|
| Resolución del symlink por Apache | ✅ Correcta |
| Acceso HTTP al archivo enlazado | ✅ Funciona correctamente |
| `FollowSymLinks` / comportamiento equivalente operativo | ✅ Compatible en este entorno |
| Viabilidad real del Modelo 2 en producción | ✅ Confirmada |

**Interpretación técnica:** Apache/cPanel sirve correctamente contenido enlazado mediante symlinks dentro de `public_html`. La validación ya nose limita al filesystem: quedó comprobada también en acceso web real.

> **Decisión técnica confirmada:** El despliegue del proyecto Laravel en este servidor se realizará mediante el **Modelo 2**, usando `public_html` como symlink hacia `~/laravel_app/public`.

---

## Paso 9 — Crear el proyecto Laravel (local)

Se crea un proyecto Laravel 13 en el entorno local. La versión de PHP objetivo en producción es 8.1, pero el proyecto se inicializa localmente con PHP 8.4 ya que Composer/Laravel resuelve la compatibilidad desde el `composer.lock`.

### 9.1 Crear el proyecto

```bash
composer create-project laravel/laravel example-project
cd example-project/
```

### 9.2 Instalar dependencias PHP

```bash
composer i
```

### 9.3 Instalar Node.js con nvm

> **Nota:** Si no tienes nvm instalado, ejecuta primero los siguientes comandos.

```bash
# Instalar nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Cargar nvm en la sesión actual
source ~/.bashrc

# Instalar Node 22
nvm install 22

# Activarlo
nvm use 22

# Verificar
node -v
```

### 9.4 Instalar dependencias de Node

```bash
npm i
```

---

## Paso 10 — Inicializar repositorio Git y vincularlo al remoto (local)

### 10.1 Inicializar el repositorio

```bash
git init
git branch -m main
```

### 10.2 Configurar `.gitignore`

El archivo `.gitignore` raíz del proyecto debe contener al menos lo siguiente. Nota que `/public/build` está comentado intencionalmente para incluir los assets compilados en el repositorio y evitar tener que compilar en el servidor:

```gitignore
*.log
.DS_Store
.env
.env.backup
.env.production
.phpactor.json
.phpunit.result.cache
/.cursor/
/.idea
/.nova
/.phpunit.cache
/.vscode
/.zed
/auth.json
/node_modules
#/public/build
/public/hot
/public/storage
/storage/*.key
/storage/pail
/vendor
_ide_helper.php
Homestead.json
Homestead.yaml
Thumbs.db
```

### 10.3 Hacer el commit inicial

```bash
git add .
git commit -m "Initial Laravel 13 setup with build assets"
```

**Salida:**

```
[main (root-commit) ed31aa8] Initial Laravel 13 setup with build assets
 57 files changed, 11949 insertions(+)
```

### 10.4 Vincular al repositorio remoto y hacer push

```bash
git remote add origin git@github.com:GeraZgVic/repo-de-prueba-ultrabiz.git
git push -u origin main
```

**Salida:**

```
Enumerating objects: 77, done.
Counting objects: 100% (77/77), done.
Delta compression using up to 16 threads
Compressing objects: 100% (58/58), done.
Writing objects: 100% (77/77), 81.78 KiB | 5.11 MiB/s, done.
Total 77 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), done.
To github.com:GeraZgVic/repo-de-prueba-ultrabiz.git
 * [new branch]      main -> main
Branch 'main' set up to track remote branch 'main' from 'origin'.
```

---

## Paso 11 — Clonar el repositorio en cPanel

### 11.1 Generar llave SSH en el servidor

Desde la terminal SSH del servidor cPanel:

```bash
ssh-keygen -t ed25519 -C "cPanel-deploy"
```

**Salida:**

```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/ubizdevops/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubizdevops/.ssh/id_ed25519
Your public key has been saved in /home/ubizdevops/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:T7opQy/hkIBooccBEmSxlcTYniHXHWo9oziLF4FTK64 cPanel-deploy
The key's randomart image is:
+--[ED25519 256]--+
|=**+o ...        |
|o+**..o.         |
|oB=+oo +         |
|=.*o+ . o        |
|.o = o  S .      |
|. . * o  +       |
|E. o + o. .      |
|  .   = .o       |
|       +o        |
+----[SHA256]-----+
```

### 11.2 Obtener la clave pública

```bash
cat ~/.ssh/id_ed25519.pub
```

**Salida:**

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGjxup3p9jHzsD3YMXp2vHBV9J9bp8V2JpM4lsQqmIfY cPanel-deploy
```

### 11.3 Agregar la clave a GitHub

1. Ir a [https://github.com/settings/keys](https://github.com/settings/keys)
2. Click en **New SSH key**
3. Title: `cPanel Deploy`
4. Pegar la clave pública copiada
5. Guardar

### 11.4 Verificar autenticación con GitHub

```bash
ssh -T git@github.com
```

**Salida esperada:**

```
Hi GeraZgVic! You've successfully authenticated, but GitHub does not provide shell access.
```

### 11.5 Clonar el repositorio

```bash
git clone git@github.com:GeraZgVic/repo-de-prueba-ultrabiz.git laravel_app
```

**Salida:**

```
Cloning into 'laravel_app'...
remote: Enumerating objects: 77, done.
remote: Counting objects: 100% (77/77), done.
remote: Compressing objects: 100% (57/57), done.
remote: Total 77 (delta 1), reused 77 (delta 1), pack-reused 0 (from 0)
Receiving objects: 100% (77/77), 81.78 KiB | 1.49 MiB/s, done.
Resolving deltas: 100% (1/1), done.
```

### 11.6 Instalar dependencias PHP en el servidor

```bash
cd laravel_app
composer install --no-dev --optimize-autoloader
```

---

## Paso 12 — Configurar el symlink definitivo de `public_html`

### Regla de oro

> 🔥 **El symlink siempre reemplaza `public_html`, nunca vive dentro de él.**

El comando correcto es:

```bash
ln -s ~/laravel_app/public ~/public_html
```

Este comando **solo funciona si `public_html` no existe previamente**. Por eso el flujo completo es:

```bash
# 1. Mover el docroot actual como respaldo
mv ~/public_html ~/public_html_backup

# 2. Crear el symlink
ln -s ~/laravel_app/public ~/public_html
```

### ❌ Error común a evitar

```bash
# INCORRECTO — crea una carpeta "public" dentro de public_html
ln -s ~/laravel_app/public ~/public_html/public
```

Eso genera esta estructura incorrecta:

```
public_html/
└── public -> laravel_app/public   ← rompe la raíz del dominio
```

### Verificación

```bash
ls -ld ~/public_html
```

**Salida esperada:**

```
lrwxrwxrwx 1 ubizdevops ubizdevops 35 Apr 20 22:57 /home/ubizdevops/public_html -> /home/ubizdevops/laravel_app/public
```

### Cómo revertir si algo sale mal

```bash
rm ~/public_html
mv ~/public_html_backup ~/public_html
```

### Registro de lo ocurrido en este despliegue

En la primera ejecución se cometió el error de crear el symlink dentro de `public_html` en lugar de reemplazarlo. A continuación se documenta la corrección aplicada:

```bash
# Eliminar el symlink incorrecto
rm ~/public_html/public

# Mover el directorio original como respaldo
mv ~/public_html ~/public_html_original

# Crear el symlink correcto
ln -s ~/laravel_app/public ~/public_html
```

**Verificación post-corrección:**

```bash
ls -ld ~/public_html
ls -la ~/public_html
```

**Salida:**

```
lrwxrwxrwx 1 ubizdevops ubizdevops 35 Apr 20 22:57 /home/ubizdevops/public_html -> /home/ubizdevops/laravel_app/public
```

---

## Paso 13 — Crear y configurar el archivo `.env`

### 13.1 Crear el archivo

Desde el directorio `~/laravel_app`:

```bash
touch .env
```

Completar el archivo con los valores del entorno. Como mínimo, configurar el dominio de la aplicación:

```env
APP_URL=https://ubiz-devops.xyz/
```

### 13.2 Generar la clave de aplicación

```bash
php artisan key:generate
```

**Salida:**

```
INFO  Application key set successfully.
```

---

## Próximos pasos

> Esta sección se irá completando conforme avance el proceso de instalación.

- [x] Paso 1 — Verificar PHP
- [x] Paso 2 — Instalar Composer
- [x] Paso 3 — Hacer Composer accesible globalmente
- [x] Paso 4 — Verificar Git
- [x] Paso 5 — Análisis de arquitectura y validación de document root
- [x] Paso 6 — Validar symlinks a nivel de cuenta de usuario
- [x] Paso 7 — Validar symlinks dentro de `public_html` (prueba web)
- [x] Paso 8 — Elegir y confirmar modelo de despliegue
- [x] Paso 9 — Crear el proyecto Laravel (local)
- [x] Paso 10 — Inicializar repositorio Git y vincularlo al remoto (local)
- [x] Paso 11 — Clonar repositorio en cPanel y configurar SSH
- [x] Paso 12 — Configurar el symlink definitivo de `public_html`
- [x] Paso 13 — Crear `.env` y generar clave de aplicación


---

*Última actualización: 20 de abril de 2026*
