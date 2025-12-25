# HeadOfData101

Repositorio del curso Head of Data 101 (Albert School). Proyecto end-to-end con foco en scraping, limpieza y preparacion de datos a partir de listados de coches de AutoScout24.

## Estado actual

- Notebook de scraping con pasos guiados para principiantes.
- Notebook de preprocessing para transformar datos raw en un dataset analitico.
- Datos raw y processed en `data/` con versionado por timestamp.

## Estructura del repo

```
.
├─ data/
│  ├─ raw/                # CSVs generados por scraping
│  ├─ processed/          # CSVs limpios y estructurados
│  └─ samplefiles/        # Archivos de muestra para clase
├─ notebooks/
│  ├─ scrapping/          # Scraping (AutoScout24)
│  └─ preprocessing/      # Limpieza y normalizacion
├─ reports/               # Salidas y reportes (por ahora vacio)
├─ src/                   # Codigo reutilizable (por ahora vacio)
└─ README.md
```

## Requisitos

- Python 3.x
- Jupyter (o VS Code con soporte para notebooks)
- Librerias usadas: `requests`, `beautifulsoup4`, `pandas`, `numpy`, `matplotlib`

## Como ejecutar

1. Crear y activar un entorno virtual.
2. Instalar dependencias:

```bash
pip install requests beautifulsoup4 pandas numpy matplotlib
```

3. Abrir los notebooks:

- `notebooks/scrapping/scrapping.ipynb`
- `notebooks/preprocessing/preprocessing.ipynb`

## Datos

- **Raw**: se generan en `data/raw/` con timestamp. Nunca se modifican.
- **Processed**: se guardan en `data/processed/` con timestamp.
- **Sample**: ejemplos en `data/samplefiles/`.

Columnas esperadas en el raw (segun notebook de preprocessing):
`make`, `model`, `mileage`, `price`, `registration`, `fuel`, `country`, `brand`, `page`.

## Buenas practicas de scraping

- Limitar paginas y marcas en pruebas.
- Respetar pausas entre requests.
- Revisar los Terminos de uso del sitio y la legalidad local.

## Proximos pasos (sugeridos)

- Consolidar scripts en `src/` para reutilizar scraping y limpieza.
- Añadir analisis exploratorio y visualizaciones en `reports/`.
- Definir un pipeline reproducible (make/pyproject) y tests basicos.
