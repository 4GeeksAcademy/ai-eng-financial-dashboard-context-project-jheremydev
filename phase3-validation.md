# Phase 3 — Rule Validation

## Prueba 1: no-hardcoded-period.md (no concluyente)
Se pidió corregir el periodo fijo "2024 - Full Year" en el mismo chat donde se había analizado el problema. El agente confirmó explícitamente que no consultó `.agents/rules/no-hardcoded-period.md` antes de resolverlo — llegó a la solución por el contexto previo de la conversación. La solución implementada es correcta (deriva el periodo de los datos de la API), pero no sirve como prueba de que la regla dirige el trabajo.

## Prueba 2: handle-empty-facets.md (válida)
En una sesión nueva, sin contexto previo, se pidió: "Agrega manejo explícito para cuando /api/metrics/facets reciba una lista vacía de movimientos". El log de ejecución muestra que el agente leyó `.agents/rules/` y específicamente `handle-empty-facets.md` antes de programar. La solución implementada (min_date/max_date anulables) sigue exactamente la guía accionable de la regla. Esta prueba confirma que el sistema de reglas, junto con AGENTS.md, dirige correctamente el trabajo del agente.

## Conclusión
Las reglas en `.agents/rules/` son efectivas cuando el agente parte sin contexto previo en el chat, gracias a que `AGENTS.md` le indica explícitamente revisar esa carpeta antes de actuar.