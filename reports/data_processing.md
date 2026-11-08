# Informe de procesamiento de datos

**Actividad:** GA1-220501094-01-AA1-EV06 – Limpieza, análisis y estructuración de datos con Pandas
**Proyecto:** Análisis de la siniestralidad vial en Colombia
**Notebook:** [`notebooks/pandas_labs.ipynb`](../notebooks/pandas_labs.ipynb) · **Antecedente:** [`reports/data_quality.md`](data_quality.md) (EV05)

**Pregunta orientadora (adaptada):** ¿Cómo preparar y estructurar los datos de siniestralidad vial para obtener información confiable sobre dónde y cuándo se concentran los lesionados y fallecidos?

| Proyecto guía Instacart | Este proyecto |
|---|---|
| Órdenes (`instacart_orders`) | Registros de lesionados (`lesiones`) |
| Productos (`products`) | Municipios (`dim_municipalities`) |
| Departamentos / pasillos | Departamentos (`dim_departments`) y sectores viales críticos (`sectores`) |
| Hábitos de compra | Patrones temporales y territoriales de la siniestralidad |
| Tabla de análisis por producto | Tabla de análisis por municipio (`municipality_analysis.csv`) |

---

## 1. Lección 2 – Limpieza y estandarización

### Pipeline de limpieza (Laboratorio 2)
Cada paso se implementó como una función que registra sus cambios. El informe completo está en `data/processed/cleaning_log.csv`.

| Dataset | Paso | Filas antes → después | Valores afectados | Decisión |
|---|---|---|---|---|
| lesiones | Renombrar columnas | 644.042 → 644.042 | 6 | `snake_case` |
| lesiones | Convertir fecha | = | 644.042 | texto → `datetime64`; 0 fechas inválidas |
| lesiones | Códigos DIVIPOLA | = | 80.832 | int → texto con `zfill(2)` / `zfill(5)` |
| lesiones | Normalizar texto | = | 585 | `strip`, mayúsculas, sin tildes (se conserva la Ñ) |
| lesiones | Nulos ocultos | = | 14 | `SIN ESTABLECER` → `NaN` |
| lesiones | Variables de fecha | = | 5 columnas nuevas | `anio`, `mes`, `nombre_mes`, `dia_semana`, `anio_parcial` |
| lesiones | Marcar atípicos | = | 638 | `cantidad` > percentil 99,9 (21). Se marcan, no se eliminan |
| lesiones | Duplicados | = | 294.354 | **Se conservan**: sin ID de víctima no son errores |
| lesiones | Tipos categóricos | = | 4 columnas | Memoria: 224,7 MB → 83,6 MB |
| sectores | Nulos ocultos | 316 → 316 | 217 | `<Null>` (175) y `sd` (42) → `NaN` |
| sectores | Tipos numéricos | = | 5 columnas | coma decimal → `float` |
| sectores | Código DIVIPOLA | = | 43 | `zfill(5)` y nueva columna `cod_depto` |
| sectores | Normalizar texto | = | 555 | tildes, espacios dobles, `*` en tramo |
| sectores | Imputar texto | = | 198 | `SIN TRAMO` / `SIN NOMBRE` |
| sectores | Duplicados por coordenadas | 316 → **312** | 4 | Se conserva el punto con mayor `gi_zscore` |
| sectores | Nivel de confianza | = | 312 | `gi_pvalue` → 99 % / 95 % / 90 % |

### Resultados del Reto 2
- Después de normalizar, los 6 departamentos con tilde (ATLÁNTICO, BOLÍVAR, BOYACÁ, CAQUETÁ, CÓRDOBA, QUINDÍO) **coinciden** entre las dos fuentes.
- El criterio IQR no sirve para `cantidad`: con Q1 = Q3 = 1 marcaría 79.726 registros (12 %). Se eligió el percentil 99,9, que marca 638.
- **Eliminar duplicados haría perder el 33,4 % de los lesionados** (885.829 → 589.932). Esto confirma la decisión de conservarlos.
- Nulos que quedan, todos esperables: `id_mt` (175, vías sin concesión), `pr` (42), y 3 departamentos y 11 municipios sin establecer.

## 2. Lección 3 – Filtrado, ordenamiento y agrupación

