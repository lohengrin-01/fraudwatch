# FraudWatch — Plan del proyecto

> **Estado:** definición funcional avanzada; desarrollo aún no iniciado.  
> **Propósito:** proyecto de portafolio para demostrar competencias de **ingeniería de software** mediante una plataforma bancaria ficticia de detección de fraude y revisión humana.  
> **Última revisión:** 11 de noviembre de 2026.

## 1. Visión

FraudWatch será una aplicación web local que reproduce transacciones financieras históricas anonimizadas como si llegaran en tiempo real, **una cada tres segundos**. Un modelo de Machine Learning entrenado previamente asignará una probabilidad estimada de fraude a cada transacción. Cuando alcance el umbral fijo de alerta, la operación quedará retenida en la **simulación** para que un analista la investigue y decida aprobarla o rechazarla. Las transacciones por debajo del umbral se aprobarán automáticamente en el entorno ficticio.

La plataforma mostrará indicadores, gráficos, tabla de transacciones en vivo y avisos visuales temporales. Tendrá varios analistas, un administrador limitado a gestión de cuentas (más acceso de lectura a las métricas), autenticación, reserva de alertas y trazabilidad de decisiones.

**No es un producto bancario real:** no realizará transferencias ni bloqueará dinero real. Las probabilidades del modelo no garantizan detección de fraude; las etiquetas históricas servirán para evaluación y no estarán visibles durante las decisiones de investigación.

## 2. Objetivos y criterios de éxito

1. Demostrar arquitectura de software distribuida a escala manejable, APIs REST, frontend tipado y comunicación en tiempo real.
2. Integrar un modelo ML **entrenado offline** en un servicio de predicción y mostrar explicaciones SHAP para alertas.
3. Implementar correctamente un flujo de trabajo multiusuario (reserva, revisión, decisión y recuperación de errores).
4. Ejecutar toda la aplicación **solo localmente con Docker Compose**, a partir de un único repositorio.
5. Proveer pruebas unitarias y de integración, y documentación de APIs con OpenAPI/Swagger UI.

**Fuera de la promesa del proyecto:** rendimiento en producción bancaria, operación con dinero real o validación regulatoria; despliegue público; reentrenamiento automático; notificaciones por correo/SMS; infraestructura de mensajería asíncrona.

## 3. Alcance funcional confirmado

### 3.1 Datos y simulación

- **Fuente de entrenamiento:** dataset público con **transacciones reales anonimizadas** y etiquetas históricas de fraude.
- **Requisito indispensable:** un identificador consistente de cliente que permita reconstruir su historial de transacciones; debe confirmarse su presencia real en el dataset escogido.
- **Criterio de selección:** priorizar transacciones interpretables y realistas para la demo.
- **Tipos de operación:** **pendientes de seleccionar**, según los tipos y atributos existentes en el dataset; no se forzará una variedad incompatible.
- **Partición:** separación **cronológica** entre datos de entrenamiento y datos posteriores usados en la simulación/evaluación; evitar fugas temporales y variables que revelen resultados futuros.
- **Emisión:** reproducir transacciones históricas **en orden cronológico, una cada 3 segundos**, no inventarlas dinámicamente.
- **Controles:** iniciar, pausar y reiniciar manualmente.
- **Duración:** procesar continuamente hasta que alguien pause/reinicie o se agoten las transacciones; al finalizar, detenerse y permitir un nuevo reinicio manual.
- **Sesión compartida:** todos los analistas ven el **mismo flujo**; cualquiera puede iniciar, pausar o reiniciar.
- **Reinicio:** exigir confirmación antes de borrar los datos de simulación. Eliminar transacciones, alertas y decisiones de la corrida anterior, **pero conservar las cuentas y credenciales**. Notificar a las sesiones abiertas para sincronizar la interfaz.
- **Pausa:** detener la reproducción de nuevas transacciones, **sin detener la investigación de alertas existentes**.

> Nota: la elección inicial de un “simulador en tiempo real” quedó concretada más adelante como **reproducción cronológica de un dataset histórico**, presentada como flujo en tiempo real. La fuente de datos del MVP no será un generador sintético independiente.

