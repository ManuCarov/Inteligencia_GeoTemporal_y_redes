# Informe Técnico

## 1. Descripción general

Este informe resume los resultados obtenidos en las cuatro fases del proyecto de Inteligencia GeoTemporal y Redes, aplicados a las series energéticas (`ener_*`) y agro-meteorológicas (`agro_*`). Se destacan los hallazgos principales en procesamiento de señales, análisis de grafos y modelado predictivo.

---

## 2. Fase 1 – Geo-visualización y exploración de datos

- Se trabajó inicialmente con las bases de datos limpias `ener_clean.csv` y `agro_clean.csv`.
- Se realizó una exploración descriptiva de las variables clave (demanda, generación, temperatura, NDVI, viento, etc.), incluyendo:
  - Estadísticos básicos (media, desviación estándar, rangos) y revisión de valores atípicos.
  - Inspección de valores faltantes y consistencia temporal de las series.
- A partir de las columnas de coordenadas (`Latitude`, `Longitude`) se generaron visualizaciones geo-referenciadas para ubicar espacialmente los nodos/sensores tanto energéticos como agro-meteorológicos.
- Se construyeron gráficas de series temporales y mapas temáticos que permiten identificar patrones básicos de comportamiento (tendencias, variaciones diarias, zonas de mayor/menor intensidad de las variables).
- El objetivo principal de esta fase fue entender la estructura básica de los datos y establecer una línea base limpia sobre la cual aplicar, en fases posteriores, técnicas de procesamiento de señales, análisis de redes y modelado.

---

## 3. Fase 2 – Procesamiento de señales

### 2.1 Tarea 3 – Análisis espectral energía (Ener_4)

- Se analizaron las series de generación eólica `Ener_4` en sus versiones limpia (`ener_clean.csv`) y con ruido (`ener_noise.csv`).
- Mediante FFT se estimó la densidad espectral de potencia (PSD) de ambas series y se calculó la diferencia `PSD_noise - PSD_clean` en dB.
- Se generaron espectrogramas (tiempo–frecuencia) para ambas señales y un espectrograma diferencial, lo que permitió:
  - Identificar que el ruido inyectado se concentra principalmente en ciertas bandas de frecuencia, que aparecen con mayor energía en la serie ruidosa respecto a la limpia.
  - Visualizar claramente las zonas tiempo–frecuencia donde la potencia adicional asociada al ruido es más marcada.
- Con base en estos análisis se pudo responder a la pregunta de examen sobre en qué rangos de frecuencia se concentra la mayor parte del ruido inyectado, apoyándose tanto en la PSD diferencial como en el espectrograma diferencial.

### 2.2 Tarea 4 – Filtrado Butterworth en Agro_3

- Señal de interés: humedad relativa `Agro_3` en sus versiones limpia (`agro_clean.csv`) y con ruido (`agro_noise.csv`).
- Se diseñó un filtro pasa-bajos Butterworth de orden 4 con frecuencia de corte de 5 Hz y frecuencia de muestreo de 2000 Hz, y se aplicó a la serie ruidosa (`Agro_3_noise`).
- Para evaluar el efecto del filtrado se calculó el RMSE respecto a la serie limpia:
  - RMSE entre limpia y ruidosa (antes del filtro): **≈ 3.34**
  - RMSE entre limpia y filtrada (después del filtro): **≈ 4.28**
- Conclusión: con estos parámetros de filtro, el RMSE **empeora** tras el filtrado (aumenta de ~3.34 a ~4.28). Es decir, el filtro pasa-bajos, tal como fue configurado, no mejora la aproximación a la señal original limpia y, por tanto, no incrementa la capacidad predictiva basada en esa serie. Sería necesario re-ajustar la frecuencia de corte y/o el orden para obtener beneficios reales.

---

## 4. Fase 3 – Análisis de grafos

- A partir de las columnas `Source_Node` y `Target_Node` de los cuatro archivos:
  - `Data/ener_clean.csv`, `Data/ener_noise.csv`
  - `Data/agro_clean.csv`, `Data/agro_noise.csv`
  se construyeron grafos dirigidos para cada dataset.
- Se eliminaron filas con `Source_Node` o `Target_Node` nulos para evitar nodos `NaN` en los grafos.
- Para cada grafo se calcularon:
  - Centralidad de grado (degree centrality).
  - Centralidad de intermediación (betweenness centrality).
- Se identificó para cada dataset el nodo cuello de botella como aquel con mayor betweenness, generando una tabla-resumen de nodos críticos. Estos nodos son candidatos a producir mayor impacto si fallan, dado que concentran gran parte de los caminos más cortos de la red.

---

## 5. Fase 4 – Modelado

### 4.1 P1 – Causalidad y Redes (energía)

- Se construyó un grafo dirigido de la red eléctrica a partir de `ener_clean.csv`.
- El grafo resultante tiene:
  - **70 nodos** y **865 aristas**.
- Se calcularon las centralidades y se identificó el nodo cuello de botella como aquel con mayor betweenness (en la ejecución actual, el nodo con mayor betweenness aparece como el nodo **119**).

#### 4.1.1 Causalidad de Granger (Ener_10 → Ener_9)

- Se analizó la posible causalidad de Granger desde el **Factor de Potencia** (`Ener_10`) hacia el **Voltaje** (`Ener_9`).
- Se construyó un modelo bivariado [Ener_9, Ener_10] y se aplicaron pruebas de Granger para rezagos de 1 a 4.
- Resumen de p-values del test `ssr_ftest`:
  - lag 1: p ≈ 0.30
  - lag 2: p ≈ 0.34
  - lag 3: p ≈ 0.43
  - lag 4: p ≈ 0.019