### Perfil de la siniestralidad (Laboratorio 3)
| Dimensión | Resultado |
|---|---|
| Tendencia anual | Pico en 2016 (77.037). Mínimo reciente en 2020 (27.481, pandemia). 2023–2025 entre 47.000 y 51.000 |
| Mes | Reparto parejo; marzo tiene el promedio más alto (2018–2025) |
| Día de la semana | Domingo 17,0 % y sábado 16,4 %: el fin de semana concentra la mayor parte |
| Territorio | 6 departamentos suman más del 50 % (Valle 14,5 %, Antioquia 10,8 %, Bogotá 9,4 %, Cundinamarca, Santander, Tolima) |
| Municipios | Bogotá (32.275), Cali (23.775), Medellín (20.354), Ibagué y Pereira lideran 2018–2025 |
| Sectores críticos | Valle del Cauca: 60 sectores y 684 fallecidos. El sector más crítico es Tumaco–Pedregal (52 fallecidos, z = 14,5) |

### Resultados del Reto 3
- Antioquia 2025: 4.697 lesionados; Medellín aporta 2.705 (58 %).
- Enero–septiembre 2026 frente al mismo periodo de 2025: **+11 %** a nivel nacional (36.992 → 41.042). Los mayores aumentos están en Casanare, Sucre y Risaralda.
- 69 sectores combinan 99 % de confianza y 15 o más fallecidos.
- Las vías de INVÍAS/otras nacionales suman 1.513 fallecidos y las de la ANI 1.480.
- Pasto (~70 % de Nariño) y Quibdó (~70 % de Chocó) son capitales que concentran casi toda la siniestralidad de su departamento.

## 3. Lección 4 – Combinación de datasets

### Tabla de análisis por municipio (Laboratorio 4)
`municipality_analysis.csv` tiene **1.083 municipios × 18 columnas**. Se construyó combinando `dim_municipalities` + `dim_departments` + métricas de lesiones + métricas de sectores, con `validate` en cada `merge`.

| Validación | Resultado |
|---|---|
| Una fila por municipio | ✔ |
| Filas = municipios de la dimensión | ✔ |
| Total lesionados 2003–2025 conservado | ✔ |
| Total lesionados 2025 conservado | ✔ |
| Los 312 sectores quedaron asignados | ✔ |
| Fallecidos de sectores conservados | ✔ |
| Sin departamentos nulos | ✔ |

### Resultados del Reto 4
- Los 312 sectores encuentran su municipio por código DIVIPOLA (`_merge = both` en el 100 %).
- 3 municipios tienen **nombre distinto** con el mismo código: Buga / Guadalajara de Buga, Tumaco / San Andrés de Tumaco y Patía (El Bordo) / Patía. Un cruce por nombre los habría perdido.
- *Outer join* 2025: 133 municipios tienen lesionados y sector crítico, y 730 tienen solo lesionados. Ningún municipio con sector crítico quedó sin lesionados.
- Fallecidos en sectores por cada 1.000 lesionados: Cesar (61), Arauca (42) y La Guajira (41) encabezan. Allí la siniestralidad en carretera pesa más que la urbana.
- Un `merge` con `validate="one_to_one"` sobre `sectores` lanza `MergeError` (179 claves repetidas). Esto muestra para qué sirve validar la cardinalidad.
- La prueba de ida y vuelta (guardar y volver a leer los CSV) conserva filas, columnas y ceros de DIVIPOLA.

## 4. Archivos procesados (`data/processed/`)

| Archivo | Contenido | En Git |
|---|---|---|
| `lesiones_clean.csv` | Lesiones limpias (644.042 × 12) | No (49 MB; se regenera con el notebook) |
| `sectores_clean.csv` | Sectores limpios (312 × 16) | Sí |
| `cleaning_log.csv` | Informe de cambios de la limpieza | Sí |
| `lesiones_by_year.csv` | Lesionados y registros por año (2003–2025) | Sí |
| `lesiones_by_department_year.csv` | Lesionados por departamento y año | Sí |
| `dim_departments.csv` / `dim_municipalities.csv` | Tablas de dimensión DIVIPOLA | Sí |
| `municipality_analysis.csv` | Tabla de análisis por municipio | Sí |

> Al leer los CSV procesados, use `dtype={"cod_muni": str, "cod_depto": str}` para conservar los ceros iniciales.

## 5. Conclusiones
1. Los datos quedaron **confiables y estructurados**: tipos correctos, claves DIVIPOLA consistentes y texto homogéneo. No se perdió ningún lesionado y cada decisión quedó documentada.
2. La regla más importante es **sumar `cantidad`** y **no eliminar duplicados**; de lo contrario se subestimaría un tercio de los lesionados.
3. La siniestralidad se concentra en el tiempo (fines de semana) y en el territorio (6 departamentos y sus capitales). Los sectores críticos en carretera muestran un riesgo distinto, sobre todo en Cesar, Nariño y La Guajira.
4. `municipality_analysis.csv` es la base para la siguiente etapa: visualización, mapas y modelado.