### 3.2 Evaluación de fraude

- Detección mediante **Machine Learning** (no reglas de fraude como motor principal).
- Modelo propio entrenado offline sobre dataset público, cargado en el servicio de inferencia.
- Cada transacción obtiene una **probabilidad o puntuación de fraude**; usar un **umbral fijo** seleccionado al evaluar el modelo (valor numérico por definir).
- **Bajo el umbral:** aprobación automática en la simulación, con estado visible en historial.
- **Igual o sobre el umbral:** alerta, notificación y **retención** de la transacción para decisión humana.
- **SHAP:** calcular y conservar explicaciones **solo para transacciones que generen alertas**; presentar los factores que contribuyeron a la predicción, no afirmaciones causales ni pruebas de fraude.
- **Fallo del servicio ML:** dejar la transacción retenida con estado de **error de evaluación**; no aprobarla sin puntuación.
- **Recuperación de fallo:** el analista puede solicitar **reprocesamiento manual** de esas transacciones cuando el servicio ML esté disponible nuevamente.

### 3.3 Investigación y decisiones

- El analista puede abrir el detalle de una alerta con monto, fecha/hora, identificador de transacción y cliente, probabilidad de fraude, factores SHAP e **historial reciente del cliente**.
- Decisiones habilitadas: **aprobar, rechazar o marcar «En investigación»**.
- «En investigación» mantiene retenida la operación y permite decidir definitivamente más adelante; **no habrá un expediente avanzado de evidencias/notas** en el MVP.
- **Justificación obligatoria** al registrar cada decisión.
- Cada decisión registra su analista responsable y fecha/hora.
- Para evitar que dos personas decidan simultáneamente, un analista debe **tomar/reservar** una alerta antes de trabajarla. Otros analistas no pueden decidir sobre esa reserva.
- La reserva se **libera manualmente** o se resuelve mediante una decisión; **no caduca por inactividad**.
- Al **cerrar sesión**, las reservas del analista **permanecen** para que pueda retomarlas al regresar.
- Cuando el **administrador desactiva** a un analista, se **liberan automáticamente** sus alertas reservadas.

### 3.4 Interfaz del analista

- **Dashboard compartido** entre todos los analistas; no requiere dashboard personalizado por usuario.
- Indicadores de transacciones procesadas, alertas, operaciones de alto riesgo y estados de revisión.
- Gráficos con actualización en tiempo real.
- Tabla en vivo, por defecto **más recientes primero** (sin ordenamiento configurable por ahora).
- Filtros por **estado, nivel de riesgo y fechas**; búsqueda por ID de transacción o cliente.
- **Paginación en backend:** FastAPI consulta PostgreSQL y retorna una página de resultados con sus filtros.
- **Actualización automática en cualquier página** mientras llegan operaciones (mantener filtros y paginación coherentes).
- **Notificaciones visuales temporales** cuando surjan alertas; sin centro de notificaciones ni sonido.
- Detalle de transacción, historial de cliente, factores SHAP, botones de decisión, motivo obligatorio y estado de reserva.
- **WebSockets** para transmitir cambios y alertas; **reconexión automática y resincronización** con el backend después de una interrupción.

### 3.5 Métricas de evaluación del modelo

- Sección separada de evaluación con **precision, recall, falsos positivos y falsos negativos** (y demás métricas que puedan derivarse correctamente).
- Cálculo y **actualización en tiempo real** a medida que se procesan las transacciones.
- Acceso de **solo lectura** tanto para analistas como para administrador.
- Las **etiquetas históricas** alimentan los cálculos, pero se ocultan en la vista de investigación para no sesgar al analista.
- Interpretar las métricas como evaluación sobre el subconjunto histórico reproducido hasta ese momento; pueden fluctuar mucho al inicio.

### 3.6 Usuarios, permisos y autenticación

