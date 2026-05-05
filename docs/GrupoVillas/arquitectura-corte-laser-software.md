# Arquitectura Técnica — Plataforma Corte Láser GrupoVillas
**Versión:** 2.3  
**Fecha:** Mayo 2026  
**Desarrollador:** Ing. Victor Gerardo Zúñiga Gamboa  
**Cliente:** Arq. Miguel Villanueva — GrupoVillas  
**Documento:** Uso interno de desarrollo / Confidencial

---

## 1. Stack Tecnológico

| Capa | Tecnología | Notas |
|---|---|---|
| Backend | Laravel 12 | Versión estable actual, mínimos breaking changes |
| Autenticación | Laravel 12 Starter Kit (Livewire) | Breeze ya no recibe actualizaciones en L12, se usa el starter kit oficial |
| Frontend reactivo | Livewire 4 | Estable desde enero 2026, Single File Components por defecto |
| UI Components | Mary UI + Tailwind CSS | Diseñado específicamente para Livewire |
| Templating | Blade | |
| Base de datos | MySQL 8 | |
| Hosting | cPanel compartido | Confirmar accesos con Carlos |
| Control de versiones | Git — GitHub | Ramas protegidas con GitHub Rulesets |
| Gestión de proyecto | Azure DevOps Boards | Tablero SCRUM (Features, User Stories, Tasks) |

---

## 2. Roles del Sistema

| Rol | Descripción | Acceso |
|---|---|---|
| `admin` | Administrador general del negocio | Panel completo |
| `cortador` | Operador de la cortadora láser | Actualizar estados y pedidos asignados |
| `cliente` | Cliente externo registrado | Portal del cliente |

> **Nota:** En V1 solo existirá 1 Admin y 1 Cortador. La gestión multiusuario se extiende en V2.

---

## 3. Módulos por Versión

### V1 — MVP (~71 horas disponibles)

| # | Módulo | Descripción |
|---|---|---|
| M1 | Auth & Usuarios | Login, registro de clientes, roles Admin / Cortador / Cliente |
| M2 | Portal del Cliente | Envío de solicitudes, subida de archivos DXF/DWG, seguimiento de pedido |
| M3 | Gestión de Solicitudes | Panel admin para ver, asignar y gestionar solicitudes entrantes |
| M4 | Cotización Asistida | Formulario de cotización manual con campos clave, generación de PDF |
| M5 | Estados del Pedido | Flujo de estados con historial y cambio por Admin / Cortador |
| M6 | Notificaciones por Correo | Envío automático al cliente en cambios de estado clave (sin costo) |
| M7 | Gestión de Clientes | Alta, edición y vista de historial básico de clientes |
| M8 | FAQ con Búsqueda Simple | Sección de preguntas frecuentes con filtrado en tiempo real por palabra clave (Livewire). El Admin administra las preguntas desde el panel. Sin integración externa ni costo adicional. |

> **Nota chatbot:** En caso de requerirse un chatbot en V1, el alcance se limita a este FAQ con búsqueda
> simple por palabras clave. Cualquier integración con servicios externos, IA conversacional o WhatsApp
> Business API queda fuera del alcance de V1 por implicar costos o complejidad adicional.
> Se evaluará una solución más elaborada en V2 o V3.

---

### V2 — CRM y Analítica

| # | Módulo | Descripción |
|---|---|---|
| M9 | Reportes y Analítica | Reportes semanales/mensuales, exportación Excel, KPIs del negocio |
| M10 | CRM — Historial de Clientes | Perfil completo, historial de pedidos, acumulado de compras |
| M11 | Reutilización de Trabajos | Clonar trabajos anteriores como base para nuevos pedidos |
| M12 | Inventario Básico de Materiales | Control de materiales disponibles, merma y reposición |
| M13 | Multiusuario y Permisos | Más usuarios con roles configurables desde el panel |
| M14 | Chatbot Avanzado | Chat en vivo interno o integración con IA (sujeto a costo y aprobación del cliente) |

---

### V3 — Expansión

| # | Módulo | Descripción |
|---|---|---|
| M15 | Facturación CFDI | Generación de facturas electrónicas — **requiere contratación de servicio externo (ej. Facturapi)** |
| M16 | Integración WhatsApp Business API | Notificaciones por WhatsApp — **requiere contratación Meta / Twilio, costo adicional** |

> **Nota:** Los módulos de Administración de Obras y Papelería corresponden a líneas de negocio
> independientes dentro de GrupoVillas. Se consideran proyectos separados con su propio
> levantamiento de requerimientos, arquitectura y contrato. No forman parte del alcance
> de este proyecto.

---

### Módulos a Evaluar a Futuro (sin compromiso de versión)

| Módulo | Descripción |
|---|---|
| Cotización automática desde DXF | Cálculo de área y costo directo desde el archivo de diseño. Alta complejidad técnica, requiere análisis dedicado. |

---

## 4. Convenciones de Nomenclatura Laravel

Se respetan las convenciones de Laravel para aprovechar Eloquent sin configuración adicional.

