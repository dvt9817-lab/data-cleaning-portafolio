# 💧 Embalses y generación eléctrica en Colombia durante El Niño

Extracción, limpieza y preparación de datos abiertos de XM (operador del mercado eléctrico colombiano) con Python en Google Colab, para comparar los episodios de El Niño de 2015-16, 2023-24 y 2026.

## Descripción

El Niño reduce las lluvias y los aportes de agua a los embalses que alimentan la generación hidroeléctrica en Colombia. Este proyecto construye, a partir de la API pública de XM, el conjunto de datos que permite comparar cómo se comportaron los embalses, los aportes hídricos, la generación por tipo de fuente y el precio de bolsa en cada episodio, incluido el de 2026, que está en curso.

La parte más importante del proyecto es el proceso de extracción y limpieza: la API tiene límites de consulta, devuelve filas duplicadas, entrega las horas de generación como texto y algunos embalses tienen series incompletas. Cada problema se documenta abajo con su evidencia y su tratamiento.

## Objetivo

Dejar un conjunto de tablas limpias, validadas y documentadas, listas para analizar la relación entre el nivel de los embalses, los aportes hídricos, la generación por fuente y el precio de bolsa durante los tres episodios de El Niño.

## Preguntas que guían el proyecto

1. ¿Cómo se compara el nivel de los embalses en El Niño 2026 frente a 2015-16 y 2023-24?
2. ¿Qué tan por debajo de la media histórica están los aportes hídricos actuales y desde cuándo?
3. ¿Cómo cambia la generación entre hídrica, térmica y otras fuentes cuando bajan los embalses, y qué pasa con el precio de bolsa?
4. ¿Qué embalses están en el punto más crítico?
5. ¿Hay relación entre el nivel de los embalses y el precio de bolsa?

## Descripción del dataset

