# Informe Técnico de Preparación, Limpieza y Estructuración de Datos

**Evidencia de Aprendizaje:** `GA1-220501094-01-AA1-EV07 – Preparación, limpieza y estructuración de datos del proyecto individual con Pandas`  
**Programa de Formación:** Tecnólogo en Análisis y Desarrollo de Software (ADSO) - Ficha 3223873  
**Aprendiz:** Simón Carmona Toro  
**Instructor:** Carlos Enrique Navia Torres  
**Fecha:** Noviembre de 2026  
**Repositorio GitHub:** [proyecto-ciencia-datos-ev05](https://github.com/simoncarmona-dotcom/proyecto-ciencia-datos-ev05.git)  

---

## 1. Introducción y Contexto del Proyecto Individual

El presente informe documenta la ejecución técnica y los hallazgos obtenidos en la evidencia **GA1-220501094-01-AA1-EV07**, donde se aplican las técnicas de limpieza, filtrado, agrupación y combinación de datos de las Lecciones 2, 3 y 4 con **Python** y **Pandas** sobre el proyecto individual de investigación: **Ciencia de Datos Aplicada al Balance de Campeones, Tasas de Selección/Bloqueo (*Pick/Ban Rate*), Dificultad Mecánica y Escalado Estadístico por Clases en *League of Legends***.

### 1.1 Pregunta Analítica del Proyecto (EV02 / EV03 / EV05)
> *«¿De qué manera la clase de rol de un campeón (Asesino, Mago, Tirador, Tanque, Luchador o Soporte), su índice de dificultad mecánica y la duración total del encuentro influyen en la variación de su tasa de victoria (Win Rate), tasa de selección (Pick Rate) y tasa de bloqueo (Ban Rate) en el metajuego clasificatorio de League of Legends?»*

### 1.2 Conformidad de las Fuentes de Datos con la Guía de Aprendizaje
En cumplimiento estricto con la **página 6 de la guía de aprendizaje**, los tres conjuntos de datos crudos almacenados en `data/raw/` provienen de los portales de datos abiertos autorizados:
1. **`lol_champion_meta_stats.csv` (100 filas $\times$ 11 columnas):** Estadísticas del metajuego competitivo por campeón y carril (Portal: **Kaggle Datasets** – `vivovinco/league-of-legends-champion-stats`).
2. **`lol_champions_base_attributes.csv` (70 filas $\times$ 14 columnas):** Catálogo oficial de parámetros de diseño de fábrica, dificultad mecánica (1 a 10) y crecimiento de estadísticas por nivel (Portal: **Riot Games Data Dragon / Kaggle** – `gyejr95/league-of-legends-champion-stats`).
3. **`lol_ranked_matches_performance.csv` (500 filas $\times$ 11 columnas):** Registros de desempeño individual por partida clasificatoria con duración en segundos y daño total infligido (Portal: **Google Dataset Search / Kaggle** – `paololol/league-of-legends-ranked-matches`).

---

## 2. Lección 2: Limpieza y Estandarización de Datos Crudos

### 2.1 Pipeline Modular con `.pipe()` y Registro de Estado (`anotar()`)
Tomando como punto de partida el diagnóstico de calidad desarrollado en la **EV05** ([`notebooks/data_quality.ipynb`](../notebooks/data_quality.ipynb) y [`reports/data_quality.md`](data_quality.md)), se construyó un pipeline modular encadenado con `.pipe()` en [`notebooks/data_preparation.ipynb`](../notebooks/data_preparation.ipynb) que ejecuta las siguientes acciones justificadas:

1. **Deduplicación de clave compuesta `(Name, Role)` en `meta_stats`:** Se identificaron 4 pares campeón-carril repetidos (*Akali MID*, *Aurelion Sol MID*, *Cho'Gath TOP* y *Diana JUNGLE*). Se ordenó por frecuencia en el carril (`Role %`) y se conservó el registro principal de cada par, consolidando **96 combinaciones campeón-rol únicas**.
2. **Filtrado de partidas canceladas (*remakes*) en `matches_perf`:** Se detectaron 3 partidas con duración menor a 300 segundos (`duration_sec` de 213 s, 217 s y 234 s), correspondientes a cierres anticipados (*remakes*) al minuto 3 que distorsionaban el cálculo de daño por minuto (`DPM`). Al aplicar el filtro `duration_sec >= 300`, se consolidaron **497 partidas clasificatorias válidas**.
3. **Normalización de llaves de unión (`champion_key`):** Se resolvieron las discrepancias entre `id` y `name` en los 7 campeones con espacios o apóstrofos (*Aurelion Sol, Cho'Gath, Kai'Sa, Kha'Zix, Lee Sin, Master Yi, Miss Fortune*) mediante la función `normalizar_llave()`, que genera una clave alfanumérica en minúsculas (`aurelionsol`, `chogath`, `kaisa`, etc.).
4. **Tratamiento justificado de valores faltantes:**
   - En `meta_stats`: los 4 nulos de `Class` (*Camille, Maokai, Yasuo, Ezreal*) se imputaron dinámicamente cruzando con la clase primaria de `base_attributes['tags']`; los 3 nulos de `Tier` se imputaron con la moda estadística (`'A'`); los 7 nulos de `Trend` se imputaron con `0.0`.
   - En `base_attributes`: los 5 nulos de `mp` y `mpperlevel` (*Garen, Katarina, Mordekaiser, Riven, Yasuo*) se imputaron con `0.0` y los 4 nulos de `partype` con `'None'`, unificando la codificación de campeones sin recurso de maná.
   - En `matches_perf`: los 9 nulos de `role` se imputaron con el carril principal del campeón en el metajuego y los 7 nulos de `damage_dealt` con la mediana de daño del campeón en dicho carril.
5. **Conversión de tipos inconsistentes y optimización de memoria:**
   - Conversión de `Win %`, `Role %`, `Pick %` y `Ban %` de texto (`object`) a proporción decimal (`float32` en $[0.0, 1.0]$).
   - Conversión de `tier` a categoría ordenada (`S+ > S > A > B > C > D`) y de variables de baja cardinalidad (`champion_class`, `role`, `partype`, `tags`, `patch`) a `category`.

### 2.2 Comparación Cuantitativa de Calidad (Antes vs. Después)

| Dataset | Paso del Pipeline | Filas | Columnas | Nulos Totales | Memoria (KB) | Ahorro / Variación |
|---|---|:---:|:---:|:---:|:---:|---|
| **base_attributes** | `0_crudo` | 70 | 14 | 14 | 10.5 KB | 5 campeones sin maná con `NaN` en `mp`/`mpperlevel` y 4 en `partype`. |
| **base_attributes** | `2_limpio_y_tipado` | 70 | 16 | 0 | 7.5 KB | **28.6% de ahorro en memoria**; llave `champion_key` y `primary_class` añadidas. |
| **meta_stats** | `0_crudo` | 100 | 11 | 14 | 12.7 KB | 4 pares `(Name, Role)` repetidos y porcentajes como `object`. |
| **meta_stats** | `2_limpio_y_tipado` | 96 | 12 | 0 | 6.0 KB | **52.8% de ahorro en memoria**; clave compuesta única y `tier` ordenado. |
| **matches_perf** | `0_crudo` | 500 | 11 | 16 | 48.1 KB | 3 partidas *remake* ($< 300$ s) y nulos en `role` (9) y `damage_dealt` (7). |
| **matches_perf** | `2_limpio_y_metricas`| 497 | 16 | 0 | 32.7 KB | **32.0% de ahorro en memoria**; 497 partidas válidas con `DPM`, `OPM` y `KDA`. |

---

## 3. Lección 3: Filtrado, Ordenamiento, Agrupación e Indicadores

### 3.1 Filtrado Avanzado y Selección de Extremos
- **Máscaras booleanas + `.isin(['TOP', 'MID'])` + `.between(0.51, 0.60)`:** Se identificaron **8 campeones dominantes en líneas solitarias** con alta popularidad (`pick_rate >= 8%`), liderados por *Cho'Gath TOP* (54.36% Win Rate), *Swain MID* (54.09%) y *Orianna MID* (53.55%).
- **Filtro con `.query("ban_rate >= 0.25 and win_rate >= 0.51")`:** Se detectaron **15 registros con alerta de desbalance competitivo**, candidatos a ajustes (*nerfs*), destacando *Kai'Sa MID* (33.50% Ban Rate, 53.19% Win Rate) y *Cho'Gath TOP* (30.30% Ban Rate, 54.36% Win Rate).

### 3.2 Indicador 1: Balance Competitivo por Clase y Rol
Resumen global por clase de campeón obtenido mediante `groupby('champion_class').agg(...)`:

| Clase de Campeón | Registros Rol | Campeones Únicos | Win Rate (%) | Pick Rate (%) | Ban Rate (%) | KDA Medio | Meta Score Medio |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Marksman** | 18 | 11 | **51.54%** | 8.54% | 22.44% | **2.94** | 67.89 |
| **Mage** | 19 | 15 | **50.65%** | 10.69% | 16.07% | 2.89 | 66.40 |
| **Fighter** | 27 | 19 | **50.56%** | 9.66% | 20.72% | 2.66 | 67.16 |
| **Assassin** | 10 | 8 | **50.28%** | 8.86% | **22.82%** | 2.80 | 64.73 |
| **Tank** | 12 | 9 | **50.17%** | **11.90%** | 20.72% | 2.52 | **71.71** |
| **Support** | 10 | 8 | **49.69%** | 10.53% | 17.06% | 2.91 | 62.83 |

#### Tabla Dinámica (`pivot_table`): Win Rate Promedio (%) por Clase y Rol

| Clase \ Rol | ADC | JUNGLE | MID | SUPPORT | TOP |
|---|:---:|:---:|:---:|:---:|:---:|
| **Assassin** | 47.88% | **51.26%** | 49.99% | — | — |
| **Fighter** | 51.48% | 50.66% | 50.37% | **53.82%** | 50.08% |
| **Mage** | — | 50.40% | 50.12% | 50.56% | **54.14%** |
| **Marksman** | **52.14%** | 50.85% | **53.19%** | 50.38% | 50.31% |
| **Support** | — | 48.60% | — | **49.97%** | 48.51% |
| **Tank** | 48.33% | 50.45% | — | 50.27% | **51.03%** |

* **Interpretación:** Los **Tiradores (`Marksman`)** lideran la efectividad global (**51.54%**) y dominan el carril inferior (`ADC`: **52.14%**). Los **Luchadores (`Fighter`)** demuestran la mayor versatilidad táctica del metajuego al sostener un Win Rate superior al 50% en los 5 carriles del mapa. Los **Asesinos (`Assassin`)** sufren la mayor tasa de bloqueo promedio (**22.82%**), siendo efectivos en `JUNGLE` (**51.26%**) pero inestables fuera de su rol natural.

---

### 3.3 Indicador 2: Curva de Efectividad según Índice de Dificultad Mecánica
Segmentación de `difficulty` (escala 1 a 10) mediante `pd.cut` y cálculo de diferenciales intragrupo con `.transform('mean')`:

| Rango de Dificultad | Registros Rol | Campeones Únicos | Dificultad Media | Win Rate (%) | Pick Rate (%) | Ban Rate (%) | KDA Medio |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Baja (1-3)** | 16 | 12 | 2.50 | **51.18%** | 9.91% | 16.02% | 2.78 |
| **Media (4-6)** | 49 | 34 | 5.02 | **50.57%** | 9.33% | **20.82%** | **2.81** |
| **Alta (7-10)** | 31 | 24 | 7.65 | **50.33%** | **10.92%** | 20.64% | 2.73 |

* **Interpretación (La Paradoja de la Dificultad):** Los campeones de **Baja Dificultad (1–3)** superan en tasa de victoria promedio (**51.18%**) a los de **Media (50.57%)** y **Alta Dificultad (50.33%)**. Una menor complejidad de ejecución reduce los errores mecánicos en clasificatorias. Sin embargo, los campeones de **Alta Dificultad (7–10)** tienen la mayor tasa de selección (**10.92%**) y un bloqueo elevado (**20.64%**), impulsados por especialistas que superan ampliamente el promedio de su segmento (*Swain MID* $+3.76\%$, *Yone MID* $+3.67\%$, *Azir TOP* $+3.58\%$).

---

### 3.4 Indicador 3: Escalado Temporal según Duración del Encuentro
Sobre las **497 partidas clasificatorias válidas** (`duration_sec >= 300`), se segmentó la duración en tres tramos con `pd.cut`:

| Clase de Campeón | Tramo de Duración | Partidas | Distribución Clase (%) | Win Rate Partidas (%) | Daño Medio | Oro Medio | Daño por Minuto (DPM) | KDA Medio |
|---|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Assassin** | Corta (< 25 min) | 12 | 21.4% | 33.33% | 28,461 | 14,734 | **1,200.8** | **7.56** |
| **Assassin** | Media (25-35 min) | 25 | 44.6% | 52.00% | 29,288 | 15,834 | 980.6 | 6.32 |
| **Assassin** | Larga (> 35 min) | 19 | 33.9% | 57.89% | 32,775 | 17,806 | 834.4 | 5.76 |
| **Fighter** | Corta (< 25 min) | 19 | 14.0% | 5.26% | 28,308 | 14,754 | **1,205.0** | 6.06 |
| **Fighter** | Media (25-35 min) | 66 | 48.5% | 45.45% | 29,248 | 15,903 | 971.9 | 5.51 |
| **Fighter** | Larga (> 35 min) | 51 | 37.5% | 39.22% | 33,836 | 17,975 | 880.0 | 6.35 |
| **Mage** | Corta (< 25 min) | 7 | 6.6% | 100.00% | 24,583 | 14,100 | 1,036.3 | 5.19 |
| **Mage** | Media (25-35 min) | 67 | **63.2%** | 58.21% | 30,295 | 16,107 | 1,003.0 | 5.89 |
| **Mage** | Larga (> 35 min) | 32 | 30.2% | **75.00%** | 33,668 | 17,928 | 876.8 | 5.47 |
| **Marksman** | Corta (< 25 min) | 14 | 18.2% | 57.14% | 29,486 | 15,210 | **1,251.2** | **9.13** |
| **Marksman** | Media (25-35 min) | 36 | 46.8% | 47.22% | 28,857 | 15,680 | 970.8 | 6.55 |
| **Marksman** | Larga (> 35 min) | 27 | 35.1% | **62.96%** | 33,730 | 18,152 | 862.0 | 5.57 |
| **Support** | Corta (< 25 min) | 10 | 17.5% | 40.00% | 22,926 | 13,268 | 988.8 | 3.66 |
| **Support** | Media (25-35 min) | 28 | 49.1% | 42.86% | 29,151 | 15,677 | 1,010.2 | 6.67 |
| **Support** | Larga (> 35 min) | 19 | 33.3% | 26.32% | 36,606 | 19,027 | 914.2 | 6.78 |
| **Tank** | Corta (< 25 min) | 11 | 16.9% | 63.64% | 23,115 | 13,726 | 984.6 | 5.63 |
| **Tank** | Media (25-35 min) | 18 | 27.7% | 38.89% | 31,954 | 16,509 | 1,065.5 | 6.34 |
| **Tank** | Larga (> 35 min) | 36 | **55.4%** | **61.11%** | 34,021 | 18,068 | 866.3 | 6.12 |

* **Interpretación:** En partidas largas ($> 35$ min, con inventarios superiores a 17,900 de oro), los **Magos (`75.00%`)**, **Tiradores (`62.96%`)** y **Tanques (`61.11%`, donde ocurre el 55.4% de sus encuentros)** consolidan las mayores tasas de victoria gracias al escalado sinérgico de daño sostenido y absorción de daño en peleas grupales 5v5.

---

## 4. Lección 4: Combinación y Estructuración de Datasets

### 4.1 Auditoría de Unión y Validación de Cardinalidad
Se integraron los tres datasets mediante `champion_key`, comprobando con `indicator=True` un emparejamiento del **100% (`both: 96` en metajuego y `both: 497` en partidas)**, sin registros huérfanos (`left_only: 0`):
1. **Ficha Consolidada de Campeones (`lol_champions_consolidated.csv` y `.parquet`):** `96 filas x 31 columnas` que unen `meta_limpio` con `base_limpio` (`validate='m:1'`) y las métricas agregadas de las partidas (`validate='m:1'`).
2. **Tabla Analítica Maestra de Partidas (`lol_master_analytics.csv` y `.parquet`):** `497 filas x 40 columnas` que unen cada encuentro clasificatorio válido con los atributos base de fábrica y el resumen de metajuego del campeón (`validate='m:1'`).
3. **Estructuración Adicional:** Se aplicó `pd.concat()` para comparar a los líderes de baja vs. alta dificultad y `.melt()` para estructurar las tasas competitivas en formato largo `(18, 3)` listo para visualización.
