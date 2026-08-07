# 🚑 Llamadas de Urgencias y Emergencias — Línea 123 (Bogotá)

**Autor:** Daniela Vargas

**Herramientas:** Python (Pandas) en Google Colab

**Fuente:** [Datos Abiertos Bogotá — Secretaría Distrital de Salud](https://datosabiertos.bogota.gov.co/dataset/llamadas-de-urgencias-y-emergencias-que-ingresan-a-traves-de-la-linea-123)

**Periodo:** Enero – Junio 2026 (6 archivos mensuales combinados, ~56,000 registros)

## 🎯 Objetivo

Descargar, combinar y limpiar los registros mensuales de llamadas de urgencias y emergencias que ingresan a la línea 123 de Bogotá, dejando el dataset listo para análisis y visualización.

## 📥 Obtención de los datos

El portal (CKAN) publica un archivo CSV por mes con URL de descarga independiente. Los datos se descargaron y combinaron automáticamente con un script en Python (`requests` + `pandas`), en vez de descargarlos manualmente mes por mes.

## 🔎 Problemas de calidad detectados y decisiones tomadas

| Problema | Decisión |
|---|---|
| **Separador incorrecto**: los archivos usan `;` en vez de `,` | Se especificó `sep=';'` al leer el CSV |
| **Encoding incorrecto**: el archivo está en `cp850` (no `utf-8` ni `latin-1`), causando caracteres corruptos en tildes y ñ | Se identificó el encoding correcto comparando los patrones de corrupción de caracteres, y se decodificó explícitamente como `cp850` |
| **`LOCALIDAD` y `TIPO_INCIDENTE` duplicados por inconsistencia de tildes/encoding** (ej. `ENGATIVA` vs `ENGATIVÁ`) | Se unificaron a un solo valor estándar por categoría |
| **`PRIORIDAD_FINAL` en marzo 2026 viene codificada con números (1-4) en vez de texto**, sin diccionario oficial de equivalencia en los metadatos del portal | **No se infirió el mapeo.** Aunque el análisis de proporciones sugería una posible equivalencia, no había forma de confirmarla con la fuente oficial, así que esas filas se marcaron como `"No verificable"` y se excluyen de cualquier análisis por prioridad — aunque se conservan para conteos generales (por localidad, tipo de incidente, etc.) |
| **`GENERO`, `UNIDAD` y `EDAD` nulos en 19,012 filas** | Se investigó el patrón: el 100% de estos casos también carecen de `RECEPCION`, indicando que son llamadas que no llegaron a completarse (no se registraron datos del paciente). Se marcaron `GENERO` y `UNIDAD` como `"No registrado"`; `EDAD` y `RECEPCION` se dejaron como nulos por ser campos numéricos/fecha |
| **Fechas mezclaban dos formatos** (ISO `2026-01-01 19:25:31` y colombiano `3/04/2026 8:46`) según el mes de origen | Se usó `pd.to_datetime` con detección automática de formato por fila (`format='mixed'`) |
| **277 filas duplicadas** | Eliminadas |

## 📁 Archivos en esta carpeta

- `data/raw/llamadas123_2026_enero_junio_raw.csv` — datos originales combinados, sin modificar
- `data/processed/llamadas123_2026_enero_junio_clean.csv` — dataset limpio, listo para análisis (55,795 filas)
- `notebooks/Llamadas_123.ipynb` — notebook con el proceso completo de descarga y limpieza

## 📊 Resultado

Dataset final: **55,795 filas × 11 columnas**, listo para análisis de volumen de llamadas por localidad, tipo de incidente y prioridad (excluyendo marzo 2026 en este último caso).
