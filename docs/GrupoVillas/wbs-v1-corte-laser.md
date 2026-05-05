# WBS — V1 MVP · Plataforma Corte Láser GrupoVillas
**Versión:** 1.3  
**Fecha:** Mayo 2026  
**Desarrollador:** Ing. Victor Gerardo Zúñiga Gamboa  
**Documento:** Uso interno de desarrollo / Confidencial  

> **Tiempo disponible V1:** ~71 horas  
> **Estimado total V1:** ~53 horas  
> **Colchón para arranque V2:** ~18 horas  

---

## Resumen de Features V1

| # | Feature | Horas estimadas |
|---|---|---|
| F1 | Setup e Infraestructura | 5 hrs |
| F2 | Auth & Usuarios | 4 hrs |
| F3 | Portal del Cliente | 8 hrs |
| F4 | Gestión de Solicitudes | 6 hrs |
| F5 | Cotización Asistida | 10 hrs |
| F6 | Estados del Pedido | 7 hrs |
| F7 | Notificaciones por Correo | 4 hrs |
| F8 | Gestión de Clientes | 5 hrs |
| F9 | FAQ con Búsqueda Simple | 4 hrs |
| | **Total estimado V1** | **53 hrs** |

---

## F1 — Setup e Infraestructura
> Configuración del repositorio, ambientes y base del proyecto Laravel.  
> **Estimado:** 5 horas  
> ⚡ Flujo ya validado en proyecto de prueba — estimado reducido.

---

**US01 · Como desarrollador, quiero configurar el repositorio Git en GitHub con la estructura de ramas definida, para tener control de versiones desde el inicio del proyecto.**

- Estimado: 1 hr
- Criterios:
  - Repositorio creado en GitHub
  - Ramas `main` y `develop` creadas y protegidas con GitHub Rulesets
  - Convención de ramas documentada: `feature/*`, `fix/*`, `hotfix/*`

---

**US02 · Como desarrollador, quiero configurar los tres ambientes en cPanel (Production, Staging, Development), para poder trabajar y validar cambios sin afectar el sistema en producción.**

- Estimado: 2 hrs
- Criterios:
  - Subdominios creados: `app.`, `staging.`, `dev.`
  - 3 bases de datos MySQL independientes configuradas
  - Archivos `.env` independientes por ambiente

---

**US03 · Como desarrollador, quiero instalar y configurar Laravel 12 con el starter kit de Livewire, Mary UI y Tailwind CSS, para tener la base del proyecto lista para desarrollar.**

- Estimado: 2 hrs
- Criterios:
  - Laravel 12 instalado con starter kit Livewire
  - Livewire 4, Mary UI y Tailwind CSS configurados
  - `spatie/laravel-permission` instalado
  - `barryvdh/laravel-dompdf` instalado
  - Proyecto desplegado en ambiente Development

---

## F2 — Auth & Usuarios
> Login, registro y control de roles del sistema.  
> **Estimado:** 4 horas

---

**US04 · Como administrador, quiero iniciar sesión con correo y contraseña, para acceder al panel de administración del sistema.**

- Estimado: 1 hr
- Criterios:
  - Login funcional con redirección al panel admin
  - Sesión persistente
  - Mensaje de error si credenciales incorrectas

---

**US05 · Como cliente, quiero registrarme con nombre, correo y contraseña, para poder acceder al portal y enviar solicitudes.**

- Estimado: 1 hr
- Criterios:
  - Formulario de registro con validación
  - Rol `cliente` asignado automáticamente al registrarse
  - Redirección al portal del cliente tras registro exitoso

---

**US06 · Como desarrollador, quiero implementar middleware de roles, para que cada usuario solo pueda acceder a las secciones que le corresponden según su rol.**

- Estimado: 2 hrs
- Criterios:
  - Rol `admin` accede a `/admin/*`
  - Rol `cortador` accede a `/admin/*` con permisos limitados (solo estados y pedidos)
  - Rol `cliente` accede a `/` portal del cliente
  - Redirección automática si intenta acceder a ruta no permitida

