Inteligencia GeoTemporal y Redes
================================

Este repositorio contiene el desarrollo de una serie de tareas prácticas de análisis de datos geo-temporales, procesamiento de señales, análisis de grafos y modelado predictivo aplicados a dos dominios principales:

- **Sistema eléctrico** (serie `ener_*`)
- **Sistema agro-meteorológico** (serie `agro_*`)

El trabajo está organizado en cuatro notebooks principales (Fase 1 a Fase 4), que van desde la exploración y limpieza hasta el modelado ARIMAX con variables topológicas de red.


Estructura del proyecto
-----------------------

- `Fase1_GeoVisualizacion_Clean.ipynb`  
	Exploración y geovisualización básica de los datos limpios de energía y agro.

- `Fase2_ProcesamientoSenales.ipynb`  
	Procesamiento de señales unidimensionales:
	- Análisis espectral (FFT, densidad espectral de potencia, espectrogramas) para comparar series limpias (`ener_clean.csv`) y ruidosas (`ener_noise.csv`).
	- Identificación de bandas de frecuencia donde el ruido inyectado es dominante.
	- Diseño y aplicación de filtros pasa-bajos (Butterworth) sobre series agro (`Agro_3`), con evaluación mediante RMSE frente a la serie limpia (`agro_clean.csv`).

- `Fase3_AnalisisGrafos.ipynb`  
	Construcción y análisis de redes dirigidas a partir de los archivos:
	- `Data/ener_clean.csv`, `Data/ener_noise.csv`
	- `Data/agro_clean.csv`, `Data/agro_noise.csv`
	Se construyen grafos dirigidos `Source_Node -> Target_Node`, se calculan centralidades (grado y betweenness) y se identifican nodos cuello de botella para cada dataset.

- `Fase4_Modelado.ipynb`  
	Integración de análisis de redes, causalidad temporal y modelado predictivo:
	- **P1 – Causalidad y Redes**:  
		Construcción del grafo de energía a partir de `ener_clean.csv`, cálculo de betweenness, identificación del nodo cuello de botella y prueba de causalidad de Granger entre `Ener_10` (Factor de Potencia) y `Ener_9` (Voltaje). Se simula la falla del nodo con mayor betweenness y se evalúa el impacto en la conectividad (componentes fuertemente conexas).
	- **P2 – Optimización Geo-Agrónoma**:  
		Uso de `agro_noise.csv` para filtrar ruido GPS (suavizado rolling), definir un proxy de pendiente y una medida de varianza de viento (`Agro_10`). Se localizan sensores con bajo NDVI (`Agro_5`) y se analiza su coincidencia con zonas de alta pendiente/alta varianza de viento, generando una recomendación automática de inversión en infraestructura hídrica y rompevientos.
	- **P3 – Analítica Predictiva**:  
		Ajuste de modelos ARIMAX para la demanda (`Ener_1`) con exógenas: temperatura (`Ener_3`) y centralidad de betweenness del nodo de origen. Se comparan los AIC de un modelo base (solo temperatura) y de un modelo extendido (temperatura + centralidad) para evaluar si la importancia topológica mejora el ajuste.


Datos
-----

La carpeta `Data/` contiene los archivos CSV utilizados en todas las fases:

- `ener_clean.csv`, `ener_noise.csv` – series energéticas limpias y con ruido.
- `agro_clean.csv`, `agro_noise.csv` – series agro-meteorológicas limpias y con ruido.

Cada archivo incluye medidas físicas (potencia, tensión, temperatura, NDVI, viento, etc.), coordenadas GPS (`Latitude`, `Longitude`) y columnas de red (`Source_Node`, `Target_Node`).


Entorno de ejecución
--------------------

Se recomienda usar un entorno virtual de Python dentro del proyecto (`.venv`). El notebook `Fase4_Modelado.ipynb` ya está configurado para usar el kernel `.venv` (Python 3.14.x).

Paquetes principales utilizados:

- `numpy`, `pandas`
- `matplotlib`, `plotly`
- `scipy` (FFT, filtros, espectrogramas)
- `networkx` (análisis de grafos)
- `statsmodels` (ADF, Granger, SARIMAX/ARIMAX)
- `jupyter`, `ipykernel`

Instalación básica en `.venv` (ejemplo en PowerShell):

1. Crear y activar entorno (si no existe):

	 - `python -m venv .venv`
	 - `.venv\\Scripts\\Activate.ps1`

2. Instalar dependencias mínimas:

	 - `pip install numpy pandas matplotlib scipy networkx statsmodels plotly jupyter ipykernel`

3. Registrar el kernel (opcional si ya aparece en VS Code):

	 - `python -m ipykernel install --user --name .venv --display-name ".venv"`


Cómo ejecutar las notebooks
---------------------------

1. Abrir el proyecto en VS Code.
2. Asegurarse de que el entorno `.venv` está activado o seleccionado como kernel en cada notebook.
3. Ejecutar los notebooks en el siguiente orden recomendado:
	 - Fase 1 (visualización)
	 - Fase 2 (procesamiento de señales)
	 - Fase 3 (análisis de grafos)
	 - Fase 4 (modelado y generación de informe)

En `Fase4_Modelado.ipynb`, se sugiere ejecutar las celdas en orden (importes, P1, P2, P3 y la celda de resumen de informe) para obtener todos los resultados actualizados.


Resultados clave
----------------

- Identificación de bandas de frecuencia donde el ruido inyectado es dominante en las series energéticas.
- Evaluación del efecto de filtros pasa-bajos sobre señales agro y su impacto en el RMSE frente a la serie limpia.
- Detección de nodos cuello de botella en las redes de energía y agro mediante betweenness centrality.
- Verificación de causalidad de Granger entre variables energéticas (`Ener_10 -> Ener_9`).
- Análisis espacial del bajo NDVI en relación con relieve y viento, con una recomendación automática de inversión.
- Comparación de modelos ARIMAX con y sin centralidad del nodo de origen para la predicción de la demanda.

# Inteligencia_GeoTemporal_y_redes