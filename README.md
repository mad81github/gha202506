
# Tarea Técnica: Implementar filtro HTTP para captura y propagación de contexto del cliente

## Descripción

Se requiere implementar la infraestructura necesaria en el microservicio para capturar información relevante del cliente desde las cabeceras HTTP entrantes, almacenarla temporalmente por petición, y propagar dicha información como cabeceras personalizadas en llamadas HTTP salientes hacia servicios externos específicos (`pmt`, `pmt-payment`).

Esta funcionalidad reemplaza el `ClientContextInterceptor` actualmente usado en Apache CXF.

---

## Objetivos de la tarea

1. Capturar datos del cliente desde las cabeceras HTTP de las peticiones entrantes.
2. Almacenar los datos en un contexto (`ClientContextHolder`) asociado al hilo de ejecución.
3. Propagar los datos como cabeceras personalizadas en las peticiones salientes, únicamente para los clientes `pmt` y `pmt-payment`.

---

## Componentes a desarrollar

### 1. ClientContext (modelo)
- Crear clase inmutable con los siguientes atributos:
  - `clientId`, `requestId`, `clientIp`, `userId`, `appVersion`, `osVersion`, `device`, `amznTraceId`, `salesforceToken`
- Usar patrón Builder para su construcción.

### 2. ClientContextHolder
- Clase `ThreadLocal` que mantiene el contexto del cliente por hilo.
- Métodos: `setContext(ClientContext)`, `getContext()`, `clear()`.

### 3. HttpServerFilter (ClientContextCaptureServerFilter)
- Anotado con `@Filter("/**")`.
- Captura los siguientes headers:
  - `Request-ClientId`, `Request-RequestId`, `X-Forwarded-For`, `X-User-Id`, `X-Request-AppVersion`, `X-Request-OsVersion`, `X-Request-Device`, `X-Amzn-Trace-Id`, `X-Salesforce-Token`
- Construye un `ClientContext` y lo guarda en el `ClientContextHolder`.
- Limpia el contexto al finalizar la petición (`doFinally(ClientContextHolder::clear)`).

### 4. HttpClientFilter (ClientContextPropagationFilter)
- Anotado con `@Filter(serviceIdPattern = "pmt|pmt-payment")`.
- Extrae el `ClientContext` y agrega sus valores como cabeceras salientes:
  - `Request-ClientId`, `Request-RequestId`, `Request-ClientIp`, `X-User-Id`, `X-Request-AppVersion`, `X-Request-OsVersion`, `X-Request-Device`, `X-Amzn-Trace-Id`, `X-Salesforce-Token`

---

## Criterios de Aceptación

- [ ] `ClientContextHolder` almacena los datos por petición sin fuga entre hilos.
- [ ] Los headers relevantes son capturados correctamente desde las peticiones entrantes.
- [ ] Las peticiones salientes a `pmt` y `pmt-payment` contienen todos los headers necesarios.
- [ ] Se incluyen pruebas unitarias para los componentes `ClientContext` y `ClientContextHolder`.
- [ ] Se incluye al menos una prueba de integración donde se verifique que una petición entrante con cabeceras produce una petición saliente con cabeceras propagadas.

---

## Notas de testing / QA

- Usar herramientas como Postman para simular peticiones entrantes con diferentes headers.
- Verificar mediante logs o puntos de ruptura que el contexto se propaga correctamente.
- Si usas OpenTelemetry, confirmar que los atributos están disponibles también como `baggage` si se desea trazabilidad extendida.
