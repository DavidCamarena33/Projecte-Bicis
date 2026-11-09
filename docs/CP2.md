# BiciTierra Market · CP2
**Setmana 2: Preparar les dades i establir una referència**

## 1. Notebook amb preparació reproduïble i decisions de qualitat

El análisis se realiza sobre una copia de los datos; los CSV originales se conservan intactos. En [el notebook de trabajo](../tests/probes.ipynb) se ordenan las filas por canal, producto, región y mes, se comprueba la cobertura y se crean las particiones temporales.

Se detectaron estos valores ausentes:

| Campo | Nulos | Decisión |
|---|---:|---|
| `marketing_budget` | 12 | Conservar. No asumir que nulo significa cero; si se usa en un modelo, justificar la imputación y ajustarla solo con entrenamiento. |
| `planned_discount` | 12 | Conservar. No asumir que nulo significa descuento cero; revisar antes de imputar. |
| `marketing_spend` | 18 | Excluir como predictor del mismo mes porque se conoce al finalizarlo. No hace falta imputarlo para estas referencias. |
| `avg_discount` | 24 | Excluir como predictor del mismo mes porque es un dato observado durante o al finalizarlo. |
| `website_sessions` | 18 | Excluir como predictor del mismo mes porque el total de visitas se conoce después de transcurrir el mes. |

El notebook todavía contiene un relleno general de nulos con cero sobre la copia de trabajo. Se considera **provisional y no justificado**; no debe interpretarse como decisión final. Las dos referencias de esta semana no necesitan esas columnas.

## 2. Tabla de particiones, predictores y resultados de las referencias

### Particiones temporales

| Partición | Periodo | Filas | Uso |
|---|---|---:|---|
| Histórico inicial | 2021-01 a 2021-12 | 216 | Proporciona el año previo para los retardos de 12 meses. |
| Entrenamiento | 2022-01 a 2024-12 | 648 | Calcular la media histórica de cada serie. |
| Validación | 2025-01 a 2025-12 | 216 | Comparar las referencias. |
| Prueba reservada | 2026-01 a 2026-06 | 108 | Se guarda para la evaluación final de la semana 5. |

### Predictores según cuándo se conocen

| Decisión | Variables | Motivo |
|---|---|---|
| Admitir | `channel`, `product_line`, `region`, calendario, `marketing_budget`, `planned_discount`, `stock_units`, `list_price` y ventas anteriores de la misma serie. | Se conocen antes del mes, según el documento del proyecto. Los nulos de presupuesto y descuento previsto quedan pendientes de tratamiento justificado. |
| Excluir | `revenue`, `website_sessions`, `marketing_spend`, `avg_discount`, `closing_stock_units` y `stockout` del mismo mes. | Se conocen durante o después del mes que se quiere predecir. |
| Objetivo | `sales_units` del mes que se predice. | Es el valor que queremos estimar; no se usa como predictor de ese mismo mes. |
| Pendiente | `campaign`. | Confirmar en el diccionario si se conoce antes del mes o si describe una campaña ya ejecutada. |

### Referencias medidas en validación

Se compararon la media de ventas 2022–2024 de cada combinación de canal, producto y región, y las ventas del mismo mes de 2024. Ambas se evaluaron sobre las mismas 216 filas de 2025.

| Referencia | MAE (unidades) | WAPE |
|---|---:|---:|
| Media histórica por serie | 19,92 | 19,19 % |
| Mismo mes del año anterior | 18,94 | 18,25 % |

En esta validación, la referencia del mismo mes del año anterior obtiene un error menor: aproximadamente 0,98 unidades menos de MAE y 0,94 puntos porcentuales menos de WAPE. El notebook ya calcula el MAE; el WAPE de esta tabla se calculó con las mismas filas y queda pendiente incorporarlo al notebook para que todo el resultado sea reproducible allí. No se ha evaluado la partición de prueba.

## 3. Un ejemplo de cómo una variable posterior puede falsear el resultado

Imaginemos que el 31 de diciembre queremos predecir las ventas de enero. El `revenue` y las `website_sessions` de enero todavía no se conocen: solo estarán disponibles cuando enero haya transcurrido. Si los usamos para predecir las ventas de ese mismo enero, el cálculo utiliza información del futuro y el error parecerá mejor de lo que sería en una predicción real. Por eso se excluyen como predictores del mismo mes.
