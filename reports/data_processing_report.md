# Informe de Procesamiento, Limpieza y Estructuración de Datos con Pandas

**Evidencia de Aprendizaje:** `GA1-220501094-01-AA1-EV06 – Limpieza, análisis y estructuración de datos con Pandas (Instacart)`  
**Programa:** Tecnólogo en Análisis y Desarrollo de Software (ADSO)  
**Autor:** Simón Carmona  
**Fecha:** Noviembre de 2026  
**Repositorio GitHub:** [proyecto-ciencia-datos-ev05 (Rama ev06)](https://github.com/simoncarmona-dotcom/proyecto-ciencia-datos-ev05.git)  

---

## 1. Introducción y Objetivos

El presente informe documenta el desarrollo técnico de la evidencia **EV06**, en la cual se aplica el ciclo completo de ingeniería de datos sobre los conjuntos de datos del caso de estudio **Instacart Market Basket Analysis** utilizando la biblioteca **Pandas** en **Python 3.12**.

El proyecto resuelve la pregunta orientadora planteada en la guía formativa del SENA:
> *¿Cómo preparar y estructurar los datos de Instacart para obtener información confiable sobre los hábitos de compra?*

Para ello, se ejecutaron las tres fases estipuladas en el material formativo:
1. **Lección 2:** Limpieza y estandarización de datos crudos mediante funciones modulares y un pipeline con informe de cambios cuantificable.
2. **Lección 3:** Filtrado, ordenamiento y agregación para la perfilación de clientes y hábitos de compra.
3. **Lección 4:** Estructuración y combinación de múltiples datasets relacionales en una tabla analítica maestra y una ficha por producto, persistiendo los resultados en formatos optimizados (`Parquet` y `CSV`).

---

## 2. Lección 2: Limpieza y Estandarización de Datos Crudos

### 2.1 Pipeline Modular de Limpieza y Función `anotar`
Se diseñó un pipeline automatizado basado en el método `.pipe()` de Pandas, donde cada transformación está encapsulada en una función pura que recibe y retorna un `DataFrame`. Intercalada entre cada etapa, la función `anotar` capturó métricas del estado del dataset (filas, nulos totales y memoria consumida en megabytes).

```python
def anotar(df, tabla, paso):
    registro.append({
        'tabla': tabla,
        'paso': paso,
        'filas': len(df),
        'nulos': int(df.isna().sum().sum()),
        'memoria_mb': round(df.memory_usage(deep=True).sum() / 1e6, 1),
    })
    return df
```

### 2.2 Informe Cuantitativo de Cambios
El pipeline arrojó las siguientes variaciones entre el estado crudo y limpio:

| Tabla | Paso del Pipeline | Filas | Valores Nulos | Memoria (MB) | Variación e Impacto |
|---|---|---|---|---|---|
| **orders** | `0 crudo` | 478,967 | 28,819 | 23.0 MB | Estado de ingesta inicial. |
| **orders** | `1 sin duplicados` | 478,952 | 28,817 | 23.0 MB | Se eliminaron 15 pedidos duplicados exactos (2 de ellos eran primeros pedidos). |
| **orders** | `2 tipos corregidos` | 478,952 | 28,817 | 7.2 MB | **Ahorro del 68.7% en memoria** mediante casting a `int32`, `int16`, `int8` e `Int16`. |
| **products** | `0 crudo` | 49,694 | 1,258 | 3.1 MB | 1,258 productos sin nombre (`product_name` nulo). |
| **products** | `1 nombres limpios` | 49,694 | 0 | 3.1 MB | Nulos imputados con `'Unknown'` y texto formateado a `Title Case`. |
| **order_prods**| `0 crudo` | 4,545,007 | 836 | 145.4 MB | Estado de ingesta inicial. |
| **order_prods**| `1 tipos corregidos` | 4,545,007 | 836 | 54.5 MB | **Ahorro del 62.5% en memoria** (`int32`, `int8`, `Int16`). Nulos preservados deliberadamente. |

### 2.3 Solución y Conclusiones del Reto 2
* **Ejercicio 2.1:** La tabla `order_products` no posee filas duplicadas exactas ni repetición de claves compuestas (`order_id + product_id`). La columna `reordered` contiene exclusivamente valores binarios `0` y `1`.
* **Ejercicio 2.2:** Los catálogos `aisles` (134 registros) y `departments` (21 registros) no tienen duplicados ni espacios en blanco iniciales o finales, conservando integridad estructural limpia.
* **Ejercicio 2.3:** Los 836 valores nulos en `add_to_cart_order` se concentran en 70 pedidos. Cada uno de ellos contiene entre 65 y 127 artículos (promedio de 77 artículos). Esto evidencia una restricción en la base de datos origen que solo registraba hasta la posición 64 del carrito.
* **Ejercicio 2.4:** La estandarización de nombres en `products` corrigió 6,321 registros que presentaban caracteres invisibles (`\xa0`), espacios múltiples o inconsistencias de mayúsculas.
* **Ejercicio 2.5:** La conversión de los nombres de los días a tipo `category` ordenado redujo el consumo en memoria de 7.01 MB a 0.48 MB, logrando un **ahorro de memoria del 93.2%**.
* **Ejercicio 2.6:** Los productos `'Unknown'` aparecen 9,547 veces en 8,254 pedidos distintos. **No deben eliminarse**, pues corresponden a compras efectivamente pagadas y eliminarían artículos reales de la cesta.
* **Ejercicio 2.7:** Reemplazar `'missing'` por `pd.NA` en departamentos generaría que `groupby` excluya dichas ventas por defecto (`dropna=True`), invisibilizando esas transacciones en los reportes de ventas.

---

## 3. Lección 3: Filtrado, Ordenamiento y Agrupación de Datos

### 3.1 Segmentación de Clientes y Hábitos de Compra
A partir de la frecuencia de pedidos de los 157,437 usuarios únicos de `orders`, se construyeron tres segmentos mediante `pd.cut`:
- **Ocasional (1 pedido):** 55,357 clientes (35.2%), con una mediana de intervalo de recompra de 16 días.
- **Regular (2 a 4 pedidos):** 71,553 clientes (45.4%), con una mediana de intervalo de recompra de 11 días.
- **Frecuente (≥ 5 pedidos):** 30,527 clientes (19.4%), que generan el 48.4% de todos los pedidos (231,946 transacciones) con una mediana de recompra de apenas 6 días.

### 3.2 Análisis Horario por Franja
Al cruzar los segmentos con las franjas horarias mediante `pd.crosstab`:
- **Tarde (12:00 a 17:59):** Concentra el 48.4% - 49.4% de los pedidos en todos los segmentos.
- **Mañana (06:00 a 11:59):** Concentra el 35.2% de las compras en clientes frecuentes, frente al 30.0% en clientes ocasionales.

### 3.3 Solución y Conclusiones del Reto 3
* **Ejercicio 3.1:** Las tres horas pico los domingos son las 15:00 (7,585 pedidos), 14:00 (7,533 pedidos) y 13:00 (7,483 pedidos).
* **Ejercicio 3.2:** La mayor espera media entre pedidos ocurre a las 00:00 h (medianoche, 12.52 días) y la menor ocurre a las 22:00 h (10.62 días).
* **Ejercicio 3.3:** Un total de 6,327 usuarios tienen 10 pedidos o más (4.02% del total de clientes).
* **Ejercicio 3.4:** 44,577 pedidos se realizaron exactamente a los 7 días del anterior, representando el **9.31% del total** y confirmando el hábito de abastecimiento semanal.
* **Ejercicio 3.5:** 49,420 pedidos contienen 20 o más productos. El pedido récord corresponde a `order_id = 61355` con **127 productos**.
* **Ejercicio 3.6:** La tasa de recompra es monotónicamente decreciente con la posición en el carrito:
  - Posición 1: **67.86%**
  - Posición 2: **67.71%**
  - Posición 3: **65.90%**
  - Posición 4: **63.75%**
  - Posición 5: **61.75%**  
  *Conclusión:* Los productos prioritarios colocados primero en el carrito son artículos habituales y recurrentes.
* **Ejercicio 3.7:** La mediana de espera de pedidos en fin de semana es de 8.0 días, frente a 7.0 días en días laborables.

---

## 4. Lección 4: Combinación y Estructuración de Múltiples Datasets

### 4.1 Construcción de la Tabla Analítica Maestra
Se integraron las 5 tablas sobre la tabla de hechos `order_products` empleando uniones de tipo `left` con validación estricta de cardinalidad `m:1`:

$$\text{maestra} = \text{order\_products} \bowtie_{m:1} \text{products} \bowtie_{m:1} \text{aisles} \bowtie_{m:1} \text{departments} \bowtie_{m:1} \text{orders}$$

Se verificó mediante aserción que la tabla maestra conserve exactamente las **4,545,007 filas** originales sin duplicación ni pérdida accidental de registros, alcanzando 14 columnas analíticas consolidadas.

### 4.2 Ficha Analítica por Producto y Ranking Departamental
Agregando la tabla maestra a nivel de producto (`product_id`), se derivó la ficha con métricas de volumen, clientes únicos, tasa de recompra y posición media en el carrito.
Filtrando los 739 productos con alta significancia estadística ($\ge 1,000$ pedidos) y clasificándolos con `.rank(method='first')`, se identificaron los productos con mayor fidelidad de cada departamento:
- **dairy eggs:** *Whole Organic Omega 3 Milk* (tasa de recompra: **86.31%**)
- **produce:** *Banana* (66,050 pedidos, tasa de recompra: **84.43%**)
- **snacks:** *Extra Fancy Unsalted Mixed Nuts* (tasa de recompra: **82.24%**)
- **beverages:** *Zero Calorie Cola* (tasa de recompra: **81.85%**)
- **bakery:** *Organic Honey Sweet Whole Wheat Bread* (tasa de recompra: **79.90%**)

### 4.3 Solución y Conclusiones del Reto 4
* **Ejercicio 4.1:** Los tres pasillos con más de 10,000 artículos y mayor tasa de recompra son `milk` (78.13%), `water seltzer sparkling water` (73.06%) y `fresh fruits` (71.82%). El pasillo con menor tasa es `spices seasonings` (15.24%).
* **Ejercicio 4.2:** En la madrugada (0 a 5 h), el departamento de `beverages` entra al top 3 con 8.49% de participación, desplazando a `snacks` (que ocupa el top 3 en el total global con 8.91%).
* **Ejercicio 4.3:** Los tres snacks más solicitados son `Lightly Salted Baked Snap Pea Crisps` (3,462 pedidos), `Original Veggie Straws` (2,331 pedidos) y `Sea Salt Pita Chips` (2,275 pedidos).
* **Ejercicio 4.4:** La tabla resumen por usuario cuenta con 149,626 clientes. El promedio es de 30.38 artículos por usuario. El comprador con mayor volumen es `user_id = 103634` con 580 artículos.
* **Ejercicio 4.5:** La unión interna `inner` entre `orders` y los pedidos de `order_products` devuelve 450,046 filas, omitiendo **28,906 pedidos (6.03%)** que no tienen artículos asociados en la base de datos.
* **Ejercicio 4.6:** Se combinaron verticalmente con `pd.concat` los 3 productos top de frutas/verduras (`Banana`, `Bag Of Organic Bananas`, `Organic Strawberries`) y los 3 de lácteos (`Organic Whole Milk`, `Organic Half & Half`, `Half & Half`).
* **Ejercicio 4.7:** La tabla dinámica departamento $\times$ día tiene dimensiones `(21, 7)`. El día con mayor volumen de compra de alcohol es el **Viernes (día 5)** con 3,814 artículos.

---

## 5. Artefactos Generados en `data/processed/`

Los conjuntos de datos procesados fueron generados y almacenados con éxito:

1. `cleaned_instacart_orders.csv` (19.36 MB): Pedidos sin duplicados, con tipado optimizado, segmentos de clientes y franjas horarias.
2. `cleaned_products.csv` (2.18 MB): Catálogo de productos con nombres estandarizados y sin valores nulos.
3. `ficha_productos.parquet` (1.69 MB) y `ficha_productos.csv` (3.51 MB): Métricas consolidadas por producto (volumen, clientes, tasa de recompra, posición media).
4. `lideres_por_departamento.csv`: Productos con mayor tasa de recompra por cada departamento.
5. `resumen_departamentos.csv`: Resumen global de artículos y tasa de recompra por categoría departamental.
6. `perfil_habitos_compra.csv`: Distribución de clientes, pedidos y mediana de espera por segmento.
7. `informe_limpieza.csv`: Auditoría cuantitativa del pipeline de limpieza.
