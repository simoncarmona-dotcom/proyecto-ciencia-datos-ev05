# Proyecto de Ciencia de Datos: Balance de Campeones y Escalado por Clases en League of Legends

* **Programa de Formación:** Tecnólogo en Análisis y Desarrollo de Software (`228118`)
* **Ficha:** `3223873`
* **Evidencia:** `GA1-220501094-01-AA1-EV06` – Diagnosticar la calidad de datos: preparación, exploración y documentación
* **Aprendiz:** Simón Carmona Toro
* **Instructor:** Carlos Enrique Navia Torres

---

## 1. Descripción y propósito del proyecto

Este proyecto de ciencia de datos tiene como propósito evaluar el equilibrio competitivo de los campeones de **League of Legends** a través de sus seis clases principales (*Fighter*, *Mage*, *Assassin*, *Marksman*, *Tank* y *Support*) y cinco carriles del mapa (*TOP*, *JUNGLE*, *MID*, *ADC* y *SUPPORT*).

En esta fase se realiza el **diagnóstico inicial de calidad de los datos** utilizando **Python** y **Pandas**, examinando la estructura de los conjuntos de datos seleccionados, los tipos de variables, la presencia de valores faltantes y las posibles inconsistencias antes de proceder con las etapas de limpieza y análisis.

---

## 2. Pregunta analítica

> *«¿De qué manera la clase de rol de un campeón (Asesino, Mago, Tirador, Tanque, Luchador o Soporte), su índice de dificultad mecánica y la duración total del encuentro influyen en la variación de su tasa de victoria (Win Rate), tasa de selección (Pick Rate) y tasa de bloqueo (Ban Rate) en el metajuego clasificatorio de League of Legends?»*

---

## 3. Datasets utilizados

Los conjuntos de datos crudos se encuentran en la carpeta `data/raw/`:

| Archivo | Dimensiones | Descripción | Origen / Fuente |
| :--- | :---: | :--- | :--- |
| `data/raw/lol_champion_meta_stats.csv` | `100 × 11` | Estadísticas de rendimiento del metajuego por campeón y carril (`Name`, `Class`, `Role`, `Tier`, `Score`, `Trend`, `Win %`, `Role %`, `Pick %`, `Ban %`, `KDA`). | Kaggle Datasets (`vivovinco/league-of-legends-champion-stats`) |
| `data/raw/lol_champions_base_attributes.csv` | `70 × 14` | Atributos base de fábrica, clases (`tags`), dificultad mecánica (`1` a `10`) y crecimiento de estadísticas por nivel (`hp`, `mp`, `armor`, `attackdamage`). | Riot Games Data Dragon & Kaggle (`gyejr95/league-of-legends-champion-stats`) |
| `data/raw/lol_ranked_matches_performance.csv` | `500 × 11` | Desempeño individual por partida clasificatoria, daño infligido (`damage_dealt`), oro (`gold_earned`) y duración del encuentro en segundos (`duration_sec`). | Google Dataset Search & Kaggle (`paololol/league-of-legends-ranked-matches`) |

---

## 4. Estructura del repositorio

```text
proyecto-ciencia-datos/
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   └── raw/
│       ├── lol_champion_meta_stats.csv
│       ├── lol_champions_base_attributes.csv
│       └── lol_ranked_matches_performance.csv
├── notebooks/
│   └── diagnostico_calidad.ipynb
└── reports/
    └── informe_diagnostico.md
```

* **`notebooks/diagnostico_calidad.ipynb`:** Cuaderno de Jupyter ejecutable con la carga de los tres datasets y la inspección estructural y de calidad mediante `head()`, `tail()`, `shape`, `columns`, `dtypes`, `info()`, `describe()` y `describe(include='object')`.
* **`reports/informe_diagnostico.md`:** Informe técnico individual con los resultados de la inspección estructural, identificación de valores faltantes, tipos de datos, posibles inconsistencias, acciones de limpieza propuestas y conclusiones.

---

## 5. Instrucciones de instalación y ejecución

1. **Clonar el repositorio:**
   ```bash
   git clone https://github.com/simoncarmona-dotcom/proyecto-ciencia-datos-ev05.git
   cd proyecto-ciencia-datos-ev05
   ```

2. **Crear y activar un entorno virtual (opcional pero recomendado):**
   ```bash
   python -m venv .venv
   # En Windows:
   .venv\Scripts\activate
   ```

3. **Instalar las dependencias necesarias:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Ejecutar el Notebook de diagnóstico:**
   ```bash
   jupyter notebook notebooks/diagnostico_calidad.ipynb
   ```
   También puede abrirse directamente en **Visual Studio Code** o **Google Colab**.
