# 🌿 Detección de Restauración y Recuperación de Cobertura Forestal
### Mediante SAR y Persistencia de Índices de Vegetación — Guatemala

> **Caso de uso institucional** | MARN · INAB · CONAP  
> Monitoreo REDD+ · NDC · Plan de Fomento Forestal 2026–2030

---

## 📋 Descripción

Este repositorio contiene el flujo de procesamiento geoespacial para generar una **capa anual de áreas en recuperación y restauración de cobertura forestal para Guatemala**, integrando datos de radar de apertura sintética (SAR) Sentinel-1 con índices espectrales derivados de Sentinel-2.

La metodología combina:
- Incremento de retrodispersión SAR (polarizaciones VV y VH mediante **RVI S1**)
- Persistencia temporal de índices de vegetación (NDVI, EVI, NBR)
- Umbrales de referencia calibrados con inventarios de incentivos forestales (PROBOSQUE / PINPEP)
- Ponderación multivariable para categorizar grado de recuperación

Los resultados soportan la toma de decisiones sobre monitoreo de incentivos, priorización territorial para regeneración asistida, y reporte a nivel de municipio, departamento, cuenca y área protegida.

---

## 🗂️ Estructura del repositorio

```
├── data/
│   ├── inputs/                  # Insumos base (no incluidos — ver sección Datos)
│   └── outputs/                 # Salidas geojson/shp y CSV por unidad administrativa
├── scripts/
│   ├── gee/                     # Scripts de Google Earth Engine (.js)
│   │   ├── 01_sar_increment.js
│   │   ├── 02_vi_persistence.js
│   │   ├── 03_slope_elevation.js
│   │   └── 04_composite_score.js
│   ├── python/                  # Pipeline de post-procesamiento (.py)
│   │   ├── processing.py
│   │   ├── thresholds.py
│   │   └── export_stats.py
│   └── utils/
│       └── helpers.py
├── docs/
│   ├── ficha_metodologica.pdf   # Ficha técnica completa
│   └── diagrama_flujo.png
├── wireframe/                   # Mockup de página Experience Builder
└── README.md
```

---

## 🔄 Flujo de procesamiento

```
┌─────────────────┐     ┌──────────────────────┐     ┌──────────────────────┐
│  1. ALCANCE     │────▶│  2. DATOS E INSUMOS  │────▶│  3. PROCESAMIENTO    │
│                 │     │                      │     │                      │
│ Capa anual de  │     │ • Máscara Bosque/No  │     │ • Incremento SAR     │
│ recuperación/  │     │   Bosque 2020         │     │   (VV, VH, RVI S1)  │
│ restauración   │     │ • Sentinel-1 / S-2    │     │ • Persistencia VI   │
│ forestal GT    │     │ • Polígonos PROBOSQUE │     │ • % pendiente/elev  │
│                │     │ • Límites municipios, │     │ • Umbrales ref.     │
│                │     │   AP y cuencas        │     │ • Ponderación vars  │
└─────────────────┘     └──────────────────────┘     └──────────────────────┘
                                                               │
          ┌────────────────────────────────────────────────────┘
          ▼
┌──────────────────────┐     ┌──────────────────────┐     ┌─────────────────────┐
│  4. BD & GEOSERV.   │────▶│  5. INFRAESTRUCTURA  │────▶│  6. USO INST.       │
│                      │     │                      │     │                     │
│ • GeoJSON / SHP      │     │ • Python + GEE       │     │ • Estadísticas      │
│ • CSV por municipio, │     │ • GitHub (este repo) │     │   de ganancias      │
│   AP, depto, cuenca  │     │ • Experience Builder │     │ • Monitoreo areas   │
│ • Publicación en     │     │                      │     │   con incentivos    │
│   ArcGIS Enterprise  │     │                      │     │ • Priorización      │
│                      │     │                      │     │   territorial       │
└──────────────────────┘     └──────────────────────┘     └─────────────────────┘
```

---

## 📦 Datos de entrada requeridos

| Insumo | Fuente | Resolución | Notas |
|--------|--------|-----------|-------|
| Máscara Bosque/No Bosque 2020 | MARN / INAB / CONAP | 30 m | Mapa oficial tripartita |
| Sentinel-1 GRD (VV, VH) | Copernicus / GEE | 10 m | Órbitas ascendentes y descendentes |
| Sentinel-2 SR | Copernicus / GEE | 10–20 m | Colección armonizada con filtro de nubes |
| Polígonos de plantaciones forestales | INAB — PROBOSQUE / PINPEP | Vector | A nivel nacional |
| Límites municipales | IGN Guatemala | Vector | |
| Áreas protegidas | CONAP | Vector | |
| Cuencas hidrográficas | MARN / INSIVUMEH | Vector | |

