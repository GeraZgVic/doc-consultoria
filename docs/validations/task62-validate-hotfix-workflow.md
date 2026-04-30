# Task 62 — Validate hotfix/* Workflow (Minimal Test)

**Elaborado por:** Ing. Victor Gerardo Zuñiga Gamboa  
**Fecha:** 30 de abril de 2026  
**Versión:** v1.0  
**Estado:** Resolved  

---

## Objetivo

Validar la mecánica especial del flujo `hotfix/*`, que difiere del flujo estándar `feature → develop → main`. Una rama `hotfix/*` se crea directamente desde `main` y, una vez resuelta, debe fusionarse en ambas ramas — `main` y `develop` — para garantizar que el fix no se pierda en el siguiente ciclo de desarrollo.

---

## Contexto

El bloqueo de push directo a ramas protegidas (`main` y `develop`) fue validado previamente en la **Task 60**. Las reglas configuradas en el Ruleset `protect-main-develop` siguen activas y aplican igualmente en este flujo.

---

## Flujo validado

### Paso 1 — Actualizar main local

```bash
git checkout main
git pull origin main
```

**Salida:**

```
From github.com:GeraZgVic/repo-de-prueba-ultrabiz
 * branch            main       -> FETCH_HEAD
   5fc9b33..c1f2f4c  main       -> origin/main
Updating 1ba4a79..c1f2f4c
Fast-forward
 .gitignore        | 1 +
 workflow-test.txt | 2 ++
 2 files changed, 3 insertions(+)
 create mode 100644 workflow-test.txt
```

### Paso 2 — Crear rama hotfix desde main

```bash
git checkout -b hotfix/test-hotfix-workflow
```

**Salida:**

```
Switched to a new branch 'hotfix/test-hotfix-workflow'
```

> **Importante:** La rama se crea desde `main`, no desde `develop`. Este es el punto de partida diferenciador del flujo `hotfix/*`.

### Paso 3 — Aplicar cambio mínimo y commit

```bash
echo "hotfix test" > hotfix-test.txt
git add hotfix-test.txt
git commit -m "test: validate hotfix workflow"
```

**Salida:**

```
[hotfix/test-hotfix-workflow e815bdb] test: validate hotfix workflow
 1 file changed, 1 insertion(+)
 create mode 100644 hotfix-test.txt
```

### Paso 4 — Subir la rama al remoto

```bash
git push -u origin hotfix/test-hotfix-workflow
```

**Salida:**

```
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 16 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 312 bytes | 312.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Create a pull request for 'hotfix/test-hotfix-workflow' on GitHub by visiting:
remote: https://github.com/GeraZgVic/repo-de-prueba-ultrabiz/pull/new/hotfix/test-hotfix-workflow
Branch 'hotfix/test-hotfix-workflow' set up to track remote branch 'hotfix/test-hotfix-workflow' from 'origin'.
```

### Paso 5 — Pull Request hacia main

Configuración:
- **Base branch:** `main`
- **Compare:** `hotfix/test-hotfix-workflow`

Validar:
- Sin conflictos
- Cambio visible correctamente
- PR creado sin bloqueos

**Resultado:** Merge exitoso hacia `main`.

### Paso 6 — Pull Request hacia develop

Configuración:
- **Base branch:** `develop`
- **Compare:** `hotfix/test-hotfix-workflow`

Validar:
- Sin conflictos
- Cambio visible correctamente
- PR creado sin bloqueos

**Resultado:** Merge exitoso hacia `develop`.

### Paso 7 — Eliminar la rama hotfix

Rama eliminada desde GitHub con el botón **Delete branch** tras el segundo merge.

### Paso 8 — Confirmar que ambas ramas contienen el fix

```bash
git checkout main
git pull origin main
ls
```

**Salida:** `hotfix-test.txt` presente ✅

```bash
git checkout develop
git pull origin develop
ls
```

**Salida:** `hotfix-test.txt` presente ✅

---

## Resultado final

| Criterio | Resultado |
|---|---|
| Rama `hotfix/*` creada desde `main` | ✅ |
| Cambio mergeado hacia `main` vía PR | ✅ |
| Cambio mergeado hacia `develop` vía PR | ✅ |
| Fix presente en `main` | ✅ |
| Fix presente en `develop` | ✅ |
| Rama `hotfix/*` eliminada post-merge | ✅ |
| Bloqueo de push directo a ramas protegidas | ✅ Validado en Task 60 |

---

## Conclusión

El flujo `hotfix/*` funciona correctamente. El fix quedó disponible en ambas ramas, garantizando que no se pierda en el siguiente ciclo de desarrollo. La mecánica especial de este flujo queda validada y documentada como parte de la propuesta de arquitectura de ambientes.

---

*Última actualización: 30 de abril de 2026*
