# BiciTierra Market · CP3
## Setmana 4: segmentació de clients i decisió de model abans del test

> **Abast.** Aquest document recull la preparació i la segmentació de clients desenvolupades a [tests/semana4.ipynb](../tests/semana4.ipynb), i la comparació de regressors de [tests/semana3.ipynb](../tests/semana3.ipynb). La partició de test de gener a juny de 2026 continua reservada: **no s'hi ha d'escollir el model ni ajustar-ne els paràmetres**.

## 1. Preparació i qualitat de les dades de clients

El fitxer conté 2.000 clients. La segmentació busca grups accionables segons el seu comportament de compra, no segons la seva identitat ni les seves característiques personals.

### Variables incloses i excloses

| Variable | Decisió | Justificació |
|---|---|---|
| `days_since_last_order` | Incloure | Recència: quant de temps fa que el client no compra. Ajuda a distingir clients actius i possibles inactius. |
| `orders_last_12m` | Incloure | Freqüència: nombre de comandes en els darrers 12 mesos. |
| `annual_spend` | Incloure | Valor econòmic anual del client. |
| `returns_ratio` | Incloure | Proporció de comandes retornades; pot indicar friccions de producte o servei que cal investigar. |
| `avg_order_value` | Excloure del clustering | El valor anual és aproximadament el tiquet mitjà multiplicat per les comandes. Incloure les tres variables duplicaria part del senyal de despesa i li donaria massa pes. Es pot fer servir per reconstruir despesa absent, no com a quarta variable de clustering. |
| `customer_id`, `snapshot_date` | Excloure | Són identificador i data de fotografia, no comportament del client. |
| Edat, ciutat, canal i interessos | Excloure del clustering principal | L'objectiu és formar segments conductuals. Es poden emprar després per descriure els segments, però no s'incorporen distàncies categòriques al K-Means. L'edat té 20 nuls i no és necessària per a aquest objectiu. |

### Nuls i valors extrems

S'han detectat 30 nuls a `annual_spend`, 20 a `days_since_last_order` i 20 a `age`. **Inicialment es va plantejar substituir per zero els valors absents de despesa**, però la revisió de `data/customers.csv` mostra que no seria coherent: els 30 clients amb `annual_spend` absent tenen com a mínim una comanda en els darrers 12 mesos (`orders_last_12m` va d'1 a 10) i també tenen informat `avg_order_value`. Per tant, no es pot interpretar el nul com si no haguessin comprat o gastat res. Amb les dades disponibles, el notebook reconstrueix aquests imports amb `avg_order_value × orders_last_12m`; els imports reconstruïts van de 99,84 € a 7.080,05 €. Aquesta imputació és més coherent amb l'activitat observada, tot i que cal tenir present que és una estimació derivada i no el valor original registrat.

