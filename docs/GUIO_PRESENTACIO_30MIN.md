# Repartiment i Guió de Defensa · BiciTierra Market (30 min)

## Recomanació de Divisió (sense canvis de torn incòmodes)
Per a parlar d'una tirada i sense haver de passar-se el micro dues voltes:

* **Company 1 (00:00 – 10:00 | ~10 min):** Diapositives **1, 2, 3 i 4** *(Portada, Índex, Context/Dades i EDA/Qualitat)*
* **Company 2 (10:00 – 20:00 | ~10 min):** Diapositives **5, 6 i 7** *(Comparativa 15 Models, Prova Cega 2026/Errors i Permutació/Juliol 2026)*
* **Tu — Arnau (20:00 – 30:00 | ~10 min):** Diapositives **8, 9, 10 i 11** *(Segmentació K-Means, Power BI, Autocrítica i Recomanacions/Tancament)*

---

## GUIÓ COMPANY 1 (Minuts 00:00 – 10:00)

### Diapositiva 1 · Portada (1,5 minuts)
«Bon dia. Som Frank Sabater, David Camarena i Arnau Payá, i anem a presentar el projecte **BiciTierra Market**, desenvolupat a parts iguals entre els tres.  
BiciTierra és una distribuïdora de bicicletes amb botiga física i venda online que tenia tres grans objectius: anticipar les vendes mensuals per a no quedar-se sense estoc a l'estiu, entendre quins perfils de client té i transferir tot açò a un quadre de comandament en Power BI amb recomanacions reals i honestes.»

### Diapositiva 2 · Índex (1 minut)
«Ací teniu el full de ruta de l'exposició en 9 apartats: jo us explicaré les dades inicials, la neteja i com vam protegir el flux contra trampes; després el meu company passarà a la comparativa dels 15 models i la prova cega de 2026; i finalment Arnau tancarà amb la segmentació de clients, el Power BI, els errors comesos i les recomanacions per a direcció.»  
*(Fes clic en **Apartat 01** per a saltar directament).*

### Diapositiva 3 · Context i Dades (3 minuts)
«Les tres preguntes estratègiques que ens planteja la direcció són clares: podem predir les vendes mensuals per a evitar ruptures d'estoc?, quins grups de clients tenim? i quins límits tenen les nostres dades?  
Per a respondre-les, treballem amb dos fitxers sintètics completament independents:
1. **`sales_history.csv`** amb **1.188 registres mensuals** des de gener de 2021 fins a juny de 2026. Són 18 sèries comercials (2 canals, 3 línies de bicicleta i 3 regions), on l'objectiu és predir **`sales_units`** (unitats venudes al mes).
2. **`customers.csv`** amb una fotografia fixa de **2.000 clients** a 30 de juny de 2026, amb la seua despesa anual, comandes, dies sense comprar i devolucions.  
Un punt fonamental, tal com ens va recalcar el professorat: **no hi ha cap relació entre els dos CSVs**. Són dades independents i mai les hem creuat ni unit.»

