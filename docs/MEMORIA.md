# Projecte d'IA i Big Data · Curs 2026–2027
# Memòria del Projecte: BiciTierra Market
### Optimització predictiva de la demanda i segmentació conductual de clients per a la presa de decisions estratègiques

**Autors:**
* Frank Sabater
* David Camarena
* Arnau Payá

**Centre:** IES Jaume II el Just (Tavernes de la Valldigna)  
**Mòdul:** Curs d'Especialització en Intel·ligència Artificial i Big Data  
**Data:** Novembre 2026  
**Repositori:** `https://github.com/fransabater/Projecte-Bicis`

---

## Índex
1. **Resum Executiu i Formulació del Repte**
2. **Dades, Qualitat i Disseny de la Partició Temporal**
3. **Modelatge Predictiu i Validació de Vendes**
4. **Segmentació Conductual de Clients (K-Means)**
5. **Traducció a Negoci i Arquitectura del Panell Executiu (Power BI)**
6. **Límits del Sistema, Riscos i Línies Futures**
7. **Organització de l'Equip i Autoavaluació**
8. **Bibliografia i Referències Tècniques**

---

## 1. Resum Executiu i Formulació del Repte

### 1.1 Context empresarial
BiciTierra Market és una empresa comercial dedicada a la distribució i venda de bicicletes i accessoris ciclistes que opera mitjançant un model híbrid de venda minorista: botigues físiques (*retail*) i comerç electrònic (*ecommerce*). La companyia gestiona tres famílies de productes (bicicletes urbanes, elèctriques i de muntanya) repartides en tres grans regions geogràfiques (*central*, *coast* i *metro*).

El creixement continu de la demanda i la diversitat de canals han generat tensions en la cadena de subministrament. La direcció de l'empresa planteja tres preguntes estratègiques essencials:
1. **Anticipació de la demanda:** Podem predir les vendes mensuals amb prou exactitud per a planificar les compres a proveïdors i evitar tant les ruptures d'estoc (*stockouts*) com el cost de sobreemmagatzematge?
2. **Coneixement del client:** Quins perfils de client tenim en base al seu comportament real de compra i quines accions comercials diferenciades tenen sentit per a cadascun?
3. **Criteri de decisió:** Quines decisions operatives podem justificar a partir dels resultats de la intel·ligència artificial i quins límits hem d'assumir amb aquestes dades?

### 1.2 Objectius del projecte
* **Objectiu principal:** Desenvolupar i validar un flux complet d'aprenentatge automàtic capaç d'anticipar la variable objectiu `sales_units` (unitats venudes al mes per canal, línia i regió), segmentar la base de clients en perfils accionables i consolidar els resultats en un quadre de comandament executiu que permeta decisions informades.
* **Unitat d'observació:** Cada registre representa l'activitat mensual agregada d'una combinació única de canal, línia de producte i regió geogràfica (18 sèries temporals simultànies).

---

## 2. Dades, Qualitat i Disseny de la Partició Temporal

### 2.1 Naturalesa de les dades i integritat referencial
El projecte es fonamenta en dos conjunts de dades sintètics elaborats per a l'entorn docent:
1. `sales_history.csv`: 1.188 files que cobreixen el període comprés entre gener de 2021 i juny de 2026 per a les 18 combinacions de mercat.
2. `customers.csv`: 2.000 clients registrats amb dades agregades a data 30 de juny de 2026.

> **Regla de desacoblament:** Ambdós conjunts són independents. No existeix una clau forana transaccional que permeta vincular clients individuals amb les files mensuals de vendes. La despesa dels clients no suma exactament els ingressos de `sales_history`.

### 2.2 Blindatge metodològic contra la fuga de dades (*Data Leakage*)
Un dels errors crítics en la modelització predictiva comercial és utilitzar informació que només es coneix a posteriori. Per a garantir una avaluació realista, les variables s'han segregat estrictament en funció del moment en què estan disponibles:

