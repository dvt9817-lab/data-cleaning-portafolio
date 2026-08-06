# 🛒 Supermarket Sales — Data Cleaning with Python

Proyecto de limpieza y preparación de datos usando Python (Pandas) en Google Colab.

---

## 📋 Descripción

Este proyecto toma un dataset de ventas de supermercado con nombres de columnas confusos, tipos de dato incorrectos y sin variables derivadas, y lo transforma en un dataset limpio y estructurado listo para análisis en Power BI.

---

## 🗂️ Dataset

- **Registros:** 1,000 transacciones
- **Columnas originales:** 12
- **Fuente:** Caso práctico de análisis de datos
- **Contexto:** Ventas de una cadena de supermercados con 3 sucursales en Nueva York, Houston y Chicago

---

## 🛠️ Herramientas

| Herramienta | Uso |
|---|---|
| Python (Pandas) | Limpieza y transformación |
| Google Colab | Entorno de ejecución |
| Power BI | Análisis y visualización |

---

## 🧹 Proceso de limpieza

### Problemas encontrados
- Nombres de columnas con códigos sin sentido (`city-ghh-998`, `type-usr-search`, etc.)
- Columnas `date` y `time` en formato string
- Sin variables derivadas para el análisis

### Transformaciones aplicadas

**1. Renombrado de columnas** — estandarización a snake_case descriptivo

| Antes | Después |
|---|---|
| `plk-invoice-number` | `invoice_id` |
| `xyz-branch` | `branch` |
| `city-ghh-998` | `city` |
| `type-usr-search` | `customer_type` |
| `category-catalog-dsp` | `category` |
| `cost of goods sold` | `cogs` |
| `dateddmmyyy` | `date` |
| `payment-type-full` | `payment_method` |

**2. Corrección de tipos de dato**
- `date`: string → datetime
- `time`: string → time

**3. Creación de columnas derivadas**
- `gross_margin` — margen bruto (sale - cogs)
- `month` — mes extraído de la fecha
- `day_of_week` — día de la semana
- `hour` — hora de la transacción

---

## ✅ Resultado

| | Antes | Después |
|---|---|---|
| Columnas | 12 | 16 |
| Nombres de columnas | Confusos | Claros en snake_case |
| Tipo de `date` | String | Datetime |
| Tipo de `time` | String | Time |
| Valores nulos | 0 | 0 |
| Columnas derivadas | 0 | 4 |

---

## 📁 Archivos

| Archivo | Descripción |
|---|---|
| `supermarket_limpieza.ipynb` | Notebook con el proceso de limpieza documentado |
| `Supermarket sales.csv` | Dataset original sin la limpieza |
| `supermarket_clean.csv` | Dataset limpio listo para análisis |


---

## 📊 Próximos pasos

El dataset limpio fue cargado en Power BI para construir un dashboard con análisis de ventas, márgenes, satisfacción del cliente y tendencias temporales.

---
