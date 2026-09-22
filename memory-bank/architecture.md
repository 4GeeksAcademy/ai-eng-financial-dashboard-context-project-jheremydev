# Arquitectura

## Stack

- **Frontend:** TypeScript, React 19 y React DOM. Vite gestiona el desarrollo y el build; Tailwind CSS se integra mediante `@tailwindcss/vite` ([frontend/package.json](../frontend/package.json), [frontend/vite.config.ts](../frontend/vite.config.ts)).
- **Visualizacion y UI:** Recharts para los graficos, `lucide-react` para iconos y utilidades `clsx`/`tailwind-merge` para clases ([frontend/package.json](../frontend/package.json)).
- **Pruebas frontend:** Vitest y pruebas unitarias de las funciones financieras ([frontend/package.json](../frontend/package.json), [frontend/src/lib/financial-utils.test.ts](../frontend/src/lib/financial-utils.test.ts)).
- **Backend:** Python 3.13, FastAPI, Pydantic y Uvicorn. `pytest`, `pytest-cov` y `httpx` soportan las pruebas del backend ([backend/Dockerfile](../backend/Dockerfile), [backend/requirements.txt](../backend/requirements.txt), [backend/app/routes.py](../backend/app/routes.py)).
- **Infraestructura local:** Docker Compose levanta dos servicios, `frontend` en el puerto 5173 y `backend` en el 8000 ([docker-compose.yml](../docker-compose.yml)). El backend tambien expone el puerto 5678 para `debugpy` ([backend/Dockerfile](../backend/Dockerfile)).

## Separacion de responsabilidades

El frontend separa la entrada de la aplicacion (`App.tsx`), los componentes del dashboard (`components/dashboard`), los tipos (`financial-types.ts`) y la logica pura de calculo y formato (`financial-utils.ts`). Los tipos de dominio restringen `operation_type`, `category` y `business_type` mediante uniones literales ([frontend/src/lib/financial-types.ts](../frontend/src/lib/financial-types.ts)).

El backend separa la configuracion de FastAPI y middleware en `main.py` de los modelos Pydantic, generacion de datos, filtros, calculos y rutas en `routes.py` ([backend/app/main.py](../backend/app/main.py), [backend/app/routes.py](../backend/app/routes.py)).

## Flujo frontend-backend

1. `App` llama a `${VITE_API_BASE_URL}/api/metrics`; si no se define la variable, usa una URL relativa ([frontend/src/App.tsx](../frontend/src/App.tsx)).
2. En desarrollo, Vite redirige las rutas `/api` a `http://backend:8000` mediante su proxy ([frontend/vite.config.ts](../frontend/vite.config.ts)).
3. FastAPI atiende `/api/metrics`, genera movimientos con `generate_mock_movements(seed=42)`, aplica filtros opcionales y devuelve una lista validada por el modelo Pydantic `FinancialMovement` ([backend/app/routes.py](../backend/app/routes.py)).
4. El frontend transforma la lista en KPI, datos mensuales y periodo, y los pasa a las tarjetas y graficos ([frontend/src/App.tsx](../frontend/src/App.tsx), [frontend/src/lib/financial-utils.ts](../frontend/src/lib/financial-utils.ts)).

## Contratos y endpoints del backend

El backend define, ademas de `/api/metrics`, endpoints para facets, resumen por dia/semana/mes, categorias principales, comparacion, alertas y subconjuntos B2B/B2C ([backend/app/routes.py](../backend/app/routes.py)). Sus parametros usan `Literal` y restricciones de `Query` para varios valores aceptados. La existencia de estas rutas no implica que todas esten conectadas a la UI actual.

## Ejecucion local

El README prescribe `docker compose up --build`. El frontend se sirve en `http://localhost:5173`, el backend en `http://localhost:8000` y la documentacion de FastAPI en `http://localhost:8000/docs` ([README.md](../README.md)).
