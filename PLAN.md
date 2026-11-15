# Plan de plataforma agrícola

## Objetivo y alcance

Construir una plataforma de comercialización agrícola para comunidades rurales de Colombia, dirigida a personas con distintos niveles de experiencia tecnológica y con conectividad limitada. El producto debe facilitar que productores publiquen cosechas, que compradores encuentren oferta y soliciten pedidos, y que comerciantes, asociaciones y administradores participen con permisos claros.

Este documento conserva el roadmap y registra el estado real del repositorio. La primera entrega de identidad implementa una base de cuenta y permisos; los cortes de productos y catálogo agregan captura privada y publicación explícita, y el flujo básico de pedidos permite solicitar y decidir pedidos. El catálogo enlaza cada producto publicado y disponible a su solicitud por ID. Pagos en línea, logística, precios de referencia, facturación y verificación de identidad no se consideran requisitos confirmados; deben definirse antes de incorporarlos.

## Estado observado del repositorio

- El repositorio ya tiene un scaffold de Next.js App Router, TypeScript, Tailwind CSS y ESLint, una portada neutral en español, scripts de lint/tipos y CI para lint, tipos y build.
- `.gitignore` excluye `.env*`. Las credenciales se revisaron sin mostrar ni registrar valores; no se ha conectado a Neon ni a servicios de correo o imágenes.
- Se inició la primera parte de identidad/permisos: autenticación por correo y contraseña, sesiones persistidas, roles básicos y migración versionada. Los cortes de productos y catálogo agregan captura privada y publicación explícita bajo una hipótesis de visibilidad pendiente de revisión; el flujo básico de pedidos está implementado. No se implementan perfiles comerciales, asociaciones ni sincronización offline. El primer corte offline posterior solo ofrece un shell estático genérico, sin cachear catálogo ni datos privados.

## Progreso de fundaciones e identidad