- **Varios analistas** pueden iniciar sesión simultáneamente.
- Existe también un **administrador**, introducido posteriormente a la idea inicial de “único rol de analista”.
- **Rol analista:** dashboard, controles compartidos del simulador, investigación, reservas, decisiones y consulta de métricas.
- **Rol administrador:** crear, desactivar y gestionar cuentas; consultar dashboard y métricas en lectura. **No** aprobar, rechazar ni investigar transacciones. (El detalle exacto de controles del simulador para administradores puede definirse después; el control está garantizado para analistas.)
- Gestión de cuentas y **restablecimiento de contraseña por administrador**; sin registro público de usuarios.
- Contraseña temporal asignada por el administrador: **cambio obligatorio en el primer inicio de sesión**.
- Usar **JWT con access tokens y refresh tokens**.
- Permitir cerrar **solo la sesión actual** o **todas las sesiones activas**.
- **Cambiar contraseña** invalida todas las sesiones anteriores.
- **Desactivar una cuenta** revoca inmediatamente su capacidad de operar, invalida la renovación y desconecta WebSockets; libera las alertas reservadas.
- Tras **5 intentos de contraseña incorrecta**, bloquear el inicio de sesión durante **15 minutos**.
- Almacenar contraseñas como **hash seguro**, nunca texto plano.

## 4. Arquitectura

La arquitectura confirmada, el diagrama conceptual y la estructura propuesta del monorepo se encuentran en [architecture.md](./architecture.md).

## 5. Flujos principales

### 5.1 Transacción de riesgo bajo

1. El simulador emite la siguiente operación cronológica.
2. El servicio ML devuelve puntuación de fraude.
3. Si está **por debajo** del umbral, Core API registra **aprobación automática**.
4. Se actualizan PostgreSQL, dashboard y tabla vía WebSockets.

### 5.2 Transacción sospechosa

1. El simulador emite una operación; ML devuelve puntuación sobre el umbral.
2. Se crea una **alerta retenida** y se calcula explicación SHAP.
3. Los analistas reciben una **notificación visual temporal** y ven la alerta en el dashboard.
4. Un analista **reserva** el caso y consulta historial, riesgo y explicaciones.
5. Registra justificación y decide **aprobar**, **rechazar** o pasar a **En investigación**.
6. La transacción y su decisión se actualizan para todos los usuarios.

### 5.3 Fallo al evaluar

1. La llamada REST al servicio ML falla o no produce una evaluación válida.
2. La transacción se retiene como **Error de evaluación**.
3. Un analista solicita **reprocesar** manualmente.
4. Si ML responde, se aplica el flujo normal de clasificación; si vuelve a fallar, sigue retenida.

### 5.4 Administración de cuenta

1. Administrador crea/restablece cuenta de analista con contraseña temporal.
2. En el primer ingreso, el analista debe cambiarla antes de acceder al sistema.
3. La nueva contraseña revoca las sesiones previas.
4. Si el administrador desactiva una cuenta, se bloquea el acceso inmediatamente y se liberan reservas.

### 5.5 Reinicio global

1. Cualquier analista solicita reiniciar.
2. Aparece confirmación explícita con advertencia de pérdida de trabajo activo.
3. Si confirma, se detiene la reproducción y se eliminan **solo los datos de la simulación** (transacciones, alertas y decisiones), conservando las cuentas.
4. Todos los clientes conectados resincronizan su estado.
5. Se permite empezar otra vez desde el principio del conjunto de reproducción.

## 6. Reglas de negocio y consistencia importantes

Estas reglas materializan decisiones del alcance; los detalles de implementación se resolverán durante el diseño:

- **Exclusividad de reservas:** una alerta solo puede tener un analista reservante activo; la reserva debe hacerse de forma atómica para evitar que dos analistas la tomen al mismo tiempo.
- **Autorización:** solo el titular de la reserva puede ejecutar decisiones sobre ella; los administradores no pueden decidir alertas.
- **Inmutabilidad de decisiones finales:** una transacción aprobada o rechazada no se debe decidir nuevamente por una solicitud duplicada.
- **Retención segura:** ninguna transacción sin evaluación ML válida se aprueba automáticamente.
- **Sin filtraciones de etiquetas:** las etiquetas reales no se incluyen en las respuestas de detalle que usa el analista.
- **Resincronización:** después de reconectar WebSockets, consultar el estado persistido para no depender de mensajes perdidos.
- **Reinicio coherente:** proteger operaciones concurrentes para que no aparezcan decisiones huérfanas ni transacciones de una corrida anterior después del borrado.
- **Datos efímeros de demo:** conservar usuarios y configuración del sistema, no el historial de operaciones tras el reinicio solicitado.
- **Estado «En investigación»:** continúa retenido hasta resolución posterior; aún no hay evidencias avanzadas.

