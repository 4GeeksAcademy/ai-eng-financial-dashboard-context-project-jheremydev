# Engineering Findings & Proposed Rules — Phase 2

## Convenciones detectadas

- **Separación por capas**: frontend separa entrada (`main.tsx`), componentes (`App.tsx`, `dashboard/`), tipos y lógica (`financial-types.ts`, `financial-utils.ts`), UI reutilizable (`card.tsx`). Backend separa app/middleware (`main.py`) de modelos/rutas (`routes.py`).
- **Naming coherente**: archivos en kebab-case (`kpi-card.tsx`), componentes/tipos en PascalCase (`KPICard`, `FinancialMovement`), funciones/variables en camelCase (`computeKPIs`, `fetchFinancialData`).
- **Tipado por uniones literales**: dominio restringe valores válidos (`financial-types.ts:1-3`, `routes.py:7-14`).
- **Componentes con props explícitas y estados de carga** (`kpi-card.tsx:5-12`, `35-49`).
- **Manejo básico de errores de red centralizado** en `App.tsx:24-40`.
- **Validación de parámetros en endpoints** vía `Literal`/`Query` (`routes.py:12-14, 268-277, 287-293, 342-348`).
- **Funciones de dominio puras y testeadas** (`financial-utils.ts:21-81`, `financial-utils.test.ts:25-99`).
- **Cobertura de tests backend** sobre filtros y endpoints (`test_routes.py:29-180`).

---

## Reglas propuestas

### Arquitectura / Contratos de datos

**Regla 1 — No fijar valores derivables de datos como texto estático.**
El encabezado del frontend muestra "2024 - Full Year" como texto fijo, mientras el backend genera fechas relativas al día actual (`App.tsx:53` vs `routes.py:54-59`). Cualquier cambio a la generación de datos debe ir acompañado de la actualización del periodo mostrado, derivándolo dinámicamente en vez de hardcodearlo.

**Regla 2 — No usar `mock-data.ts` como fuente de datos del frontend.**
`mock-data.ts:1-12` contiene un dataset local desconectado del flujo real; los datos provienen de `/api/metrics` (`App.tsx:15-21`). No conectar componentes nuevos a `mock-data.ts` sin verificar primero si sigue siendo necesario.

**Regla 3 — Validar la respuesta de la API en runtime antes de usarla.**
`FinancialMovement` es solo un tipo de compilación (`financial-types.ts:5-12`); no hay validación runtime antes de pasar los datos a `computeKPIs`/`computeMonthlyData` (`App.tsx:31-35`). Si el contrato del backend cambia, el frontend puede mostrar métricas incorrectas sin detectarlo. Añadir validación runtime (ej. con Zod o comprobaciones explícitas) antes de usar la respuesta.

### Manejo de fechas

**Regla 4 — No usar `new Date()` directamente sobre fechas ISO sin hora.**
`financial-utils.ts:43` convierte fechas `YYYY-MM-DD` con `new Date()`, lo que puede desplazar el día en zonas horarias occidentales y agrupar movimientos en el mes equivocado. Usar un parser que fuerce interpretación UTC o extraiga año/mes directamente del string.

### Backend / Estado y concurrencia

**Regla 5 — No usar `random.seed()` global para generar datos deterministas.**
`routes.py:73-75` modifica el estado global de `random`, usado por todos los endpoints vía `generate_mock_movements(seed=42)` (`routes.py:248-259, 268-285`). Si en el futuro hay ejecución concurrente, el resultado dependerá del orden de llamadas. Usar una instancia local (`random.Random(seed)`) en vez del módulo global.

**Regla 6 — Manejar listas vacías en `build_metrics_facets`.**
La función accede directamente al primer y último elemento sin comprobar longitud (`routes.py:145-155`), lo que lanza `IndexError` si la fuente de datos devuelve una lista vacía. Añadir manejo explícito del caso vacío antes de modificar esta función o la fuente de datos.

**Regla 7 — Preservar la semántica acumulativa del cálculo de alertas, o documentar el cambio explícitamente.**
El cálculo actual compara cada periodo contra la media histórica de todos los periodos anteriores, no contra el periodo inmediatamente anterior (`routes.py:219-224`). Cualquier cambio a `threshold` o a esta lógica debe declarar explícitamente si mantiene la semántica acumulativa o la cambia a ventana móvil.

### Seguridad / Configuración

**Regla 8 — No mantener CORS abierto (`allow_origins=["*"]`) fuera de desarrollo local.**
`main.py:7-13` permite todos los orígenes, métodos, headers y credenciales. Antes de cualquier despliegue a producción, restringir los orígenes permitidos explícitamente.

### Testing

**Regla 9 — Cubrir casos límite además del caso feliz.**
Los tests actuales de `build_metrics_facets` (`test_routes.py:85-98`) y de fechas (`financial-utils.test.ts:45-71`) no prueban el caso de dataset vacío ni el cambio de zona horaria. Nuevas funciones que dependan de datos externos deben incluir un test de caso límite (lista vacía, fecha en el borde de zona horaria) antes de mergear.

### DX / Debugging

**Regla 10 — No descartar el detalle técnico del error sin loguearlo.**
El `catch` en `App.tsx:31-36` descarta el error recibido tras construirlo con el status HTTP (`App.tsx:16-20`). Está bien no exponerlo al usuario final, pero debe loguearse (consola o servicio de logging) para no perder capacidad de depuración.

---

## Síntesis

Las convenciones más sólidas son la separación por capas, los tipos literales compartidos, los componentes con props explícitas y las funciones de dominio testeables — estas deben mantenerse en cualquier código nuevo.

Los riesgos principales no son de estilo, son **contratos implícitos**: el año fijo en la UI, la conversión local de fechas ISO, el estado global de `random`, la duplicación de datos mock y la falta de validación runtime del JSON. Un futuro contribuidor o agente de IA puede romper estos contratos sin que el código deje de compilar ni los tests actuales fallen — por eso se convierten en reglas explícitas en la Fase 3.