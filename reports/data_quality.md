# Informe técnico – Diagnóstico inicial de calidad de datos

**Actividad:** GA1-220501094-01-AA1-EV05 – Diagnosticar la calidad de datos: preparación, exploración y documentación
**Proyecto:** Análisis de la siniestralidad vial en Colombia
**Herramientas:** Python 3 · Pandas · Jupyter Notebook
**Notebook de soporte:** [`notebooks/data_quality.ipynb`](../notebooks/data_quality.ipynb)

---

## 1. Nombre del proyecto y pregunta analítica

**Proyecto:** Análisis de la siniestralidad vial en Colombia.

**Propósito:** identificar los territorios y sectores viales con mayor siniestralidad para apoyar la priorización de acciones de seguridad vial.

**Pregunta analítica:**
> ¿Qué departamentos, municipios y sectores viales concentran la mayor cantidad de lesionados y fallecidos por accidentes de tránsito en Colombia, y cómo ha evolucionado esa siniestralidad en el tiempo?

## 2. Descripción de los datasets

| # | DataFrame | Archivo | Contenido | Relación con el problema |
|---|---|---|---|---|
| 1 | `df_lesiones` | `data/raw/lesiones_accidentes_transito.csv` | Lesionados en accidentes de tránsito por fecha del hecho, departamento y municipio (con códigos DIVIPOLA) y `CANTIDAD` de personas. Periodo 01/01/2003 – 30/09/2026. | Responde **cuántos**, **dónde** y **cuándo** hay lesionados. |
| 2 | `df_sectores` | `data/raw/sectores_criticos_siniestralidad_vial.csv` | 316 puntos críticos de la red vial identificados con análisis de puntos calientes Getis-Ord Gi* (`GiZScore`, `GiPValue`), número de `Fallecidos`, entidad responsable (ANI, INVÍAS, departamento, ciudad), tramo, PR, coordenadas y municipio. | Responde **qué sectores viales** concentran fallecidos y permite cruzarlos con el municipio. |

Ambos archivos provienen del portal de Datos Abiertos de Colombia (descarga del 03/11/2026), en formato CSV con codificación UTF-8.

## 3. Resultados de la inspección estructural

Comandos aplicados a cada DataFrame: `head()`, `tail()`, `shape`, `columns`, `dtypes`, `info()`, `describe()` y `describe(include='object')`.

| Dataset | Filas | Columnas | Tipos de datos | Observaciones |
|---|---|---|---|---|
| `df_lesiones` | 644.042 | 6 | 3 texto (`FECHA HECHO`, `DEPARTAMENTO`, `MUNICIPIO`), 3 enteros (`COD_DEPTO`, `COD_MUNI`, `CANTIDAD`) | Sin nulos explícitos. La fecha queda como texto. Los códigos DIVIPOLA se leen como enteros y pierden el cero inicial. `CANTIDAD`: mín. 1, mediana 1, media 1,38, máx. 91. 34 departamentos y 1.089 códigos de municipio. |
| `df_sectores` | 316 | 13 | 11 texto, 2 enteros (`Fallecidos`, `divipola`) | `GiZScore`, `GiPValue`, `Latitud` y `Longitud` quedan como texto por usar coma decimal. `PR` mezcla números y `sd`. Nulos en `Tramo` y `Nombre`. `Fallecidos`: mín. 1, mediana 10, máx. 52. 22 departamentos y 133 municipios. |

**Características principales**

- **`df_lesiones`** es un registro largo (una fila por evento-municipio-día). Su variable de medida es `CANTIDAD`; el análisis debe usar la **suma** de `CANTIDAD`, no el número de filas.
- **`df_sectores`** es una tabla pequeña y sin fecha (una foto del estado de los sectores críticos). Todos los puntos tienen `GiZScore` positivo (1,66 a 14,54), es decir, son puntos calientes.

## 4. Valores faltantes, tipos de datos e inconsistencias

### 4.1 Valores faltantes

| Dataset | Columna | Tipo de faltante | Registros | % |
|---|---|---|---|---|
| `df_lesiones` | `DEPARTAMENTO` | Texto `SIN ESTABLECER` (código 0) | 3 | < 0,01 |
| `df_lesiones` | `MUNICIPIO` | Texto `SIN ESTABLECER` | 11 | < 0,01 |
| `df_sectores` | `Nombre` | Nulo (`NaN`) | 175 | 55,38 |
| `df_sectores` | `Tramo` | Nulo (`NaN`) | 23 | 7,28 |
| `df_sectores` | `ID_MT` | Texto `<Null>` | 175 | 55,38 |
| `df_sectores` | `PR` | Texto `sd` (sin dato) | 42 | 13,29 |

