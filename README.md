# BiciTierra Market - Predicción de Ventas y Segmentación

## 📌 Objetivo del Proyecto
El objetivo de este proyecto es ayudar a la empresa BiciTierra Market (tienda física y comercio electrónico) a mejorar sus decisiones estratégicas. Para ello, desarrollaremos modelos predictivos para anticipar las ventas mensuales (optimizando la planificación de stock) y técnicas de clustering para segmentar perfiles de clientes y proponer acciones comerciales específicas.

## 👥 Miembros del Equipo
* [Frank Sabater] 
* [David Camarena]
* [Arnau Payá]

## 📂 Estructura del Repositorio
Siguiendo las directrices del proyecto, el repositorio se organiza de la siguiente manera:
* `docs/`: Memoria del proyecto, decisiones justificadas y registro de seguimiento.
* `data/`: Datos originales conservados (intactos) y datos derivados diferenciados.
* `notebooks/`: Cuadernos Jupyter para exploración (EDA), modelado y segmentación.
* `src/`: Scripts y funciones de Python reutilizables.
* `tests/`: Comprobaciones de calidad de datos y código.
* `evidencies/`: Métricas, predicciones exportadas, capturas y archivos del dashboard final.

## ⚙️ Dependencias y Entorno
Para ejecutar este proyecto es necesario contar con:
* Python 3.12+
* Librerías principales: `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`.
* Entorno interactivo: `jupyter notebook`.

## 🚀 Orden de Ejecución
Para reproducir los resultados, los notebooks deben ejecutarse en el siguiente orden:
1. `notebooks/01_eda.ipynb`: Análisis exploratorio y diagnóstico de calidad de datos.
2. `notebooks/02_modelatge_baseline.ipynb`: Preparación de datos, creación de variables temporales, evaluación del baseline y entrenamiento del modelo predictivo.
3. `notebooks/03_segmentacio.ipynb`: Limpieza de datos de clientes y clustering.

## ⚠️ Limitaciones
* Trabajamos con datos sintéticos generados para el caso; ciertas correlaciones matemáticas podrían no implicar causalidad en un entorno de mercado real.
* La predicción ciega para julio de 2026 no dispone de la venta real para validarse.
* Solo se utilizarán predictores conocidos *antes* del inicio del mes a predecir para evitar la fuga de datos (Data Leakage).

---

## 📋 Entrega i Comprovació · CP1 (Setmana 1)

| Apartado | Respuesta del equipo |
| :--- | :--- |
| **Pregunta principal y decisión de negocio** | ¿Podemos predecir cuántas unidades (`sales_units`) de cada línea de producto se venderán en cada región y canal el próximo mes usando solo datos hasta el mes anterior? **Decisión:** Ajustar las compras a proveedores y la planificación del inventario físico. |
| **Pregunta secundaria** | ¿Cómo podemos agrupar o segmentar a nuestros clientes actuales basándonos en su comportamiento de compra (gasto anual, valor medio de pedido, frecuencia y ratio de devoluciones) para diseñar campañas comerciales personalizadas y mejorar su retención? |
| **Variable objetivo y unidad de observación** | **Variable a predecir:** `sales_units`. <br>**Unidad de observación:** Cada fila representa la agrupación de un mes, un canal, una línea de producto y una región. |
| **Predictores iniciales y momento en que se conocen** | **Conocidos a inicio de mes (Útiles):** Histórico de ventas (meses anteriores), stock inicial, línea de producto, canal, región, estacionalidad (mes del año). <br>**Conocidos a fin de mes (Excluidos para predecir):** Ingresos (`revenue`), roturas de stock (`stockout`), stock de cierre, sesiones web. |
| **Problemas de calidad detectados y ejemplos** | Hemos detectado incongruencias en el inventario. <br>**Ejemplo:** Existen casos donde las unidades vendidas (`sales_units`) superan al stock inicial (`stock_units`), o donde el stock de cierre no coincide con la resta del stock inicial menos las ventas. [AÑADIR: Faltan X valores nulos en la columna Y]. |
| **Baseline, modelo alternativo y métrica que proponemos** | **Baseline:** Usar la media histórica de ventas o repetir las ventas del mismo mes del año anterior. <br>**Modelo:** Regresión Lineal Regularizada (Ridge) con Scikit-learn. <br>**Métricas:** MAE (Error Absoluto Medio en unidades) y WAPE (Porcentaje de error ponderado). |
| **Variables y técnica inicial de segmentación** | **Variables:** Gasto anual, frecuencia de compra (pedidos últimos 12 meses), ratio de devoluciones y recencia (días desde último pedido). <br>**Técnica:** Agrupamiento no supervisado mediante algoritmo K-Means. |
| **Hipótesis 1 y Hipótesis 2** | **H1:** La estacionalidad de ventas (picos en primavera/verano) es mucho más pronunciada en el canal físico (retail) que en el online (ecommerce). <br>**H2:** Quedarse sin stock (`stockout`) reduce las ventas del mes en curso, pero genera un repunte o efecto rebote de compras acumuladas en el mes siguiente. |