---

## F3 — Portal del Cliente
> Interfaz del cliente para enviar solicitudes y dar seguimiento a sus pedidos.  
> **Estimado:** 8 horas

---

**US07 · Como cliente, quiero ver un dashboard con el resumen de mis solicitudes activas, para saber rápidamente el estado de mis pedidos al ingresar al portal.**

- Estimado: 2 hrs
- Criterios:
  - Vista con listado de solicitudes activas del cliente
  - Muestra estado actual de cada pedido
  - Acceso rápido a crear nueva solicitud

---

**US08 · Como cliente, quiero enviar una nueva solicitud de corte láser desde el portal, para no tener que hacerlo por WhatsApp.**

- Estimado: 4 hrs
- Criterios:
  - Formulario con campos: `description`, `material_type`, `thickness`, `piece_count`, `scale`, `schedule` (regular/urgent), `notes`
  - Subida de archivo DXF/DWG almacenado en `storage/app/job-requests/` — máx. 5MB
  - Validación de formato de archivo (`.dxf`, `.dwg` únicamente)
  - Confirmación visual al enviar la solicitud
  - Registro creado en tabla `job_requests` con `status` inicial y `user_id` del cliente autenticado

---

**US09 · Como cliente, quiero consultar el estado actual de mis pedidos desde el portal, para saber en qué etapa se encuentra mi trabajo sin tener que preguntar por WhatsApp.**

- Estimado: 2 hrs
- Criterios:
  - Vista de detalle del pedido con `current_status`
  - Historial de cambios de estado desde `order_statuses`
  - Indicación de `estimated_delivery_date` si está definida

---

## F4 — Gestión de Solicitudes
> Panel interno para que el Admin y Cortador gestionen las solicitudes entrantes.  
> **Estimado:** 6 horas

---

**US10 · Como administrador, quiero ver el listado de todas las solicitudes recibidas con su estado actual, para tener una visión general de la operación.**

- Estimado: 2 hrs
- Criterios:
  - Tabla con registros de `job_requests` ordenados por `created_at`
  - Filtro por `status`
  - Indicador visual de solicitudes sin atender
  - Acceso al detalle de cada solicitud

---

**US11 · Como administrador, quiero ver el detalle completo de una solicitud incluyendo el archivo adjunto, para poder revisarla antes de generar una cotización.**

- Estimado: 2 hrs
- Criterios:
  - Vista de detalle con todos los campos del modelo `JobRequest`
  - Descarga del archivo desde `file_path`
  - Opción de cambiar `status` a "En Revisión" o "Requiere Información"
  - Campo para agregar notas internas (`notes`)

---

**US12 · Como administrador, quiero crear solicitudes en nombre del cliente desde el panel, para registrar pedidos que llegan por WhatsApp o llamada.**

- Estimado: 2 hrs
- Criterios:
  - Formulario de captura equivalente al del portal del cliente
  - Selector de `User` existente (rol `cliente`) o creación rápida de cliente nuevo
  - Registro en `job_requests` con el `user_id` del cliente seleccionado

---

## F5 — Cotización Asistida
> Generación manual de cotizaciones con exportación a PDF.  
> **Estimado:** 10 horas

---

**US13 · Como administrador, quiero generar una cotización para una solicitud con los campos clave del cálculo, para formalizar el precio acordado con el cliente.**

- Estimado: 5 hrs
- Criterios:
  - Formulario crea registro en tabla `quotes` con campos: `material_cost`, `machine_time_cost`, `labor_cost`, `waste_factor`, `file_adjustment_fee`, `subtotal`, `total`
  - Indicador de `schedule` (regular/urgent) heredado del `JobRequest` relacionado
  - `folio` generado automáticamente
  - `validity_days` y `payment_conditions` configurables
  - `status` de la `Quote` inicia en `draft`
  - Cambio automático de `status` en `JobRequest` a "Cotización Lista" al guardar

