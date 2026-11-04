# Informe Técnico Individual: Diagnóstico de Calidad de Datos

* **Denominación del Programa de Formación:** Tecnólogo en Análisis y Desarrollo de Software (`228118`)
* **Ficha:** `3223873`
* **Competencia:** `220501094` - Estructurar propuesta técnica de servicio de tecnología de la información según requisitos técnicos y normativa
* **Resultado de Aprendizaje:** `220501094-01` - Definir especificaciones técnicas del software de acuerdo con las características del software a construir
* **Evidencia de Aprendizaje:** `GA1-220501094-01-AA1-EV06` – Diagnosticar la calidad de datos: preparación, exploración y documentación
* **Aprendiz:** Simón Carmona Toro
* **Instructor:** Carlos Enrique Navia Torres

---

## 1. Nombre del proyecto y pregunta analítica

* **Nombre del proyecto:** Ciencia de Datos Aplicada al Balance de Campeones, Tasas de Selección/Bloqueo (*Pick/Ban Rate*) y Escalado Estadístico por Clases en *League of Legends*.
* **Propósito del proyecto:** Evaluar de manera objetiva mediante Python y Pandas el equilibrio competitivo de los campeones de *League of Legends* en función de su clase de rol, su dificultad mecánica (1 a 10) y la duración de las partidas, identificando anomalías de balance (*Buffs* y *Nerfs*) y estructurando las bases de un sistema de recomendación por carril.
* **Pregunta analítica:**
  > *«¿De qué manera la clase de rol de un campeón (Asesino, Mago, Tirador, Tanque, Luchador o Soporte), su índice de dificultad mecánica y la duración total del encuentro influyen en la variación de su tasa de victoria (Win Rate), tasa de selección (Pick Rate) y tasa de bloqueo (Ban Rate) en el metajuego clasificatorio de League of Legends?»*

---

## 2. Descripción de los datasets utilizados

El proyecto integra tres conjuntos de datos en formato CSV almacenados en `data/raw/`:

1. **`lol_champion_meta_stats.csv` (`df_meta`):**
   * **Fuente:** Kaggle Datasets (`vivovinco/league-of-legends-champion-stats`).
   * **Descripción:** Estadísticas consolidadas del metajuego competitivo por campeón y carril (`TOP`, `JUNGLE`, `MID`, `ADC`, `SUPPORT`). Incluye variables categóricas (`Name`, `Class`, `Role`, `Tier`) y métricas de rendimiento (`Score`, `Trend`, `Win %`, `Role %`, `Pick %`, `Ban %`, `KDA`).
   * **Relación con la pregunta analítica:** Suministra las variables dependientes centrales (`Win %`, `Pick %` y `Ban %`) segmentadas por carril y clase.
2. **`lol_champions_base_attributes.csv` (`df_attributes`):**
   * **Fuente:** Riot Games Data Dragon & Kaggle (`gyejr95/league-of-legends-champion-stats`).
   * **Descripción:** Catálogo estático oficial de atributos base de fábrica de los campeones (`id`, `name`, `title`, `tags`, `partype`, `difficulty`, `hp`, `hpperlevel`, `mp`, `mpperlevel`, `armor`, `armorperlevel`, `attackdamage`, `attackdamageperlevel`).
   * **Relación con la pregunta analítica:** Aporta el índice oficial de dificultad mecánica (`difficulty`, escala de 1 a 10) y las estadísticas de crecimiento por nivel (`hpperlevel`, `armorperlevel`, `attackdamageperlevel`).
3. **`lol_ranked_matches_performance.csv` (`df_matches`):**
   * **Fuente:** Google Dataset Search & Kaggle (`paololol/league-of-legends-ranked-matches`).
   * **Descripción:** Registros individuales de desempeño por partida clasificatoria (`match_id`, `champion_name`, `role`, `win`, `kills`, `deaths`, `assists`, `damage_dealt`, `gold_earned`, `duration_sec`, `patch`).
   * **Relación con la pregunta analítica:** Aporta la duración exacta del encuentro en segundos (`duration_sec`) y el daño total infligido (`damage_dealt`) para evaluar el escalado temporal en partidas cortas vs. largas.

---

## 3. Resultados de la inspección estructural

Mediante la ejecución de los comandos `head()`, `tail()`, `shape`, `columns`, `dtypes`, `info()`, `describe()` y `describe(include='object')` en el cuaderno `notebooks/diagnostico_calidad.ipynb`, se obtuvo el siguiente diagnóstico estructural:

| Dataset | Filas | Columnas | Tipos de datos | Observaciones |
| :--- | :---: | :---: | :--- | :--- |
| **`lol_champion_meta_stats.csv`** (`df_meta`) | 100 | 11 | `float64` (3), `object` (8) | Contiene 100 registros correspondientes a 70 campeones únicos (`Name`), ya que algunos campeones se juegan en múltiples carriles. Presenta valores faltantes en `Class` (4), `Tier` (3) y `Trend` (7). Las columnas `Win %`, `Role %`, `Pick %` y `Ban %` están tipificadas como `object`. |
| **`lol_champions_base_attributes.csv`** (`df_attributes`) | 70 | 14 | `int64` (5), `float64` (4), `object` (5) | Contiene 1 fila única por cada uno de los 70 campeones evaluados. La dificultad (`difficulty`) tiene media de `5,44` y mediana de `5,50` (rango `1` a `10`). Presenta 5 valores ausentes en `mp` y `mpperlevel`. |
| **`lol_ranked_matches_performance.csv`** (`df_matches`) | 500 | 11 | `int64` (7), `float64` (2), `object` (2) | Contiene 500 registros de desempeño en partidas clasificatorias (parches `14.19` y `14.20`). Presenta valores faltantes en `role` (9) y `damage_dealt` (7). La duración (`duration_sec`) promedia `1.933,90` s (~32,2 min), con mínimo de `213` s y máximo de `2.578` s. |

---

## 4. Identificación de valores faltantes, tipos de datos y posibles inconsistencias

### 4.1. Valores faltantes (`NaN`)
* En **`df_meta`:**
  * `Class`: **4 valores faltantes** (`96 non-null` de `100`).
  * `Tier`: **3 valores faltantes** (`97 non-null` de `100`).
  * `Trend`: **7 valores faltantes** (`93 non-null` de `100`).
* En **`df_attributes`:**
  * `mp` y `mpperlevel`: **5 valores faltantes** en cada columna (`65 non-null` de `70`), asociados a campeones sin recurso de maná (`Garen`, `Katarina`, `Mordekaiser`, `Riven`, `Yasuo`).
* En **`df_matches`:**
  * `role`: **9 valores faltantes** (`491 non-null` de `500`).
  * `damage_dealt`: **7 valores faltantes** (`493 non-null` de `500`).

### 4.2. Problemas en tipos de datos (`dtypes`)
* **Porcentajes almacenados como texto (`object`) en `df_meta`:** Las variables `Win %`, `Role %`, `Pick %` y `Ban %` incluyen el carácter `'%'` (ej. `'46.80%'`), lo que hace que Pandas las clasifique como `object` y las excluya del resumen estadístico numérico `describe()`.
* **Conversión implícita a `float64` por valores nulos:** En `df_matches`, la columna `damage_dealt` contiene valores enteros pero fue tipificada como `float64` debido a los 7 registros `NaN`.
* **Versión del parche como decimal en `df_matches`:** La variable `patch` se importó como `float64` (`14.19`, `14.20`) en lugar de texto categórico (`object`).

### 4.3. Posibles inconsistencias y limitaciones
* **Doble codificación para ausencia de maná en `df_attributes`:** Algunos campeones sin maná registran `mp = 0.0` (`Aatrox`, `Sett`, `Viego`, `Yone`), mientras que otros registran `NaN` (`Garen`, `Katarina`, `Mordekaiser`, `Riven`, `Yasuo`).
* **Inconsistencia en escritura de nombres entre tablas:** En `df_attributes`, la columna `id` omite espacios y apóstrofos (`Kaisa`, `Chogath`, `Khazix`, `LeeSin`), mientras que `name` sí los incluye (`Kai'Sa`, `Cho'Gath`, `Kha'Zix`, `Lee Sin`). En `df_matches` también aparecen registros sin apóstrofo, lo que causaría pérdida de coincidencias al ejecutar `pd.merge()`.
* **Registros de partidas canceladas (*remakes*) en `df_matches`:** El valor mínimo de `duration_sec` es `213` segundos (3,55 minutos). Las partidas menores a 300 segundos corresponden a *remakes* por desconexión inicial y no representan partidas reales.
* **Relación `1:N` entre atributos y metajuego:** `df_attributes` tiene 70 filas (1 por campeón) y `df_meta` tiene 100 filas (campeones en múltiples carriles), por lo que la llave primaria de `df_meta` es compuesta (`Name` + `Role`).

---

## 5. Principales hallazgos de calidad por dataset

