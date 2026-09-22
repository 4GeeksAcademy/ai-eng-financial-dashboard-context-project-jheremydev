# Estado actual

## Funciona hoy

- La aplicacion tiene un frontend React/TypeScript y un backend FastAPI empaquetados como servicios separados en Docker Compose ([docker-compose.yml](../docker-compose.yml)).
- `App` carga movimientos desde `/api/metrics`, muestra estado de carga, calcula cuatro KPI, calcula series mensuales y muestra un mensaje generico si falla la carga ([frontend/src/App.tsx](../frontend/src/App.tsx)).
- El backend genera un dataset determinista de 360 movimientos ordenados, soporta filtros por fechas, categoria y tipo de operacion, y valida la respuesta mediante Pydantic ([backend/app/routes.py](../backend/app/routes.py), [backend/tests/test_routes.py](../backend/tests/test_routes.py)).
- La API tambien implementa facets, resumen por periodo, categorias principales, comparacion, alertas y endpoints B2B/B2C ([backend/app/routes.py](../backend/app/routes.py)).
- Hay pruebas unitarias de calculos y formateadores del frontend y pruebas de endpoints y filtros del backend ([frontend/src/lib/financial-utils.test.ts](../frontend/src/lib/financial-utils.test.ts), [backend/tests/test_routes.py](../backend/tests/test_routes.py)).

## Gaps y riesgos conocidos

Los siguientes puntos estan respaldados por [engineering-findings.md](../engineering-findings.md) y por las reglas activas en [.agents/rules](../.agents/rules):

- **Contrato de datos en runtime:** `App` pasa el JSON de red directamente a los calculos; las interfaces TypeScript no validan el payload en runtime. La regla `api-source-and-runtime-validation.md` pide validar que la respuesta tenga forma y campos validos antes de calcular.
- **Fechas date-only:** `computeMonthlyData` construye un `Date` desde `YYYY-MM-DD` y luego lee valores locales. `parse-date-only-values.md` identifica el riesgo de desplazamiento de dia o mes por zona horaria.
- **Estado global de random:** `generate_mock_movements` usa `random.seed(seed)` y operaciones del modulo global. `use-local-random-generator.md` senala que esto puede volver el comportamiento dependiente del orden de llamadas cuando exista concurrencia.
- **CORS permisivo:** `main.py` configura `allow_origins=["*"]`, credenciales, metodos y headers abiertos. `restrict-production-cors.md` exige una lista explicita fuera del desarrollo local.
- **Diagnostico de errores:** el `catch` del frontend ignora el error capturado aunque la peticion construye un mensaje con el status. `preserve-error-diagnostics.md` pide conservar ese detalle para desarrollo sin exponerlo necesariamente al usuario.
- **Casos limite sin cobertura:** la regla `handle-empty-facets.md` documenta el contrato necesario para listas vacias; `test-boundary-cases.md` tambien identifica faltantes para fechas sensibles a zona horaria, payloads invalidos y semantica de alertas.
- **Semantica de alertas:** `detect_outcome_alerts` compara cada periodo con el promedio acumulado de periodos anteriores. `preserve-alert-semantics.md` pide mantener y probar explicitamente esa semantica si se modifica.
- **Fuentes mock duplicadas:** `mock-data.ts` contiene un dataset local que no participa en el flujo actual, mientras `App` usa la API. La regla `api-source-and-runtime-validation.md` pide no crear una segunda fuente activa sin una decision explicita.

## Prioridades razonables

Estas prioridades se derivan de los riesgos anteriores, no de funcionalidades futuras declaradas:

1. Validar el contrato de `/api/metrics` en runtime y conservar diagnosticos tecnicos de errores sin cambiar el mensaje seguro mostrado en pantalla.
2. Corregir el tratamiento de fechas sin hora y añadir pruebas para limites de mes, ano y zonas horarias.
3. Hacer determinista y aislado el generador de datos usando una instancia local de `random`, y cubrir el comportamiento de listas vacias en facets.
4. Definir configuracion de CORS por entorno antes de usar el backend fuera del desarrollo local.
5. Completar pruebas de contratos: payloads incompletos, filtros sin resultados y la media acumulada usada por las alertas.

Estas prioridades no implican que exista ya una base de datos, autenticacion, despliegue productivo o integracion contable; el repositorio no aporta evidencia de esas capacidades.