**Fuente:** API pública de XM, consultada con la librería [`pydataxm`](https://github.com/EquipoAnaliticaXM/API_XM). No requiere usuario ni llave.
**Periodo:** 2015-01-01 a 2026-10-04 (4,295 días).

| Métrica (`MetricId`) | Entidad | Unidad | Uso |
|---|---|---|---|
| `PorcVoluUtilDiar` | Sistema y Embalse | fracción 0-1 (el catálogo la rotula `%`) | Volumen útil diario |
| `AporEner` | Sistema | kWh | Aportes hídricos reales |
| `AporEnerMediHist` | Sistema | kWh | Aportes, media histórica |
| `PrecBolsNaci` | Sistema | COP/kWh | Precio de bolsa nacional |
| `Gene` | Recurso | kWh (24 columnas de hora) | Generación por planta |
| `CapaUtilDiarEner` | Embalse | kWh | Capacidad útil, para el ranking de embalses |
| `ListadoEmbalses`, `ListadoRecursos` | Sistema | n/a | Catálogos (región hídrica, tipo de fuente) |

**Ventanas de comparación** (definidas para este proyecto a partir de los periodos reportados para cada evento):

| Ventana | Evento | Inicio | Fin |
|---|---|---|---|
| `nino_2015_16` | El Niño 2015-16 | 2015-03-01 | 2016-05-31 |
| `nino_2023_24` | El Niño 2023-24 | 2023-06-01 | 2024-04-30 |
| `nino_2026` | El Niño 2026 (en curso) | 2026-06-01 | 2026-10-04 |

Volumen, aportes y precio cubren todo el periodo 2015-2026. La generación por recurso se descarga solo para estas tres ventanas.

## Herramientas

Python (pandas), `pydataxm`, Google Colab y GitHub.

## Proceso de limpieza

### Pipeline

1. **Catálogo de métricas:** se lista con `get_collections()` y se confirman los `MetricId` exactos antes de descargar.
2. **Descarga por bloques:** una función parte cada rango en bloques de 31 días (límite de la API en granularidad diaria), los descarga y los une.
3. **Ranking de embalses:** se ordenan por capacidad útil (`CapaUtilDiarEner`, 2026-10-04), excluyendo `AGREGADO BOGOTA`. Los 15 primeros cubren el **95.72 %** de la capacidad útil.
4. **Series diarias del sistema:** volumen, aportes reales, aportes de media histórica y precio, 2015-2026. Se eliminan duplicados y se valida contra los 4,295 días del rango.
5. **Volumen por embalse:** se descarga para todos los embalses y se filtra al top 15.
6. **Catálogo de recursos:** se normaliza el tipo de fuente.
7. **Generación por recurso en las 3 ventanas:** las horas se convierten a número, se suman a kWh diarios por recurso y se agrupan por tipo de fuente.
8. **Tablas finales y validación:** se arman las tablas y una celda compara los conteos con los de la primera corrida.

### Hallazgos de calidad de datos

| # | Hallazgo | Evidencia | Tratamiento |
|---|---|---|---|
| 1 | Límite de 31 días por consulta diaria | Restricción de la API | Descarga por bloques de 31 días |
| 2 | Filas duplicadas | En la serie de volumen del sistema, 4,447 filas contra 4,295 días esperados (152 de más). Los bloques con filas de más se repiten en jun-ago 2019 y ene-abr 2024 en volumen, aportes, precio y volumen por embalse | `drop_duplicates` (copias exactas) y validación contra el conteo exacto de días |
| 3 | Las 24 columnas de hora de `Gene` llegan como **texto** | En una semana de prueba (3,771 filas), 33,354 celdas no se convierten a número y todas son cadenas vacías, concentradas entre las 19:00 y las 06:00 y en recursos solares (12.5 por fila en promedio) | Se convierten con `pd.to_numeric` y el vacío se trata como 0 kWh. Sumar sin convertir concatena el texto en vez de sumar |
| 4 | Categorías inconsistentes en el catálogo de recursos | `SOLAR` (2,177) y `Solar` (37) aparecen como categorías distintas | `.str.upper()` antes de agrupar |
| 5 | Plantas de las ventanas antiguas ausentes del catálogo vigente | Recursos sin tipo: 15 de 191 (0.99 % de la generación) en 2015-16, 9 de 324 (0.09 %) en 2023-24 y 0 de 606 en 2026 | Quedan como `SIN CLASIFICAR`, sin descartarlos |
| 6 | El volumen útil se rotula `%` pero viene como fracción | Valores entre 0 y 1 (por ejemplo 0.79719) | Se conserva como fracción y se documenta la unidad |
| 7 | Embalses con series incompletas | `EL QUIMBO`: 3,991 días, con datos desde 2015-11-01 y sin huecos. `MIRAFLORES`: 3,954 días, con 341 faltantes (2016-05-10; 2024-04-27 a 2025-02-03; 2025-02-05 a 2025-04-02) | Sin interpolar: quedan como vacíos |
| 8 | Volumen de la generación por recurso | Unos 2 millones de filas en el periodo completo | Se acota a las 3 ventanas de El Niño |
| 9 | `AGREGADO BOGOTA` aparece junto a los embalses individuales | Figura en el catálogo y en las series de embalses | Se excluye del ranking individual para no duplicar capacidad |

### Decisiones y supuestos

- **Top 15 de embalses** por capacidad útil; el resto no entra al detalle por embalse.
- **Precio diario** como promedio simple de las 24 horas, sin ponderar por demanda.
- **Horas vacías = 0 kWh** en la generación.
- **Sin interpolación** de huecos y **sin filtrado de valores atípicos**: los datos se conservan como los entrega la fuente.

## Tablas resultantes

**`data/processed/`**

| Archivo | Columnas | Filas |
|---|---|---|
| `fact_sistema_diario.csv` | `Date`, `Volumen_Util_Fraccion`, `Aportes_kWh`, `Aportes_MediaHist_kWh`, `Precio_Bolsa_COP_kWh` | 4,295 (una por día, sin nulos) |
| `volumen_util_por_embalse.csv` | `Name`, `Date`, `Value` (fracción 0-1) | Top 15 embalses, hasta 4,295 días cada uno |
| `generacion_diaria_por_tipo.csv` | `Ventana`, `Date`, `Tipo`, `Generacion_Diaria_kWh`, `Generacion_GWh` | Una fila por ventana, día y tipo de fuente |
| `dim_embalse.csv` | `Name`, `Capacidad_Util` (kWh), `Region_Hidrica` | 15 |
| `dim_ventana.csv` | `Ventana`, `Evento`, `Inicio`, `Fin` | 3 |

**`data/reference/`**: `listado_recursos.csv`, `precio_bolsa_diario.csv`, `volumen_historico_sistema.csv`, `aportes_reales.csv` y `aportes_media_historica.csv`, para contrastar la fuente con las tablas procesadas.

## Validación

La última celda del notebook compara los resultados con los de la primera corrida y marca `OK` o `REVISAR`:

| Verificación | Esperado |
|---|---|
| Días de la serie del sistema | 4,295, sin duplicados ni nulos |
| Embalses del top 15 encontrados | 15 |
| Días faltantes de `MIRAFLORES` | 341 |
| Filas de generación por ventana | 63,144 (2015-16), 82,478 (2023-24), 63,390 (2026) |
| Días por ventana | 458, 335 y 126 |

## Limitaciones

- **Los catálogos reflejan el día de la consulta.** `ListadoRecursos` y `ListadoEmbalses` devuelven el catálogo vigente, así que los recursos sin clasificar pueden variar en una ejecución posterior.
- **La ventana de 2026 está en curso.** No incluye todavía la temporada seca de diciembre a marzo, por lo que no es directamente comparable con los episodios completos.
- **La fuente puede revisar datos históricos.** Una ejecución futura podría no coincidir exactamente con los conteos de la primera corrida.
- **Los huecos de `MIRAFLORES` no tienen causa identificada** en los datos.


```

## Cómo reproducirlo

1. Abre `notebooks/01_extraccion_limpieza_xm.ipynb` en Google Colab.
2. Ejecuta todas las celdas en un runtime nuevo. Las descargas por bloques tardan varios minutos.
3. Revisa que la última celda marque `OK` en todas las verificaciones.
4. El notebook genera y descarga `datos_embalses_nino.zip` con `data/processed/` y `data/reference/`.

## Fuente y uso de los datos

Datos abiertos de XM, obtenidos con `pydataxm`. Antes de reutilizarlos, revisa los términos de uso del portal de XM.
