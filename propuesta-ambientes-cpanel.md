# Propuesta base de arquitectura de ambientes

## Objetivo
Definir una base técnica clara para separar el sistema del cliente en tres ambientes:

- Production
- Staging
- Development

La finalidad es agilizar el trabajo, reducir riesgos al publicar cambios y establecer un flujo ordenado de desarrollo, validación y liberación.

---

## Contexto
El sistema actual parece estar montado en un hosting con cPanel y probablemente fue desarrollado con CodeIgniter. A partir de este escenario, la propuesta considera una estrategia compatible con:

- subdominios por ambiente
- un solo repositorio Git
- ramas por flujo de trabajo
- configuraciones independientes por entorno

---

## Propuesta de ambientes

### Production
Ambiente estable y público.

Ejemplo:
- `app.cliente.com`

Características:
- recibe únicamente cambios aprobados
- usa configuración productiva
- conecta a su propia base de datos
- no debe usarse para pruebas técnicas

### Staging
Ambiente de validación previa a producción.

Ejemplo:
- `staging.cliente.com`

Características:
- replica el comportamiento esperado de producción
- sirve para QA, validación funcional y revisión con cliente
- usa base de datos independiente
- despliega la rama `develop`

### Development
Ambiente técnico para pruebas de desarrollo.

Ejemplo:
- `dev.cliente.com`

Características:
- se usa para revisar cambios en construcción
- puede desplegar temporalmente ramas `feature/*`, `fix/*` o `hotfix/*`
- usa base de datos independiente
- no requiere una rama fija llamada `development`

---

## Diagrama
flowchart TD

    DEV[Desarrollador]

    subgraph REPO[Repositorio Git]
        F[feature/*<br/>fix/*<br/>hotfix/*]
        D[develop]
        M[main]
    end

    subgraph ENVS[Ambientes]
        E1[Development<br/>dev.cliente.com]
        E2[Staging<br/>staging.cliente.com]
        E3[Production<br/>app.cliente.com]
    end

    subgraph DBS[Bases de datos]
        DB1[(DB Development)]
        DB2[(DB Staging)]
        DB3[(DB Production)]
    end

    DEV -->|crear rama desde develop| F
    F -->|PR base: develop| D
    F -->|pruebas técnicas opcionales| E1
    D -->|deploy| E2
    D -->|PR base: main| M
    M -->|deploy| E3

    E1 --> DB1
    E2 --> DB2
    E3 --> DB3


## Repositorio y estrategia Git

### Recomendación
Usar **un solo repositorio** para todo el sistema.

### Justificación
Tener un solo repositorio ayuda a:

- mantener una sola línea de evolución del código
- facilitar trazabilidad de cambios
- evitar divergencias entre ambientes
- simplificar mantenimiento y soporte

No se recomienda usar un repositorio por ambiente, ya que eso suele generar desalineación entre versiones y trabajo duplicado.

---

## Ramas propuestas

### `main`
Rama estable de producción.

Uso:
- solo recibe cambios ya validados
- alimenta el ambiente Production

### `develop`
Rama de integración.

Uso:
- concentra cambios aprobados provenientes de ramas de trabajo
- alimenta el ambiente Staging

### `feature/*`
Ramas para nuevas funcionalidades.

Ejemplos:
- `feature/nuevo-login`
- `feature/reporte-clientes`

### `fix/*`
Ramas para correcciones normales.

Ejemplos:
- `fix/error-formulario`
- `fix/validacion-correo`

### `hotfix/*`
Ramas para correcciones urgentes.

Ejemplos:
- `hotfix/error-pago`
- `hotfix/fallo-login-prod`

---

## Flujo de trabajo propuesto

### 1. Inicio de trabajo
El desarrollador se posiciona en `develop`, actualiza la rama y crea una rama de trabajo.

Ejemplo conceptual:
- partir de `develop`
- crear `feature/*` o `fix/*`

### 2. Desarrollo
Los cambios se trabajan en la rama correspondiente.

### 3. Integración
Cuando el cambio está listo, se crea un Pull Request hacia `develop`.

Ejemplo:
- base: `develop`
- compare: `feature/nueva-funcionalidad`

### 4. Validación en Staging
Una vez integrado en `develop`, el ambiente Staging recibe el cambio.

Aquí se realizan:
- pruebas funcionales
- QA
- validación interna
- revisión con cliente si aplica

### 5. Promoción a producción
Cuando lo integrado en Staging queda validado, se crea un Pull Request de `develop` hacia `main`.

Ejemplo:
- base: `main`
- compare: `develop`

### 6. Liberación
Production despliega la rama `main`.

---

## Relación entre ramas y ambientes

Importante:

- `develop` es una **rama**
- Development es un **ambiente**

No son lo mismo.

Relación propuesta:

- Development puede desplegar temporalmente una rama de trabajo (`feature/*`, `fix/*`, `hotfix/*`)
- Staging despliega `develop`
- Production despliega `main`

---

## Despliegue por ambiente

### Production
- subdominio independiente
- base de datos independiente
- credenciales propias
- logs propios
- rama desplegada: `main`

### Staging
- subdominio independiente
- base de datos independiente
- credenciales propias
- servicios externos idealmente en modo sandbox
- rama desplegada: `develop`

### Development
- subdominio independiente
- base de datos independiente
- configuración aislada
- rama desplegada: variable según la prueba técnica

---

## Recomendaciones de configuración

Cada ambiente debe tener su propia configuración para:

- URL base
- credenciales de base de datos
- correo
- APIs externas
- pasarelas de pago
- almacenamiento de archivos
- logs y depuración

Esto evita que un ambiente afecte a otro.

---

## Manejo de base de datos

Se recomienda que cada ambiente tenga su propia base de datos:

- DB Production
- DB Staging
- DB Development

Notas:
- Staging puede usar copia controlada de producción si es necesario
- Development debe usar datos de prueba
- no se deben compartir bases de datos entre ambientes

---

## Manejo de incidencias en Staging

Si un cambio integrado en `develop` rompe Staging:

1. identificar el commit o PR responsable
2. revertir el cambio en `develop`
3. desplegar nuevamente Staging
4. corregir el problema en una nueva rama de trabajo

Regla recomendada:
- en ramas compartidas como `develop` y `main`, usar `git revert`
- evitar reescritura de historial

---

## Beneficios de esta propuesta

- mayor control sobre cambios
- menor riesgo al publicar a producción
- validación previa antes de liberar
- mejor trazabilidad de código
- estructura escalable para futuras mejoras
- base clara para automatizar despliegues después

---

## Implementación sugerida por fases

### Fase 1
Separación de ambientes y control de versiones.

Incluye:
- creación de subdominios
- configuración de bases de datos por ambiente
- definición del repositorio Git
- estructura de ramas
- despliegue manual controlado

### Fase 2
Estandarización operativa.

Incluye:
- documentación de despliegue
- checklist de validación antes de producción
- política de backups
- criterios de rollback

### Fase 3
Automatización gradual.

Incluye:
- integración con repositorio remoto
- webhook o flujo de deploy
- estandarización de promociones entre ambientes

---

## Conclusión
La propuesta recomendada para este proyecto es:

- un solo repositorio Git
- ramas de trabajo (`feature/*`, `fix/*`, `hotfix/*`)
- rama `develop` para integración y Staging
- rama `main` para Production
- ambiente Development para pruebas técnicas, sin necesidad de una rama fija llamada `development`

Esta estructura ofrece orden, trazabilidad y una base sólida para crecer sin complicar innecesariamente la operación.

