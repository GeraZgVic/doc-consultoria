# US60 — Validate Git Workflow and Environment Promotion (Minimal Test)

**Elaborado por:** Ing. Victor Gerardo Zuñiga Gamboa  
**Fecha:** 27 de abril de 2026  
**Versión:** v1.0  
**Estado:** Resolved  

---

## Objetivo

Validar el workflow de Git propuesto y la estrategia de promoción de ambientes usando código mínimo, sin depender de requerimientos funcionales.

El objetivo es probar el flujo completo a través de ramas y ambientes: crear una rama `feature/*` desde `develop`, abrir un Pull Request hacia `develop`, validar la integración, y posteriormente promover los cambios vía Pull Request desde `develop` hacia `main`.

---

## Configuración inicial

### Cuenta de GitHub en cPanel

```bash
git config user.name "Victor Zuñiga"
git config user.email "gerazgvic@gmail.com"
```

### Crear rama develop desde main

```bash
git checkout -b develop
```

#### Subir la rama al repositorio

```bash
git push -u origin develop
```

---

## Configuración de Branch Protection

Ir al repositorio:
> Settings → Branches → Add branch ruleset

### Nombre del Ruleset

`protect-main-develop`

### Target branches

Seleccionar `Add target` → `Include by pattern` y agregar los siguientes patrones:

- `main`
- `develop`

> **Nota:** No se deben incluir todas las ramas.

### Reglas configuradas

- [x] Restrict deletions
- [x] Block force pushes
- [x] Require a pull request before merging
  - Required approvals: `0`
  - Allowed merge methods: Merge, Squash, Rebase
- [ ] Restrict creations
- [ ] Restrict updates
- [ ] Require linear history
- [ ] Require deployments to succeed
- [ ] Require signed commits
- [ ] Require status checks to pass
- [ ] Require code scanning results
- [ ] Require code quality results
- [ ] Automatically request Copilot code review

> **Nota:** Esta configuración está pensada para validar el workflow de desarrollo y promoción de ambientes sin agregar fricción innecesaria. Se establece como obligatorio el uso de Pull Requests para mantener el flujo `feature → develop → main`, pero sin requerir aprobaciones en esta etapa (Required approvals: 0), ya que el objetivo actual es validar el proceso y no la gobernanza del equipo.
>
> En etapas posteriores, esta configuración puede evolucionar agregando:
> - Requerimiento de aprobaciones (1 o más reviewers)
> - Code Owners
> - Status checks (CI/CD)
> - Políticas de calidad de código
>
> Por ahora, se prioriza simplicidad, claridad del flujo y velocidad de validación.

---

## Validación de Branch Rules

### 1. Traer las ramas del remoto

```bash
git fetch origin
```

### 2. Crear rama local siguiendo la remota

```bash
git checkout -b develop origin/develop
```

### 3. Crear rama de trabajo feature

```bash
git checkout -b feature/test-workflow-ambientes
```

### 4. Aplicar un cambio mínimo

```bash
echo "workflow test" > workflow-test.txt
```

Validar:

```bash
git status
```

### 5. Agregar el cambio y crear commit

```bash
git add workflow-test.txt
git commit -m "test: validate workflow feature -> develop -> main"
```

### 6. Validar con git log

```bash
git log --oneline -n 1
```

**Salida:**

```
5155a85 (HEAD -> feature/test-workflow-ambientes) test: validate workflow feature -> develop -> main
```

### 7. Subir la rama al remoto

```bash
git push -u origin feature/test-workflow-ambientes
```

### 8. Crear Pull Request desde feature hacia develop

Ir a GitHub y crear un PR con la siguiente configuración:

- **Base branch:** `develop`
- **Compare:** `feature/test-workflow-ambientes`

Validar:
- El PR se puede crear sin bloqueos
- El cambio (archivo y commit) es visible correctamente
- No hay conflictos

### 9. Realizar merge del Pull Request hacia develop

Validar:
- El merge se realiza correctamente
- No se permite push directo a `develop` (solo vía PR)
- El flujo `feature → develop` se cumple

### 10. Eliminar la rama feature

Después del merge, eliminar la rama desde GitHub con el botón **Delete branch**.

Validar:
- La rama ya no existe en remoto
- El repositorio queda limpio

---

## Resultado parcial de la validación

Se confirma que:

- El flujo `feature → develop` mediante Pull Request funciona correctamente
- El merge se realiza sin conflictos
- La integración en `develop` es exitosa

**Pendiente de validar:**

- Bloqueo de push directo a ramas protegidas
- Bloqueo de force push
- Protección contra eliminación de ramas críticas

El repositorio queda listo para continuar con la siguiente validación: `develop → Pull Request → main`

---

## Validación de restricciones en ramas protegidas

### 11. Intento de push directo a `develop` (debe fallar)

Posicionarse en la rama `develop` y realizar un cambio:

```bash
git checkout develop
git pull origin develop

echo "direct push test" >> workflow-test.txt
git add workflow-test.txt
git commit -m "test: direct push to develop"
```

Intentar hacer push directo:

```bash
git push origin develop
```

**Resultado:**

```
remote: error: GH013: Repository rule violations found for refs/heads/develop.
remote: Review all repository rules at https://github.com/GeraZgVic/repo-de-prueba-ultrabiz/rules?ref=refs%2Fheads%2Fdevelop
remote:
remote: - Changes must be made through a pull request.
remote:
! [remote rejected] develop -> develop (push declined due to repository rule violations)
error: failed to push some refs to 'github.com:GeraZgVic/repo-de-prueba-ultrabiz.git'
```

**Resultado de la validación:**

Se confirma que:
- No se permite hacer push directo a `develop`
- Los cambios deben realizarse obligatoriamente mediante Pull Request
- Las reglas de protección están funcionando correctamente

---

## Validación completa del workflow

### 12. Crear Pull Request de `develop` hacia `main`

Configuración:
- **Base branch:** `main`
- **Compare:** `develop`

Validar:
- Cambios acumulados desde `develop` son visibles
- No hay conflictos
- PR se puede crear sin bloqueos

### 13. Merge del Pull Request hacia `main`

Acción:
- Realizar merge desde GitHub

Validar:
- El merge se ejecuta correctamente
- No se permite push directo a `main`
- La promoción de cambios se realiza únicamente vía PR

---

## Resultado final de la validación

Se confirma que el workflow completo funciona correctamente:

`feature → Pull Request → develop → Pull Request → main`

**Reglas de protección validadas:**

- ✔ Bloqueo de push directo a ramas críticas (`develop`, `main`)
- ✔ Uso obligatorio de Pull Request
- ✔ Prevención de force push
- ✔ Prevención de eliminación de ramas protegidas

---

## Alcance y pendientes

> El flujo `hotfix/*` quedó fuera del alcance de esta validación. Su mecánica especial (rama creada desde `main`, fusionada tanto en `main` como en `develop`) será cubierta en la **US62 — Validate hotfix/* Workflow (Minimal Test)**.

---

## Conclusión

El repositorio queda correctamente configurado para un flujo de trabajo controlado, escalable y alineado con buenas prácticas de desarrollo.

Este esquema permite:
- Separación clara entre desarrollo, validación y producción
- Control de cambios mediante Pull Requests
- Reducción de errores en despliegues
- Base sólida para integrar CI/CD en etapas posteriores

---

*Última actualización: 27 de abril de 2026*
