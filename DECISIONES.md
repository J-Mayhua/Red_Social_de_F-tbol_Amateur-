# Decisiones técnicas

[← Volver al README](../README.md)

Registro de las decisiones de diseño más relevantes. Cada una indica **qué elegí**, **por qué**, **qué descarté** y **el costo asumido**. Explica el razonamiento, no la implementación.

---

## 1. La seguridad vive en la base de datos

- **Qué elegí:** políticas de seguridad por fila (RLS) que leen la identidad y el rol del JWT.
- **Por qué:** la regla se aplica siempre, incluso si alguien llama a la API sin pasar por la interfaz. Ocultar botones en la interfaz es solo experiencia de usuario, no seguridad.
- **Qué descarté:** proteger rutas con middleware en la aplicación web, porque duplicaría la regla y dejaría la base sin protección propia.
- **Costo:** las reglas se escriben en SQL y exigen disciplina: cada permiso necesita su propio otorgamiento además de la política.

## 2. Microservicios con base de datos propia; reglas entre servicios en el BFF

- **Qué elegí:** una base por servicio, sin consultas cruzadas. Una regla que involucra dos servicios (por ejemplo, "solo amigos al equipo") se aplica en el BFF.
- **Por qué:** independencia de despliegue y de fallos.
- **Qué descarté:** fusionar bases para poder hacer joins, y políticas con referencias cruzadas.
- **Costo:** esas reglas del BFF no las impone la base de datos, así que no protegen frente a llamadas directas a la API. Es un punto a reforzar.

## 3. Los servicios sin lógica propia no llevan aplicación

- **Qué elegí:** Retos y Notificaciones son solo base de datos más PostgREST. Las reglas viven en la base (triggers, funciones, políticas).
- **Por qué:** menos piezas que mantener, desplegar y asegurar.
- **Qué descarté:** un contenedor de aplicación "por simetría" con otros servicios.

## 4. Sesión deslizante con refresh rotativo

- **Qué elegí:** token de acceso de vida corta y refresh de vida larga, guardado hasheado y rotado en cada uso. Cookies HttpOnly gestionadas por el BFF.
- **Por qué:** la sesión se siente permanente sin ampliar la ventana de uso de un token robado.
- **Qué descarté:** subir la duración del token de acceso.

## 5. Noticias propias en lugar de una API externa

- **Qué elegí:** dos fuentes: crónica automática diaria (a partir del ranking de jugadas) y artículos de editores verificados.
- **Por qué:** no depender de la cuota, los precios ni los términos de un tercero.
- **Qué descarté:** servicios de noticias externos.

## 6. El job de noticias es un script plano con cron

- **Qué elegí:** un script PHP pequeño, con consultas preparadas, validación de lo que recibe e idempotencia (no repite una noticia dentro de una ventana de tiempo).
- **Por qué:** su tarea es una llamada HTTP más una inserción; un framework completo añadiría cientos de paquetes sin aportar nada.
- **Qué descarté:** Laravel completo y Node.

## 7. Se revisa quién es editor, no cada artículo

- **Qué elegí:** una persona solicita el rol de editor, un administrador lo aprueba una vez y luego sus artículos se publican al instante. Administración conserva intervención posterior.
- **Por qué:** la plataforma no pretende hacer trabajo editorial artículo por artículo.
- **Qué descarté:** una cola de aprobación por artículo y un rol nuevo de "periodista".

## 8. Contadores en vistas, no en triggers

- **Qué elegí:** likes y comentarios se calculan al leer.
- **Por qué:** evita estado duplicado que pueda desincronizarse.
- **Excepción consciente:** los puntos de equipo se acumulan con trigger y se auditan en un historial, porque son un acumulado sobre un historial creciente y se usan para ordenar el ranking.

## 9. Anti-trampa de resultados en la base de datos

- **Qué elegí:** quien reporta un resultado no puede confirmarlo; quien confirma no puede alterar lo reportado; solo se puede reportar sobre un reto aceptado. Las columnas sensibles de los equipos (puntos, capitán, marca de liga) no se editan por API y solo cambian mediante mecanismos internos.
- **Por qué:** la integridad del ranking es el valor del módulo de torneos.
- **Qué descarté:** confiar en la política por fila para proteger columnas, porque esas políticas no distinguen columnas.

## 10. Notificaciones: escritura de confianza y lectura aislada

- **Qué elegí:** solo el backend crea avisos, con una cuenta de escritura dedicada. Los usuarios leen y marcan como leídas únicamente las suyas. Se evita duplicar avisos sin leer del mismo origen.
- **Por qué:** nadie puede fabricar avisos ni leer los de otra persona. Un fallo al notificar nunca debe romper la acción original.
- **Qué descarté:** permitir que los clientes inserten notificaciones, y guardar el nombre del actor en cada aviso (se desactualiza si el usuario lo cambia; se resuelve al leer).
- **Límite conocido:** la deduplicación no es atómica; dos peticiones simultáneas podrían duplicar un aviso.

## 11. Amigos dentro del servicio de identidad; sugerencias por texto

- **Qué elegí:** las amistades viven junto a los usuarios, con amistad mutua (hay que aceptar). Las sugerencias comparan el texto de ubicación del perfil.
- **Por qué:** permite integridad referencial real, y evita coordenadas exactas y datos sensibles.
- **Qué descarté:** un microservicio nuevo para amistades y la proximidad por GPS.

## 12. Cambios de base de datos idempotentes; la base viva manda

- **Qué elegí:** un script ya aplicado no se edita; los cambios van en archivos nuevos que se pueden repetir sin romper nada.
- **Por qué:** trazabilidad y reproducibilidad con una base que se evoluciona a mano.
- **Costo:** no hay un registro automático de qué scripts están aplicados. Un runner de migraciones es una mejora pendiente.

## 13. Alcance acotado

- **Qué elegí:** dejar fuera por ahora mensajes directos, streaming en vivo y publicaciones de texto. Cerrar primero Amigos y Notificaciones, luego una ronda de pulido y después una prueba integral.
- **Por qué:** los mensajes exigen tiempo real y moderación; el streaming exige un proveedor externo de pago; pulir antes obligaría a rehacer el trabajo.
- **Monetización:** se pospone hasta tener comunidad. Cobrar sin usuarios ahuyenta a los primeros.
