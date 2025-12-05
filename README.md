# Análisis de ventas en Python

Este repositorio contiene la entrega de la actividad de análisis y limpieza de datos usando **Python** y **pandas**.  
El objetivo es cargar, inspeccionar, limpiar y transformar tres ficheros CSV (`ventas`, `clientes` y `productos`) y generar nuevos ficheros limpios y listos para análisis.

## Contenido del repositorio

Archivos principales:

- `entrega_extra_ev_Manal_Zahi.ipynb`  
  Notebook donde hago todo el trabajo paso a paso:
  - Carga de los CSV originales.
  - Revisión y ajuste de tipos de datos.
  - Eliminación de duplicados.
  - Relleno de valores faltantes siguiendo las reglas del enunciado.
  - Filtrado de filas según condiciones lógicas.
  - Creación de columnas derivadas.
  - Exportación de los resultados a nuevos CSV.

- `ventas_limpias.csv`  
  DataFrame de ventas limpio, con:
  - Tipos de datos corregidos (fechas como `datetime`, IDs como enteros, etc.).
  - Sin duplicados según la clave (`venta_id`, `cliente_id`, `producto_id`, `fecha_venta`).
  - Valores faltantes tratados:
    - `cantidad` rellenada con 0 si estaba vacía.
    - `precio` rellenado con la media del producto (`groupby("producto_id")`).
    - `categoria` vacía como `"SinCategoria"`.
    - `descuento` rellenado según la categoría:
      - Electrónica → 0.25  
      - Hogar → 0.10  
      - Deporte → 0.20  
      - SinCategoria → 0.00  
    - `fecha_venta` vacía rellenada con la fecha mínima del propio DataFrame.
    - Filas con `cliente_id` vacío eliminadas.
  - Columnas nuevas:
    - `importe_bruto = cantidad * precio`
    - `importe_neto = importe_bruto * (1 - descuento)`

- `clientes_limpios.csv`  
  DataFrame de productos limpio (a partir de `df_productos_new`, se guarda con este nombre porque así lo pide el enunciado), con:
  - `stock` vacío rellenado con 0.
  - `precio` vacío rellenado como `coste * 1.5` (si hiciera falta).
  - `categoria` vacía rellenada como `"SinCategoria"`.
  - Columnas nuevas:
    - `margen = precio - coste`
    - `porcentaje_margen = margen / precio`

- `productos_limpios.csv`  
  DataFrame de clientes limpio (a partir de `df_clientes_new`), con:
  - `edad` vacía rellenada con:
    - Media de edad de la ciudad correspondiente.
    - Si falta ciudad, media global de edad.
    - Después redondeada al entero más cercano.
  - `fecha_registro` vacía rellenada con la fecha mínima del propio DataFrame.
  - `email` vacío rellenado con `"desconocido@example.com"`.
  - `activo` vacío rellenado con 0.
  - Columna nueva:
    - `es_mayor_edad = edad >= 18` (booleano)

## Resumen de la lógica aplicada

### 1. Carga e inspección de datos

- Uso `pd.read_csv` para cargar:
  - `ventas.csv` → `df_ventas`
  - `clientes.csv` → `df_clientes`
  - `productos.csv` → `df_productos`
- Reviso las primeras filas con `head()` y los tipos con `dtypes`.
- Ajusto tipos:
  - Fechas (`fecha_venta`, `fecha_registro`, `fecha_alta`) a `datetime`.
  - `cliente_id` como entero nullable (`Int64`) en ventas.
  - `edad` como float y luego entero en clientes.
  - `activo` como entero nullable.
  - `stock` como numérico.

### 2. Duplicados

- En ventas, considero duplicadas las filas con la misma combinación:
  - `venta_id`, `cliente_id`, `producto_id`, `fecha_venta`
- En clientes, considero duplicadas las filas con:
  - `nombre`, `apellido`, `fecha_registro`
- Aplico `duplicated()` y `drop_duplicates()` para quedarme con la primera aparición.

### 3. Tratamiento de valores faltantes

#### Ventas (`df_ventas_clean`)

- `cantidad` → `fillna(0)`
- `precio` → media por `producto_id`:
  - `groupby("producto_id")["precio"].transform("mean")`
- `categoria` → `"SinCategoria"` si está vacía.
- `descuento`:
  - Relleno según la categoría con un diccionario:
    - Electrónica: 0.25
    - Hogar: 0.10
    - Deporte: 0.20
    - SinCategoria: 0.00
  - Si la categoría es `"SinCategoria"`, fuerzo `descuento = 0.0`.
- `fecha_venta`:
  - Relleno con la fecha mínima del propio DataFrame.
- `cliente_id`:
  - Elimino las filas donde está vacío (`dropna(subset=["cliente_id"])`).

#### Clientes (`df_clientes_clean`)

- `edad`:
  - Calculo la media por ciudad y la media global.
  - Si hay ciudad, uso la media de esa ciudad.
  - Si no hay ciudad, uso la media global.
  - Redondeo al entero más cercano.
- `fecha_registro`:
  - Relleno con la fecha mínima del DataFrame.
- `email`:
  - Relleno con `"desconocido@example.com"`.
- `activo`:
  - Relleno con 0.

#### Productos (`df_productos_clean`)

- `stock`:
  - Relleno con 0.
- `precio`:
  - Si estuviera vacío, lo calculo como `coste * 1.5`.
- `categoria`:
  - Relleno con `"SinCategoria"` si faltara.

### 4. Filtrado

- `df_ventas_filtrado`:
  - `cantidad > 0`
  - `descuento > 0`
  - `categoria != "SinCategoria"`
  - `fecha_venta >= "2024-01-10"`
- `df_clientes_activos`:
  - `activo == 1`
  - `edad >= 30`

### 5. Columnas derivadas

- En ventas (`df_ventas_new`):
  - `importe_bruto = cantidad * precio`
  - `importe_neto = importe_bruto * (1 - descuento)`

- En productos (`df_productos_new`):
  - `margen = precio - coste`
  - `porcentaje_margen = margen / precio`

- En clientes (`df_clientes_new`):
  - `es_mayor_edad = edad >= 18`

## Cómo reproducir el análisis

1. Clonar o descargar este repositorio.
2. Abrir el notebook `entrega_extra_ev_Manal_Zahi.ipynb` en Jupyter, VS Code o similar.
3. Asegurarse de tener instalado:
   - `python`
   - `pandas`
   - `numpy`
4. Ejecutar las celdas en orden.  
   Al final se generarán de nuevo los CSV:
   - `ventas_limpias.csv`
   - `clientes_limpios.csv`
   - `productos_limpios.csv`

---
Este proyecto forma parte de una práctica de análisis y limpieza de datos con Python en el contexto de un máster, enfocada en trabajar solo con **Python básico + pandas** y dejar el código bien comentado y reproducible.
