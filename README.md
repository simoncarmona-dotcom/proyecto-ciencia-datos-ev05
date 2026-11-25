# Proyecto de Ciencia de Datos: Balance de Campeones, Dificultad Mecánica y Escalado en League of Legends

* **Programa de Formación:** Tecnólogo en Análisis y Desarrollo de Software (`228118`) - Ficha `3223873`
* **Competencia:** `220501094` - Estructurar propuesta técnica de servicio de tecnología de la información según requisitos técnicos y normativa
* **Evidencias Formativas Desarrolladas:**
  - `GA1-220501094-01-AA1-EV05`: Diagnosticar la calidad de datos: preparación, exploración y documentación
  - `GA1-220501094-01-AA1-EV06`: Limpieza, análisis y estructuración de datos con Pandas (Instacart)
  - `GA1-220501094-01-AA1-EV07`: Preparación, limpieza y estructuración de datos del proyecto individual con Pandas (*League of Legends*)
* **Aprendiz:** Simón Carmona Toro
* **Instructor:** Carlos Enrique Navia Torres
* **Repositorio GitHub:** [proyecto-ciencia-datos-ev05](https://github.com/simoncarmona-dotcom/proyecto-ciencia-datos-ev05.git)

---

## 1. Descripción del Proyecto Individual

Este proyecto de ciencia de datos analiza de forma objetiva, utilizando **Python** y **Pandas**, el equilibrio competitivo de los campeones de **League of Legends** en función de su clase de rol (*Assassin*, *Fighter*, *Mage*, *Marksman*, *Support*, *Tank*), su índice oficial de dificultad mecánica (escala de 1 a 10) y la duración de los encuentros clasificatorios.

En la evidencia **EV07**, se ejecuta el flujo completo de **limpieza, estandarización, filtrado, agrupación, cálculo de indicadores e integración relacional** sobre los tres datasets del proyecto individual, dando continuidad directa al diagnóstico de calidad realizado en la **EV05**.

---

## 2. Pregunta Analítica del Proyecto (EV02 / EV03 / EV05 / EV07)

> *«¿De qué manera la clase de rol de un campeón (Asesino, Mago, Tirador, Tanque, Luchador o Soporte), su índice de dificultad mecánica y la duración total del encuentro influyen en la variación de su tasa de victoria (Win Rate), tasa de selección (Pick Rate) y tasa de bloqueo (Ban Rate) en el metajuego clasificatorio de League of Legends?»*

---

## 3. Fuentes de Datos Oficiales (Conformidad con Guía EV03, EV05 y EV07)

En cumplimiento estricto con los portales de datos abiertos autorizados en la **página 6 de la guía de aprendizaje**, los datasets originales ubicados en `data/raw/` fueron obtenidos de:

| Archivo en `data/raw/` | Dimensiones Crudas | Dimensiones Limpias | Fuente / Portal Autorizado | Descripción |
| :--- | :---: | :---: | :--- | :--- |
| `lol_champion_meta_stats.csv` | `100 x 11` | `96 x 12` | **Kaggle Datasets** (`vivovinco/league-of-legends-champion-stats`) | Estadísticas del metajuego por campeón y carril (`Win %`, `Pick %`, `Ban %`, `Role %`, `Tier`, `Score`, `Trend`, `KDA`). |
| `lol_champions_base_attributes.csv` | `70 x 14` | `70 x 16` | **Riot Games Data Dragon / Kaggle** (`gyejr95/league-of-legends-champion-stats`) | Catálogo oficial de atributos base de fábrica, índice de dificultad mecánica (`1` a `10`) y crecimiento estadístico por nivel. |
| `lol_ranked_matches_performance.csv` | `500 x 11` | `497 x 16` | **Google Dataset Search / Kaggle** (`paololol/league-of-legends-ranked-matches`) | Desempeño individual por partida clasificatoria, daño total infligido (`damage_dealt`), oro (`gold_earned`) y duración (`duration_sec`). |

---

## 4. Estructura del Repositorio

El repositorio conserva los archivos de las actividades anteriores y amplía la estructura del proyecto utilizando nombres en inglés conforme a la página 25 de la guía:

```text
proyecto-ciencia-datos/
├── README.md                                      # Descripción, fuentes, transformaciones e instrucciones
├── .gitignore                                     # Exclusiones de Git (entornos virtuales, caches)
├── requirements.txt                               # Dependencias (pandas, numpy, pyarrow, jupyter, ipykernel)
│
├── data/
│   ├── raw/                                       # Datasets originales sin modificaciones
│   │   ├── lol_champion_meta_stats.csv            # Metajuego competitivo por campeón y rol (100 filas)
│   │   ├── lol_champions_base_attributes.csv      # Atributos base y dificultad mecánica (70 filas)
│   │   ├── lol_ranked_matches_performance.csv     # Desempeño en partidas clasificatorias (500 filas)
│   │   └── (datasets de práctica Instacart EV06)
│   │
│   └── processed/                                 # Datasets limpios, validados y estructurados
│       ├── lol_master_analytics.csv               # Tabla maestra de partidas (497 filas x 40 columnas)
│       ├── lol_master_analytics.parquet           # Tabla maestra de partidas en formato binario Parquet
│       ├── lol_champions_consolidated.csv         # Ficha consolidada campeón-rol (96 filas x 31 columnas)
│       ├── lol_champions_consolidated.parquet     # Ficha consolidada campeón-rol en formato Parquet
│       ├── cleaned_lol_champion_meta.csv          # Dataset de metajuego limpio y tipado (96 filas)
│       ├── cleaned_lol_base_attributes.csv        # Catálogo base limpio con llave normalizada (70 filas)
│       ├── cleaned_lol_matches_performance.csv    # Partidas válidas >= 300s con DPM y KDA (497 filas)
│       ├── resumen_clase_rol.csv                  # Indicador 1: Balance competitivo por Clase y Rol
│       ├── resumen_dificultad.csv                 # Indicador 2: Curva de efectividad por Dificultad
│       ├── resumen_escalado_duracion.csv          # Indicador 3: Escalado temporal por Duración
│       ├── informe_limpieza_lol.csv               # Registro cuantitativo del pipeline de limpieza
│       └── (archivos procesados de Instacart EV06)
│
├── notebooks/
│   ├── data_quality.ipynb                         # Diagnóstico de calidad del proyecto individual (EV05)
│   ├── data_preparation.ipynb                     # Cuaderno principal EV07: Limpieza, Indicadores y Fusión LoL
│   ├── diagnostico_calidad.ipynb                  # Copia en español del diagnóstico EV05
│   └── pandas_labs.ipynb                          # Laboratorios guiados 2, 3, 4 y 21 retos Instacart (EV06)
│
└── reports/
    ├── data_quality.md                            # Informe técnico de diagnóstico de calidad inicial (EV05)
    ├── data_preparation_report.md                 # Informe técnico completo de preparación y análisis (EV07)
    ├── informe_diagnostico.md                     # Copia en español del informe diagnóstico EV05
    └── data_processing_report.md                  # Informe técnico de procesamiento Instacart (EV06)
```

---

## 5. Resumen de Transformaciones y Resultados Analíticos (EV07)

El cuaderno **[`notebooks/data_preparation.ipynb`](notebooks/data_preparation.ipynb)** desarrolla paso a paso las tres lecciones aplicadas al proyecto individual:

### Lección 2: Limpieza y Estandarización de Datos
* **Pipeline modular con `.pipe()` y monitoreo con `anotar()`:**
  1. **Deduplicación de clave compuesta `(Name, Role)`:** Eliminación de 4 registros secundarios repetidos en `meta_stats` (*Akali MID, Aurelion Sol MID, Cho'Gath TOP, Diana JUNGLE*), consolidando **96 combinaciones campeón-rol únicas**.
  2. **Filtrado de partidas canceladas (*remakes*):** Exclusión de las 3 partidas con `duration_sec < 300` (213 s, 217 s y 234 s) en `matches_perf`, consolidando **497 partidas clasificatorias válidas**.
  3. **Normalización de llave relacional (`champion_key`):** Estandarización en minúsculas sin espacios ni apóstrofos (`chogath`, `kaisa`, `khazix`, `leesin`) para resolver las 7 discrepancias entre `id` y `name`.
  4. **Imputación justificada de valores faltantes:** Recuperación de los 4 nulos de `Class` desde `base_attributes['tags']`; imputación de `Tier` con la moda (`'A'`) y `Trend` con `0.0`; unificación de campeones sin maná con `mp = 0.0` y `partype = 'None'`; imputación de `role` por rol dominante y `damage_dealt` por mediana del campeón y carril.
  5. **Optimización de tipos y validación con `assert`:** Conversión de porcentajes de texto (`object`) a proporción decimal (`float32`), `tier` a categoría ordenada (`S+ > S > A > B > C > D`) y reducción de memoria del **52.8%** en `meta_stats`, **28.6%** en `base_attributes` y **32.0%** en `matches_perf`, con **0 valores nulos** verificados con `assert`.

### Lección 3: Filtrado, Ordenamiento, Agrupación e Indicadores
* **Filtrado y ordenamiento:** Aplicación de máscaras booleanas, `.isin()`, `.between()`, `.query()`, `.sort_values()`, `.nlargest()` y `.nsmallest()` para identificar los 8 campeones dominantes de líneas solitarias (`TOP`/`MID`) y los 15 campeones con alerta de bloqueo crítico (`Ban Rate >= 25%` y `Win Rate >= 51%`).
* **Indicador 1 (Balance Competitivo por Clase y Rol):** Los **Tiradores (`Marksman`)** lideran el Win Rate global (**51.54%**, y **52.14%** en `ADC`); los **Luchadores (`Fighter`)** mantienen $> 50\%$ de Win Rate en los 5 carriles del mapa; los **Asesinos (`Assassin`)** registran la mayor presión de bloqueo (**22.82% Ban Rate**), rindiendo en `JUNGLE` (**51.26%**) pero cayendo en `ADC` (**47.88%**).
* **Indicador 2 (Curva de Efectividad según Dificultad Mecánica):** Mediante `pd.cut` y `.transform('mean')`, se demuestra la *Paradoja de la Dificultad*: los campeones de **Baja Dificultad (1–3)** alcanzan el mayor Win Rate promedio (**51.18%**) frente a los de **Media (50.57%)** y **Alta Dificultad (50.33%)**, aunque los de alta complejidad concentran la mayor selección (**10.92%**) y alto bloqueo (**20.64%**).
* **Indicador 3 (Escalado Temporal según Duración del Encuentro):** Sobre las 497 partidas válidas, en encuentros largos ($> 35$ min, ~18,000 de oro) escalan con fuerza los **Magos (75.00% Win Rate)**, **Tiradores (62.96% Win Rate)** y **Tanques (61.11% Win Rate, donde ocurre el 55.4% de sus partidas)**, mientras que los **Asesinos** alcanzan su máximo ritmo ofensivo en partidas cortas ($< 25$ min) con **1,200.8 DPM**, descendiendo a **834.4 DPM** en juego tardío.

### Lección 4: Combinación y Estructuración de Datasets
* **Integración con `pd.merge()` y validación (`validate='m:1'`, `indicator=True`):** Emparejamiento del **100%** (`both: 96` y `both: 497`) para construir la **Ficha Consolidada de Campeones** (`96 x 31`) y la **Tabla Analítica Maestra de Partidas** (`497 x 40`).
* **Apilado y reestructuración (`pd.concat()` y `.melt()`):** Comparativa de líderes de baja vs. alta dificultad y transformación a formato largo `(18, 3)` exportada a `data/processed/` en formatos **CSV** y **Parquet**.

---

## 6. Instrucciones de Ejecución

1. **Clonar el repositorio:**
   ```bash
   git clone https://github.com/simoncarmona-dotcom/proyecto-ciencia-datos-ev05.git
   cd proyecto-ciencia-datos-ev05
   ```
2. **Instalar las dependencias:**
   ```bash
   pip install -r requirements.txt
   ```
3. **Ejecutar los Notebooks:**
   ```bash
   jupyter notebook
   ```
   - Abrir [`notebooks/data_quality.ipynb`](notebooks/data_quality.ipynb) para consultar el diagnóstico inicial de calidad (EV05).
   - Abrir [`notebooks/data_preparation.ipynb`](notebooks/data_preparation.ipynb) para ejecutar el pipeline completo de limpieza, cálculo de indicadores y estructuración (EV07).
