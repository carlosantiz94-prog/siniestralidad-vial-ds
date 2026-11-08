# Siniestralidad vial en Colombia – Proyecto de ciencia de datos

Proyecto individual de ciencia de datos con Python y Pandas (SENA).

| Actividad | Contenido | Notebook | Informe |
|---|---|---|---|
| **EV05** – Diagnóstico de calidad | Carga, inspección y diagnóstico de los datasets | [`notebooks/data_quality.ipynb`](notebooks/data_quality.ipynb) | [`reports/data_quality.md`](reports/data_quality.md) |
| **EV06** – Limpieza, análisis y estructuración | Lecciones 2, 3 y 4: limpieza, filtrado/agrupación y combinación | [`notebooks/pandas_labs.ipynb`](notebooks/pandas_labs.ipynb) | [`reports/data_processing.md`](reports/data_processing.md) |

## Descripción

El proyecto estudia los lesionados por accidentes de tránsito y los sectores críticos de la red vial de Colombia. Sigue la estructura del proyecto guía *Instacart*, adaptada a los datos de siniestralidad vial.

**Pregunta analítica:**
> ¿Qué departamentos, municipios y sectores viales concentran la mayor cantidad de lesionados y fallecidos por accidentes de tránsito en Colombia, y cómo ha evolucionado esa siniestralidad en el tiempo?

## Datasets

### Originales (`data/raw/`) – no se modifican
| Archivo | Descripción | Filas × columnas |
|---|---|---|
| `lesiones_accidentes_transito.csv` | Lesionados en accidentes de tránsito por fecha, departamento y municipio (2003 – sep. 2026) | 644.042 × 6 |
| `sectores_criticos_siniestralidad_vial.csv` | Sectores críticos de siniestralidad vial (puntos calientes Getis-Ord Gi*) con fallecidos y coordenadas | 316 × 13 |

Fuente: portal de Datos Abiertos de Colombia ([datos.gov.co](https://www.datos.gov.co)), descarga del 03/11/2026.

### Procesados (`data/processed/`) – generados por `pandas_labs.ipynb`
| Archivo | Descripción |
|---|---|
| `lesiones_clean.csv` | Lesiones limpias, con variables de fecha (no se sube a Git por tamaño: 49 MB) |
| `sectores_clean.csv` | Sectores críticos limpios (312 filas) |
| `cleaning_log.csv` | Informe de cambios del pipeline de limpieza |
| `lesiones_by_year.csv` | Lesionados por año |
| `lesiones_by_department_year.csv` | Lesionados por departamento y año |
| `dim_departments.csv`, `dim_municipalities.csv` | Tablas de dimensión DIVIPOLA |
| `municipality_analysis.csv` | Tabla de análisis por municipio (1.083 municipios) |

## Estructura del repositorio

```
siniestralidad-vial-ds/
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   ├── raw/
│   │   ├── lesiones_accidentes_transito.csv
│   │   └── sectores_criticos_siniestralidad_vial.csv
│   └── processed/
│       └── (archivos generados)
├── notebooks/
│   ├── data_quality.ipynb      # EV05
│   └── pandas_labs.ipynb       # EV06
└── reports/
    ├── data_quality.md         # EV05
    └── data_processing.md      # EV06
```

## Requisitos

- Python 3.10 o superior
- Dependencias en `requirements.txt` (pandas, numpy, jupyter, ipykernel)

## Cómo ejecutar

```bash
# 1. Clonar el repositorio y entrar a la rama de trabajo
git clone https://github.com/<tu-usuario>/siniestralidad-vial-ds.git
cd siniestralidad-vial-ds
git switch develop

# 2. Crear y activar el entorno virtual
python -m venv .venv
.venv\Scripts\activate          # Windows
source .venv/bin/activate       # macOS / Linux

# 3. Instalar dependencias
pip install -r requirements.txt

# 4. Ejecutar los notebooks en orden
jupyter notebook
```

Orden de ejecución (en Jupyter: **Kernel → Restart & Run All**):
1. `notebooks/data_quality.ipynb` – diagnóstico (EV05).
2. `notebooks/pandas_labs.ipynb` – limpieza, análisis y combinación (EV06). Genera todos los archivos de `data/processed/`.

Sin abrir Jupyter:
```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/data_quality.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/pandas_labs.ipynb
```

> Al leer los CSV procesados, use `dtype={"cod_muni": str, "cod_depto": str}` para conservar los ceros de los códigos DIVIPOLA.

## Flujo de ramas (Git)

- `main`: versión entregada de cada actividad.
- `develop`: integración.
- `feature/pandas-data-processing`: desarrollo de la EV06. Se fusiona en `develop`.

## Resultados principales

**EV05 – Diagnóstico:** fechas leídas como texto, códigos DIVIPOLA sin ceros, coma decimal, faltantes ocultos (`<Null>`, `sd`, `SIN ESTABLECER`) y nombres con o sin tilde entre fuentes.

**EV06 – Procesamiento:**
- **Limpieza:** no se pierde ningún lesionado (885.829 en total). Los duplicados se conservan porque eliminarlos haría perder el 33,4 %. En sectores se eliminan 4 puntos repetidos (316 → 312).
- **Análisis:** el pico es 2016 y el mínimo reciente 2020. El domingo es el día con más lesionados. Seis departamentos suman más de la mitad del total. Enero–septiembre 2026 tiene 11 % más lesionados que el mismo periodo de 2025.
- **Combinación:** la tabla por municipio está validada (sin duplicados ni pérdidas). Los 312 sectores se asignan por código DIVIPOLA, y 3 municipios se habrían perdido si se cruzara por nombre.

## Autor

Carlos Antiz · Aprendiz SENA