## 7. Decisiones pendientes y riesgos

### Pendientes que sí afectan la construcción

1. **Dataset real**: encontrar uno público y legalmente utilizable con etiquetas, orden temporal, identificador consistente de cliente y variables interpretables. Es la dependencia más importante: los datos anonimizados pueden no incluir todas estas características simultáneamente.
2. **Tipo de operaciones**: depende del dataset seleccionado (transferencias, pagos con tarjeta, etc.).
3. **Features del modelo y simulador**: usar las mismas variables y transformaciones en entrenamiento e inferencia; evitar features con información futura.
4. **Modelo y umbral fijo**: elegir algoritmo, validación temporal, métrica principal y umbral según resultados reales sobre datos retenidos.
5. **SHAP**: comprobar compatibilidad y coste computacional con el modelo elegido.
6. **División de responsabilidades entre servicios**: especificar contratos REST, secuencia de eventos, idempotencia y manejo de fallos; no incorporar colas salvo cambio expreso de alcance.
7. **Esquema PostgreSQL y migraciones**: scripts SQL vs Alembic (pregunta 69 aplazada).
8. **Auth entre servicios y WebSockets**: decidir manejo seguro de credenciales, revocación de access tokens y refresh tokens, cookies/almacenamiento, y cierre de conexiones activas.
9. **Acceso del administrador a controles de simulación**: no definido explícitamente; al menos los analistas pueden controlarla.
10. **Métricas en tiempo real**: definir el cálculo sobre etiquetas históricas *solo de transacciones ya procesadas*, su frecuencia y cómo separar evaluación de la operación.
11. **Aspectos visuales**: diseño de pantallas, rutas, componentes, comportamiento preciso de filtros y actualizaciones.
12. **Logs y monitoreo**: aplazado por decisión explícita.

### Riesgos de alcance

- Aunque el proyecto busca mantenerse pequeño, **tres microservicios, WebSockets, RBAC, JWT/refresh tokens, sesiones revocables, SHAP y concurrencia multiusuario** son un alcance significativo. Evitar agregar funciones sin necesidad.
- Un dataset de fraude real anonimizado con **clientes identificables de manera persistente + tiempo ordenable + etiquetas** puede ser difícil de conseguir. **No sustituir silenciosamente por datos sintéticos**: discutir alternativas si no se encuentra.
- No confundir “transacciones históricas reproducidas cada 3 segundos” con un motor de pagos realmente conectado a un banco.

## 8. Plan de implementación — empezar por el frontend

El desarrollo comenzará por el frontend, en tareas pequeñas y autocontenidas descritas en el [backlog](./tasks.md). La primera tarea será crear un proyecto vacío de React + TypeScript + Tailwind CSS con una prueba que pase; todavía no se implementarán funciones de negocio.

### Fase 0 — Base del frontend y contratos provisionales

- Crear el proyecto vacío y documentar cómo ejecutar su prueba inicial.
- Definir rutas, componentes y contratos tipados de la interfaz, con estados de carga, vacío y error.
- Introducir un adaptador de datos intercambiable y fixtures deterministas para desarrollar sin backend.
- Identificar claramente el modo de demostración con datos ficticios; estos fixtures no sustituyen el dataset real del MVP ni sirven como resultados de evaluación del modelo.
- Mantener provisionales los tipos de operación, variables, factores SHAP y niveles de riesgo hasta validar los datos y el modelo.

**Entregable:** base de interfaz verificable sin servicios ni base de datos.

### Fase 1 — Interfaz navegable con datos de prueba

- Implementar login, cambio de contraseña y navegación por rol con sesiones simuladas.
- Construir dashboard, gráficos, tabla paginada con filtros, detalle e historial del cliente.
- Incorporar reservas, decisiones con motivo, reprocesamiento y notificaciones mediante el adaptador de prueba.
- Construir controles de inicio, pausa y reinicio con confirmación, evaluación del modelo y administración de cuentas.
- Verificar estados y comportamientos con pruebas de componentes; los permisos simulados no sustituyen la autorización del backend.

