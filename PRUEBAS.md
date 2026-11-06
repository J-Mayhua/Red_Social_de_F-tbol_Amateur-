# Calidad y pruebas

[← Volver al README](README.md)

Este documento separa con claridad **qué existe**, **qué se verificó** y **qué falta**. No se presentan cifras que no se hayan podido comprobar.

## Cómo se contaron las cifras

Los números de este documento salen de **lectura estática de los archivos de prueba** del servicio de identidad (inventario por archivo) y de los registros de verificación del proyecto. **No se ejecutó la suite** al preparar esta documentación, así que no se afirma cuántas pruebas pasan hoy.

## Resumen

| Área | Tipo de verificación | Estado |
|---|---|---|
| Servicio de identidad | Pruebas automatizadas (Feature y Unit) | Existen; con fallos preexistentes registrados |
| Contenido, Noticias, Retos, Notificaciones | Pruebas manuales de extremo a extremo por API | Realizadas en los flujos principales; sin suite automatizada |
| Frontend | Compilación de producción | Pasa sin errores |
| Frontend | Pruebas JS automatizadas | No existen todavía |
| Interfaz completa | Verificación visual y de humo con sesión real | Pendiente |

## Pruebas automatizadas del servicio de identidad

- **11 archivos de prueba**: 9 con casos reales y 2 plantillas de ejemplo.
- Áreas que cubren: registro y verificación de correo, inicio y cierre de sesión con renovación, límites de intentos de inicio de sesión, permisos y roles, recuperación de contraseña (incluidos casos límite), exportación y borrado de cuenta, inicio de sesión social y búsqueda de amigos.
- La búsqueda de amigos tiene **6 pruebas**: mínimo de caracteres, escape de comodines, límite de peticiones, autenticación obligatoria, ids inválidos y límite de ids.


### Estado conocido

- Hay **13 pruebas con fallos preexistentes** (inicio y cierre de sesión, verificación de correo, recuperación de contraseña, registro y perfil), registrados antes de que el proyecto tuviera control de versiones. Por eso no se compararon con un estado anterior y **no se usan como señal de regresión**: se verifica con pruebas dirigidas.
- Una migración antigua falla al ejecutarse sobre SQLite. En la base actual no afecta, pero sí en una instalación nueva. Está registrada como deuda técnica.
- Parte de la prueba de búsqueda simula la consulta en lugar de ejecutarla contra una base real.

## Verificación manual de extremo a extremo

Estas verificaciones se hicieron con peticiones reales y evidencia (estado HTTP, respuestas y consultas):

- **Torneos:** crear equipos, retar, aceptar, reportar un resultado, comprobar que el reportante no puede confirmarlo, confirmarlo desde el otro capitán, y verificar puntos, historial e insignia. Se repitió tras cambios de interfaz.
- **Restricciones de torneos:** un usuario solo capitanea un equipo.
- **Amistades entre servicios:** solicitar, aceptar y agregar al equipo; agregar a alguien que no es amigo se rechaza con un mensaje claro.
- **Notificaciones:** like que genera aviso al autor, aislamiento por usuario y deduplicación de avisos sin leer.
- **Noticias:** operaciones principales de publicación, likes, comentarios y permisos.

## Pendiente de verificar

- Notificaciones: agrupación, acciones desde la lista, resolución automática y contador, con sesión real y consulta a la base.
- Interfaz de Amigos, Torneos, Inicio y navegación con sesión autenticada.
- Anchos de pantalla de 360, 768 y 1280 px, y estados de carga, vacío y error.
- Pruebas con datos realistas (del orden de cien publicaciones y cincuenta usuarios) y en un teléfono real.
- Prueba integral de los cinco servicios.

## Humo mínimo que no debe romperse

1. Iniciar sesión.
2. Dar like y comentar.
3. Crear una noticia siendo editor.
4. Aceptar un reto o una amistad desde Notificaciones.
5. Reportar y confirmar un resultado con dos cuentas.
6. Cerrar sesión.

## Plan de calidad

1. Reparar los fallos de la suite de identidad y la migración sobre SQLite.
2. Añadir un runner de pruebas para el frontend y convertir las comprobaciones temporales en pruebas persistentes.
3. Cubrir con pruebas los flujos críticos de seguridad.
4. Ejecutar la ronda de pulido y la prueba integral.