- Conclusión: en al menos un rezago (lag 4) el p-value es < 0.05, por lo que **sí hay evidencia de causalidad de Granger** de `Ener_10` hacia `Ener_9`; es decir, el factor de potencia ayuda a predecir el voltaje en la red bajo este modelo.

#### 4.1.2 Falla del nodo cuello de botella

- Se simuló la falla del nodo con mayor betweenness eliminándolo del grafo y se comparó la cantidad de componentes fuertemente conexas antes y después:
  - Antes de la falla: **70** componentes fuertemente conexas.
  - Después de eliminar el nodo cuello de botella: **69** componentes fuertemente conexas.
- Interpretación: lejos de fragmentar la red en más componentes, la eliminación de ese nodo reduce ligeramente el número de componentes fuertemente conexas. En este caso particular, el nodo identificado no actúa como un puente crítico que dispare la fragmentación de la red; su falla no incrementa el número de islas fuertemente conectadas y, por tanto, no genera un aumento significativo del riesgo estructural según esta métrica.

---

### 4.2 P2 – Optimización Geo-Agrónoma (agro_noise)

- Se trabajó con el dataset agro-meteorológico ruidoso `agro_noise.csv`.
- Se aplicó un suavizado (media móvil) a las coordenadas GPS (`Latitude`, `Longitude`) para filtrar ruido de posicionamiento y se definió un indicador de pendiente (`slope_proxy`) basado en la magnitud del gradiente espacial.
- Se calculó la varianza móvil del viento (`Agro_10`) como indicador de estrés eólico.
- Se definieron sensores de **bajo NDVI** a partir del percentil 10 de `Agro_5`:
  - Umbral NDVI (10 %): **≈ 0.6967**.
  - Número de observaciones con NDVI ≤ umbral: **200**.
- Sobre este subconjunto de bajo NDVI:
  - Proporción ubicada en zonas de alta pendiente (≥ percentil 75 de `slope_proxy`): **≈ 25.5 %**.
  - Proporción en zonas de alta varianza de viento (≥ percentil 75 de `wind_var`): **≈ 32.5 %**.
- Conclusión principal:
  - No se observa que la **mayoría** de los puntos de bajo NDVI coincida simultáneamente con alta pendiente y alta varianza de viento. El solapamiento existe, pero no es masivo.
  - La recomendación automática generada es **no** realizar una intervención homogénea en toda la finca, sino **priorizar microzonas** donde coinciden bajo NDVI, mayor pendiente y mayor variabilidad del viento, concentrando allí la inversión en infraestructura hídrica (riego localizado, micro-reservorios) y rompevientos.

---

### 4.3 P3 – Analítica Predictiva (ARIMAX con centralidad)

- Variable dependiente (endógena): **Demanda** (`Ener_1`).
- Variables exógenas consideradas:
  - Modelo 1 (base): Temperatura (`Ener_3`).
  - Modelo 2 (extendido): Temperatura (`Ener_3`) + Centralidad de betweenness del nodo de origen (`centrality`, obtenida a partir de `Source_Node` y el grafo de energía de P1).
- Antes del ajuste, se aplicó la prueba ADF a `Ener_1`, resultando **no estacionaria** al 5 %; por ello se trabajó con la serie diferenciada y exógenas alineadas.
- Se ajustaron dos modelos ARIMAX (implementados vía SARIMAX sin estacionalidad) con orden `(1, 0, 1)`:
  - **Modelo sin centralidad** (Ener_1 ~ Ener_3):
    - AIC ≈ **8749.99**.
  - **Modelo con centralidad** (Ener_1 ~ Ener_3 + centrality):
    - AIC ≈ **8751.99**.
- Comparación de AIC:
  - El AIC **aumenta** al incluir la centralidad del nodo de origen (de 8749.99 a 8751.99).
  - Además, el coeficiente estimado para la variable de centralidad resulta prácticamente **nulo**, indicando que la topología local del nodo de origen, tal como se codificó, no aporta señal explicativa adicional en este esquema.
- Conclusión:
  - Bajo la especificación ARIMAX `(1,0,1)` usada, **no se observa mejora en el ajuste** al incorporar la importancia del nodo en el grafo como variable exógena. El modelo con centralidad es ligeramente peor según AIC, por lo que la centralidad, en esta configuración, no incrementa la capacidad predictiva sobre la demanda.

---

## 6. Conclusiones globales

- El análisis espectral permitió identificar las bandas de frecuencia donde el ruido inyectado afecta más intensamente a las series energéticas, facilitando la interpretación de la calidad de la señal y la relación señal–ruido.
- Los experimentos de filtrado sobre señales agro demostraron que el diseño de filtros debe estar cuidadosamente adaptado al contenido espectral de la señal; un filtro pasa-bajos mal elegido puede empeorar el ajuste frente a la serie limpia.
- El análisis de grafos permitió identificar nodos cuello de botella en todas las redes (energía y agro), aunque en el caso concreto del nodo de mayor betweenness en la red de energía, su falla no produjo una fragmentación severa según el número de componentes fuertemente conexas.
- La prueba de causalidad de Granger aportó evidencia de que el factor de potencia (`Ener_10`) ayuda a predecir el voltaje (`Ener_9`) en ciertos rezagos, reforzando la interpretación física del sistema eléctrico.
- En el ámbito geo-agro, el cruce de bajo NDVI con indicadores de pendiente y estrés eólico orientó la toma de decisiones hacia intervenciones focalizadas, en lugar de inversiones generalizadas.
- Finalmente, el contraste de modelos ARIMAX con y sin centralidad del nodo mostró que, para este conjunto de datos y especificación, la importancia topológica de los nodos no mejora la capacidad predictiva sobre la demanda, lo que sugiere explorar otras formas de integrar la información de red o probar órdenes/modelos alternativos.
