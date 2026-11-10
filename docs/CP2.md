# BiciTierra Market · CP2
**Setmana 2: Preparar les dades i establir una referència**

## 1. Notebook amb preparació reproduïble i decisions de qualitat

El análisis se realiza sobre una copia de los datos; los CSV originales se conservan intactos. El trabajo propio está organizado en dos notebooks: [Semana 1](../tests/semana1.ipynb) y [Semana 2](../tests/semana2.ipynb). Los notebooks de `notebooks/` son materiales guía proporcionados para el proyecto.

Se detectaron estos valores ausentes:

| Campo | Nulos | Decisión |
|---|---:|---|
| `marketing_budget` | 12 | Conservar. No asumir que nulo significa cero; si se usa en un modelo, justificar la imputación y ajustarla solo con entrenamiento. |
| `planned_discount` | 12 | Conservar. No asumir que nulo significa descuento cero; revisar antes de imputar. |
| `marketing_spend` | 18 | Excluir como predictor del mismo mes porque se conoce al finalizarlo. No hace falta imputarlo para estas referencias. |
| `avg_discount` | 24 | Excluir como predictor del mismo mes porque es un dato observado durante o al finalizarlo. |
| `website_sessions` | 18 | Excluir como predictor del mismo mes porque el total de visitas se conoce después de transcurrir el mes. |

Los notebooks propios todavía contienen un relleno general de nulos con cero sobre la copia de trabajo. Se considera **provisional y no justificado**; no debe interpretarse como decisión final. Las dos referencias de esta semana no necesitan esas columnas.

## 2. Particiones, predictores y resultados de las referencias

El enunciado junta tres evidencias en un mismo punto de entrega, pero cumplen funciones distintas: la **tabla de particiones** indica qué meses se usan para entrenar, validar y probar; la **lista de predictores** indica qué columnas se pueden conocer al hacer la predicción. No son la misma tabla ni una se calcula a partir de la otra. Se documentan aquí juntas porque el CP2 pide ambas.

### Particiones temporales

| Partición | Periodo | Filas | Uso |
|---|---|---:|---|
| Histórico inicial | 2021-01 a 2021-12 | 216 | Proporciona el año previo para los retardos de 12 meses. |
| Entrenamiento | 2022-01 a 2024-12 | 648 | Calcular la media histórica de cada serie. |
| Validación | 2025-01 a 2025-12 | 216 | Comparar las referencias. |
| Prueba reservada | 2026-01 a 2026-06 | 108 | Se guarda para la evaluación final de la semana 5. |

### Lista de variables admitidas y excluidas

Esta tabla responde a otra pregunta: **¿estaría disponible esta variable antes de empezar el mes cuyas ventas queremos anticipar?** Las decisiones temporales siguen las reglas del caso y deben aplicarse igual en entrenamiento y validación.

| Variable | Decisión | Cuándo se conoce y motivo |
|---|---|---|
| `month`, `channel`, `product_line`, `region`, `days_in_month` | Admitir | Calendario y características de la serie conocidos antes del mes. |
| `stock_units`, `marketing_budget`, `planned_discount`, `list_price` | Admitir, con nulos pendientes de resolver donde corresponda | El diccionario define estos datos como disponibles antes del mes. No se debe asumir que los nulos de presupuesto o descuento previsto significan cero. |
| Ventas anteriores de la misma serie | Admitir cuando ya estén observadas | Son datos históricos. Para la referencia anual, se usa el mismo mes del año anterior. |
| `sales_units` del mes que se predice | Objetivo, no predictor | Es el resultado que queremos estimar; usarlo como entrada sería dar al cálculo la respuesta. |
| `revenue` del mismo mes | Excluir | Se conoce al cerrar el mes y está ligado a la actividad de ventas que queremos anticipar. |
| `website_sessions` del mismo mes | Excluir | El total se acumula durante el mes y no está disponible antes de que empiece. El CSV también contiene valores para filas `store`, así que no justificamos la exclusión diciendo que solo exista para ecommerce. |
| `marketing_spend`, `avg_discount`, `closing_stock_units`, `stockout` del mismo mes | Excluir | Son valores observados durante o al finalizar el mes. |
| `campaign` | Admitir | El diccionario la define como un plan simulado conocido antes del mes. |

### Referencias medidas en validación

Se compararon la media de ventas 2022–2024 de cada combinación de canal, producto y región, y las ventas del mismo mes de 2024. Ambas se evaluaron sobre las mismas 216 filas de 2025.

| Referencia | MAE (unidades) | WAPE |
|---|---:|---:|
| Media histórica por serie | 19,92 | 19,19 % |
| Mismo mes del año anterior | 18,94 | 18,25 % |

En esta validación, la referencia del mismo mes del año anterior obtiene un error menor: aproximadamente 0,98 unidades menos de MAE y 0,94 puntos porcentuales menos de WAPE. Las métricas se calculan en el notebook de semana 2 sobre las mismas filas de validación. No se ha evaluado la partición de prueba.

## 3. Un ejemplo de cómo una variable posterior puede falsear el resultado

Imaginemos que el 31 de diciembre queremos predecir las ventas de enero. El `revenue` y las `website_sessions` de enero todavía no se conocen: solo estarán disponibles cuando enero haya transcurrido. Si los usamos para predecir las ventas de ese mismo enero, el cálculo utiliza información del futuro y el error parecerá mejor de lo que sería en una predicción real. Por eso se excluyen como predictores del mismo mes.