| Variable | Moment de disponibilitat | Decisió metodològica | Motiu tècnic |
| :--- | :---: | :---: | :--- |
| `month`, `channel`, `product_line`, `region` | Abans de començar el mes | **Admesa** | Metadades del calendari i de la sèrie. |
| `stock_units` | Abans de començar el mes | **Admesa** | Estoc físic inicial disponible al magatzem. |
| `list_price` | Abans de començar el mes | **Admesa** | Preu de tarifa oficial catalogat. |
| `marketing_budget` | Abans de començar el mes | **Admesa** | Pressupost assignat prèviament a màrqueting. |
| `planned_discount` | Abans de començar el mes | **Admesa** | Descompte comercial programat per campanya. |
| `campaign` | Abans de començar el mes | **Admesa** | Planificació de campanya estacional coneguda. |
| Retards històrics (`lag_12`) | Abans de començar el mes | **Admesa** | Vendes passades completament consolidades. |
| `sales_units` (mes actual) | Al finalitzar el mes | **Variable objectiu** | Target a predir; utilitzar-lo seria donar la resposta. |
| `revenue` | Al finalitzar el mes | **Exclosa** | Es calcula sobre les vendes ja tancades. |
| `marketing_spend` | Al finalitzar el mes | **Exclosa** | Despesa real liquidada després de la campanya. |
| `avg_discount` | Al finalitzar el mes | **Exclosa** | Descompte real ponderat aplicat en les vendes. |
| `website_sessions` | Al finalitzar el mes | **Exclosa** | Les visites web s'acumulen al llarg dels 30 dies. |
| `closing_stock_units`, `stockout` | Al finalitzar el mes | **Exclosa** | Es deriven de l'estoc final i de la demanda atesa. |

### 2.3 Diagnòstic de qualitat i tractament justificat de nuls
L'exploració inicial va detectar valors absents en diverses columnes del fitxer de vendes: `avg_discount` (24 nuls), `marketing_spend` (18 nuls), `website_sessions` (18 nuls), `marketing_budget` (12 nuls) i `planned_discount` (12 nuls).
* **Decisió:** Les tres primeres variables van ser descartades com a predictores per motius de separació temporal, de manera que no requereixen imputació per al modelatge. Per a `marketing_budget` i `planned_discount`, els nuls coincideixen amb períodes vall sense campanya comercial activa. No es va aplicar cap mitjana artificial per a no distorsionar les sèries; es van conservar i tractar únicament amb informació d'entrenament.

### 2.4 Disseny de la partició temporal estricta
Les dades de sèries temporals mai s'han de dividir amb particions aleatòries (`train_test_split` clàssic), ja que trencaria l'ordre cronològic i permetria al model aprendre de patrons del futur. Es va dissenyar una partició en blocs temporals tancats:

| Partició | Període cronològic | Files | Propòsit tècnic |
| :--- | :---: | :---: | :--- |
| **Històric inicial** | 2021-01 a 2021-12 | 216 | Generació de retards estacionals de 12 mesos (`lag_12`). |
| **Entrenament (Train)** | 2022-01 a 2024-12 | 648 | Ajust dels models, escalats i transformacions. |
| **Validació (Val)** | 2025-01 a 2025-12 | 216 | Comparació justa de models i selecció del candidat. |
| **Prova reservada (Test)** | 2026-01 a 2026-06 | 108 | Avaluació final cega i verificació de robustesa. |
| **Previsió cega** | 2026-07 | 18 | Inferència operativa per a la planificació d'estoc. |

![Evolució mensual de vendes per canal](01_estacionalitat_vendes.png)
*Figura 1: Evolució temporal de vendes (2021–2026) per canal, mostrant l'acusada estacionalitat estival i les particions del projecte.*

---

## 3. Modelatge Predictiu i Validació de Vendes

### 3.1 Baselines de referència
Abans de proposar cap model d'intel·ligència artificial, es van mesurar dues referències heurístiques senzilles sobre les 216 observacions del conjunt de validació (any 2025):
1. **Mitjana històrica per sèrie:** Promig de vendes 2022–2024 de cada combinació de canal, producte i regió.
   * `MAE = 19,92 unitats` | `WAPE = 19,19%`