### Diapositiva 4 · Exploració, Qualitat i Control de Fuga (4,5 minuts)
*(Fes clic en la **Figura 1** per ampliar-la uns segons i assenyalar els pics d'estiu).*  
«En l'anàlisi exploratòria vam veure una estacionalitat molt marcada: les vendes pugen fort entre maig i juliol i baixen a l'hivern, sent el canal web i la zona metropolitana (*metro*) els que lideren el volum.  
En qualitat de dades vam aplicar la regla **"Nul no és zero"**:
* En clients, teníem 30 nuls de despesa anual (`annual_spend`), però eixos clients tenien fins a 10 comandes; posar 0 € hauria sigut un error greu. Els vam reconstruir multiplicant tiquet mitjà per comandes. I els 20 nuls de recència els vam omplir amb la **mediana (44 dies)**.
* En vendes, els 12 mesos sense pressupost ni descompte previst corresponen a mesos vall sense campanya.  
Finalment, vam aplicar un control estricte de fuga de dades: per a predir les vendes el dia 1 de cada mes, **només admetem el que se sap abans de començar el mes** (estoc inicial, preu tarifa, pressupost, descompte previst, campanya, any i número de mes). Excloem tot el que se sap en tancar el mes (ingressos, visites web o ruptura d'estoc), i separem per dates cronològiques sense usar `train_test_split` aleatori. Li passe el torn al meu company.»

---

## GUIÓ COMPANY 2 (Minuts 10:00 – 20:00)

### Diapositiva 5 · Comparativa de 15 Models (3,5 minuts)
*(Fes clic en la **Figura 2** per ampliar el rànquing).*  
«Gràcies. En la Setmana 2 vam fixar dues referències bàsiques (*baselines*) sobre l'any 2025: la mitjana històrica (`19,92` unitats d'error) i repetir el mateix mes de l'any anterior (`18,94` unitats).  
En la Setmana 3 vam entrenar **15 models diferents** de `scikit-learn` amb dades de 2022–2024 i els vam avaluar tots amb les mateixes regles sobre 2025. Com es veu en la gràfica, el guanyador absolut va ser **`LinearRegression`**, amb un error percentual (**WAPE**) del **`13,12%`**, un **MAE de `13,62` unitats** i un **$R^2$ de `0,8472`**, reduint un **28,1%** l'error del millor baseline.  
El triem pel principi de senzillesa: millora a models complexos com *GradientBoosting* o *RandomForest*, no memoritza soroll i ens permet auditar fàcilment els seus coeficients.»

### Diapositiva 6 · Prova Cega en Test 2026 i Anàlisi d'Errors (3,5 minuts)
«Una vegada decidit el model sense mirar 2026, en la Setmana 5 el vam reentrenar amb tot l'històric fins a desembre de 2025 (864 files) i vam avaluar **una única vegada** el test reservat de gener–juny de 2026 (108 files).  
Els resultats confirmen que el model és molt sòlid: un error percentual (**WAPE**) del **`14,65%`** (més del 85% de fiabilitat), un **MAE de `17,47` unitats** i un **$R^2$ de `0,83`**, sense sobreajustament.  
A més, vam revisar els **2 majors errors** comesos en 2026:
1. **Abril 2026 (`ecommerce · urban · metro`):** Venda real 328 u. vs predicció 246,5 u. (`-81,5 u.`). La combinació de campanya de primavera, web i àrea metro dispara la demanda de forma no lineal, i una recta ho suavitza per davall.
2. **Juny 2026 (`ecommerce · mountain · metro`):** Venda real 198 u. vs predicció 140,2 u. (`-57,8 u.`). Ací hi va haver una **ruptura física d'estoc real**: es van vendre totes les 198 unitats disponibles, censurant la demanda real que no es va poder atendre.»

### Diapositiva 7 · Factors que Manen (Permutació) i Juliol 2026 (3 minuts)
*(Fes clic en la **Figura 3** per ampliar la gràfica).*  
«Per a explicar què variables pesen més en el model, vam aplicar **Importància per Permutació** sobre el test de 2026, desordenant cada variable per a mesurar quant puja l'error.  
El factor més determinant és el **preu de tarifa (`list_price`, +64,11 u.)**, seguit de les línies **elèctrica (`+48,06 u.`)** i **urbana (`+29,65 u.`)**, i l'**estoc inicial (`stock_units`, +24,42 u.)**.  
Finalment, vam aplicar el model a les **18 combinacions de juliol de 2026**, guardant el fitxer `predicciones_julio_2026.csv`. Cal recordar que són **estimacions sense venda real encara**, i que mostren associació predictiva en dades simulades, no causalitat directa en el mercat real. Ara passo la paraula a Arnau per a explicar la segmentació de clients, el Power BI i les conclusions finals.»

---

## BLOC D'ARNAU (Minuts 20:00 – 30:00 · Sense guió, només referència de temps)
* **Diapositiva 8 · Segmentació K-Means (`k=4`) (~3 min):** Mostrar Figura 4 ampliada, explicar els 4 perfils mitjans (`Estàndard 40%`, `VIP 32,2%`, `Risc 15,9%`, `Inactiu 11,9%`), accions + controls previs, escalat i casos fronterers (`226` i `258`).
* **Diapositiva 9 · Quadre de Comandament Power BI (~3,5 min):** Mostrar les 2 vistes ampliades (o Power BI en directe filtrant anys/canals).
* **Diapositiva 10 · Autocrítica: Errors i Solucions (~1,5 min):** Repassar els 4 errors reals del projecte i com es van corregir.
* **Diapositiva 11 · Recomanacions, Límits i Millores Futures (~2 min):** +25% estoc d'estiu, bloqueig de campanyes sense estoc, cartera VIP, Docker i tests A/B de màrqueting. Tancament i preguntes.