- **Hecho:** scaffold mínimo de Next.js 16.4.6, React 19.3, TypeScript, Tailwind CSS 4 y ESLint; scripts `lint` y `typecheck`; página inicial provisional en español sin flujos de producto; documentación de desarrollo básica; CI de lint, tipos y build.
- **Hecho:** dependencias instaladas limpiamente con `npm ci` y una caché npm aislada; se regeneró el lockfile tras detectar que el primer intento había dejado entradas transitivas incompletas.
- **Verificado desde una instalación limpia:** `npm ci`, `npm run lint`, `npm run typecheck` y `npm run build` finalizan correctamente; el build de Next.js 16.4.6 prerenderiza `/` y `/_not-found` como rutas estáticas.
- **Hecho en una rama de seguimiento basada en `main`:** prueba smoke con `node:test` que inicia el servidor de producción y valida el estado HTTP, el idioma español y el texto provisional de la portada; CI ejecuta `npm test` después del build.
- **Verificado en la rama de seguimiento:** `npm ci`, `npm run lint`, `npm run typecheck`, `npm run build` y `npm test` pasan; la prueba smoke pasa (1/1). La portada es únicamente un scaffold técnico provisional, no el producto agrícola terminado.
- **Nota técnica:** se mantiene ESLint 9.39.5 porque los plugins incluidos por `eslint-config-next` aceptan ESLint hasta la versión 9 y npm marca ESLint 9 como fuera de soporte. La versión 10 actual produce pares de dependencias inválidos con esos plugins; revisar la actualización del stack cuando Next la soporte o aprobar otro linter.
- **Identidad ya presente en `main`:** Better Auth con Prisma/PostgreSQL, sesiones validadas en servidor y roles múltiples; el registro asigna comprador y solo permite agregar productor o comerciante desde una acción autenticada. Asociación y administrador no se autoasignan; su proceso de aprobación sigue pendiente.
- **Hipótesis provisional de identidad heredada:** se mantiene por ahora el supuesto de roles múltiples; es flexible y sigue sujeto a revisión de producto.
- **Seguimiento de variables (solo nombres):** el proyecto requiere `AUTH_SECRET` y `DATABASE_URL`; la errata observada `COUDINARY_API_KEY` debe aclararse antes de integrar Cloudinary. No se copian valores ni se configura el proveedor en este corte.
- **Alcance pequeño de productos/inventario:** productores autenticados pueden crear y editar sus propios registros privados con nombre, categoría, cantidad, unidad, precio COP y municipio/localidad. La lectura y escritura filtran por propietario en servidor. El estado no se guarda por separado: cantidad mayor que cero significa disponible y cero significa agotado. No se publica el producto ni se decrementa inventario por ventas.
- **Hipótesis de visibilidad pública pendiente de revisión:** la vista web del catálogo muestra solo registros cuya persona propietaria publicó explícitamente y cuya cantidad es mayor que cero. Los campos propuestos para exposición son nombre, categoría, cantidad/unidad, precio COP por unidad y municipio; se excluyen teléfono, correo, dirección precisa e identificadores/datos privados del productor. La migración agrega `isPublished` con valor por defecto `false`; solo el productor propietario puede publicar o retirar. Esto no es una política aprobada ni una decisión confirmada.
- **Hipótesis provisionales de producto:** categorías iniciales (frutas, hortalizas, tubérculos, granos/cereales, otro); unidades iniciales (kg, unidad, arroba, bulto, canastilla); el precio en pesos enteros COP corresponde a una unidad de medida; cantidad admite hasta tres decimales; municipio/localidad es texto libre. No son taxonomía, unidades ni formatos aprobados: confirmar con usuarios antes de ampliar el flujo.
- **Verificado en el corte privado de productos:** el cliente Prisma se genera y el esquema valida sin conexión; 5 pruebas nuevas cubren campos, cantidades, permisos de productor/propietario y estado derivado. No se conectó ni escribió en Neon y no hay base PostgreSQL local desechable disponible para probar la migración.
- **Preview de Vercel pendiente:** el check de Vercel falló, pero no se pudieron consultar sus logs porque el CLI no tiene credenciales configuradas en este entorno. No se modificó configuración ni se leyeron valores de secretos; no es posible confirmar si falta alguna variable. El build local de producción sí pasa con placeholders locales.
- **Pendiente de prueba de integración:** la migración se generó desde el esquema sin ejecutarla contra Neon. No hay `psql` local ni daemon de Docker disponible para una base PostgreSQL desechable.
- **Integraciones excluidas:** no se habilitan verificación ni recuperación de correo (Brevo queda fuera) ni fotos (Cloudinary queda fuera hasta aclarar la errata y diseñar carga segura).
- **Validación local del flujo de pedidos:** Prisma generate/validate, lint, typecheck, 13 pruebas unitarias, build de producción con placeholders locales y smoke HTTP (1/1) pasan. La prueba PostgreSQL se omite si no se configura `TEST_DATABASE_URL` para una base desechable local.
- **Integración real/concurrencia pendiente:** en esta sesión no hay `psql`, el daemon de Docker no está activo, `TEST_DATABASE_URL` no está configurada y no existe `.env` local. La prueba de integración reportó `SKIP`; no se ejecutó ni se declara cubierta por mocks. Ninguna migración se aplicó a una base.
- **Integración catálogo-pedido:** los PR #5 (catálogo) y #6 (pedidos) ya están fusionados en `main`. Este corte añade a cada producto publicado y disponible un enlace accesible a `/catalogo/[productId]/pedido`; la proyección pública expone solo el ID requerido y los campos públicos aprobados. Los productos privados/retirados o agotados siguen excluidos tanto del catálogo como de la acción de solicitud. No se modifican pagos, contacto, notificaciones ni funcionamiento offline.
- **Primer corte offline seguro:** se añade un manifest y un service worker propio compatible con el stack instalado (Next.js 16.4.6 y TypeScript 5; no se añade plugin ni dependencia). En la instalación se guarda una única respuesta HTML estática generada dentro del worker, sin hacer `fetch` ni cachear respuestas del servidor. Solo las navegaciones GET exactas a `/` y `/catalogo`, sin query string, se intentan en red y reciben esa pantalla genérica tras un fallo de red. La pantalla no contiene productos ni stock y avisa que se necesita conexión para solicitar.
- **Privacidad y límites offline:** no se cachean páginas/RSC, assets descargados, catálogo, autenticación, `/mi-cuenta`, pedidos, APIs, mutaciones ni URLs con query. No se envían cookies para el único objeto cacheado (shell sintético con `credentials: omit`), no se cachean respuestas de red y la política rechaza `Set-Cookie`. La caché contiene una sola pantalla y al activar una versión nueva se elimina la caché antigua de este shell. El indicador visible advierte que disponibilidad puede estar desactualizada y que las solicitudes no se guardan ni confirman offline. El registro inicial requiere conexión. No hay lectura de catálogo offline ni borradores/sincronización.
- **Pruebas offline:** se agregan pruebas de allowlist/exclusiones y de fallback de navegación con fallo de red; `npm test` también comprueba en producción que se sirven el worker y el manifest. La prueba de fallback verifica la política en Node, no sustituye pruebas de navegador/dispositivo.
- **Validación local de este corte:** `npm ci` terminó correctamente (525 paquetes; npm reportó 8 vulnerabilidades moderadas y un aviso de engine para `@prisma/streams-local@0.1.11`, que pide Node 22; este proyecto corre con Node 20.20.2). Versiones instaladas: Next.js 16.4.6 y TypeScript 5.9.3. Pasaron `npm run db:generate` (Prisma Client 7.10.1), `npx prisma validate`, `npm run lint`, `npm run typecheck`, `npm run test:unit` (10/10), `npm run build` y `npm test` (5/5: 3 pruebas de política offline y 2 smoke HTTP de producción). El build y Prisma usaron solo el URL/secreto placeholder del proceso local; no se conectó a ninguna base. No se ejecutó integración/concurrencia PostgreSQL por no disponer de una base descartable.
- **CI y Preview del PR #8:** la ejecución CI `43677478758` terminó correctamente; pasaron `npm ci`, Prisma generate, lint, typecheck, unitarias, build y `npm test`. El check de Vercel falló con el mensaje exacto `Authorization required to deploy.`; no hay Preview verificado. No se modificó ni se intentó autorizar la integración de Vercel.

