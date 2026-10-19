# Unidad 2_ Adquisision y Normalizacion con Pandas

Consolidación de las ventas de febrero 2026 de una cadena de comida rápida a partir de fuentes en distintos formatos (CSV y Excel), para obtener un dataset limpio, integrado y listo para el análisis. Los datos están anonimizados.

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `Unidad_2__Adquisision_y_Normalizacion_con_Pandas.ipynb` | Notebook con el proceso completo |
| `Ventas_Febrero.csv` | Fuente: transacciones (producto × tienda × unidades) |
| `Maestro de Productos.xlsx` | Fuente: catálogo de productos con precio |
| `Maestro de Tiendas.xlsx` | Fuente: datos de cada tienda |
| `ventas_consolidadas.parquet` | Salida: dataset final |
| `requirements.txt` | Librerías necesarias |

## Requisitos

- Python 3.10 o superior
- Librerías: `pandas`, `openpyxl` (lectura de Excel), `pyarrow` (exportación a Parquet), `matplotlib`

```bash
pip install -r requirements.txt
```

## Cómo reproducir

**Local:**
1. Clonar el repositorio.
2. Instalar las dependencias.
3. Abrir el notebook y ejecutar todas las celdas en orden. Las fuentes se leen con rutas relativas, así que deben estar en la misma carpeta que el notebook.

**Google Colab:**
1. Abrir el notebook en Colab.
2. Subir los tres archivos fuente al almacenamiento de la sesión (panel de archivos, a la izquierda).
3. Ejecutar *Entorno de ejecución → Ejecutar todo*. Colab ya trae instaladas las librerías.

## Proceso

**1. Adquisición.** El CSV fue exportado desde Excel con configuración regional argentina, por lo que se lee con `sep=";"` y `thousands="."`. Sin este último, los valores mayores a 1.000 se interpretaban como decimales.

**2. Limpieza de ventas.**
- Se eliminan las 3 filas del pie del export ("Total", fila vacía, "Filtros aplicados").
- Los IDs y las unidades se convierten a `int64`, tras validar que no hubiera decimales.
- Se eliminan las filas con 0 unidades.
- Las unidades negativas se mantienen como devoluciones o cancelaciones, por lo que el total calculado es **venta neta**.
- Se agrega la columna `fecha` y se convierte a `datetime`.

**3. Limpieza del maestro de productos.**
- Se eliminan los códigos nulos (productos duplicados que ya existen con un código válido) y un producto sin nombre ni ventas.
- Las categorías nulas y los valores `"-"` se imputan como `"SIN CLASIFICAR"`.
- El código pasa a `int64`, el mismo tipo que en ventas, y las columnas de texto a `str`.

**4. Integración.** Merge `left` de ventas con productos y con tiendas por `id_producto` e `id_tienda`, validado como relación muchos a uno. El 100% de las ventas encontró su producto y su tienda.

**5. Normalización.**
- Se excluyen los registros de familia SISTEMAS (separadores, mensajes de cocina, ajustes administrativos), que no son ventas de producto.
- Se crea `total_venta = cantidad × precio_unitario`.

**6. Exportación.** El dataset final (142.922 filas × 14 columnas, sin nulos) se exporta a `ventas_consolidadas.parquet`. Pesa 0,79 MB, contra 2,23 MB del CSV original, que tiene solo 3 columnas.

## Limitaciones

- **Fecha:** el export de origen es mensual, porque el límite de extracción de 150 mil filas impide abrirlo por día. Se asigna `2026-02-01` como fecha representativa del período.
- **Unidades negativas:** se tratan como devoluciones por simplicidad. En la operación real pueden corresponder a contadores o ajustes de sistema.
- **Precio 0:** algunos productos (modificadores, componentes de combos) tienen precio 0 en el maestro, por lo que su `total_venta` es 0.
- **Precio con IVA:** `precio_unitario` incluye IVA.

