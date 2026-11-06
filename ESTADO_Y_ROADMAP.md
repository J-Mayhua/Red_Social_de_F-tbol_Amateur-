# Estado y roadmap

[← Volver al README](README.md)

**Fuente del estado:** registros internos del proyecto fechados el 2026-10-15 (el servicio de identidad se revisó de nuevo el 2026-11-04). Versión y fecha de este documento: [COMPLETAR].

## Leyenda

- ✅ Verificado con evidencia (consulta, respuesta HTTP o prueba real)
- 🟡 Implementado, pendiente de verificación completa
- ⬜ Pendiente
- ⚠️ Deuda técnica o riesgo conocido

## Estado por servicio

| Servicio | Estado | Detalle |
|---|---|---|
| Identidad | ✅ Núcleo cerrado | Registro, sesión, roles y solicitud de rol de editor verificados. 🟡 Amistades, búsqueda y avatares. ⚠️ Inicio de sesión social, exportación y borrado de cuenta y foto de perfil en revisión |
| Contenido (jugadas) | ✅ Cerrado | Feed, comentarios, subida, ranking y likes sin duplicados |
| Noticias | ✅ Cerrado | Publicación, likes, comentarios, editor de texto enriquecido, portada y crónica automática. Suscripciones: base lista ✅, interfaz ⬜ por confirmar |
| Retos (torneos) | ✅ Base de datos cerrada | Reglas verificadas por API. 🟡 Interfaz a la espera de pruebas manuales con dos cuentas |
| Notificaciones | 🔵 En curso | Base de datos y avisos de like y comentario ✅. 🟡 Amistad, torneos, noticias, agrupación, acciones, contador y página de lista |
| Inicio social | 🟡 Implementado | Feed mixto con mezcla verificada con pruebas temporales; falta verificación visual y con sesión |
| Rediseño de interfaz | 🟡 Fases 0 a 2 | Tokens, navegación y componentes compartidos construidos y compilados; falta verificación visual |

## Deuda técnica conocida

- ⚠️ Suite de pruebas del servicio de identidad con fallos preexistentes y una migración antigua incompatible con instalaciones nuevas.
- ⚠️ Revisión del reparto de permisos entre administrador y moderador.
- ⚠️ Los permisos de roles existen como etiquetas y todavía no se aplican en todos los puntos.
- ⚠️ Deduplicación de notificaciones no atómica.
- ⚠️ Los scripts SQL se aplican a mano, sin registro automático de cuáles están aplicados.
- ⚠️ Hallazgos de la revisión interna de seguridad en proceso de corrección (ver [Seguridad](SEGURIDAD.md)).

## Roadmap

### Corto plazo
1. Verificar con sesión real Notificaciones (amistad, torneos, noticias, contador y página), Amigos e Inicio.
2. Comprobar la interfaz a 360, 768 y 1280 px y en sus estados de carga, vacío y error.
3. Pruebas manuales de Torneos con dos cuentas.

### Medio plazo
4. Ronda única de pulido: camino de un usuario nuevo, mensajes de error en español en toda la app, datos realistas y prueba en teléfono.
5. Rediseño de Retos, Amigos, Feed y Perfil (fases 3 a 5).
6. Endurecimiento de seguridad y reparación de la suite de pruebas.
7. Foto de perfil funcional.

### Largo plazo
8. Prueba integral de los cinco servicios.
9. Límite de integrantes por equipo (depende de definir la modalidad del deporte) y borrado lógico.
10. Monetización, solo con comunidad activa: patrocinio de equipos de liga, torneos con inscripción, reserva de canchas y planes para editores. Son ideas, no compromisos.

## Fuera de alcance por ahora

Mensajes directos, streaming en vivo, fotos y publicaciones de texto, y nuevos microservicios. Se reconsiderarán con una comunidad activa.