## Decisiones provisionales de productos, inventario y catálogo

El corte de gestión captura datos privados para que el productor administre su oferta. El pequeño catálogo de solo lectura es una hipótesis de producto que debe revisarse antes de considerarla política aprobada.

1. **Campos mínimos:** nombre, categoría, cantidad disponible, unidad de medida, precio entero en COP por unidad de medida y municipio/localidad en texto libre.
2. **Taxonomía y unidades:** las opciones de categoría y unidad son una lista inicial técnica para hacer el formulario consistente; requieren validación con productores y comerciantes. No añadir nuevas opciones sin confirmar necesidad.
3. **Disponibilidad:** se deriva de la cantidad guardada (mayor que cero: disponible; cero: agotado), evitando un estado independiente que pueda contradecir el inventario.
4. **Propiedad:** toda consulta y mutación se limita al usuario autenticado; las acciones de producto requieren rol productor. No hay decremento por pedidos, que quedan fuera de alcance.
5. **Imágenes:** no se guarda ni carga fotografía. No integrar Cloudinary hasta aclarar la errata observada en la variable `COUDINARY_API_KEY` y diseñar carga segura.
6. **Hipótesis de catálogo público (pendiente):** solo mostrar productos publicados explícitamente y con cantidad mayor que cero; exponer nombre, categoría, cantidad/unidad, precio COP y municipio. No exponer contacto ni dirección precisa. Los registros existentes y los nuevos comienzan privados; publicar/retirar exige sesión, rol productor y propiedad verificada en servidor. Esta visibilidad no está confirmada por responsables de producto o usuarios.

## Requisitos

### Funcionales recibidos

- Perfiles y permisos para productores, compradores, comerciantes, administradores y asociaciones.
- Publicación y gestión de productos agrícolas e inventario; catálogo y búsqueda para compradores; flujo de pedidos.
- Enlaces compartibles para productos/ofertas y distribución por redes sociales.
- Notificaciones a usuarios.
- Interfaz en español natural de Colombia, priorizando teléfonos y personas con baja familiaridad digital.
- Funcionamiento útil en condiciones de conectividad intermitente.

### No funcionales recibidos

