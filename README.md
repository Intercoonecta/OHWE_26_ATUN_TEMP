# Temperatura superficial del mar y captura por lance de atún aleta amarilla en el Domo Térmico de Costa Rica

Equipo: 
Silvia Ramírez,
Barbara Závala y 
Jesús C. Compaire.

Este proyecto se desarrolló para la hackatón AICED 2026. Se intenta explora la relación entre las anomalías mensuales de temperatura superficial del mar y la captura por lance de atún aleta amarilla (*Thunnus albacares*), para ello se ultilizó datos pesqueros de la CIAT y temperatura de Copernicus Marine.

El análisis principal se realiza con valores mensuales representativos de una misma área. Como complemento, se compara la captura por lance entre las fases de un patrón regional de variación de temperatura, resumido mediante PC1.

## Pregunta de investigación

¿Cómo se relacionan las anomalías mensuales de temperatura superficial del mar con la captura por lance de atún aleta amarilla (*Thunnus albacares*) en el área de estudio del Domo Térmico de Costa Rica durante el período histórico con datos coincidentes?

## Área y período

- **Área de estudio:** rectángulo de 7 a 11° N y de 87 a 93° O. Àrea delimitada por Shi y Wang (2024) donde establecen el núcleo climatológico en este cuadrante.
- **Temperatura disponible:** enero de 1960 a junio de 2025, a 1 m de profundidad, como aproximación a la temperatura superficial.
- **Datos coincidentes utilizados:** se encontraron 720 meses entre abril de 1960 y mayo de 2025. Hay meses sin registros pesqueros; estos no se reemplazaron por ceros.
- **Referencia para las anomalías locales:** periodo comprendido entre 1991–2020, con 360 meses completos.

El rectángulo es una delimitación fija de estudio y no representa el borde exacto del Domo en cada fecha ya que durante el año se desplaza. La pesca se selecciona por centros de cuadrícula de 1°. La temperatura y la pesca se emparejan por mes dentro del rectángulo; NO se emparejan por cada ubicación de pesca.

## Archivos necesarios

| Archivo | Contenido y uso |
| --- | --- |
| `Exploracion_atun_CR.ipynb` | Cuaderno con exploración, mapas, análisis ambiental y modelos estadísticos. |
| `PublicPSTunaSetType.csv` | Registros de captura y esfuerzo pesquero de la CIAT. |
| `cmems_obs-ins_glo_phy-temp-sal_my_cora-oa_P1M_TEMP_96.00W-77.00W_2.10N-16.00N_1.00m_1960-01-01-2025-06-01.nc` | Temperatura mensual de Copernicus; el nombre debe coincidir exactamente con `archivo_nc` en el cuaderno. |

El cuaderno genera `atun_aleta_amarilla_region.csv` y `atun_domo_aleta_amarilla.csv` durante la exploración. No son entradas adicionales obligatorias si está disponible la base original.

El README no incluye ni descarga los archivos de datos. Deben estar disponibles localmente en la carpeta de trabajo del cuaderno. Si el NetCDF tiene un sufijo como `(1)`, se debe ajustar `archivo_nc` al nombre real, sin modificar sus datos.

## Bibliotecas utilizadas

Para el análisis estadístico y los gráficos se utilizan Python, pandas, NumPy, xarray, netCDF4, SciPy, Matplotlib y statsmodels. Los mapas y los EOF también requieren Cartopy, cmocean y eofs.

Si faltan bibliotecas, se pueden instalar desde una celda del cuaderno:

```python
%pip install pandas numpy xarray netCDF4 scipy matplotlib statsmodels cartopy cmocean eofs
```

La biblioteca `copernicusmarine` y una cuenta de Copernicus solo son necesarias para repetir la descarga opcional. No se necesitan para leer el NetCDF ya descargado.

## Cómo ejecutar la versión actual del cuaderno

**El cuaderno contiene salidas guardadas de una sesión interactiva. La comparación de fases está situada antes de la creación de algunas variables que utiliza. Por ello, no debe asumirse que ejecutar todas las celdas desde arriba funcionará sin reorganizarlas.**

Para ejecutar sin depender de variables antiguas, guardar el cuaderno y reiniciar el kernel. Después seguir este orden:

1. **Modelo de anomalías locales.** Ejecutar el bloque que comienza con el título `MODELO PARA ANOMALÍAS LOCALES`, desde las importaciones y la lectura de los dos archivos hasta la corrección HAC y su conclusión. Este bloque crea `datos_analisis`, `tiempo_anios` y los modelos, y carga pandas, xarray, Matplotlib y statsmodels.
2. **Preparación ambiental regional.** Volver a la celda que abre el NetCDF y dibuja los 12 mapas de climatología. Define `var_temp`, `climatologia_mensual` y los límites del rectángulo. Mantener `ds` abierto hasta terminar los cálculos: posponer la instrucción `ds.close()` para el final del trabajo.
3. **EOF y PC1.** Ejecutar la preparación de climatología regional, la celda que contiene `solver = Eof(...)`, el gráfico de EOF1 y PC1 y la celda que define las fases y los mapas compuestos.
4. **Comparación entre fases.** Ejecutar desde `tabla_fases = pc1.to_dataframe()` hasta el modelo de fases y sus intervalos HAC. Este bloque utiliza `datos_analisis` y `tiempo_anios`, que ya deben existir.
5. **Exploración y mapas adicionales.** Para reproducir la exploración pesquera inicial y el mapa de lances, ejecutar sus celdas en orden antes del bloque de descarga. El mapa de zonas económicas exclusivas consulta un servicio de ArcGIS; Cartopy puede necesitar descargar sus capas geográficas.

Las celdas de comprobación que imprimen si `pc1` existe son auxiliares; sus salidas antiguas no indican el estado actual del kernel.

**Celdas que se pueden omitir:**

- La lectura de `RONI.ascii.txt`: aún aparece en esta versión, pero RONI no se utiliza en ninguno de los dos modelos.
- La llamada a `copernicusmarine.subset(...)`: omitirla si ya está disponible el NetCDF; de lo contrario solicitará credenciales.

Las celdas se ejecutan con **Shift + Enter**. Al terminar, guardar el cuaderno para conservar las salidas actualizadas. Si se reorganiza para ejecutar todo de arriba hacia abajo, el bloque de comparación de fases debe quedar después de la creación de los datos mensuales y de PC1.

## Procedimiento del análisis principal

1. Seleccionar los registros pesqueros dentro del rectángulo del Domo.
2. Agrupar por año y mes, sumar captura y lances y calcular:

   **Captura por lance mensual = captura mensual total de YFT / lances mensuales totales.**

3. Seleccionar la temperatura a 1m  dentro del mismo rectángulo y comprobar valores faltantes.
4. Calcular una temperatura media espacial por mes, con pesos de coseno de latitud y pesos reducidos para las celdas de los bordes.
5. Calcular los 12 promedios mensuales de referencia de 1991–2020 y restarlos a cada mes de toda la serie para obtener anomalías en °C.
6. Unir pesca y temperatura por fecha, comprobando que cada fecha aparezca una sola vez en cada tabla.
7. Explorar distribuciones, gráficos, correlaciones de Pearson y Spearman, variación temporal y residuos.
8. Ajustar una regresión simple y después una regresión que considera el mes del año y la tendencia temporal lineal:

```python
captura_por_lance ~ anomalia_c + C(Month) + tiempo_anios
```

9. Calcular intervalos y valores p aproximados con errores HAC, rezago de 12 meses calendario, pesos Bartlett y aproximación normal. El índice temporal conserva los huecos entre meses.

El mes de agosto de 2017 contiene 230 toneladas en un lance. Se conserva en el modelo principal y se excluye únicamente en una comprobación de sensibilidad. No se incorpora el tipo de lance como predictor; la respuesta es captura por lance nominal con los tipos agrupados.

## Resultados principales

| Análisis | Resultado |
| --- | --- |
| Pearson, todos los meses | 0,101 |
| Pearson, sin agosto de 2017 | 0,133 |
| Spearman, todos los meses | 0,166 |
| Pendiente del modelo simple | +2,155 t/lance por °C |
| R² del modelo simple | 0,0101, aproximadamente 1,01 % |
| Pendiente del modelo ajustado | −1,061 t/lance por °C |
| IC95 % HAC de la pendiente ajustada | −2,489 a 0,366 |
| Valor p aproximado | 0,145 |
| R² del modelo ajustado completo | 0,0809, aproximadamente 8,09 % |
| Pendiente ajustada sin agosto de 2017 | −0,791 t/lance por °C |