**Entregable:** prototipo local navegable que permite revisar todos los flujos antes de conectar servicios reales.

### Fase 2 — Viabilidad de datos y modelo

- Realizar una revisión temprana y acotada del dataset durante el trabajo de interfaz; no es necesario esperar a terminar todas las pantallas.
- Confirmar licencia, transacciones reales anonimizadas, identificador consistente de cliente, orden temporal y etiquetas; discutir alternativas si no existe una fuente compatible.
- Preparar particiones cronológicas de entrenamiento, validación y reproducción/evaluación, sin fugas de información futura.
- Entrenar un modelo base y elegir el umbral fijo usando validación, preservando los datos posteriores de evaluación.
- Comprobar compatibilidad y coste de SHAP y reconciliar los contratos provisionales con las variables reales.

**Entregable:** fuente validada, pipeline reproducible, modelo serializado y contrato de variables; requisito previo a la integración real de datos e inferencia.

### Fase 3 — Servicios, persistencia y seguridad

- Resolver contratos REST, responsabilidades, estados, idempotencia, autenticación entre servicios y estrategia de migraciones.
- Crear los tres servicios FastAPI, PostgreSQL y Docker Compose local.
- Implementar autenticación JWT y sesiones revocables, contraseñas temporales, bloqueo temporal, permisos y gestión de cuentas desde el inicio de las APIs protegidas.
- Incorporar reproducción cada 3 segundos, inferencia, retención por errores y reprocesamiento manual.
- Implementar reservas exclusivas, decisiones trazables, reinicio coherente y métricas sin exponer etiquetas en investigación.
- Añadir pruebas unitarias y de integración junto con cada comportamiento y mantener OpenAPI/Swagger UI.

**Entregable:** APIs verificables con datos reales y reglas de seguridad y concurrencia aplicadas en el servidor.

### Fase 4 — Integración, tiempo real y presentación

- Sustituir el adaptador de prueba por las APIs reales sin cambiar los flujos de pantalla.
- Conectar WebSockets con autenticación, revocación, reconexión y resincronización del estado persistido.
- Verificar actualizaciones en cualquier página de la tabla, métricas, reservas y reinicio entre sesiones.
- Completar pruebas de integración de fallos y concurrencia; no incorporar una suite E2E completa al alcance.
- Documentar ejecución con Docker Compose, APIs, arquitectura, descarga de datos y guion de demo.

**Entregable:** monorepo documentado y demo completa ejecutable únicamente en local por otra persona.

## 9. Guion corto para mostrar el proyecto

1. Ejecutar `docker compose up --build` y abrir la aplicación local.
2. Iniciar sesión como analista.
3. Pulsar **Iniciar simulación** y observar una transacción cada 3 segundos.
4. Mostrar la actualización de indicadores, gráficos, tabla y aparición de una alerta.
5. Reservar una alerta, revisar historial del cliente, puntuación y factores SHAP.
6. Marcar **En investigación** con motivo y luego aprobar/rechazar con justificación.
7. Abrir una segunda sesión de analista para enseñar que el caso queda reservado.
8. Mostrar la sección de métricas en tiempo real sin exponer las etiquetas al analista durante su revisión.
9. Pausar la simulación y demostrar que las investigaciones continúan.
10. Mostrar, si es conveniente, un fallo controlado del ML y el reprocesamiento manual.
11. Confirmar un reinicio, verificar borrado de operaciones y conservación de cuentas.
12. Enseñar Swagger UI, Docker Compose, estructura del monorepo y pruebas.

## 10. Registro consolidado de decisiones tomadas

