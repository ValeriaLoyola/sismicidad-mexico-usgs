# Análisis de la actividad sísmica en México y su entorno

Proyecto de análisis exploratorio y aprendizaje no supervisado sobre eventos sísmicos de magnitud **M ≥ 5** en México y su entorno.

**Autora:** Karla Valeria Loyola Alarcón  
**Periodo analizado:** 1 de enero de 2015 al 19 de septiembre de 2026  
**Fuente:** USGS Earthquake Catalog (ANSS Comprehensive Earthquake Catalog, ComCat)  
**Herramienta principal:** R / R Markdown  
**Visualización final:** Flexdashboard

---

## 1. Objetivo

El objetivo del proyecto es estudiar si los eventos sísmicos de magnitud **M ≥ 5** presentan perfiles estadísticos diferenciados a partir de su **localización y profundidad**, y evaluar posteriormente si dichos perfiles también muestran diferencias en magnitud.

El proyecto tiene un alcance **descriptivo, exploratorio y de aprendizaje no supervisado**. No pretende predecir cuándo ni dónde ocurrirá un sismo.

---

## 2. Pregunta de investigación

> **¿Se pueden identificar perfiles estadísticos diferenciados de sismicidad de magnitud ≥ 5 en México y su entorno a partir de la localización y profundidad de los eventos, y presentan esos perfiles diferencias en magnitud?**

La magnitud **no se utiliza para construir los clústeres**. Se reserva como una variable externa para caracterizar y contrastar posteriormente los perfiles obtenidos.

---

## 3. Datos

Los datos se obtuvieron del **USGS Earthquake Catalog**, consultando registros del catálogo integral de USGS, **ComCat**.

### Criterios de extracción

- **Magnitud mínima:** 5
- **Periodo:** 2015-01-01 a 2026-09-19
- **Latitud:** 14° a 33° N
- **Longitud:** -118° a -86°
- **Unidad de análisis:** un evento sísmico
- **Base analítica final:** 422 eventos

El archivo utilizado para reproducir exactamente el análisis se conserva en:

```text
data/raw/usgs_sismos_2015_2026-09-19.csv
```

Se guarda una copia local porque el catálogo de USGS puede actualizar posteriormente algunos parámetros de los eventos.

---

## 4. Preparación y calidad de los datos

Antes del análisis se revisaron:

- estructura y tipos de variables;
- valores faltantes;
- IDs duplicados;
- consistencia de fechas y coordenadas;
- profundidades negativas;
- eventos fuera del dominio definido;
- valores atípicos;
- heterogeneidad del tipo de magnitud (`magType`).

No se eliminaron observaciones únicamente por ser valores atípicos, ya que un evento extremo puede ser válido y relevante desde el punto de vista sísmico.

No fue necesario excluir registros por ausencia en las variables esenciales utilizadas en el análisis.

---

## 5. Metodología

### Análisis exploratorio

Se estudió:

- distribución de magnitud;
- distribución de profundidad;
- frecuencia acumulada conforme aumenta el umbral de magnitud;
- distribución espacial;
- comportamiento anual de los eventos;
- relación entre profundidad y magnitud.

Para evaluar la relación entre profundidad y magnitud se utilizó **correlación de Spearman**.

### K-Means

K-Means se utilizó como técnica principal de agrupamiento.

Variables utilizadas:

- coordenada proyectada **X**;
- coordenada proyectada **Y**;
- **profundidad**.

Las coordenadas geográficas se transformaron a **EPSG:6372 (Mexico ITRF2008 / Lambert Conformal Conic)** y las variables fueron estandarizadas antes del agrupamiento.

La cantidad de clústeres se evaluó mediante:

- método del codo;
- Silhouette;
- Gap Statistic;
- parsimonia e interpretabilidad.

La solución final utiliza **K = 3**.

### Validación externa con magnitud

La magnitud se analizó después de formar los perfiles mediante:

- prueba de **Kruskal-Wallis**;
- tamaño de efecto **epsilon cuadrada**;
- comparaciones post hoc de **Wilcoxon con ajuste de Holm**.

