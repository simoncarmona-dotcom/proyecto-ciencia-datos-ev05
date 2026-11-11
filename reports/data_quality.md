# Informe de Calidad de Datos - Instacart Market Basket Analysis

**Evidencia de Aprendizaje:** `GA1-220501094-01-AA1-EV04 / EV06`  
**Dataset:** Instacart Market Basket Analysis (5 datasets relacionales)  
**Entorno:** Python 3.12, Pandas  

---

## 1. Resumen Ejecutivo del Diagnóstico de Calidad

Durante la fase de exploración y diagnóstico de calidad de datos de los 5 archivos CSV de Instacart (`instacart_orders.csv`, `products.csv`, `order_products.csv`, `aisles.csv`, `departments.csv`), se identificaron las siguientes características y anomalías estructurales:

| Dataset | Registros | Columnas | Duplicados Exactos | Valores Faltantes (Nulos) | Columnas Afectadas |
|---|---|---|---|---|---|
| `instacart_orders.csv` | 478,967 | 6 | 15 filas | 28,819 nulos | `days_since_prior_order` |
| `products.csv` | 49,694 | 4 | 0 filas | 1,258 nulos | `product_name` |
| `order_products.csv` | 4,545,007 | 4 | 0 filas | 836 nulos | `add_to_cart_order` |
| `aisles.csv` | 134 | 2 | 0 filas | 0 nulos | Ninguna |
| `departments.csv` | 21 | 2 | 0 filas | 0 nulos | Ninguna |

---

## 2. Diagnóstico Detallado de Anomalías

### 2.1 Duplicados en `instacart_orders.csv`
- Se identificaron **15 registros completamente duplicados**.
- Al analizar los identificadores `order_id`, se observó que provienen de errores de ingesta o doble envío de eventos.
- **Acción requerida:** Eliminación de los 15 duplicados mediante `drop_duplicates().reset_index(drop=True)`, reduciendo el dataset a **478,952 filas únicas**.

### 2.2 Valores Faltantes en `days_since_prior_order`
- Se detectaron 28,819 nulos en la versión cruda (28,817 tras remover los duplicados).
- **Causa raíz:** Cada valor nulo coincide exactamente con `order_number == 1`. Es decir, representan la primera compra histórica del cliente, por lo que no existe un pedido previo.
- **Estrategia adoptada:** Preservar los valores nulos como tipo entero nullable (`Int16`) o mantenerlos sin imputar a cero arbitrariamente, ya que un cero indicaría compra el mismo día.

### 2.3 Valores Faltantes en `products.csv`
- 1,258 productos carecen de nombre (`product_name` es NaN).
- Todos pertenecen al pasillo `aisle_id = 100` (`missing`) y al departamento `department_id = 21` (`missing`).
- **Estrategia adoptada:** Imputar el nombre con el valor representativo `'Unknown'` y estandarizar el texto con formato tipo título (`.str.title()`).

### 2.4 Valores Faltantes en `order_products.csv`
- 836 registros presentan nulo en `add_to_cart_order`.
- Estos nulos se concentran en 70 pedidos que contienen 65 o más artículos (hasta 127 artículos).
- **Causa raíz:** Truncamiento técnico en el sistema transaccional para carritos que superaron 64 elementos.
- **Estrategia adoptada:** Mantener los registros en la tabla usando tipo `Int16`, ya que representan transacciones reales y productos efectivamente adquiridos.

---

## 3. Estado de Partida para EV06
Este informe y el notebook `data_quality.ipynb` establecen la línea base para el desarrollo del pipeline automatizado de limpieza, análisis de hábitos y estructuración de la tabla analítica maestra en `pandas_labs.ipynb`.
