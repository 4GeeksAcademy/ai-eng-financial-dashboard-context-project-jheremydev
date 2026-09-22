# Contexto del producto

## Que hace la aplicacion

El repositorio contiene un dashboard de metricas financieras. La pantalla principal muestra un encabezado de resumen, cuatro KPI y dos graficos mensuales:

- `Total Income`, `Total Outcome`, `Profit` y `Profit Margin` se renderizan desde `KPIRow` ([frontend/src/components/dashboard/kpi-row.tsx](../frontend/src/components/dashboard/kpi-row.tsx)).
- `Income vs. Outcome` presenta la evolucion mensual de ingresos y gastos ([frontend/src/components/dashboard/income-outcome-chart.tsx](../frontend/src/components/dashboard/income-outcome-chart.tsx)).
- `Profit Margin %` presenta el margen de beneficio mensual ([frontend/src/components/dashboard/profit-percent-chart.tsx](../frontend/src/components/dashboard/profit-percent-chart.tsx)).

`App` obtiene los movimientos desde `/api/metrics`, calcula los KPI y los datos mensuales con `computeKPIs` y `computeMonthlyData`, y deriva el periodo visible con `formatPeriod` ([frontend/src/App.tsx](../frontend/src/App.tsx), [frontend/src/lib/financial-utils.ts](../frontend/src/lib/financial-utils.ts)).

## Para quien parece estar hecha

El texto visible del producto la orienta a una lectura ejecutiva de resultados financieros: el encabezado dice `Financial Overview` y `Executive metrics dashboard`, y las tarjetas usan etiquetas agregadas como ingresos totales, gastos totales, beneficio y margen ([frontend/src/components/dashboard/dashboard-header.tsx](../frontend/src/components/dashboard/dashboard-header.tsx), [frontend/src/components/dashboard/kpi-row.tsx](../frontend/src/components/dashboard/kpi-row.tsx)). El repositorio no identifica un sector, empresa o rol de usuario mas especifico, por lo que no se afirma uno.

## Problema de negocio que aborda

La aplicacion organiza movimientos de ingresos y egresos para hacer visibles tres preguntas basicas de seguimiento financiero: cuanto ingreso hubo, cuanto se gasto y cual fue el resultado neto y su margen. Esta relacion esta explicitada en el calculo de `computeKPIs` y en los textos auxiliares de las tarjetas ([frontend/src/lib/financial-utils.ts](../frontend/src/lib/financial-utils.ts), [frontend/src/components/dashboard/kpi-row.tsx](../frontend/src/components/dashboard/kpi-row.tsx)). Los graficos agregan esos movimientos por mes para observar su evolucion ([frontend/src/lib/financial-utils.ts](../frontend/src/lib/financial-utils.ts)).

## Limites conocidos del contexto

Los datos actuales no proceden de una base de datos o de una integracion contable: el backend genera 360 movimientos mock deterministas con `seed=42` para los endpoints ([backend/app/routes.py](../backend/app/routes.py)). El frontend de `App` consume directamente `/api/metrics`; aunque el backend define endpoints adicionales de resumen, categorias, comparacion y alertas, no hay evidencia en los componentes actuales de que formen parte de la pantalla principal.