| Dataset | Problema identificado | Evidencia encontrada | Posible impacto | Acción propuesta |
| :--- | :--- | :--- | :--- | :--- |
| **`lol_champion_meta_stats.csv`** | `Win %`, `Role %`, `Pick %` y `Ban %` como tipo `object`. | `df_meta.dtypes` e `info()` reportan las 4 columnas como `object` por el símbolo `'%'`. | Imposibilidad de calcular medias, correlaciones y filtros numéricos sobre las tasas de victoria y bloqueo. | Retirar `'%'` con `.str.replace('%', '')` y convertir a `float64` mediante `.astype(float)`. |
| **`lol_champion_meta_stats.csv`** | Valores ausentes en `Class` (4), `Tier` (3) y `Trend` (7). | `df_meta.info()` muestra `96`, `97` y `93` valores no nulos respectivamente. | Pérdida de 4 registros al agrupar por clase de campeón (`groupby('Class')`). | Imputar `Class` cruzando con `df_attributes['tags']`, `Tier` con la moda (`'A'`) y `Trend` con `0.0`. |
| **`lol_champions_base_attributes.csv`** | Valores ausentes e inconsistencia (`0.0` vs `NaN`) en `mp` y `mpperlevel`. | `df_attributes.info()` reporta `65 non-null` de `70`, y `describe()` reporta `min = 0.0`. | Exclusión de 5 campeones sin maná en análisis numéricos de atributos base. | Reemplazar los valores `NaN` en `mp` y `mpperlevel` por `0.0` usando `.fillna(0.0)`. |
| **`df_attributes` y `df_matches`** | Diferencias en formato de nombres de campeones (`Kai'Sa` vs `Kaisa`). | Comparación entre `id`, `name` y `champion_name` en `describe(include='object')`. | Fallos de unión relacional (`pd.merge`) en campeones con apóstrofos o espacios. | Crear una columna llave normalizada (`.str.lower().str.replace("'", "").str.replace(" ", "")`). |
| **`lol_ranked_matches_performance.csv`** | Faltantes en `role` (9) y `damage_dealt` (7), y partidas *remake* (`duration_sec < 300`). | `df_matches.info()` (`491` y `493 non-null`) y `describe()` (`duration_sec min = 213`). | Distorsión del promedio de daño por minuto y duración de partida por cierres anticipados. | Filtrar `duration_sec >= 300`, imputar `role` con la moda del campeón y `damage_dealt` con la mediana por rol. |

---

## 6. Acciones de limpieza y preparación propuestas

Para la siguiente fase del proyecto (procesamiento y análisis exploratorio), se propone ejecutar el siguiente protocolo en Pandas:

1. **Transformación de tipos de datos:**
   * En `df_meta`, transformar `Win %`, `Role %`, `Pick %` y `Ban %` de `object` a `float64` eliminando el carácter `'%'`.
   * En `df_matches`, convertir `patch` a tipo `object` (`astype(str)`) para tratarlo como versión categórica.
2. **Tratamiento de valores faltantes:**
   * En `df_meta`, recuperar los 4 valores faltantes de `Class` a partir del catálogo oficial `df_attributes['tags']`, imputar `Tier` con la moda y `Trend` con `0.0`.
   * En `df_attributes`, aplicar `.fillna(0.0)` sobre `mp` y `mpperlevel`.
   * En `df_matches`, imputar los 9 registros sin `role` según el rol principal del campeón y los 7 registros sin `damage_dealt` con la mediana del campeón y carril correspondiente.
3. **Normalización de llaves y filtrado de registros inválidos:**
   * Estandarizar los nombres de campeones eliminando espacios y apóstrofos antes de unir las tres tablas.
   * Excluir las partidas con `duration_sec < 300` para eliminar el ruido de los *remakes*.

---

## 7. Conclusiones sobre las condiciones actuales de los datos para continuar con el proyecto

1. **Adecuación funcional completa:** Los tres datasets evaluados cumplen con la cobertura temática y las variables requeridas para responder la pregunta analítica formulada, abarcando las 6 clases de campeones, los 5 carriles del mapa, la dificultad mecánica del 1 al 10 y la duración en segundos de las partidas.
2. **Baja proporción de datos ausentes y alta recuperabilidad:** Los valores faltantes identificados representan fracciones menores (entre el 1,4 % y el 7,1 % según la columna) y pueden corregirse sin pérdida de información aprovechando la complementariedad entre el catálogo primario (`df_attributes`) y las tablas secundarias (`df_meta` y `df_matches`).
3. **Importancia del diagnóstico previo:** Realizar esta auditoría estructural con `info()`, `dtypes` y `describe()` antes de iniciar el modelado permitió detectar que las tasas porcentuales estaban almacenadas como texto (`object`), que existían diferencias de escritura en nombres con apóstrofos y que habían partidas canceladas (`duration_sec < 300`), garantizando que las etapas posteriores de limpieza y visualización produzcan resultados confiables y reproducibles.