`info()` no muestra estos marcadores de texto como nulos: son **faltantes ocultos**. Los `<Null>` de `ID_MT` corresponden a vías sin contrato de concesión (INVIAS-OTROS 126, DEPTO 48, CIUDAD 1), por lo que coinciden con los nulos de `Nombre`.

**Impacto:** en `df_lesiones` es mínimo. En `df_sectores` más de la mitad de los puntos no tiene nombre de proyecto, lo que limita la descripción, pero no el conteo de fallecidos ni su ubicación (las coordenadas están completas).

### 4.2 Tipos de datos

| Variable | Tipo leído | Tipo correcto | Evidencia |
|---|---|---|---|
| `FECHA HECHO` | texto | `datetime` | Las 644.042 fechas se convierten sin error con formato `%d/%m/%Y`. |
| `COD_DEPTO`, `COD_MUNI` | `int64` | texto de 2 y 5 dígitos | 80.832 registros pierden el cero inicial (ej. Medellín `05001` → `5001`). |
| `divipola` | `int64` | texto de 5 dígitos | 43 códigos con 4 dígitos. |
| `GiZScore`, `GiPValue`, `Latitud`, `Longitud` | texto | `float` | Coma decimal; al reemplazarla por punto, el 100 % convierte a número. |
| `PR` | texto | numérico | 42 valores `sd`. |

### 4.3 Posibles inconsistencias

- **Filas idénticas en `df_lesiones`:** 294.354 (45,7 %). El dataset no tiene un identificador de víctima o evento, así que dos lesionados en el mismo municipio y día producen filas iguales. No se pueden tratar como errores sin confirmar con la fuente.
- **Cambio en la forma de registro:** entre 2010 y 2017 hay en promedio 2,4 a 2,8 lesionados por fila, frente a cerca de 1,0 en los demás años. Contar filas en lugar de sumar `CANTIDAD` produce una tendencia falsa.
- **Año 2026 incompleto:** la fecha máxima es 30/09/2026 (9 meses).
- **Valores extremos:** `CANTIDAD` = 91 en Bogotá el 25/04/2016, y valores de 50 a 63 en Bogotá, Cali y Medellín. Son plausibles en ciudades grandes, pero deben verificarse.
- **Nombres de municipio repetidos:** 65 nombres (VILLANUEVA, LA UNIÓN, BUENAVISTA, etc.) corresponden a más de un código. Hay 1.003 nombres únicos frente a 1.089 códigos.
- **Tildes entre datasets:** `df_lesiones` escribe sin tilde y `df_sectores` con tilde. Seis departamentos no coinciden por nombre (ATLÁNTICO, BOLÍVAR, BOYACÁ, CAQUETÁ, CÓRDOBA, QUINDÍO). Por código DIVIPOLA **todos** los municipios de `df_sectores` sí existen en `df_lesiones`.
- **Coordenadas repetidas en `df_sectores`:** 4 pares de puntos (Coyaima, Apartadó, Turbo, Cotorra) con las mismas coordenadas y distinto número de fallecidos. Puede haber doble conteo.
- **Texto:** 15 valores de `Tramo` truncados con `*` ("El Hatillo - … - Med*") y 25 nombres con espacios dobles.
- **Validaciones correctas:** todas las coordenadas están dentro de Colombia y todos los p-values están entre 0 y 1. 60 puntos tienen p-value > 0,05, es decir, no son significativos al 95 % (sí al 90 %).

## 5. Principales hallazgos de calidad por dataset

