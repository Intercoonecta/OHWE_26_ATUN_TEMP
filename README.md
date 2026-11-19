# Relación de anomalías mensuales de temperatura superficial del mar con la captura por lance de atún aleta amarilla (Thunnus albacares) en la zona del Domo Térmico de Costa Rica

## Integrantes

- Jesús C. Compaire
- Barbara Zavala
- Silvia Ramírez

## Herramientas utilizadas

El trabajo se desarrolla en Python mediante cuadernos de Jupyter.

Bibliotecas utilizadas:

- pandas: carga y organización de los datos.
- matplotlib: elaboración de gráficos.
- cartopy: elaboración de mapas.

La consulta de los límites marítimos requiere conexión a internet.

## Descripción

Este proyecto explora datos históricos de captura y esfuerzo pesquero
de atún aleta amarilla (Thunnus albacares). Se busca relacionar la
captura por lance con las anomalías mensuales de temperatura superficial
del mar en un área de referencia del Domo Térmico de Costa Rica.

## Pregunta de investigación

¿Cómo se relacionan las anomalías mensuales de temperatura superficial
del mar con la captura por lance de atún aleta amarilla en el área de
estudio del Domo Térmico de Costa Rica durante el período histórico
con datos coincidentes?

## Área de estudio

Se utiliza un rectángulo entre 7–11° de latitud norte y 87–93° de
longitud oeste, tomando como referencia el núcleo climatológico
descrito por Shi y Wang (2024).

Esta área fija permite comparar los registros entre años. No representa
el límite exacto del Domo en cada fecha ni corresponde exclusivamente
a aguas costarricenses.

## Datos pesqueros

La fuente es la Comisión Interamericana del Atún Tropical (CIAT).

Cada fila agrupa información por año, mes, cuadrícula de 1° × 1°
y tipo de lance.

Variables utilizadas:

- Year: año.
- Month: mes.
- SetType: tipo de lance.
- LatC1 y LonC1: coordenadas del centro de la cuadrícula.
- NumSets: número de lances.
- YFT: captura de aleta amarilla en toneladas métricas.

La captura por lance se calcula como YFT / NumSets, para registros
con NumSets mayor que cero. Se conservan las capturas de valor cero.

## Archivos

- Exploracion_atun_CR.ipynb: cuaderno de exploración de datos.
- IntroducciónP3.ipynb: cuaderno adicional de trabajo.
- atun_aleta_amarilla_region.csv: subconjunto de la selección regional inicial.
- atun_domo_aleta_amarilla.csv: subconjunto correspondiente al área de estudio del Domo.

## Cómo utilizar los datos

Descargar el cuaderno y el CSV necesario en la misma carpeta.
Abrir el cuaderno en Jupyter y ejecutar las celdas en orden.

Para cargar directamente el subconjunto del Domo:

    import pandas as pd
    atun_domo = pd.read_csv("atun_domo_aleta_amarilla.csv")
    atun_domo.head(6)

Las celdas que parten de la base completa requieren también el archivo
PublicPSTunaSetType.csv, disponible en el portal de la CIAT.

## Alcance y estado del proyecto

El proyecto está en desarrollo. El período final se definirá según
la coincidencia entre los datos pesqueros y los de temperatura.

La captura por lance representa rendimiento pesquero y no una medición
directa de abundancia. El análisis permitirá evaluar asociaciones,
sin demostrar por sí solo relaciones causales.

## Fuentes

- CIAT, datos públicos:
  https://www.iattc.org/en-US/Data/Public-domain

- Shi, W., & Wang, M. (2024). Ocean variability in the Costa Rica
  Thermal Dome region from 2012 to 2021. Remote Sensing, 16(8), 1340.
  https://doi.org/10.3390/rs16081340