- Una experiencia mobile-first, de baja fricción y bajo consumo de datos.
- Seguridad y privacidad adecuadas para datos personales, roles e inventario.
- Pruebas unitarias, de integración, autorización, concurrencia, funcionamiento offline y E2E.
- Next.js App Router, TypeScript y Tailwind son tecnologías posibles; Neon PostgreSQL y Prisma son el punto de partida; Brevo y Cloudinary son servicios disponibles esperados. La selección y configuración final debe verificarse antes de implementarlos.

### Decisiones de producto pendientes

Resolver con las personas responsables del producto, antes de cerrar los flujos y el esquema:

1. ¿Se confirma que un usuario pueda tener varios roles? ¿Quién puede crear asociaciones, aprobar su verificación y administrar sus miembros?
2. ¿Los comerciantes publican oferta propia, compran, o ambas cosas? ¿Qué operaciones puede hacer cada rol?
3. ¿Qué unidades, cantidades mínimas, disponibilidad por temporada y ubicaciones se necesitan?
4. ¿Cuándo se compromete el inventario: al solicitar, al aceptar un pedido o en otro momento? ¿Qué estados y reglas de cancelación aplican?
5. ¿Cómo se acuerdan entrega/recogida, precio final, contacto y pago? No se presume pago en línea.
6. ¿Qué información de un producto o productor puede ver cualquier persona que abra un enlace compartido?
7. ¿Qué canales y tipos de notificación son prioritarios? ¿Se requiere correo, SMS, WhatsApp u otro canal?
8. ¿Qué restricciones de privacidad, conservación de datos y tratamiento de información personal deben adoptarse para Colombia?
9. ¿Qué alcance offline se espera y qué dispositivos/navegadores deben soportarse?

## Arquitectura propuesta

### Stack inicial

- **Aplicación web:** Next.js App Router + TypeScript, con vistas renderizadas en servidor para el catálogo público y pantallas autenticadas para operaciones. Tailwind CSS para una interfaz adaptable y consistente.
- **Backend:** Server Actions para mutaciones iniciadas por formularios y componentes de la aplicación, con autorización y validación en servidor. Route Handlers para webhooks de proveedores, endpoints explícitos de integración y vistas/operaciones compartibles que necesiten una URL HTTP estable. No se recomienda agregar tRPC inicialmente: para un cliente web del mismo Next.js añadiría una capa y contratos duplicados sin una necesidad confirmada de varios clientes/API pública.
- **Persistencia:** PostgreSQL administrado en Neon; Prisma como ORM y migraciones versionadas. Usar conexión apropiada para ejecución serverless y conexión directa para tareas de migración, según la configuración recomendada vigente de Neon/Prisma.
- **Imágenes:** Cloudinary para carga y transformación de imágenes. La autorización de carga debe emitirse desde el servidor; limitar formatos, tamaño y cantidad, y persistir identificadores/URLs y metadatos necesarios, nunca secretos del proveedor en el navegador.
- **Correo/notificaciones:** Brevo como opción de entrega de correo transaccional, desacoplada mediante un servicio interno. Registrar estado e intentos de entrega sin guardar más datos personales de los necesarios.
- **Despliegue:** Vercel está previsto, con Next.js y Neon, entornos separados, migraciones controladas y secretos en el gestor de configuración del entorno.

### Límites y flujo de datos

El navegador se comunica con la aplicación Next.js; el servidor valida sesión, rol, entrada y reglas de negocio antes de consultar Prisma/PostgreSQL o invocar proveedores. El cliente no recibe acceso directo a la base de datos ni secretos de Brevo/Cloudinary. Las operaciones de inventario y pedido que dependan entre sí se realizan en transacciones de base de datos. Webhooks se validan y procesan de forma idempotente.

### Estructura de carpetas propuesta

La estructura es una guía para iniciar el repositorio vacío; no representa archivos ya existentes.

```text
src/
  app/
    (public)/                 # Inicio, catálogo y páginas compartidas
    (auth)/                   # Inicio/cierre de sesión
    (dashboard)/              # Áreas según permisos
    api/                      # Route Handlers y webhooks
  components/                 # Componentes de interfaz reutilizables
  features/
    auth/
    catalog/
    inventory/
    orders/
    notifications/
    organizations/
  lib/
    auth/                     # Sesión y políticas de autorización
    db/                       # Cliente Prisma
    validation/               # Esquemas de entrada
    integrations/              # Adaptadores Brevo y Cloudinary
  server/
    actions/                  # Mutaciones de interfaz
    services/                 # Casos de uso y reglas de negocio
prisma/
  schema.prisma
  migrations/
tests/
  unit/
  integration/
  e2e/
```

