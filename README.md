# Proyecto de Ciencia de Datos: Limpieza, Análisis y Estructuración con Pandas (Instacart)

* **Programa de Formación:** Tecnólogo en Análisis y Desarrollo de Software (ADSO) - Ficha 3223873
* **Evidencia:** `GA1-220501094-01-AA1-EV06 – Limpieza, análisis y estructuración de datos con Pandas (instacart)`
* **Aprendiz:** Simón Carmona Toro
* **Instructor:** Carlos Enrique Navia Torres
* **Repositorio:** [proyecto-ciencia-datos-ev05](https://github.com/simoncarmona-dotcom/proyecto-ciencia-datos-ev05.git)
* **Rama de Trabajo:** `ev06`

---

## 1. Descripción del Proyecto

Este repositorio contiene la solución completa de la evidencia formativa **GA1-220501094-01-AA1-EV06**, en la cual se prepara y estructura el conjunto de datos de **Instacart Market Basket Analysis** mediante la biblioteca **Pandas** en Python 3.12.

Se abordan de forma rigurosa las lecciones 2, 3 y 4 del material formativo *Preparar y estructurar datos con Python*, desarrollando los **3 laboratorios guiados** y los **21 ejercicios de retos prácticos** (7 por cada lección), junto con la generación y almacenamiento de datasets procesados y la documentación analítica.

---

## 2. Pregunta Orientadora

> *¿Cómo preparar y estructurar los datos de Instacart para obtener información confiable sobre los hábitos de compra?*

---

## 3. Estructura del Repositorio

El proyecto conserva la organización estándar de proyectos de ciencia de datos solicitada en la guía:

```text
proyecto-ciencia-datos/
├── README.md                          # Instrucciones, contexto y resultados del proyecto
├── .gitignore                         # Exclusiones de Git (entornos virtuales, caches)
├── requirements.txt                   # Dependencias requeridas (pandas, numpy, pyarrow, jupyter)
│
├── data/
│   ├── raw/                           # Datasets originales sin modificar
│   │   ├── instacart_orders.csv       # 478,967 registros históricos de pedidos
│   │   ├── order_products.csv         # 4,545,007 transacciones de artículos en carritos
│   │   ├── products.csv               # 49,694 productos del catálogo general
│   │   ├── aisles.csv                 # 134 pasillos de la tienda
│   │   └── departments.csv            # 21 departamentos o categorías
│   │
│   └── processed/                     # Datasets procesados y optimizados generados
│       ├── cleaned_instacart_orders.csv   # Pedidos limpios con segmentos y franjas
│       ├── cleaned_products.csv           # Catálogo sin nulos y con nombres estandarizados
│       ├── ficha_productos.parquet        # Métricas agregadas por producto (Parquet)
│       ├── ficha_productos.csv            # Métricas agregadas por producto (CSV)
│       ├── lideres_por_departamento.csv   # Producto con mayor fidelidad por departamento
│       ├── resumen_departamentos.csv      # Volúmenes y tasa de recompra departamental
│       ├── perfil_habitos_compra.csv      # Distribución de clientes y compras por segmento
│       └── informe_limpieza.csv           # Auditoría cuantitativa del pipeline de limpieza
│
├── notebooks/
│   ├── data_quality.ipynb             # Exploración inicial y diagnóstico de calidad
│   └── pandas_labs.ipynb              # Desarrollo secuencial de Lecciones 2, 3 y 4 (Labs y Retos)
│
└── reports/
    ├── data_quality.md                # Informe diagnóstico de calidad inicial
    └── data_processing_report.md      # Informe completo de limpieza, hábitos y tabla maestra
```

---

## 4. Contenido Técnico Desarrollado

### Lección 2: Limpieza y Estandarización
- **Laboratorio Guiado 2:** Construcción de un pipeline modular encadenado con `.pipe()` y monitoreado paso a paso con la función `anotar()`.
  - Eliminación de 15 pedidos duplicados exactos.
  - Corrección de tipos a enteros pequeños (`int32`, `int16`, `int8`, `Int16`), logrando un **ahorro de memoria del 68.7% en `orders` y 62.5% en `order_products`**.
  - Estandarización de 6,321 nombres de productos en `products.csv` e imputación de 1,258 nulos como `'Unknown'`.
  - Validación automatizada mediante aserciones (`assert`).
- **Reto 2 (7 ejercicios resueltos):**
  1. Integridad referencial y unicidad en `order_products`.
  2. Verificación de limpieza en catálogos `aisles` y `departments`.
  3. Causa raíz de los 836 nulos de `add_to_cart_order` (truncamiento en carritos $\ge 65$ ítems).
  4. Conteo y muestra de nombres estandarizados.
  5. Series categórica de días de la semana ordenada con **93.2% de reducción de memoria**.
  6. Justificación de preservación de transacciones con producto `'Unknown'` (9,547 filas en 8,254 pedidos).
  7. Análisis de la implicación de nulos reales en categorías antes de agrupar.

### Lección 3: Filtrado, Ordenamiento y Agrupación
- **Laboratorio Guiado 3:** Segmentación de clientes (`ocasional`, `regular`, `frecuente`) mediante `pd.cut` y cruce multivariable con franjas horarias (`madrugada`, `mañana`, `tarde`, `noche`) usando `pd.crosstab`.
- **Reto 3 (7 ejercicios resueltos):**
  1. Horas pico dominicales: 15:00 (7,585), 14:00 (7,533) y 13:00 (7,483).
  2. Extremos de espera: medianoche (12.52 días) vs 22:00 h (10.62 días).
  3. Identificación de 6,327 compradores de alta recurrencia ($\ge 10$ pedidos).
  4. Detección de 44,577 pedidos con ciclo semanal exacto de 7 días (9.31% del total).
  5. Identificación de 49,420 pedidos grandes ($\ge 20$ productos) y pedido récord de 127 artículos.
  6. Análisis de la tasa de recompra decreciente por posición en el carrito (de 67.86% en pos 1 a 61.75% en pos 5).
  7. Mediana de espera en fin de semana (8.0 días) vs semana (7.0 días).

### Lección 4: Combinación y Estructuración de Datasets
- **Laboratorio Guiado 4:**
  - Construcción de la tabla analítica maestra sobre `order_products` con validación `m:1` y verificación de integridad (`assert len(maestra) == 4,545,007`).
  - Ficha descriptiva por producto (`ficha_productos`), filtrado de significancia estadística ($\ge 1,000$ pedidos) y clasificación con `.rank()`.
  - Exportación en formato binario columnar `Parquet` y tabular `CSV`.
- **Reto 4 (7 ejercicios resueltos):**
  1. Pasillos líderes en recompra: `milk` (78.13%), `water seltzer sparkling water` (73.06%) y `fresh fruits` (71.82%); pasillo menor: `spices seasonings` (15.24%).
  2. Variación de madrugada: `beverages` entra al top 3 en la madrugada (8.49%), desplazando a `snacks`.
  3. Top 3 snacks más vendidos: `Lightly Salted Baked Snap Pea Crisps`, `Original Veggie Straws`, `Sea Salt Pita Chips`.
  4. Tabla analítica de 149,626 clientes (promedio: 30.38 artículos, máximo: 580 artículos).
  5. Análisis del impacto de unión `inner` vs `left` (identificación de 28,906 pedidos sin productos registrados).
  6. Apilado horizontal con `pd.concat` de líderes en `produce` y `dairy eggs`.
  7. Tabla dinámica $(21 \times 7)$ departamental: el **Viernes (día 5)** lidera compras de alcohol (3,814 artículos).

---

## 5. Instrucciones de Ejecución

### Prerrequisitos
Tener instalado Python 3.10 o superior y un gestor de paquetes `pip`.

### 1. Clonar el repositorio y ubicarse en la rama `ev06`:
```bash
git clone https://github.com/simoncarmona-dotcom/proyecto-ciencia-datos-ev05.git
cd proyecto-ciencia-datos-ev05
git checkout ev06
```

### 2. Instalar dependencias:
```bash
pip install -r requirements.txt
```

### 3. Ejecutar los Notebooks:
Iniciar el entorno de Jupyter:
```bash
jupyter notebook
```
Abrir y ejecutar:
- `notebooks/data_quality.ipynb`: para revisar la exploración y calidad base.
- `notebooks/pandas_labs.ipynb`: para ejecutar el pipeline de limpieza, análisis y estructuración.

---

## 6. Control de Versiones en GitHub

Siguiendo la metodología estipulada en la guía de aprendizaje:
* **Rama creada:** `ev06` (y `feature/pandas-data-processing`).
* **Mensajes de confirmación (commits):**
  - `feat: complete lesson 2 data cleaning`
  - `feat: complete lesson 3 data analysis`
  - `feat: complete lesson 4 data merging`
  - `docs: update README and processing report`