---

**US14 · Como administrador, quiero generar y descargar el PDF de la cotización, para enviárselo al cliente o imprimirlo en hoja membretada.**

- Estimado: 3 hrs
- Criterios:
  - PDF generado con `barryvdh/laravel-dompdf` vía acción `GenerateQuotePdf`
  - Incluye datos del negocio (logo, nombre, teléfono, correo), detalle del trabajo, desglose de costos y total
  - `folio` visible en el PDF
  - `payment_conditions` y `validity_days` incluidos
  - Ruta del archivo guardada en `quotes.pdf_path`
  - Descarga disponible desde panel admin y portal del cliente

---

**US15 · Como cliente, quiero descargar mi cotización en PDF desde el portal, para tener el documento formal del precio acordado.**

- Estimado: 2 hrs
- Criterios:
  - PDF disponible en el detalle del pedido una vez que `quotes.status` sea `sent` o superior
  - Descarga directa desde `quotes.pdf_path` sin necesidad de contactar al equipo

---

## F6 — Estados del Pedido
> Flujo de estados con historial y auditoría.  
> **Estimado:** 7 horas

---

**US16 · Como administrador, quiero cambiar el estado de un pedido desde el panel, para mantener actualizado el seguimiento del trabajo.**

- Estimado: 3 hrs
- Criterios:
  - Estados disponibles en `orders.current_status`: `received` → `under_review` → `requires_info` → `quote_ready` → `deposit_pending` → `in_production` → `ready_for_delivery` → `delivered` → `cancelled`
  - Solo se puede avanzar al siguiente estado válido según el flujo definido
  - Campo opcional de `notes` al cambiar estado
  - Registro automático en `order_statuses` con `changed_by` (FK → `users`) y `created_at`

---

**US17 · Como cortador, quiero actualizar el estado de los pedidos asignados a mí, para mantener informado al administrador sobre el avance de producción.**

- Estimado: 2 hrs
- Criterios:
  - El rol `cortador` solo puede transicionar entre: `in_production` → `ready_for_delivery`
  - No puede modificar registros en `quotes` ni datos del modelo `User`
  - Registro en `order_statuses` igual que el admin

---

**US18 · Como administrador o cliente, quiero registrar la evidencia de pago del anticipo, para poder autorizar el inicio de producción.**

- Estimado: 2 hrs
- Criterios:
  - **Flujo Admin:** formulario en el panel permite subir imagen o PDF como comprobante y guardarlo en `orders.payment_proof_path`
  - **Flujo Cliente:** el portal permite subir el comprobante directamente desde el detalle del pedido cuando `current_status` es `deposit_pending`
  - Ambos flujos escriben en el mismo campo `orders.payment_proof_path`
  - **Transición a `in_production` — lógica flexible:**
    - Si `payment_proof_path` no es nulo → transición disponible directamente
    - Si `payment_proof_path` es nulo (pago en efectivo u otro medio sin evidencia digital) → el Admin puede forzar la transición, pero el sistema solicita confirmación explícita antes de proceder
    - El campo `notes` en `order_statuses` se usa para que el Admin deje constancia del motivo (ej. "Anticipo recibido en efectivo")
    - El rol `cortador` no puede ejecutar esta transición bajo ningún escenario
  - Comprobante visible en el detalle del pedido para Admin y Cortador cuando existe

---

## F7 — Notificaciones por Correo
> Envío automático de correos al cliente en cambios de estado clave.  
> **Estimado:** 4 horas

---

**US19 · Como cliente, quiero recibir un correo automático cuando mi cotización esté lista, para saber que ya puedo revisarla y aprobarla.**

- Estimado: 1 hr
- Criterios:
  - Correo enviado automáticamente al cambiar `current_status` a `quote_ready`
  - Incluye enlace directo al portal para ver la cotización
  - Registro en `notification_logs`

---

