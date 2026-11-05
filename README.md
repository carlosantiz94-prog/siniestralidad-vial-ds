# Siniestralidad vial en Colombia – Diagnóstico de calidad de datos

Proyecto individual de ciencia de datos · **GA1-220501094-01-AA1-EV05**

## Descripción

Este repositorio contiene el diagnóstico inicial de calidad de los datasets del proyecto **Análisis de la siniestralidad vial en Colombia**. El diagnóstico se hace con Python y Pandas: carga, inspección estructural, identificación de valores faltantes, tipos de datos e inconsistencias, y propuesta de acciones de limpieza.

**Pregunta analítica:**
> ¿Qué departamentos, municipios y sectores viales concentran la mayor cantidad de lesionados y fallecidos por accidentes de tránsito en Colombia, y cómo ha evolucionado esa siniestralidad en el tiempo?

## Datasets

| Archivo | Descripción | Filas × columnas |
|---|---|---|
| `data/raw/lesiones_accidentes_transito.csv` | Lesionados en accidentes de tránsito por fecha, departamento y municipio (2003 – sep. 2026) | 644.042 × 6 |
| `data/raw/sectores_criticos_siniestralidad_vial.csv` | Sectores críticos de siniestralidad vial (puntos calientes Getis-Ord Gi*) con fallecidos y coordenadas | 316 × 13 |

Fuente: portal de Datos Abiertos de Colombia ([datos.gov.co](https://www.datos.gov.co)), descarga del 03/11/2026.

## Estructura del repositorio

```
siniestralidad-vial-ds/
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   ├── raw/                      # Datos originales (no se modifican)
│   │   ├── lesiones_accidentes_transito.csv
│   │   └── sectores_criticos_siniestralidad_vial.csv
│   └── processed/                # Datos limpios (siguiente etapa)
├── notebooks/
│   └── diagnostico_calidad.ipynb # Notebook del diagnóstico
└── reports/
    └── informe_diagnostico.md    # Informe técnico
```

## Requisitos

- Python 3.10 o superior
- Dependencias en `requirements.txt` (pandas, jupyter, ipykernel)

## Cómo reproducir el diagnóstico

```bash
# 1. Clonar el repositorio
git clone https://github.com/<carlosantiz94-prog>/siniestralidad-vial-ds.git
cd siniestralidad-vial-ds

# 2. Crear y activar el entorno virtual
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS / Linux:
source .venv/bin/activate

# 3. Instalar dependencias
pip install -r requirements.txt

# 4. Abrir el notebook
jupyter notebook notebooks/diagnostico_calidad.ipynb
```

En Jupyter, ejecutar **Kernel → Restart & Run All**. También se puede ejecutar sin abrir Jupyter:

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/diagnostico_calidad.ipynb
```

## Resultados principales

- **Lesiones:** sin nulos explícitos, pero la fecha se lee como texto y los códigos DIVIPOLA pierden el cero inicial. El 45,7 % de las filas son idénticas porque no hay identificador de víctima. Entre 2010 y 2017 cada fila agrupa más lesionados, por lo que se debe sumar `CANTIDAD` y no contar filas. 2026 llega hasta septiembre.
- **Sectores críticos:** las coordenadas y métricas usan coma decimal y se leen como texto. Hay faltantes ocultos (`<Null>` en 55 % de `ID_MT`, `sd` en 13 % de `PR`), nulos en `Nombre` y `Tramo`, y 4 pares de coordenadas repetidas.
- **Ambos:** los nombres de departamento difieren por tildes, así que el cruce debe hacerse por código DIVIPOLA.

El detalle completo está en [`reports/informe_diagnostico.md`](reports/informe_diagnostico.md).

## Autor

Carlos Antiz · Aprendiz SENA