Mantener la lógica de negocio en servicios del servidor reutilizables por Server Actions, Route Handlers y pruebas; no duplicar reglas en componentes.

## Modelo de datos inicial

Entidades candidatas, sujetas a las decisiones pendientes:

- **User**, **Role** y **UserRole**: identidad, estado de cuenta y asignación de uno o más roles con restricciones en servidor.
- **ProducerProfile**: datos públicos/comerciales necesarios para la oferta, separados de los datos privados de cuenta.
- **Organization** (asociación) y **OrganizationMember**: asociación y relación de usuarios con ella; definir si existen permisos jerárquicos.
- **Product**: nombre, descripción, categoría, unidad, origen/ubicación aproximada, imágenes y estado de publicación.
- **InventoryItem** o inventario asociado al producto/oferta: cantidad disponible, unidad, versión y marcas de actualización.
- **Order**: comprador, productor, producto opcional, instantánea de nombre/cantidad/unidad/precio, estado y marcas de tiempo. El corte actual admite una unidad de producto por pedido.
- **OrderHistory**: eventos de solicitud, aceptación y rechazo con actor/estado/fecha; no registra datos de contacto.
- **InventoryMovement**: registro auditable de ajustes, reservas, confirmaciones y liberaciones.
- **ShareLink** (o slug público): referencia opaca/estable a contenido publicable, con vigencia o revocación si el producto lo requiere.
- **Notification** y **NotificationAttempt**: destinatario, tipo, referencia, estado e intentos acotados de entrega.
- **AuditEvent**: operaciones sensibles, actor y entidad afectada, sin incluir credenciales o contenido innecesario.

Aplicar claves foráneas, restricciones de cantidades y estados, índices para filtros del catálogo/pedidos y marcas de tiempo. Evitar almacenar información personal que no se requiera para operar. No añadir tablas de pagos, transporte o facturación hasta confirmar esos casos.

## Identidad, autorización y flujo de pedidos

- Better Auth administra creación, lectura, vencimiento y revocación de sesiones en servidor; el adaptador Prisma las persiste en PostgreSQL. La política y el alcance son provisionales y requieren revisión.
- Comprobar autorización en cada caso de uso/Server Action/Route Handler; ocultar botones en la interfaz es solo una ayuda de UX, no control de acceso.
- Definir reglas explícitas por rol y organización, y probar tanto acceso permitido como denegado. No confiar en un `role` enviado por el cliente.
- Hipótesis inicial de pedido: al solicitar, el comprador mueve cantidad libre a reservada mediante una actualización condicional atómica. El productor la acepta (consume reserva) o rechaza (libera reserva); los cambios de estado e inventario se confirman juntos. El pedido conserva una instantánea del producto. La regla de reservar al solicitar requiere aprobación del producto.
- Usar clave de idempotencia o protección equivalente para reintentos de creación/confirmación; limitar transacciones y revisar conflictos de escritura. Definir cancelaciones y liberación de inventario junto con el flujo final.

## Experiencia mobile-first y modo offline