| Pregunta | Decisión |
|---|---|
| 1 | Simulación de transacciones en tiempo real (posteriormente concretada como reproducción histórica). |
| 2 | Detección por Machine Learning, no reglas + ML. |
| 3 | Decisiones manuales del analista. |
| 4 | Dashboard + tabla en vivo + notificaciones. |
| 5 | Detalle de operación, historial, factores y decisiones. |
| 6 | Autenticación (inicialmente analista; posteriormente se añadió administrador). |
| 7 | Iniciar, pausar y reiniciar simulador. |
| 8 | Borrar los datos de simulación al reiniciar. |
| 9 | Entrenar modelo propio con dataset público. |
| 10 | Ejecución únicamente local con Docker Compose. |
| 11 | Actualizaciones por WebSockets. |
| 12 | PostgreSQL. |
| 13 | React + TypeScript + Tailwind CSS. |
| 14 | Arquitectura de microservicios. |
| 15 | REST entre microservicios. |
| 16 | Pruebas unitarias y de integración. |
| 17 | Observabilidad/logs: aplazados. |
| 18 | Varios analistas. |
| 19 | Indicadores y gráficos en tiempo real. |
| 20 | Aprobar/rechazar/investigar con motivos. |
| 21 | Entrenamiento offline. |
| 22 | Monorepo. |
| 23 | Tres microservicios. |
| 24 | Umbral fijo para alertas. |
| 25 | Retener operaciones sospechosas para revisión. |
| 26 | Analista toma/reserva alerta. |
| 27 | Liberación manual de reservas. |
| 28 | Investigación sigue disponible al pausar simulador. |
| 29 | «En investigación» como estado, sin expediente avanzado. |
| 30 | Justificación obligatoria de decisiones. |
| 31 | Probabilidad + SHAP. |
| 32 | Aprobar automáticamente las operaciones de bajo riesgo. |
| 33 | Retener si falla ML. |
| 34 | Reprocesar manualmente si falla ML. |
| 35 | Gestión de cuentas mediante administrador. |
| 36 | Administrador gestiona usuarios; no decide operaciones. |
| 37 | Desactivación libera reservas. |
| 38 | Simulación compartida, controlable por cualquier analista. |
| 39 | Confirmación antes de reiniciar. |
| 40 | Una transacción cada 3 segundos. |
| 41 | Emisión continua hasta pausa/reinicio (limitada por el fin del dataset, según 47). |
| 42 | Tipos de transacciones pendientes según dataset. |
| 43 | Prioridad al realismo del dataset. |
| 44 | Dataset de transacciones reales anonimizadas. |
| 45 | Historial por cliente indispensable. |
| 46 | Reproducción cronológica del dataset. |
| 47 | Parada al acabar dataset y reinicio manual opcional. |
| 48 | Separación cronológica train/simulación. |
| 49 | SHAP solo para alertas. |
| 50 | Errores y métricas en sección de evaluación. |
| 51 | Métricas de evaluación actualizadas en tiempo real. |
| 52 | Métricas visibles para analistas y administradores. |
| 53 | Dashboard compartido entre analistas. |
| 54 | Notificaciones visuales temporales. |
| 55 | Filtros de estado, riesgo, fecha e ID. |
| 56 | Paginación en backend FastAPI/PostgreSQL. |
| 57 | Actualización automática de tabla en cualquier página. |
| 58 | Orden: más recientes primero. |
| 59 | Reconexión automática WebSocket y sincronización. |
| 60 | JWT + refresh tokens. |
| 61 | Revocación inmediata para usuarios desactivados. |
| 62 | Logout de sesión actual o todas las sesiones. |
| 63 | Restablecimiento de contraseñas por administrador. |
| 64 | Bloqueo temporal 15 minutos tras 5 fallos. |
| 65 | Reservas permanecen al cerrar sesión. |
| 66 | Cambio obligatorio de contraseña temporal en primer login. |
| 67 | Cambio de contraseña invalida sesiones anteriores. |
| 68 | OpenAPI/Swagger UI automático. |
| 69 | **No decidido**: scripts SQL o migraciones Alembic. |

---

**Siguiente paso recomendado:** crear el proyecto frontend vacío con una prueba que pase y avanzar con contratos provisionales y fixtures identificados. Validar temprano que existe un dataset compatible antes de cerrar los campos reales y desarrollar su integración; si no lo hay, replantear explícitamente la fuente o el requisito de historial por cliente, sin sustituir silenciosamente el dataset por datos sintéticos.