Els 20 nuls de `days_since_last_order` també corresponen a clients amb comandes positives (d'1 a 11) i despesa anual informada, de manera que posar-hi zero implicaria afirmar que acaben de comprar. El notebook els imputa amb la mediana; això manté una recència típica, però redueix la variabilitat d'aquests casos i s'ha de documentar com a limitació. `age` no s'utilitza ni cal imputar-la per al clustering. Abans del lliurament cal comprovar que les imputacions no deixen nuls residuals i confirmar que el producte de tiquet mitjà i freqüència representa adequadament la despesa anual en aquest conjunt de dades.

Les estadístiques disponibles mostren aquests extrems:

| Variable | Mediana | Màxim | Lectura / comprovació pendent |
|---|---:|---:|---|
| `days_since_last_order` | 44 dies | 364 dies | Revisar si 364 és un límit del sistema o un valor vàlid. |
| `orders_last_12m` | 5 | 17 | Comprovar que el màxim és coherent amb l'interval de 12 mesos. |
| `annual_spend` | 766,71 € | 13.968,60 € | El màxim és molt superior a la mediana: validar-lo, però no eliminar-lo només per ser extrem; podria ser un client realment valuós. |
| `returns_ratio` | 0,00 | 1,00 | Comprovar el càlcul del quocient i si el valor 1,00 correspon a totes les comandes retornades. |

No s'eliminen extrems sense validar-los. `StandardScaler` posa les variables en escales comparables, però no és robust als extrems. Com a comprovació de sensibilitat, convé comparar la solució amb `RobustScaler` o amb `log1p` per a la despesa i revisar si els perfils canvien substancialment. Si el canvi és gran, s'ha de documentar i triar l'escala amb criteri de negoci.

## 2. Referència heurística

Les regles manuals del notebook s'utilitzen com a primera referència interpretable, no com a resultat de clustering. La distribució guardada és:

| Segment heurístic | Clients | Percentatge | Regla resumida |
|---|---:|---:|---|
| Ocasional o baix valor | 1.532 | 76,6% | No compleix els criteris de premium ni de valor mitjà-alt. |
| Valor mitjà-alt | 288 | 14,4% | Despesa anual ≥ 1.200 € i almenys 8 comandes. |
| Premium | 180 | 9,0% | Despesa anual ≥ 3.000 € i devolucions ≤ 2%. |

La segmentació heurística ajuda a tenir una referència de negoci fàcil d'explicar. K-Means, en canvi, crea grups a partir de distàncies entre totes les variables conductuals seleccionades. No s'espera que els grups coincideixin exactament.

## 3. Tècnica, escala i nombre de grups

S'aplica K-Means sobre `days_since_last_order`, `orders_last_12m`, `annual_spend` i `returns_ratio`, després d'estandarditzar-les amb `StandardScaler`. L'estandardització és necessària perquè euros, dies, comandes i proporcions tenen unitats i magnituds diferents.

S'han provat valors de $k$ entre 2 i 6. Els resultats guardats són:

| Nombre de grups ($k$) | Inèrcia | Silhouette |
|---:|---:|---:|
| 2 | 5.862,3 | 0,2786 |
| 3 | 4.549,6 | 0,3329 |
| 4 | 3.498,0 | 0,3451 |
| 5 | 2.782,8 | 0,3621 |
| 6 | 2.432,0 | 0,3702 |

La solució de quatre grups es manté **com a decisió provisional d'interpretabilitat comercial**, no perquè sigui la que maximitza la silhouette: aquesta mètrica creix fins a $k=6$. Cal comparar les mides i els perfils de $k=4$, $k=5$ i $k=6$ abans de tancar la decisió, i explicar per què quatre grups són més accionables si es manté $k=4$. Una silhouette de 0,3451 suggereix separació moderada, no grups perfectament separats.

El notebook indica que la tècnica inicial és K-Means. Si el lliurament requereix acord explícit amb el professorat, cal confirmar-lo i deixar-ne constància; no es pot deduir aquest acord dels materials disponibles.

## 4. Perfils de K-Means i interpretació

Les mitjanes i mides següents corresponen als resultats guardats del model amb $k=4$ sobre 2.000 clients:

| Cluster / perfil | Clients | % | Despesa anual mitjana | Comandes mitjanes | Dies des de l'última compra | Devolucions mitjanes | Interpretació |
|---|---:|---:|---:|---:|---:|---:|---|
| 0 · Estàndard satisfet | 800 | 40,0% | 693,83 € | 3,61 | 51,40 | 0,00 | Grup més gran: despesa i freqüència moderades, compra relativament recent i cap devolució de mitjana. |
| 1 · VIP / alt valor | 644 | 32,2% | 2.866,48 € | 8,43 | 38,93 | 0,05 | Més despesa i freqüència, i recència baixa; és el grup de més valor actual. |
| 2 · Risc de devolució | 318 | 15,9% | 687,36 € | 4,86 | 56,27 | 0,30 | Recència semblant a la resta d'actius, però amb una proporció de devolucions marcadament superior. Cal investigar-ne les causes abans d'etiquetar els clients com a problemàtics. |
| 3 · Inactiu / possible churn | 238 | 11,9% | 603,66 € | 2,65 | 247,93 | 0,02 | Molts més dies sense comprar i menys freqüència; és el perfil més clar de possible inactivitat. |

La lectura conjunta diferencia especialment VIP i estàndard per despesa/freqüència, i el grup inactiu per recència. El grup de devolucions es diferencia principalment per `returns_ratio`, no per una recència molt alta. Els noms són etiquetes descriptives dels valors mitjans, no diagnòstics individuals: un client concret pot no encaixar amb el perfil mitjà del seu cluster.

## 5. Taula de segments i accions comercials

| Perfil | Acció proposada | Què ha de comprovar l'empresa abans d'aplicar-la |
|---|---|---|
| Estàndard satisfet (800; 40,0%) | Fer venda creuada d'accessoris útils (cascos, llums o manteniment) i recomanacions segons la compra anterior. | Estoc disponible, compatibilitat amb la bicicleta, marge de cada producte i que la campanya no sigui massa repetitiva. Mesurar conversió i marge incremental. |
| VIP / alt valor (644; 32,2%) | Proposar avantatges de fidelització, servei de taller o accés anticipat; prioritzar el reconeixement abans que descomptes generals. | Marge i cost de l'avantatge, capacitat del taller, historial i consentiment de contacte. Verificar que el benefici augmenta retenció o valor i no subvenciona compres que igualment s'haurien produït. |
| Risc de devolució (318; 15,9%) | Investigar devolucions per producte i millorar informació, guia de talles, control de qualitat o assistència de compra. | Motius reals de devolució, defectes o incidències de proveïdor, política de devolucions i possibles errors en el càlcul del ratio. No penalitzar el client ni restringir-li ofertes sense aquesta comprovació. |
| Inactiu / possible churn (238; 11,9%) | Provar una campanya de recuperació personalitzada; reservar el cupó per als casos on sigui necessari. | Cicle normal de reposició, llindar d'inactivitat, consentiment i canal de contacte, marge després del descompte i grup de control per mesurar l'efecte incremental. |

Les accions són hipòtesis per provar, no efectes causals demostrats pel clustering. És recomanable fer una prova controlada i comparar conversió, marge, devolucions i retenció amb un grup de control.

## 6. Casos fronterers i limitacions

El notebook identifica 226 clients amb 100–160 dies des de l'última compra i 258 clients amb `returns_ratio` entre 0,08 i 0,18. Aquests intervals assenyalen situacions ambigües entre actiu/inactiu o devolució baixa/alta. S'hi mostren exemples, però encara falta resumir-ne l'assignació a cada cluster i explicar quins factors —despesa, freqüència, recència o ratio— fan que el cas sigui fronterer. No s'han de prendre decisions individuals només a partir d'aquests talls.

Abans de lliurar, afegir per a cada interval una taula amb el nombre i percentatge de clients per cluster i, si escau, revisar una mostra de casos. Els llindars de 100–160 dies i 8–18% són intervals exploratoris: cal contrastar-los amb el cicle de compra i amb l'historial real de l'empresa.

## 7. Comparativa de models de la setmana 3

La comparació del notebook de setmana 3 fa servir les mateixes 216 observacions de validació de 2025. Les mètriques següents són les que consten als resultats guardats del notebook; s'han de tornar a produir en una execució neta abans de considerar-les definitives.

| Model | MAE (menor és millor) | RMSE (menor és millor) | R² (major és millor) |
|---|---:|---:|---:|
| LinearRegression | **13,62** | **18,31** | **0,8472** |
| KernelRidge | 13,97 | 18,36 | 0,8464 |
| SGDRegressor | 14,12 | 18,56 | 0,8430 |
| Ridge | 14,27 | 18,63 | 0,8417 |
| GradientBoosting | 14,34 | 19,71 | 0,8228 |
| Lasso | 14,81 | 19,50 | 0,8266 |
| RandomForest | 14,96 | 20,41 | 0,8102 |
| PLSRegression | 15,04 | 19,41 | 0,8283 |
| AdaBoost | 15,18 | 19,99 | 0,8179 |
| Bagging | 15,99 | 21,78 | 0,7839 |
| GaussianProcessRegressor | 20,32 | 26,74 | 0,6741 |
| DecisionTree | 21,13 | 27,64 | 0,6517 |
| KNN | 23,12 | 31,37 | 0,5515 |
| SVR | 32,80 | 43,96 | 0,1192 |
| IsotonicRegression | 35,87 | 45,95 | 0,0373 |

Com a context, la memòria de CP2 informa que, també a la validació de 2025, la referència de mitjana històrica obté MAE 19,92 i WAPE 19,19%, i la referència del mateix mes de l'any anterior obté MAE 18,94 i WAPE 18,25%. El millor model de la taula, LinearRegression, redueix el MAE aproximadament un 28,1% respecte a la millor d'aquestes dues referències. Cal confirmar que les files i el tractament de dades són els mateixos abans d'interpretar aquesta comparació.

### Decisió abans del test

**Decisió proposada: seleccionar LinearRegression com a model candidat**, perquè a la validació guardada té el MAE més baix, el RMSE més baix i el R² més alt de la comparativa; a més, és més senzill que els models més complexos. La diferència amb KernelRidge és petita, però no justifica preferir el model més complex amb les evidències actuals.

Aquesta decisió queda condicionada a corregir i verificar la preparació abans de congelar el model: setmana 3 aplica `fillna(0)` a una còpia completa de les dades, mentre CP2 indica que no s'ha d'interpretar automàticament un valor nul de `marketing_budget` o `planned_discount` com a zero. Cal resoldre i documentar aquests nuls d'acord amb el significat de les dades, tornar a executar tots els models sobre el mateix entrenament i validació i registrar la decisió. Després, es congelen variables, imputació i paràmetres. **Només llavors s'obre el test de 2026** per a una avaluació final única; els resultats del test no s'han d'utilitzar per tornar a triar el model.

Hi ha, a més, una contradicció al final del notebook de setmana 4: diu que s'escull Ridge amb $R^2 \approx 0,22$. Això no coincideix amb la comparativa guardada de setmana 3, on Ridge té $R^2=0,8417$ i LinearRegression és el primer per les tres mètriques. Cal corregir aquella nota amb la decisió final que es prengui després de reproduir la validació; no s'ha trobat evidència del valor 0,22 en la taula comparativa actual.

## 8. Comprovacions pendents abans del lliurament

- Tornar a executar els notebooks des d'un kernel net i confirmar que les taules coincideixen amb els resultats documentats.
- Revisar la imputació de variables dels models predictius; eliminar el `fillna(0)` general si no està justificat columna per columna.
- Comparar els perfils de K-Means per a $k=4$, $k=5$ i $k=6$ i justificar el nombre final de grups.
- Comprovar la sensibilitat als extrems i a l'escalat sense descartar automàticament clients vàlids.
- Afegir els recomptes dels casos fronterers per cluster i documentar l'acord amb el professorat sobre la tècnica si és necessari.
- Corregir la nota de decisió Ridge de setmana 4 i deixar constància que el model final es tria amb validació, abans d'obrir el test.