| Dataset | Problema identificado | Evidencia encontrada | Posible impacto | Acción propuesta |
|---|---|---|---|---|
| `df_lesiones` | Fecha almacenada como texto | `dtype` str en `FECHA HECHO` | No se puede agrupar por año o mes | `pd.to_datetime(format='%d/%m/%Y')` |
| `df_lesiones` | Códigos DIVIPOLA como enteros | `int64`; 80.832 códigos sin cero inicial | Cruces erróneos con otras fuentes | `dtype=str` al leer y `zfill(2)` / `zfill(5)` |
| `df_lesiones` | Ubicación no establecida | 3 departamentos y 11 municipios `SIN ESTABLECER` | Registros sin asignación territorial | Convertir a `NaN` y excluir del análisis territorial |
| `df_lesiones` | Filas idénticas sin identificador | 294.354 duplicados (45,7 %) | No se distingue duplicado real de víctima distinta | No eliminar; documentar y usar `sum(CANTIDAD)` |
| `df_lesiones` | Cambio en la forma de registro | 2010–2017: ~2,5 lesionados/fila; resto ~1,0 | Tendencias sesgadas si se cuentan filas | Analizar siempre con la suma de `CANTIDAD` |
| `df_lesiones` | Año 2026 incompleto | Fecha máxima 30/09/2026 | 2026 subestimado en comparaciones anuales | Comparar enero–septiembre o excluir 2026 |
| `df_lesiones` | Nombres de municipio repetidos | 65 nombres con más de un código | Agrupar por nombre mezcla municipios | Agrupar por `COD_MUNI` |
| `df_lesiones` | Valores extremos de `CANTIDAD` | Máx. 91 (Bogotá, 2016); mediana 1 | Distorsionan promedios | Verificar con la fuente; usar mediana |
| `df_sectores` | Decimales con coma | 4 columnas numéricas leídas como texto | No se calculan estadísticas ni mapas | `read_csv(decimal=',')` |
| `df_sectores` | Faltantes ocultos `<Null>` | 175 en `ID_MT` (55,4 %) | Nulos no detectados por `info()` | `na_values=['<Null>']` |
| `df_sectores` | Nulos en `Nombre` y `Tramo` | 175 y 23 nulos | Menor capacidad descriptiva | Imputar "Sin nombre" o completar desde la fuente |
| `df_sectores` | `PR` con `sd` | 42 valores | PR no numérico | `na_values=['sd']` y convertir a numérico |
| `df_sectores` | Coordenadas repetidas | 4 pares con igual Lat/Lon | Posible doble conteo de fallecidos | Revisar y consolidar |
| `df_sectores` | Texto truncado / espacios extra | 15 tramos con `*`; 25 nombres con espacios dobles | Agrupaciones por tramo inconsistentes | `str.strip()`, normalizar espacios, corregir nombres |
| Ambos | Nombres con y sin tilde | BOYACÁ vs BOYACA, etc. (6 departamentos) | El cruce por nombre falla | Cruzar por DIVIPOLA o quitar tildes |

## 6. Acciones de limpieza y preparación propuestas

1. **Lectura con tipos correctos**
   ```python
   df_lesiones = pd.read_csv(ruta_lesiones, dtype={"COD_DEPTO": str, "COD_MUNI": str},
                             parse_dates=["FECHA HECHO"], date_format="%d/%m/%Y")
   df_sectores = pd.read_csv(ruta_sectores, decimal=",", na_values=["<Null>", "sd"],
                             dtype={"divipola": str})
   ```
2. **Normalizar códigos DIVIPOLA:** `zfill(2)` para departamento y `zfill(5)` para municipio, en ambos datasets.
3. **Convertir marcadores a nulos:** `SIN ESTABLECER`, `<Null>` y `sd` → `NaN`.
4. **Normalizar texto:** mayúsculas, quitar tildes (`unicodedata`), `str.strip()` y espacios dobles. Corregir el tramo truncado.
5. **Renombrar columnas** a un formato uniforme (`fecha_hecho`, `cod_depto`, `cod_muni`, `municipio`, `cantidad`…).
6. **Revisar los 4 pares de coordenadas repetidas** en `df_sectores` y decidir si se consolidan.
7. **Definir la regla de medida:** usar `sum(CANTIDAD)` como número de lesionados; no eliminar filas idénticas.
8. **Tratamiento temporal:** crear columnas `anio` y `mes`; marcar 2026 como año parcial.
9. **Integración:** unir los datasets a nivel de municipio por `cod_muni` / `divipola`.
10. **Guardar los datos limpios** en `data/processed/` sin modificar `data/raw/`.

## 7. Conclusiones

- Los dos datasets están **en condiciones de usarse** para el proyecto. No hay columnas críticas vacías: las fechas son 100 % convertibles, las coordenadas están completas y en rango, y todos los municipios de los sectores críticos existen en el dataset de lesiones por código DIVIPOLA.
- Los problemas encontrados son sobre todo de **formato y tipo de dato** (fechas como texto, códigos sin ceros, coma decimal) y de **faltantes ocultos**. Todos se corrigen al leer los archivos con los parámetros adecuados.
- Los riesgos más importantes para la pregunta analítica son metodológicos: **contar filas en lugar de sumar `CANTIDAD`**, comparar 2026 como año completo y **cruzar por nombre** en lugar de por código.
- Limitaciones que se deben declarar: `df_lesiones` no tiene fallecidos, gravedad, causa ni tipo de actor vial, y `df_sectores` no tiene fecha. Por eso la evolución temporal solo se puede estudiar con lesionados, y los fallecidos solo por sector crítico.
- Siguiente paso: implementar las acciones de la sección 6 en un notebook de limpieza y generar los datasets procesados.