| Elemento | Convención | Ejemplo |
|---|---|---|
| Modelos | PascalCase singular en inglés | `JobRequest`, `Quote` |
| Tablas | snake_case plural en inglés | `job_requests`, `quotes` |
| Claves foráneas | nombre del modelo en snake_case + `_id` | `job_request_id`, `user_id` |
| Clave primaria | `id` (Eloquent la detecta automáticamente) | `id` |
| Timestamps | `created_at`, `updated_at` (manejados por Eloquent) | — |
| Migraciones | Versionadas en el repositorio, ejecutadas por ambiente | `database/migrations/` |

> **Sobre migraciones:** Se usará el sistema de migraciones de Laravel para mantener control
> de versiones del esquema de base de datos. Toda modificación al esquema se realiza mediante
> migraciones, nunca directamente en la base de datos. El flujo correcto es:
> Development → Staging → Production.

### Tabla de modelos y tablas del proyecto

| Modelo | Tabla | Nota |
|---|---|---|
| `User` | `users` | Nativo de Laravel |
| `JobRequest` | `job_requests` | Nombrado así para evitar conflicto con `Illuminate\Http\Request` de Laravel |
| `Quote` | `quotes` | |
| `Order` | `orders` | |
| `OrderStatus` | `order_statuses` | Historial de estados del pedido |
| `NotificationLog` | `notification_logs` | |
| `Faq` | `faqs` | |

---

## 5. Modelos de Base de Datos (V1)

### `users`
```
id, name, email, password, role (admin|cortador|cliente),
phone, created_at, updated_at
```

### `job_requests`
```
id, user_id (FK → users, cliente),
service_type (laser|3d),
description, material_type, thickness,
piece_count, scale, file_path,
file_name, notes, schedule (regular|urgent),
status, attended_by (FK → users),
created_at, updated_at
```

### `quotes`
```
id, job_request_id (FK → job_requests),
generated_by (FK → users),
material_cost, machine_time_cost,
labor_cost, waste_factor,
file_adjustment_fee, subtotal, total,
validity_days, payment_conditions,
folio, pdf_path,
status (draft|sent|approved|rejected),
created_at, updated_at
```

### `orders`
```
id, job_request_id (FK → job_requests),
quote_id (FK → quotes),
deposit_required, deposit_paid,
remaining_balance, payment_proof_path,
estimated_delivery_date, current_status,
internal_notes, created_at, updated_at
```

### `order_statuses` (historial de estados)
```
id, order_id (FK → orders),
status, changed_by (FK → users),
notes, created_at
```

### `notification_logs`
```
id, order_id (FK → orders),
user_id (FK → users),
channel (email), subject,
message, sent_at, status
```

### `faqs`
```
id, question, answer,
sort_order, active (boolean),
created_at, updated_at
```

---

## 5. Flujo de Estados del Pedido (V1)

> Los valores de `current_status` en la tabla `orders` siguen la convención `snake_case` en inglés,
> consistente con las convenciones de Laravel del proyecto.

```
[received]           — Solicitud recibida
        ↓
[under_review] ──── ¿Falta información? ──→ [requires_info]
        ↓                                          ↓ (cliente responde)
        ←──────────────────────────────────────────┘
        ↓
[quote_ready] ──── cliente aprueba (portal o confirmación manual)
        ↓
[deposit_pending] ──── cliente o admin registra evidencia de pago
        │                  ↳ Con comprobante: transición directa
        │                  ↳ Sin comprobante (efectivo): Admin confirma manualmente
        ↓
[in_production] ──── cortador ejecuta el trabajo
        ↓
[ready_for_delivery] ──── se programa entrega
        ↓
[delivered] ──── pago restante confirmado + confirmación de recibido

[cancelled] ──── disponible en cualquier punto del flujo
```

---

## 6. Arquitectura de Capas — Controlador Ligero / Servicio Pesado

```
HTTP Request
     ↓
[ Controller ]      → recibe request, valida, llama al Service, retorna respuesta
     ↓
[ Service ]         → contiene toda la lógica de negocio
     ↓
[ Model / Repository ] → interacción con base de datos
     ↓
[ Response / View ]
```

> **Regla:** Los controladores no contienen lógica de negocio.
> Los componentes Livewire tampoco — también delegan a Services.

---

## 7. Estructura de Carpetas Laravel (V1)

