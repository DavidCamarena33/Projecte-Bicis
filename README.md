# BiciTierra Market

## 🎯 Objetivo
Desarrollar modelos predictivos para anticipar las ventas mensuales (optimizando la planificación de stock y reduciendo roturas) y aplicar técnicas de clustering para segmentar perfiles de clientes, culminando en un dashboard ejecutivo y una defensa de 8 a 10 minutos.

## 👥 Miembros
* [Frank Sabater]
* [David Camarena]
* [Arnau Payá]

## 📂 Estructura del Repositorio
* `README.md`: Documentación principal del proyecto (este archivo).
* `docs/`: Memoria, decisiones y seguimiento (aquí se documentan los checkpoints como CP1, CP2, etc.).
* `data/`: Datos originales conservados e intactos, y datos derivados diferenciados.
* `notebooks/`: Cuadernos Jupyter para exploración (EDA), modelado y segmentación.
* `src/`: Funciones reutilizables de Python.
* `tests/`: Comprobaciones de calidad de datos o código.
* `evidencies/`: Métricas, predicciones, capturas y resultados del panel.

## ⚙️ Dependencias y Entorno
* **Python**: Versión 3.12 o superior.
* **Librerías principales**: `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`.
* **Entorno interactivo**: Jupyter Notebook.

## 🚀 Orden de Ejecución
Para reproducir el análisis de forma correcta, los notebooks deben ejecutarse secuencialmente:
1. `notebooks/01_eda.ipynb` (Análisis exploratorio y diagnóstico de calidad).
2. `notebooks/02_modelatge_baseline.ipynb` (Preparación de datos, partición temporal y baseline).
3. `notebooks/03_segmentacio.ipynb` (Segmentación de clientes y clustering).

## ⚠️ Limitaciones
* Los datos provienen de una simulación sintética; ciertas correlaciones estadísticas pueden no implicar causalidad directa en un mercado real.
* La predicción ciega para julio de 2026 no dispone de la venta real para validarse.
* Para evitar fugas de datos (*Data Leakage*), solo se emplearán como predictores variables estrictamente conocidas antes del inicio del mes a predecir.