- Empezar por pantallas de teléfono, controles grandes, navegación corta, etiquetas comprensibles, formularios con pocos campos por paso, mensajes de error accionables y formatos locales. Validar comprensión con usuarios rurales; “español colombiano” no debe basarse en estereotipos.
- Reducir transferencia inicial: paginación, carga diferida de imágenes, tamaños responsivos y compresión; evitar descargar catálogos completos.
- **Implementado (primer corte):** service worker y manifest sin plugin, shell HTML sintético independiente de datos, con fallback solo para navegaciones GET exactas a `/` y `/catalogo` sin query. El shell informa que no hay productos ni cantidades offline y requiere conexión para solicitar. Los archivos estáticos descargados y las páginas/respuestas dinámicas no se cachean en esta fase.
- **Pendiente de decisión y diseño:** evaluar, con pruebas específicas de privacidad, una política separada para lectura de catálogo público y recursos estáticos; solo habilitarla si puede demostrarse la exclusión de cookies y respuestas sensibles. No se ofrece lectura de productos recientes en este corte. Borradores y sincronización requieren aprobación, protección local, retención y reglas explícitas de conflicto antes de implementarse.
- No afirmar que pedidos, stock o disponibilidad están confirmados mientras el dispositivo está desconectado. Las escrituras se validan en servidor al sincronizar; resolver conflictos de manera explícita y nunca sobrescribir cambios de inventario silenciosamente. Determinar retención, borrado y protección de borradores locales.
- No almacenar credenciales, datos sensibles o tokens de sesión en cachés accesibles a scripts; excluir respuestas privadas y mutaciones del caché offline hasta diseñar controles y pruebas específicos.

## Enlaces, imágenes e integraciones

- Los enlaces compartibles deben abrir una vista pública intencionalmente limitada, con metadatos de vista previa (Open Graph) adecuados y sin exponer datos de contacto/personales salvo autorización expresa.
- Validar que el objeto siga publicado y que la persona propietaria pueda revocar/actualizar su publicación. Usar identificadores no enumerables cuando corresponda y protección contra scraping/abuso.
- Cloudinary: validar autorización, tipo MIME real, tamaño y límites; usar transformaciones para miniaturas y formatos adecuados; evitar contenido ejecutable.
- Brevo: preferir correo transaccional solo después de confirmar canal, consentimiento y contenido. Implementar reintentos acotados y observabilidad; no exponer llaves del proveedor.
- Definir comportamiento degradado ante indisponibilidad de proveedores: el pedido principal no debe perderse por una falla de notificación o carga de vista previa.
- No se proponen integraciones de pago ni conexiones de prueba en esta fase.

## Seguridad, privacidad y operación

- Validación de entradas en servidor, consultas parametrizadas mediante Prisma y controles de acceso por objeto (por ejemplo, impedir modificar producto/pedido de otro productor).
- Cookies seguras y protección CSRF según el mecanismo de sesión; protección contra abuso con límites de solicitudes en rutas de autenticación, escritura, búsqueda y creación de enlaces.
- Validar firma/origen de webhooks, hacerlos idempotentes y guardar solo el resultado necesario.
- No incluir secretos en el repositorio, documentos, logs, errores al usuario o código del cliente. Mantener `.env` local ignorado y gestionar secretos fuera del repositorio; documentar únicamente nombres de variables cuando sean confirmados.
- Minimización, acceso, corrección y eliminación/retención de datos deben definirse con responsables de privacidad conforme a obligaciones aplicables en Colombia; este plan no sustituye revisión legal.
- Registrar eventos operativos y de auditoría sin contraseñas, tokens, datos de pago ni contenido privado innecesario. Definir respaldo, restauración, alertas, retención y respuesta a incidentes antes del lanzamiento.

## Estrategia de pruebas y calidad

- **Unitarias:** validadores, permisos, transiciones de pedido, cálculo de cantidades y reglas de publicación.
- **Integración:** servicios contra PostgreSQL de prueba, restricciones de esquema, transacciones y persistencia de pedidos/inventario; adaptadores de integraciones con dobles controlados, sin llamadas reales a proveedores en CI.
- **Autorización:** matriz rol/operación/propietario/organización; cubrir accesos prohibidos y acceso directo a endpoints.
- **Concurrencia:** dos solicitudes simultáneas que compiten por las últimas unidades; el resultado no puede vender más stock que el disponible. Cubrir idempotencia y reintentos.
- **Offline/sincronización:** recursos cacheables, lectura sin red según alcance aprobado, borradores, reconexión, duplicados, conflictos y estados visibles para el usuario.
- **E2E:** recorridos principales de productor (publicar/actualizar oferta), comprador (buscar/compartir/solicitar pedido) y vendedor (aceptar/rechazar), incluyendo teléfono y permisos.
- **Calidad continua:** TypeScript estricto, lint, formato y migraciones reproducibles; CI ejecuta validaciones y pruebas sin secretos de producción.
- Probar rendimiento y accesibilidad en dispositivos modestos y red limitada. Definir métricas y umbrales concretos tras acordar dispositivos, volúmenes y nivel de servicio.