**US20 · Como cliente, quiero recibir un correo cuando mi pedido entre en producción y cuando esté listo para entrega, para estar informado sin tener que preguntar.**

- Estimado: 2 hrs
- Criterios:
  - Correo al cambiar `current_status` a `in_production`
  - Correo al cambiar `current_status` a `ready_for_delivery`
  - Ambos incluyen resumen del pedido
  - Registro en `notification_logs`

---

**US21 · Como administrador, quiero que el sistema envíe un correo interno cuando llegue una nueva solicitud sin atender, para no perder ninguna solicitud entrante.**

- Estimado: 1 hr
- Criterios:
  - Correo enviado al Admin cuando se crea un nuevo registro en `job_requests`
  - Incluye datos básicos de la solicitud y enlace al panel
  - Registro en `notification_logs`

---

## F8 — Gestión de Clientes
> Alta, edición y consulta de clientes desde el panel admin.  
> **Estimado:** 5 horas

---

**US22 · Como administrador, quiero ver el listado de clientes registrados con su información de contacto, para tener un directorio del negocio.**

- Estimado: 1 hr
- Criterios:
  - Tabla con registros del modelo `User` filtrados por rol `cliente`
  - Columnas: `name`, `email`, `phone`, `created_at`
  - Búsqueda por `name` o `email`
  - Acceso al perfil completo del cliente

---

**US23 · Como administrador, quiero crear y editar clientes manualmente desde el panel, para registrar clientes que no se registraron por el portal.**

- Estimado: 2 hrs
- Criterios:
  - Formulario crea/edita registro en tabla `users` con campos: `name`, `email`, `phone` y `notes`
  - Rol `cliente` asignado automáticamente vía `spatie/laravel-permission`
  - Cliente creado manualmente puede iniciar sesión si se le asigna contraseña

---

**US24 · Como administrador, quiero ver el historial de solicitudes de un cliente desde su perfil, para conocer su actividad y antecedentes en el negocio.**

- Estimado: 2 hrs
- Criterios:
  - Listado de registros en `job_requests` filtrados por `user_id` del cliente
  - Columnas: estado (`status`), fecha (`created_at`)
  - Acceso directo al detalle de cada `JobRequest` desde el perfil
  - Total de pedidos realizados visible en el perfil

---

## F9 — FAQ con Búsqueda Simple
> Sección de preguntas frecuentes con filtrado en tiempo real, administrable desde el panel.  
> **Estimado:** 4 horas

---

**US25 · Como cliente, quiero buscar respuestas a mis dudas en una sección de preguntas frecuentes del portal, para resolver mis preguntas sin tener que contactar al negocio.**

- Estimado: 2 hrs
- Criterios:
  - Campo de búsqueda con filtrado en tiempo real vía componente Livewire `FaqWidget`
  - Consulta sobre tabla `faqs` filtrando por `active = true` y `sort_order`
  - Resultados mostrados en formato acordeón
  - Mensaje amigable si no hay resultados
  - Visible desde el portal del cliente sin necesidad de iniciar sesión

---

**US26 · Como administrador, quiero agregar, editar, ordenar y desactivar preguntas frecuentes desde el panel, para mantener el FAQ actualizado sin necesidad de tocar código.**

- Estimado: 2 hrs
- Criterios:
  - CRUD completo sobre tabla `faqs` con campos: `question`, `answer`, `sort_order`, `active`
  - Campo `sort_order` para controlar la secuencia de aparición
  - Toggle `active` para ocultar preguntas sin eliminar el registro

---

## Resumen final V1

| Feature | Horas |
|---|---|
| F1 — Setup e Infraestructura | 5 hrs |
| F2 — Auth & Usuarios | 4 hrs |
| F3 — Portal del Cliente | 8 hrs |
| F4 — Gestión de Solicitudes | 6 hrs |
| F5 — Cotización Asistida | 10 hrs |
| F6 — Estados del Pedido | 7 hrs |
| F7 — Notificaciones por Correo | 4 hrs |
| F8 — Gestión de Clientes | 5 hrs |
| F9 — FAQ con Búsqueda Simple | 4 hrs |
| **Total V1** | **53 hrs** |
| **Horas disponibles** | **71 hrs** |
| **Colchón para inicio de V2** | **~18 hrs** |

