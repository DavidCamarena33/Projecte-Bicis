# BiciTierra Market · CP4
**Setmana 5: Evaluación final en test, análisis de errores, interpretabilidad y predicción ciega**

---

## 1. Registro previo y justificación del modelo elegido
Antes de abrir el conjunto de prueba reservado de 2026 (`2026-01` a `2026-06`), se fijaron por escrito las condiciones del experimento para evitar sesgos o ajustes posteriores sobre el test:

* **Modelo elegido:** `LinearRegression` (Regresión Lineal de `scikit-learn`).
* **Motivo de elección:** En la comparativa de la Semana 3 (`tests/semana3.ipynb`) sobre el año de validación (`2025`), fue el modelo con menor error absoluto medio (**MAE = 13.62**). Es un modelo simple, interpretable y adecuado para un primer proyecto sin necesidad de hiperparámetros complejos.
* **Parámetros:** Valores por defecto (`fit_intercept=True`), sin modificar ni tunear tras ver el test.
* **Variables admitidas (conocidas antes de empezar el mes):** `channel`, `product_line`, `region`, `stock_units`, `marketing_budget`, `planned_discount`, `list_price`, `campaign`, `days_in_month`, `año` y `numero_mes`.
* **Variables excluidas (fuga de datos / cierre de mes):** `sales_units` (objetivo), `revenue`, `closing_stock_units`, `stockout`, `website_sessions`, `marketing_spend` y `avg_discount`.
* **Nota sobre partición temporal:** No se utiliza `train_test_split` aleatorio porque en datos mensuales mezclaría pasado y futuro (*data leakage*). Se respeta la separación estricta en 2 bloques: Entrenamiento con todo el histórico previo (`2022–2025`, 864 filas) y Test reservado (`2026-01 a 2026-06`, 108 filas).

---

## 2. Evaluación única en Test (2026) vs Validación (2025)
Se entrenó el modelo una sola vez con el histórico `2022–2025` y se comparó el resultado en el test reservado de 2026 frente a la validación obtenida en la Semana 3:

| Conjunto | Periodo | MAE (unidades) | RMSE | WAPE (%) | $R^2$ |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **Validación (Semana 3)** | 2025 (216 filas) | 13.62 | 18.31 | 13.12% | 0.85 |
| **Test reservado** | 2026-01 a 06 (108 filas) | **17.47** | **22.03** | **14.65%** | **0.83** |

**Lectura:** El error en test sube unas 3.8 unidades respecto a validación (del 13.12% al 14.65% de error relativo), algo normal al predecir meses nuevos del futuro. Aun así, el modelo lineal mantiene una capacidad explicativa muy alta ($R^2 = 0.83$) sin sobreajustarse.

---

## 3. Análisis de casos de error en 2026
Al revisar las filas donde el modelo lineal cometió mayores desviaciones en 2026, destacan dos casos claros:

1. **Caso 1 (`2026-04` · `ecommerce` · `urban` · `metro`):**
   * **Venta real:** 328 unidades | **Predicción:** 246.5 unidades | **Error:** 81.5 unidades.
   * **Causa probable:** La combinación de la campaña `spring` (primavera), el canal online (`ecommerce`) y la región metropolitana (`metro`) generó un pico de demanda explosivo no lineal que una recta estándar suaviza por debajo.
2. **Caso 2 (`2026-06` · `ecommerce` · `mountain` · `metro`):**
   * **Venta real:** 198 unidades | **Predicción:** 140.2 unidades | **Error:** 57.8 unidades.
   * **Causa probable:** En este mes hubo **rotura de stock (`stockout = 1`)**: se vendieron las 198 bicicletas disponibles (`stock_units = 198`), quedando demanda real sin poder atenderse, además de ser inicio de verano con fuerte demanda en bicis de montaña.

---

## 4. Factores que influyen (Permutación) y límites de interpretación
Para medir qué variables pesan más en el modelo sin contaminar el entrenamiento, se aplicó **Importancia por Permutación** (`permutation_importance`) sobre el conjunto de test de 2026 (`scoring='neg_mean_absolute_error'`):

| Posición | Variable | Aumento del error (MAE) al desordenarla |
| :---: | :--- | :---: |
| 1 | `list_price` (Precio de tarifa) | +64.11 unidades |
| 2 | `product_line_electric` (Línea eléctrica) | +48.06 unidades |
| 3 | `product_line_urban` (Línea urbana) | +29.65 unidades |
| 4 | `stock_units` (Stock inicial) | +24.42 unidades |
| 5 | `product_line_mountain` (Línea montaña) | +6.10 unidades |

**Límites de la interpretación:**  
Las relaciones aprendidas reflejan **asociaciones estadísticas predictivas** dentro de este dataset sintético, **no causalidad directa en el mercado real**. Que el precio o el stock tengan gran peso predictivo no significa que subir artificialmente el stock o modificar el precio genere automáticamente el mismo cambio de ventas en un mercado real sin demanda previa.

---

## 5. Predicción ciega para Julio de 2026 (18 combinaciones)
Se aplicó el modelo definitivo a las **18 combinaciones comerciales** (`2 canales × 3 líneas × 3 regiones`) para el mes `2026-07` y se guardó el resultado en `evidencies/predicciones_julio_2026.csv` con la clave completa (`month`, `channel`, `product_line`, `region`, `prediccion_sales_units`).  

* **Diferencia clave:** Estas 18 cifras son **predicciones estimadas** por el modelo, no ventas reales observadas (todavía no existe cierre de julio de 2026). *(Nota técnica: El notebook carga automáticamente `data/sales_forecast_input.csv` cuando esté disponible en la carpeta de datos, o genera la plantilla con la estructura de junio para validar el flujo).*

---

## 6. Recomendaciones operativas para el negocio
1. **Recomendación de Stock (Confianza alta):**  
   Aumentar el stock de seguridad en las combinaciones de verano con mayor volumen (`ecommerce` + `metro` en líneas `urban` y `mountain`). En junio de 2026 ya se registraron roturas de stock (`stockout = 1`), y la disponibilidad física (`stock_units`) es una de las variables que más limita las ventas atendidas.
2. **Recomendación de Campaña (Confianza media-baja):**  
   Mantener el foco de campañas estacionales en primavera y verano en zonas de alta demanda (`metro` y `coast`), pero verificando primero que exista stock suficiente en almacén antes de incrementar el presupuesto publicitario (`marketing_budget`), ya que su efecto aislado en el modelo es secundario frente a la disponibilidad de producto y el precio.