### DBSCAN

DBSCAN se utilizó como **análisis de sensibilidad**, aplicándolo sobre las mismas variables estandarizadas que K-Means.

Su objetivo no es reemplazar automáticamente a K-Means, sino comprobar si la estructura de tres perfiles se reproduce bajo un método basado en densidad.

---

## 6. Principales resultados

- Se analizaron **422 eventos**.
- La magnitud media fue aproximadamente **5.40** y la mediana **5.30**.
- La profundidad media fue aproximadamente **36.75 km** y la mediana **16.96 km**.
- Aproximadamente **91.7%** de los eventos tuvo magnitud menor a 6.
- Aproximadamente **85.5%** de los eventos ocurrió a menos de 70 km de profundidad.
- La relación profundidad–magnitud fue muy débil:
  - **Spearman ρ ≈ -0.09**
  - **p ≈ 0.064**
- K-Means identificó **3 perfiles** con una Silhouette promedio de aproximadamente **0.528**.
- Las profundidades medianas de los perfiles fueron aproximadamente:
  - **Perfil 1:** 22 km
  - **Perfil 2:** 103.7 km
  - **Perfil 3:** 10 km
- La magnitud presentó diferencias globales entre perfiles, pero con tamaño de efecto pequeño:
  - **Kruskal-Wallis p ≈ 0.0009**
  - **ε² ≈ 0.029**
- DBSCAN encontró **2 grupos** y **8 eventos de ruido**; 406 de los 422 eventos quedaron concentrados en un solo grupo de densidad.

### Conclusión principal

Los perfiles identificados se diferencian principalmente por **localización y profundidad**, no por magnitud.

K-Means resulta útil como una herramienta de **segmentación descriptiva**, pero los tres perfiles no deben interpretarse como tres poblaciones sísmicas naturales ni como regiones tectónicas oficiales.

---

## 7. Contexto tectónico

Los perfiles se comparan visualmente con límites de placas del modelo **PB2002**.

Este contexto tectónico se utiliza únicamente como referencia externa. Los límites de placas **no participan en la formación de los clústeres** y no se utiliza la coincidencia espacial para afirmar causalidad tectónica.

El archivo utilizado se encuentra en:

```text
data/external/PB2002_boundaries.json
```

---

## 8. Estructura del repositorio

```text
sismicidad-mexico-usgs/
│
├── ANALISIS_SISMOS_FINAL.Rmd
├── ANALISIS_SISMOS_FINAL.html
├── Flexdashboard_Sismos_EXPOSICION_FINAL.Rmd
├── Flexdashboard_Sismos_EXPOSICION_FINAL.html
├── README.md
├── .gitignore
│
├── data/
│   ├── raw/
│   │   └── usgs_sismos_2015_2026-09-19.csv
│   ├── external/
│   │   └── PB2002_boundaries.json
│   └── processed/
│       └── sismos_dashboard.csv
│
└── docs/
    ├── index.html
    └── reporte.html
```

### Descripción de archivos

- `ANALISIS_SISMOS_FINAL.Rmd`: análisis completo y reproducible.
- `ANALISIS_SISMOS_FINAL.html`: reporte renderizado.
- `Flexdashboard_Sismos_EXPOSICION_FINAL.Rmd`: código fuente de la visualización final.
- `Flexdashboard_Sismos_EXPOSICION_FINAL.html`: dashboard renderizado localmente.
- `data/raw/`: copia del catálogo utilizada en el proyecto.
- `data/external/`: información tectónica externa.
- `data/processed/sismos_dashboard.csv`: base procesada generada por el análisis y utilizada por el Flexdashboard.
- `docs/index.html`: copia del Flexdashboard utilizada para GitHub Pages.
- `docs/reporte.html`: copia del reporte final publicada junto al dashboard.

---

## 9. Requisitos

Se recomienda trabajar desde un **R Project** y ejecutar el proyecto desde su carpeta raíz.