## Fases verificables

Los nombres bajo “áreas/archivos” son destinos propuestos porque no hay estructura de aplicación existente.

| Fase | Trabajo y dependencias | Áreas/archivos propuestos | Criterios de aceptación verificables |
|---|---|---|---|
| 0. Descubrimiento y decisiones | Validar roles, flujo y estados de pedido, unidades, publicación, pagos/contacto, offline, canales, datos y criterios de lanzamiento. Depende de responsables de producto y usuarios de prueba. | `PLAN.md`; especificación aprobada en la ubicación que acuerde el equipo. | Cada decisión de producto pendiente tiene responsable y resolución; alcance y exclusiones quedan aprobados antes de cerrar esquema y UX. |
| 1. Fundaciones técnicas | Crear Next.js/TypeScript/Tailwind, calidad/CI, configuración segura, Prisma/Neon, migraciones iniciales y entornos. | `package.json`, `src/app/`, `src/lib/db/`, `prisma/`, configuración de CI y variables documentadas sin valores. | Aplicación ejecuta en desarrollo; CI valida tipos/lint/build; migraciones se aplican y revierten en una base no productiva cuando se aprueben configuración y esquema; no hay secretos en el repositorio. |
| 2. Cuenta, perfiles y permisos | **Primer corte implementado:** cuenta con correo/contraseña, sesión, ruta privada, roles globales y auto-adopción limitada. **Pendiente:** perfiles comerciales, membresías/organizaciones, aprobación de roles privilegiados y matriz de permisos final. | `src/features/auth/`, `src/lib/auth/`, modelos Prisma y pruebas de autorización. | Pruebas demuestran denegación de roles privilegiados al input público; página privada redirige sin sesión; roles se consultan en servidor. La aprobación de producto sigue pendiente. |
| 3. Productos e inventario | **Primer corte implementado:** crear/editar registros privados de producto e inventario para productores, filtrados por propietario. **Corte actual:** estado explícito de publicación con valor seguro privado. **Pendiente:** fotografía, filtros, historial y reglas de inventario. Requiere identidad/permisos. | `src/features/products/`, rutas privadas de cuenta, Prisma/migración y pruebas de autorización/validación. | Productor autorizado crea y edita sus propios registros; comprador sin rol y otro propietario reciben denegación; cantidad cero se muestra agotada y positiva disponible; ningún registro nuevo se publica por defecto. |
| 4. Descubrimiento y compartición | **Hipótesis implementada para revisión:** página web de catálogo de solo lectura sin sesión, limitada a productos publicados y con cantidad positiva, con campos mínimos propuestos. No incluye enlaces compartibles. La política de visibilidad requiere aprobación antes de considerarse definitiva. | `src/app/catalogo/`, `src/features/catalog/`, proyección pública y pruebas de privacidad/estado. | Registros privados/retirados/agotados no aparecen; respuestas públicas contienen solo los campos permitidos; publicar/retirar requiere sesión, rol productor y propietario; la vista es utilizable en teléfono. |
| 5. Pedidos y notificaciones | Implementar estados acordados, solicitudes/decisiones, consistencia de inventario, historial e integración Brevo si se confirma. Requiere fase 3 y decisiones de fase 0. | `src/features/orders/`, `src/features/notifications/`, `src/server/services/`, webhooks/Route Handlers y pruebas de integración/concurrencia. | Transiciones no autorizadas se rechazan; confirmaciones concurrentes no exceden stock; reintentos no duplican pedidos; falla de notificación no pierde el pedido. |
| 6. PWA y sincronización | **Primer corte implementado:** shell estático y fallback acotado a las rutas públicas exactas `/` y `/catalogo`; indicador de conexión. No guarda catálogo ni escrituras. Futuro caché de catálogo/borradores/sincronización requiere aprobación y controles demostrables. | `public/sw.js`, `public/offline-policy.js`, `src/components/offline-status.tsx`, `src/app/` y pruebas de política offline. | El shell genérico se muestra ante fallo de red solo en rutas admitidas; catálogo/productos, datos privados y mutaciones no se cachean; no se solicita ni confirma offline. Las futuras capacidades necesitan pruebas en navegadores/dispositivos objetivo y reglas explícitas. |
| 7. Endurecimiento y piloto | Completar pruebas, accesibilidad, rendimiento, seguridad, respaldo/restauración, monitoreo, manuales y piloto con usuarios. Requiere fases 1–6. | `tests/`, configuración de despliegue/observabilidad, documentación operativa y guía de usuario. | Recorridos E2E y controles acordados pasan; no hay bloqueos críticos de seguridad/accesibilidad; restauración y procedimiento de incidentes se ensayan; piloto obtiene aprobación de responsables y usuarios. |

