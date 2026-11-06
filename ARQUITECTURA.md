# Arquitectura

[← Volver al README](README.md)

## Principios

1. **Independencia de servicios.** Cada microservicio tiene su propia base de datos y su propio ciclo de vida. Si uno cae, los demás siguen funcionando.
2. **La seguridad vive en la base de datos.** La identidad y el rol viajan en el JWT y las políticas por fila deciden qué se puede leer o escribir.
3. **Lo simple primero.** Un servicio sin lógica propia no lleva aplicación: se expone con PostgREST. Solo se escribe código de aplicación donde hay lógica real.
4. **Las reglas entre servicios van en el BFF.** No hay consultas cruzadas entre bases de datos.
5. **Los efectos secundarios no tumban la acción principal.** Las notificaciones son *best-effort*.

## Componentes

| Componente | Responsabilidad | Tecnología |
|---|---|---|
| Web App (BFF) | Interfaz, sesión por cookies seguras, proxies hacia los servicios, reglas que cruzan servicios | Laravel + Inertia + Vue 3 |
| Auth Service | Registro, inicio de sesión, JWT, roles y permisos, solicitudes de rol, amistades, perfil | Laravel + PostgreSQL |
| Content Service | Jugadas en video, categorías, likes y comentarios | PostgreSQL + PostgREST |
| News Service | Noticias de editores, reacciones, comentarios y suscripciones | PostgreSQL + PostgREST |
| News Job | Crónica automática diaria a partir del ranking de jugadas | PHP plano + cron |
| Retos Service | Equipos, retos, resultados, puntos e insignias | PostgreSQL + PostgREST |
| Notification Service | Avisos por usuario con lectura aislada | PostgreSQL + PostgREST |
| Almacenamiento | Videos e imágenes | MinIO (compatible con S3) |

Resumen: **5 microservicios** (identidad, contenido, noticias, retos, notificaciones) y **1 aplicación web BFF**. El conteo sale de la estructura del monorepo.

## Vista general

```mermaid
flowchart TB
    subgraph Cliente
        U["Navegador<br/>Vue 3 + Inertia"]
    end

    subgraph Web["Aplicación web (BFF)"]
        W["Proxies por servicio<br/>+ reglas entre servicios"]
    end

    subgraph Servicios
        A["Auth Service<br/>(Laravel)"]
        C["Content Service<br/>(PostgREST)"]
        N["News Service<br/>(PostgREST)"]
        T["Retos Service<br/>(PostgREST)"]
        NT["Notification Service<br/>(PostgREST)"]
        J["Job diario de noticias"]
    end

    subgraph Datos
        ADB[("DB identidad")]
        CDB[("DB contenido")]
        NDB[("DB noticias")]
        TDB[("DB retos")]
        NTDB[("DB notificaciones")]
        S3[("Almacenamiento S3")]
    end

    U --> W
    W --> A & C & N & T & NT
    A --- ADB
    C --- CDB
    N --- NDB
    T --- TDB
    NT --- NTDB
    J --> C
    J --> NDB
    A --> S3
    U -. "subida directa con URL firmada" .-> S3
    W -. "avisos best-effort" .-> NTDB
    A -. "avisos best-effort" .-> NTDB
```

## Flujo de una acción con notificación

Ejemplo: una persona da like a una jugada.

```mermaid
sequenceDiagram
    actor U as Usuario
    participant V as Vue (navegador)
    participant B as BFF (Web App)
    participant C as Servicio de contenido
    participant N as Servicio de notificaciones

    U->>V: Da like a una jugada
    V->>B: Petición con cookie de sesión y token CSRF
    B->>C: Reenvía con el JWT del usuario
    C-->>B: Resultado (la seguridad por fila valida la autoría)
    B-->>V: Respuesta
    B--)N: Aviso al autor (best-effort, sin bloquear)
```

Puntos clave:

- El navegador nunca habla directamente con los servicios; siempre pasa por el BFF.
- El token vive en cookies seguras; el BFF lo renueva de forma transparente cuando expira.
- Si el aviso falla, el like se conserva y el error se registra.

## Sesión y autenticación

```mermaid
flowchart LR
    L["Inicio de sesión"] --> T["Token de acceso<br/>(vida corta)"]
    L --> R["Refresh token<br/>(vida larga, rotativo)"]
    T --> API["Peticiones a los servicios"]
    API -- "expira" --> RF["Renovación automática en el BFF"]
    RF --> T
    RF --> R
```

El mismo secreto de firma valida el JWT en el servicio de identidad y en cada PostgREST, de modo que cada servicio puede decidir por sí mismo sin consultar al de identidad en cada petición.

## Datos por servicio (a nivel conceptual)

| Servicio | Guarda |
|---|---|
| Identidad | Usuarios, roles y permisos, solicitudes de rol, amistades, perfiles, sesiones y registros de auditoría |
| Contenido | Categorías, jugadas, likes y comentarios |
| Noticias | Artículos (automáticos y de editores), reacciones, comentarios y suscripciones |
| Retos | Equipos, integrantes, retos, resultados, puntos, insignias e historial de puntos |
| Notificaciones | Avisos por usuario con estado de lectura y de resolución |

Las referencias entre servicios (por ejemplo, el autor de una jugada) son **lógicas**: se guarda el identificador del usuario, sin clave foránea entre bases.

## Contadores y vistas

Los contadores (likes, comentarios) se calculan en vistas de lectura en vez de mantenerse con triggers, para evitar estado duplicado que pueda desincronizarse. La excepción consciente son los puntos de los equipos, que se acumulan sobre un historial auditado.

## Tolerancia a fallos

- Cada servicio falla de forma aislada.
- La pantalla de Inicio consulta varias fuentes en paralelo; si una falla, muestra un aviso discreto y renderiza las demás. Si fallan todas, muestra un error claro con reintento.
- Las notificaciones y el job de noticias son *best-effort*: registran el error y continúan.

## Entorno

Actualmente el proyecto se ejecuta en desarrollo local, con Docker Compose para las bases de datos y los servicios expuestos con PostgREST.