### Paquetes utilizados en el análisis

```r
install.packages(c(
  "tidyverse",
  "lubridate",
  "knitr",
  "cluster",
  "dbscan",
  "maps",
  "sf"
))
```

### Paquetes adicionales utilizados en el Flexdashboard

```r
install.packages(c(
  "flexdashboard",
  "scales"
))
```

---

## 10. Cómo reproducir el proyecto

### Paso 1. Descargar o clonar el repositorio

Abrir el proyecto desde la carpeta raíz.

### Paso 2. Verificar los archivos de datos

Deben existir:

```text
data/raw/usgs_sismos_2015_2026-09-19.csv
data/external/PB2002_boundaries.json
```

### Paso 3. Renderizar el análisis

Abrir:

```text
ANALISIS_SISMOS_FINAL.Rmd
```

y ejecutar **Knit**.

Este archivo:

1. lee la copia local del catálogo;
2. realiza la preparación y el EDA;
3. ajusta K-Means;
4. ejecuta la validación de magnitud;
5. realiza el análisis de sensibilidad con DBSCAN;
6. genera la base procesada:

```text
data/processed/sismos_dashboard.csv
```

### Paso 4. Renderizar el Flexdashboard

Una vez generado el archivo procesado, abrir:

```text
Flexdashboard_Sismos_EXPOSICION_FINAL.Rmd
```

y ejecutar **Knit**.

El dashboard utiliza:

```text
data/processed/sismos_dashboard.csv
```

por lo que el análisis debe ejecutarse primero.

---

## 11. Visualización publicada

El Flexdashboard se publica mediante **GitHub Pages** usando:

```text
docs/index.html
```

Una vez activado GitHub Pages, la dirección tendrá la forma:

```text
https://TU_USUARIO.github.io/sismicidad-mexico-usgs/
```

El reporte completo puede publicarse como:

```text
https://TU_USUARIO.github.io/sismicidad-mexico-usgs/reporte.html
```

> Sustituir `TU_USUARIO` por el nombre de usuario de GitHub una vez configurado GitHub Pages.

---

## 12. Limitaciones

Entre las principales limitaciones del proyecto se encuentran:

- el dominio espacial es una caja rectangular y no coincide exactamente con la frontera política de México;
- solo se analizan eventos de magnitud M ≥ 5;
- 2026 es un año parcial;
- el catálogo de USGS puede actualizar parámetros preferidos;
- existen distintos tipos de magnitud (`magType`);
- K-Means favorece agrupamientos compactos;
- DBSCAN es sensible a `eps` y `minPts`;
- el análisis no modela explícitamente la dependencia entre eventos principales y réplicas;
- los límites tectónicos utilizados son una referencia bidimensional y no incorporan la geometría tridimensional de la placa subducida;
- los resultados no constituyen un modelo de pronóstico sísmico.

---

## 13. Reproducibilidad

Para conservar la reproducibilidad del proyecto se mantienen:

- la copia local del catálogo utilizada;
- los archivos externos;
- el código completo del análisis;
- el código del Flexdashboard;
- la base procesada utilizada para la visualización.

También puede añadirse posteriormente un archivo `renv.lock` para fijar versiones específicas de paquetes de R.

---

## 14. Uso de inteligencia artificial

Durante el desarrollo se utilizaron herramientas de inteligencia artificial como apoyo para:

- revisión de código;
- depuración de errores;
- organización metodológica;
- mejora de redacción;
- diseño de visualizaciones.

Las decisiones analíticas, interpretación de resultados y validación del contenido forman parte del desarrollo del proyecto.

---

## 15. Fuentes principales

- U.S. Geological Survey (USGS), Earthquake Catalog / ComCat.
- Bird, P. (2003). *An updated digital model of plate boundaries*. Geochemistry, Geophysics, Geosystems, 4, 1027.
- Instituto de Geofísica, UNAM.
- EPSG:6372 — Mexico ITRF2008 / Lambert Conformal Conic.

---

## Autora

**Karla Valeria Loyola Alarcón**