## Riesgos principales y mitigación

| Riesgo | Consecuencia | Mitigación y señal de verificación |
|---|---|---|
| Conectividad inestable o teléfonos modestos | Abandono, datos agotados o operaciones que parecen haberse perdido. | Medir en redes/dispositivos objetivo; páginas ligeras, reintentos visibles y alcance offline acotado; pruebas de desconexión y reconexión. |
| Inventario desactualizado y pedidos simultáneos | Sobreventa o pérdida de confianza. | Fuente de verdad en servidor, transacciones/condiciones de actualización, historial de movimientos y pruebas concurrentes. |
| Roles y pertenencia a asociaciones ambiguos | Exposición o modificación de datos ajenos. | Resolver matriz de permisos en fase 0; autorización por objeto en servidor; pruebas negativas para cada operación. |
| Compartición pública filtra información | Riesgo de privacidad y seguridad para productores. | Lista explícita de campos públicos, minimización, revocación y pruebas de respuesta pública antes de activar enlaces. |
| Costos, límites o indisponibilidad de terceros | Imágenes/notificaciones fallidas o costos inesperados. | Cuotas y presupuestos, límites de carga, adaptadores, reintentos acotados, alertas y degradación sin perder pedidos. |
| Sincronización duplica o contradice escrituras | Pedidos duplicados, estado confuso o stock incorrecto. | Idempotencia, servidor como autoridad, conflictos explicables y pruebas de reintento/concurrencia. |
| Producto no coincide con prácticas locales | Baja adopción o flujos que no se pueden completar. | Investigación con productores/compradores de las comunidades objetivo y piloto antes del lanzamiento amplio. |
| Repositorio sin aplicación ni documentación | Decisiones erróneas al asumir dependencias, despliegue o requisitos no observados. | Tratar el plan como propuesta, validar con responsables en fase 0 y no presentar elementos propuestos como existentes. |

## Criterio de salida de planificación

La autenticación, los roles, la captura privada inicial de producto/inventario, un catálogo público bajo hipótesis, el flujo básico de pedidos y un shell estático offline acotado están implementados. La integración enlaza cada oferta pública y disponible a su solicitud; la visibilidad pública y las demás decisiones provisionales siguen abiertas. Antes de ampliar la fase 2 hacen falta verificación/recuperación de cuenta, perfiles y el proceso confiable para asociación/administrador. La taxonomía, unidades y política de publicación requieren validación. No se implementan caché del catálogo, borradores, operaciones ni sincronización offline.

### Fuentes técnicas consultadas

- [Autenticación en Next.js](https://nextjs.org/docs/app/guides/authentication): recomienda una biblioteca para autenticación, sesiones y autorización.
- [Integración de Better Auth con Next.js](https://www.better-auth.com/docs/integrations/next): documenta Route Handlers, Server Components y la integración Next.js 16 (`proxy`).
- [Better Auth con Prisma](https://www.better-auth.com/docs/adapters/prisma): documenta Prisma 6 y Prisma 7; el adaptador actual no soporta Prisma 8.
- [Correo y contraseña en Better Auth](https://www.better-auth.com/docs/authentication/email-password): define el flujo, validaciones y requisitos de verificación/correo.
- [Límites de solicitudes en Better Auth](https://www.better-auth.com/docs/concepts/rate-limit): documenta el rate limiting integrado y el almacenamiento para serverless.
- [Prisma con Neon](https://www.prisma.io/docs/orm/v6/overview/databases/neon): documenta el adaptador serverless y la recomendación de conexión agrupada para ejecución serverless.