No se encontró una relación lineal clara entre las anomalías de temperatura y la captura por lance del mismo mes, considerando el mes del año y la tendencia temporal. El R² del modelo ajustado corresponde al conjunto de predictores, no solo a la temperatura.

## Comparación complementaria por fases de PC1

PC1 resume la evolución temporal del principal patrón espacial de variación de temperatura. En este cuaderno se calcula con una región mayor, de 2 a 16° N y de 96 a 77° O, después de retirar la climatología mensual de todo el período y una tendencia lineal en cada punto. No es la misma variable que la anomalía local del Domo respecto a 1991–2020.

El primer modo representa el **42,6 % de la variación del campo de temperatura preparado**, no de las capturas. PC1 se escala a varianza unitaria y no se expresa en °C. Se agrupan los meses como positivos cuando PC1 supera una desviación estándar, negativos cuando queda por debajo de menos una y neutros entre ambos límites.

Las fases no equivalen automáticamente a El Niño o La Niña. En la orientación del mapa EOF1 obtenido, los valores positivos dentro del Domo corresponden a una señal de calentamiento del modo y los negativos a la señal inversa. El modo representa solo parte de la variabilidad térmica.

| Fase | Meses coincidentes con pesca | Media de captura por lance | Mediana de captura por lance |
| --- | ---: | ---: | ---: |
| Negativa | 78 | 11,76 | 10,79 |
| Neutra | 545 | 12,77 | 11,25 |
| Positiva | 97 | 12,20 | 11,96 |

Las medias y medianas se expresan en t/lance y dan el mismo peso a cada mes.

Se comparan las fases mediante regresión, usando la negativa como referencia y ajustando por mes y tendencia, con la misma corrección HAC:

```python
captura_por_lance ~ C(fase, Treatment(reference='Negativa')) + C(Month) + tiempo_anios
```

| Comparación | Diferencia ajustada en t/lance | IC95 % HAC | Valor p |
| --- | ---: | --- | ---: |
| Neutra frente a negativa | −0,308 | −2,148 a 1,531 | 0,743 |
| Positiva frente a negativa | +0,019 | −2,355 a 2,393 | 0,987 |

No se encontraron diferencias claras de las fases positiva y neutra respecto a la negativa. Estos contrastes no constituyen una prueba de equivalencia, un contraste directo entre positiva y neutra ni una prueba global de todas las fases.

## Limitaciones

- Se estudia una asociación regional del mismo mes; no una relación causal ni una medida directa de abundancia de atún.
- La temperatura promedio del rectángulo puede diferir de la temperatura de las zonas donde efectivamente se pescó. El emparejamiento por cuadrícula y mes queda como posible siguiente análisis.
- La captura por lance es nominal y no controla completamente cambios de estrategia, tecnología o composición de la pesquería.
- No se evaluaron respuestas curvas ni desfases entre temperatura y pesca.
- Los intervalos no incorporan toda la incertidumbre de la reconstrucción ambiental, la climatología ni la estimación y clasificación de PC1.
- Las salidas guardadas muestran los resultados de la sesión; no sustituyen una ejecución completa en orden desde un kernel reiniciado.

## Fuentes

- [CIAT — Datos públicos](https://www.iattc.org/en-US/Data/Public-domain) y [códigos de referencia](https://www.iattc.org/es-es/Data/Reference-codes).
- [Copernicus Marine — Producto CORA de temperatura y salinidad mediante análisis objetivo](https://data.marine.copernicus.eu/product/INSITU_GLO_PHY_TS_OA_MY_013_052/description). Dataset utilizado: `cmems_obs-ins_glo_phy-temp-sal_my_cora-oa_P1M`.
- [OMM — Normales climatológicas](https://community.wmo.int/site/knowledge-hub/programmes-and-initiatives/climate-services/wmo-climatological-normals).
- Shi, W., & Wang, M. (2024). Ocean variability in the Costa Rica Thermal Dome region from 2012 to 2021. Remote Sensing, 16(8), 1340. https://doi.org/10.3390/rs16081340