---

---

## Plan de Sprints V1

> **Duración de sprint:** 1 semana  
> **Velocidad:** 6 hrs/semana  
> **Sprints de desarrollo:** 11  
> **Sprint de cierre:** 1 (deploy + presentación)  
> **Total:** 12 sprints · ~12 semanas

### Lógica de priorización

El orden de los sprints respeta las dependencias técnicas del sistema: no es posible generar una cotización sin una solicitud previa, ni gestionar estados sin una orden creada. Por ello el flujo va de infraestructura → autenticación → portal del cliente → gestión interna → cotización → estados → notificaciones → módulos de soporte → cierre.

Los **sprints 6 y 7** (Cotización Asistida) son los de mayor valor visible para el cliente: la generación del PDF con folio, logo y desglose de costos es el entregable que mayor impacto tiene en la percepción del producto. Se recomienda priorizar su demostración en una revisión intermedia.

---

### Estructura recomendada en Azure DevOps Boards

```
Épica: V1 — MVP Plataforma Corte Láser
└── Feature: F1 — Setup e Infraestructura
    └── User Story: US01 · Repositorio GitHub
        └── Task: Crear repo en GitHub
        └── Task: Configurar ramas y Rulesets
    └── User Story: US02 · Ambientes cPanel
    └── User Story: US03 · Instalación del stack
└── Feature: F2 — Auth & Usuarios
    └── ...
```

Cada User Story de este WBS corresponde a una **User Story** en el tablero. Las tareas técnicas (crear migración, crear componente Livewire, crear Service, etc.) se agregan como **Tasks** debajo de cada User Story durante el desarrollo.

---

### Tabla de sprints

| Sprint | Semana | User Stories | Hrs estimadas | Hrs disponibles | Colchón |
|---|---|---|---|---|---|
| S01 | 1 | US01, US02, US03 | 5 hrs | 6 hrs | 1 hr |
| S02 | 2 | US04, US05, US06 | 4 hrs | 6 hrs | 2 hrs |
| S03 | 3 | US07, US08 | 6 hrs | 6 hrs | — |
| S04 | 4 | US09, US10 | 4 hrs | 6 hrs | 2 hrs |
| S05 | 5 | US11, US12 | 4 hrs | 6 hrs | 2 hrs |
| S06 | 6 | US13 | 5 hrs | 6 hrs | 1 hr |
| S07 | 7 | US14, US15 | 5 hrs | 6 hrs | 1 hr |
| S08 | 8 | US16, US17 | 5 hrs | 6 hrs | 1 hr |
| S09 | 9 | US18, US19, US20, US21 | 6 hrs | 6 hrs | — |
| S10 | 10 | US22, US23, US24 | 5 hrs | 6 hrs | 1 hr |
| S11 | 11 | US25, US26 + QA flujo completo | 6 hrs | 6 hrs | — |
| S12 | 12 | Deploy a producción + ajustes + presentación al cliente | — | 6 hrs | 6 hrs |
| | | **Total estimado** | **55 hrs** | **72 hrs** | **~17 hrs** |

> **Nota:** S03 y S09 no tienen colchón por diseño — sus User Stories consumen exactamente las 6 horas disponibles. Si alguna de estas se extiende, el colchón acumulado de sprints anteriores (S02, S04, S05) lo absorbe. El colchón global de ~17 hrs cubre imprevistos de cPanel, ajustes de diseño y cualquier retraso operativo fuera del control del desarrollador.

---

*Documento de uso interno — WBS V1 Plataforma Corte Láser GrupoVillas — v1.3 Mayo 2026*