2. **Estacionalitat anual (mateix mes de l'any anterior):** Valor observat en el mateix mes de l'any 2024 per a la mateixa sèrie.
   * `MAE = 18,94 unitats` | `WAPE = 18,25%`

Aquestes xifres marquen el llindar mínim: qualsevol model de machine learning ha de millorar clarament les 18,94 unitats d'error per a justificar la seua complexitat.

### 3.2 Comparativa de models en validació (2025)
Es van avaluar 15 arquitectures d'aprenentatge automàtic emprant exactament el mateix pipeline de dades sobre les dades de validació de 2025:

| Model | MAE (unitats) | RMSE | WAPE (%) | $R^2$ |
| :--- | :---: | :---: | :---: | :---: |
| **LinearRegression** | **13,62** | **18,31** | **13,12%** | **0,8472** |
| KernelRidge | 13,97 | 18,36 | 13,46% | 0,8464 |
| SGDRegressor | 14,12 | 18,56 | 13,60% | 0,8430 |
| Ridge | 14,27 | 18,63 | 13,75% | 0,8417 |
| GradientBoosting | 14,34 | 19,71 | 13,81% | 0,8228 |
| Lasso | 14,81 | 19,50 | 14,27% | 0,8266 |
| RandomForest | 14,96 | 20,41 | 14,41% | 0,8102 |
| PLSRegression | 15,04 | 19,41 | 14,49% | 0,8283 |
| AdaBoost | 15,18 | 19,99 | 14,62% | 0,8179 |
| Bagging | 15,99 | 21,78 | 15,40% | 0,7839 |
| *Baseline mateix mes any anterior* | *18,94* | *24,12* | *18,25%* | *0,7350* |
| *Baseline mitjana històrica* | *19,92* | *25,80* | *19,19%* | *0,6980* |
| DecisionTree | 21,13 | 27,64 | 20,35% | 0,6517 |
| KNN Regressor | 23,12 | 31,37 | 22,27% | 0,5515 |
| SVR (RBF Kernel) | 32,80 | 43,96 | 31,60% | 0,1192 |

![Comparativa de models en validació](02_comparativa_models.png)
*Figura 2: Comparativa de l'error MAE en el conjunt de validació 2025. El model lineal supera a la resta d'algorismes i redueix l'error del baseline en més d'un 28%.*

### 3.3 Justificació del model elegit
L'equip va seleccionar unànimement **LinearRegression**. Els motius tècnics i de negoci són sòlids:
* **Menor taxa d'error:** Assoleix el millor equilibri tant en MAE (13,62 unitats) com en RMSE (18,31) i coeficient de determinació ($R^2 = 0,85$), reduint en un **28,1%** l'error del millor baseline.
* **Principi de parsimònia (Navalla d'Ockham):** Davant de resultats pràcticament idèntics a KernelRidge o GradientBoosting, el model lineal ordinari no requereix optimització d'hiperparàmetres complexos, s'executa en mil·lisegons i resulta completament transparent i explicable per a la direcció financera.

### 3.4 Prova final en el conjunt reservat (Test 2026)
Una vegada congelat el model i els seus paràmetres, es va obrir el bloc reservat de prova (gener a juny de 2026, 108 files) una única vegada:

| Conjunt | Període | MAE (unitats) | RMSE | WAPE (%) | $R^2$ |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Validació (2025)** | 12 mesos (216 files) | 13,62 | 18,31 | 13,12% | 0,85 |
| **Test reservat (2026)** | 6 mesos (108 files) | **17,47** | **22,03** | **14,65%** | **0,83** |

L'augment de 3,8 unitats en el MAE (de 13,62 a 17,47) reflecteix el comportament habitual en predir el futur pròxim sense reajust continu. L'indicador $R^2 = 0,83$ confirma que la capacitat explicativa del sistema es manté sòlida i no hi ha sobreajustament.

### 3.5 Anàlisi detallat de casos d'error en 2026
L'auditoria de residus va identificar dues desviacions principals:
1. **Abril 2026 (`ecommerce` · `urban` · `metro`):**
   * Venda real: 328 unitats | Predicció: 246,5 unitats (Error: -81,5 u.).
   * *Diagnòstic:* La concurrència de la campanya comercial de primavera (*spring*), el canal online i l'àrea metropolitana va produir un repunt exponencial de demanda no lineal que una recta suavitza per davall.
2. **Juny 2026 (`ecommerce` · `mountain` · `metro`):**
   * Venda real: 198 unitats | Predicció: 140,2 unitats (Error: -57,8 u.).
   * *Diagnòstic:* Es va registrar ruptura d'estoc física (`stockout = 1`). Es van esgotar exactament les 198 unitats d'estoc inicial (`stock_units = 198`), quedant demanda latent sense poder atendre's.

### 3.6 Importància de variables per permutació
Mitjançant la tècnica de *Permutation Feature Importance* sobre el test de 2026, es va quantificar quin impacte té desordenar aleatòriament cada variable sobre l'error del model:

![Importància per permutació](04_importancia_permutacio.png)
*Figura 3: Importància per permutació de les variables sobre el conjunt de test 2026.*

* **Variables més determinants:** El preu de tarifa (`list_price`), la pertinença a la línia elèctrica (`product_line_electric`) i l'estoc inicial disponible (`stock_units`).
* **Límits de la interpretació:** Les relacions reflecteixen **associacions predictives en dades simulades, mai causalitat comercial**. Incrementar l'estoc al magatzem de forma indiscriminada no augmentarà les vendes si no hi ha una demanda real que l'acompanye.

### 3.7 Prediccions cegues per a Juliol de 2026
El model final es va aplicar a les 18 combinacions de juliol de 2026 i es va exportar a `evidencies/predicciones_julio_2026.csv`. Aquestes xifres són previsions operatives sense dades reals observades que nodreixen directament el quadre de comandament executiu.

---

## 4. Segmentació Conductual de Clients (K-Means)

### 4.1 Preparació i depuració de dades de clients
La taula `customers.csv` recull 2.000 registres. La neteja es va orientar a conservar la coherència comercial:
* **Nuls en despesa anual (`annual_spend`):** 30 registres presentaven valor absent. Es va descartar imputar-los amb zero perquè tots ells tenien comandes registrades (`orders_last_12m` entre 1 i 10) i tiquet mitjà (`avg_order_value`). Es van reconstruir multiplicant `avg_order_value × orders_last_12m`.
* **Nuls en recència (`days_since_last_order`):** 20 registres sense data d'última comanda van ser imputats amb la mediana (44 dies) per a no assumir una compra immediata fictícia.
* **Exclusió de variables:** Es van excloure `customer_id` i `snapshot_date` (sense valor analític) i `avg_order_value` per a evitar multicolinealitat, ja que la despesa anual ja és el producte de la freqüència i el tiquet mitjà.

### 4.2 Tècnica, escalat i selecció de grups ($k=4$)
Les quatre mètriques seleccionades (`days_since_last_order`, `orders_last_12m`, `annual_spend`, `returns_ratio`) presenten magnituds incompatibles (euros, dies, unitats i percentatges), fet que obliga a normalitzar amb `StandardScaler`.

Es va avaluar l'algorisme K-Means per a valors de $k$ entre 2 i 6:
* $k=2$: Inèrcia = 5.862,3 | Silhouette = 0,2786
* $k=3$: Inèrcia = 4.549,6 | Silhouette = 0,3329
* **$k=4$: Inèrcia = 3.498,0 | Silhouette = 0,3451** *(Configuració seleccionada)*
* $k=5$: Inèrcia = 2.782,8 | Silhouette = 0,3621
* $k=6$: Inèrcia = 2.432,0 | Silhouette = 0,3702

Tot i que la silueta augmenta lleugerament fins a $k=6$, **es tria $k=4$ per un estricte criteri d'interpretabilitat comercial**: genera quatre grups clarament accionables amb un volum equilibrat, evitant la creació de microsegments operativament inútils.

![Scatter plot de K-Means](03_clusters_scatter.png)
*Figura 4: Dispersió dels 4 perfils de clients segons la freqüència de compra i la despesa anual.*

### 4.3 Definició dels 4 perfils i accions comercials
L'anàlisi dels centroides dels 2.000 clients revela quatre comportaments ben diferenciats:

| Clúster / Perfil | Clients | % | Despesa anual | Comandes | Recència | Devolucions | Acció comercial proposada | Comprovació prèvia de l'empresa |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :--- | :--- |
| **0 · Estàndard satisfet** | 800 | 40,0% | 693,83 € | 3,61 | 51,4 dies | 0,0% | Venda creuada d'accessoris (cascos, llums, manteniment). | Comprovar compatibilitat d'accessoris i marge unitari. |
| **1 · VIP / Alt valor** | 644 | 32,2% | 2.866,48 € | 8,43 | 38,9 dies | 5,0% | Servei de taller gratuït, accés exclusiu i tracte preferent. | Verificar capacitat del taller i no canibalitzar marges. |
| **2 · Risc de devolució** | 318 | 15,9% | 687,36 € | 4,86 | 56,3 dies | **30,0%** | Auditoria de producte, revisió de guies de talles i atenció postvenda. | Analitzar motius reals de devolució abans de penalitzar el client. |
| **3 · Inactiu / Risc de churn** | 238 | 11,9% | 603,66 € | 2,65 | **247,9 dies** | 2,0% | Campanyes de reactivació personalitzades amb incentius temporals. | Verificar consentiment RGPD i canal preferit de contacte. |

### 4.4 Anàlisi de casos fronterers
Es van detectar 226 clients situats en la franja de 100 a 160 dies sense compra (límit entre client actiu i en risc d'inactivitat) i 258 clients amb taxes de devolució entre el 8% i el 18%. Aquests casos no han de rebre tractaments comercials agressius sense una validació prèvia del seu cicle habitual de reposició.

---

## 5. Traducció a Negoci i Arquitectura del Panell Executiu (Power BI)

### 5.1 Model de dades i bones pràctiques analítiques
Per a connectar els models de machine learning amb la presa de decisions corporativa, la capa de visualització en Power BI s'ha estructurat seguint un model d'estrella analític net:
* **Taula de fets de vendes:** Integra l'històric observat (2021–2026) amb una bandera de distinció (`is_forecast = FALSE`), unida mitjançant una relació d'unió (*append*) a les 18 files de previsió de juliol de 2026 (`is_forecast = TRUE`).
* **Taula de dimensions de clients:** Incorpora el segment de clúster assignat pel model K-Means.
* **Càlcul de mesures DAX:** Es van formular mesures explícites per a evitar sumes incorrectes de preus de tarifa o taxes de descompte (utilitzant `AVERAGE`, `SUMX` i filtres temporals específics).

### 5.2 Recomanacions directives finals
1. **Pla d'estoc per a juliol de 2026 (Alta prioritat):**
   * S'ha d'incrementar un 25% l'estoc de seguretat en bicicletes elèctriques i urbanes al canal `ecommerce` per a la regió `metro`. La ruptura de juny de 2026 demostra que l'estoc inicial limita de forma crítica les vendes en temporada alta.
2. **Pla de campanyes de màrqueting (Prioritat moderada):**
   * Concentrar el pressupost de campanya en regions urbanes i costaneres en primavera/estiu, però condicionant el llançament a la disponibilitat física prèvia d'inventari als magatzems.
3. **Estratègia de clients:**
   * Fidelitzar el 32% de clients VIP mitjançant serveis de valor afegit (tallers i manteniment) sense descomptes massius, i crear una taula de treball de qualitat per a reduir el 30% de devolucions del segment 2.

---

## 6. Límits del Sistema, Riscos i Línies Futures

### 6.1 Limitacions metodològiques
* **Dades sintètiques:** Els patrons observats procedeixen d'un generador controlat. Els coeficients de regressió descriuen correlacions numèriques en la mostra, però no garanteixen relacions de causa-efecte en un mercat ciclista real.
* **Absència de validació per a juliol 2026:** Les previsions per al pròxim mes no disposen de dades de tancament real; l'empresa haurà de monitoritzar les primeres setmanes per a calibrar les desviacions.

### 6.2 Línies futures de desenvolupament
* **Evolució del modelatge:** Incorporar models especialitzats en sèries temporals multivariants (com Prophet o SARIMAX) per a modelar pics no lineals d'estacionalitat com el d'abril.
* **Clustering avançat:** Explorar algorismes basats en densitat (DBSCAN) per a aïllar millor els valors atípics de despesa i patrons de devolució anòmals.
* **Pipeline d'automatització:** Desplegar el sistema en un entorn reproductible mitjançant FastAPI i Docker per a reentrenar els models a final de cada trimestre.

---

## 7. Organització de l'Equip i Autoavaluació

### 7.1 Repartiment de tasques
El desenvolupament operatiu es va dividir d'acord amb el calendari de sis setmanes:
* **Setmanes 1 i 2 (Frank Sabater):** Formulació de preguntes de negoci, diagnòstic de qualitat de dades, partició temporal estricta, neteja de nuls, càlcul dels dos baselines heurístics i redacció del CP1 i CP2.
* **Setmana 3 (David Camarena):** Implementació de la comparativa dels 15 models regressors en scikit-learn sobre validació 2025, anàlisi de mètriques i selecció justificada de `LinearRegression`.
* **Setmana 4 (Arnau Payá):** Preparació de dades de clients, depuració de nuls de despesa, execució de l'algorisme K-Means, anàlisi d'inèrcia/silueta per a $k=4$, descripció dels perfils i redacció del CP3.
* **Setmana 5 (Treball conjunt):** Obertura única del conjunt reservat de prova 2026, anàlisi profunda dels errors d'abril i juny, importància per permutació, generació de les prediccions cegues de juliol i tancament del CP4.
* **Setmana 6 (Treball conjunt):** Construcció del quadre de comandament executiu en Power BI, redacció de la memòria final i preparació de la defensa oral.

### 7.2 Autoavaluació de l'equip
1. **Quina ha sigut la millor decisió tècnica i quina evidència la sustenta?**  
   Triar `LinearRegression` enfront de models més complexos. L'evidència és incontestable: amb un MAE de 13,62 unitats en validació i $R^2 = 0,85$, va superar algorismes d'arbre i *ensembles*, oferint màxima interpretabilitat sense risc de sobreajustament.
2. **Quin problema ha costat més i com s'ha resolt?**  
   El tractament dels nuls en el fitxer de clients (`annual_spend`). Inicialment es va plantejar posar zeros, però es va descobrir que els clients tenien comandes positives i tiquet mitjà. Es va resoldre reconstruint la despesa de manera coherent (`avg_order_value × orders_last_12m`).
3. **Què canviaríem si tornàrem a començar?**  
   Hauríem definit des del primer dia la generació automatitzada de gràfics en alta resolució per a cada fase, estalviant temps en la consolidació dels lliurables.
4. **Quin nivell d'autonomia s'ha assolit?**  
   Nivell d'autonomia molt alt en el processament de dades amb pandas, avaluació rigorosa amb scikit-learn i modelatge BI, consolidant una comprensió crítica de per què les mètriques s'han de traduir sempre a decisions útils per a l'empresa.

---

## 8. Bibliografia i Referències Tècniques
* Pedregosa, F. et al. (2011). *Scikit-learn: Machine Learning in Python*. Journal of Machine Learning Research, 12, 2825-2830.
* McKinney, W. (2010). *Data Structures for Statistical Computing in Python*. Proceedings of the 9th Python in Science Conference.
* James, G., Witten, D., Hastie, T., & Tibshirani, R. (2021). *An Introduction to Statistical Learning with Applications in R/Python*. Springer.
* Documentació oficial de Power BI: *Model de dades en estrella i funcions DAX*, Microsoft Learn.
* Materials docents del curs: *Píndoles formatives d'IA i Big Data*, IES Jaume II el Just (Curs 2026–2027).