> ⚠️ Los insumos institucionales **no están incluidos** en este repositorio. Consultar al equipo de MARN (DESI) o solicitar acceso a las capas en el portal ArcGIS Enterprise correspondiente.

---

## ⚙️ Variables del modelo compuesto

El score de recuperación forestal se calcula ponderando las siguientes variables:

| Variable | Descripción | Fuente |
|---------|-------------|--------|
| `sar_increment` | Incremento de retrodispersión SAR (RVI S1) | Sentinel-1 |
| `ndvi_persist` | Persistencia de NDVI anual o mensual | Sentinel-2 |
| `evi_persist` | Persistencia de EVI | Sentinel-2 |
| `slope_pct` | Porcentaje de pendiente | DEM (SRTM/ALOS) |
| `elevation` | Elevación media por píxel | DEM |
| `incentive_mask` | Umbral de referencia por polígonos de incentivos | INAB |
| `forest_mask_2020` | Máscara bosque/no bosque de referencia | Tripartita 2020 |

Los pesos de ponderación y los umbrales de categorización son configurables en `scripts/python/thresholds.py`.

---

## 🚀 Cómo usar

### Prerrequisitos

```bash
# Python >= 3.9
pip install earthengine-api geopandas pandas numpy rasterio

# Autenticación en GEE
earthengine authenticate
```

### Ejecutar el flujo en GEE

1. Abrir los scripts en `scripts/gee/` en el [Code Editor de GEE](https://code.earthengine.google.com/).
2. Cargar los insumos desde los assets institucionales o Google Drive.
3. Ajustar los parámetros de año y área de interés en las variables del encabezado de cada script.
4. Exportar resultados a Drive o Asset según se requiera.

### Post-procesamiento en Python

```bash
# Calcular estadísticas por unidad administrativa
python scripts/python/export_stats.py \
  --input data/outputs/score_raster.tif \
  --boundaries data/inputs/municipios_GT.shp \
  --output data/outputs/stats_municipios.csv
```

---

## 📤 Salidas generadas

| Formato | Contenido | Escala |
|--------|-----------|--------|
| `.shp` / `.geojson` | Capa de score de recuperación | Píxel (10 m) |
| `.csv` | Estadísticas de área recuperada | Municipio / Depto / AP / Cuenca |
| Geoservicio REST | Feature Layer en ArcGIS Enterprise | — |
| Dashboard web | Experience Builder (MARN Hub) | — |

---

## 📊 Uso institucional

Los productos de esta metodología apoyan directamente:

- 📌 **Estadísticas históricas de ganancias de cobertura forestal** (REDD+)
- 📌 **Monitoreo de áreas bajo incentivos** PROBOSQUE y PINPEP
- 📌 **Priorización de territorios** para asistir la regeneración natural
- 📌 **Análisis comparativo** por municipio, departamento, cuenca y área protegida
- 📌 **Reporte NDC** — contribuciones nacionales determinadas de Guatemala

---

## ✅ Estado del proyecto

| Entregable | Estado |
|-----------|--------|
| Pregunta formulada y alcance | ✅ Completado |
| Lista de datos base | ✅ Completado |
| Diagrama de flujo | ✅ Completado |
| Especificaciones BD, geoservicios y despliegue | ✅ Completado |
| Wireframe de visualización | ✅ Completado |
| Implementación en producción | 🔄 En desarrollo |

---

## 🤝 Instituciones colaboradoras

| Institución | Rol |
|------------|-----|
| **MARN** — Ministerio de Ambiente y Recursos Naturales | Coordinación técnica, infraestructura de despliegue |
| **INAB** — Instituto Nacional de Bosques | Polígonos de incentivos, validación de campo |
| **CONAP** — Consejo Nacional de Áreas Protegidas | Capas de áreas protegidas, validación |

---

## 📄 Citación

Si utilizas esta metodología o sus productos en trabajos académicos o institucionales, por favor citar como:

```
MARN / INAB / CONAP (2026). Detección de restauración y recuperación de cobertura
forestal mediante SAR y persistencia de índices de vegetación en Guatemala.
Repositorio técnico institucional. Guatemala.
```

---

## 📬 Contacto

**Dirección de Espacios Naturales, Infraestructura Verde y Cuencas (DESI)**  
Ministerio de Ambiente y Recursos Naturales — Guatemala  
🌐 [marn.gob.gt](https://www.marn.gob.gt)