```
app/
├── Http/
│   ├── Controllers/
│   │   ├── Admin/
│   │   │   ├── JobRequestController.php      → llama JobRequestService
│   │   │   ├── QuoteController.php           → llama QuoteService
│   │   │   ├── OrderController.php           → llama OrderService
│   │   │   └── ClientController.php          → llama ClientService
│   │   └── Client/
│   │       ├── JobRequestController.php
│   │       └── OrderController.php
│   └── Middleware/
│       └── RoleMiddleware.php
│
├── Services/                                  → lógica de negocio pesada
│   ├── JobRequestService.php
│   ├── QuoteService.php
│   ├── OrderService.php
│   ├── ClientService.php
│   └── NotificationService.php
│
├── Actions/                                   → acciones atómicas reutilizables
│   ├── ChangeOrderStatus.php
│   ├── GenerateQuotePdf.php
│   └── SendEmailNotification.php
│
├── Livewire/
│   ├── Admin/
│   │   ├── JobRequestList.php                → delega a JobRequestService
│   │   ├── QuoteForm.php                     → delega a QuoteService
│   │   ├── OrderKanban.php                   → delega a OrderService
│   │   └── ClientList.php
│   └── Client/
│       ├── NewJobRequestForm.php
│       ├── MyOrders.php
│       └── FaqWidget.php
│
├── Models/
│   ├── User.php
│   ├── JobRequest.php
│   ├── Quote.php
│   ├── Order.php
│   ├── OrderStatus.php
│   ├── NotificationLog.php
│   └── Faq.php
│
├── Notifications/
│   ├── JobRequestReceived.php
│   ├── QuoteReady.php
│   ├── OrderInProduction.php
│   └── OrderReadyForDelivery.php
│
└── Policies/
    ├── JobRequestPolicy.php
    └── OrderPolicy.php

resources/views/
├── layouts/
│   ├── app.blade.php
│   ├── admin.blade.php
│   └── client.blade.php
├── admin/
│   ├── job-requests/
│   ├── quotes/
│   ├── orders/
│   └── clients/
└── client/
    ├── job-requests/
    ├── orders/
    └── faq/
```

---

## 8. Convenciones de Desarrollo

| Aspecto | Convención |
|---|---|
| Idioma del código | Inglés (variables, métodos, clases, migraciones, vistas) |
| Idioma de la UI | Español |
| Nomenclatura modelos | PascalCase singular en inglés — ver sección 4 |
| Nomenclatura tablas | snake_case plural en inglés — ver sección 4 |
| Rutas admin | Prefijo `/admin/` con middleware `role:admin,cortador` |
| Rutas cliente | Prefijo `/` con middleware `role:cliente` |
| Lógica de negocio | Siempre en `Services/`, nunca en Controllers ni Livewire |
| Acciones atómicas | En `Actions/` cuando son reutilizables entre Services |
| Migraciones | Versionadas en el repositorio. Flujo: Development → Staging → Production. Nunca modificar esquema directo en BD. |
| PDF | `barryvdh/laravel-dompdf` |
| Correo | Laravel Mail con SMTP de cPanel |
| Archivos | Laravel Storage — disco `local`, carpeta `storage/app/job-requests/` |
| Formatos aceptados | `.dxf`, `.dwg` — máximo 5MB por archivo |
| Permisos y roles | `spatie/laravel-permission` |

---

## 9. Consideraciones de Despliegue en cPanel

- PHP 8.2 o superior requerido
- Crear base de datos MySQL desde el panel de cPanel
- Configurar `.env` con credenciales de BD y SMTP
- Permisos `775` en `storage/` y `bootstrap/cache/`
- `APP_ENV=production` · `APP_DEBUG=false`
- Configurar cron job: `* * * * * php /ruta/artisan schedule:run`
- Document root apuntando a `/public` del proyecto Laravel
- Subida vía Git o FTP/SFTP

---

## 10. Dependencias Clave

```json
// composer.json
"laravel/framework": "^12.0",
"livewire/livewire": "^4.0",
"robsontenorio/mary": "^*",
"barryvdh/laravel-dompdf": "^2.x",
"spatie/laravel-permission": "^6.x"

// package.json
"tailwindcss": "^3.x",
"@tailwindcss/forms": "^0.5.x"
```

---

## 11. Pendientes por Resolver

- [ ] Confirmar credenciales y plan de hosting con Carlos
- [ ] Confirmar subdominio o dominio donde vivirá el sistema
- [ ] Obtener formato de cotización CACEP para replicar estructura en PDF
- [ ] Confirmar SMTP disponible en cPanel (host, puerto, usuario)
- [ ] Definir preguntas frecuentes iniciales para el FAQ con el cliente

---

## 12. Decisiones de Alcance Documentadas

| Decisión | Definición |
|---|---|
| Notificaciones V1 | Correo electrónico vía SMTP de cPanel. Sin costo adicional. |
| WhatsApp Business API | Fuera de alcance hasta V3. Requiere contratación externa y aprobación de costo. |
| Facturación CFDI | Fuera de alcance hasta V3. Requiere servicio externo (ej. Facturapi). |
| Chatbot V1 | FAQ con búsqueda simple por palabras clave, administrable desde el panel. Sin integración externa ni costo. Solución más elaborada se evalúa en V2/V3. |
| Impresión 3D | No forma parte de ninguna versión comprometida. Se evalúa a futuro. |
| Cotización desde DXF | Alta complejidad. Sin compromiso de versión. Se evalúa a futuro. |
| Multiusuario | V1 opera con 1 Admin y 1 Cortador. Gestión ampliada en V2. |

---

*Documento de uso interno — Desarrollo Plataforma Corte Láser GrupoVillas — v2.3 Mayo 2026*
