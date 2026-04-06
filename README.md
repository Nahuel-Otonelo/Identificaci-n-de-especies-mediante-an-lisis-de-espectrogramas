# Reconocimiento de Especies de Aves a través del Análisis de Espectrogramas de Canto

Este repositorio contiene el **Análisis Exploratorio de Datos (EDA)** y el *Pipeline* (canal de procesamiento) de preprocesamiento de un conjunto de datos bioacústicos de 22 especies de aves. El fin último de este trabajo es aislar, limpiar y preparar las señales de audio ambientales para ser entrenadas por redes neuronales convolucionales (Computer Vision) mediante la conversión del sonido a espectrogramas Mel en 2D.

## Descripción del Dataset
El proyecto se basa en datos extraídos originalmente de la librería **Xeno-canto** y anotados manualmente por expertos para aislar los cantos positivos. Se compone de tres pilares de datos:

1. **`Xenocanto_metadata_qualityA_selection.csv`**: Un catálogo tabular que provee contexto (país, altitud, fecha, etc.) de los audios originales grabados en el bosque.
2. **`Audio_birds2/`**: Los archivos originales en formato altamente comprimido (`.mp3`).
3. **`Annotation/`**: Anotaciones en formato Sonic Visualiser Layer (`.svl` - XML) que marcan la "Verdad Fundamental" (Ground Truth), señalando el tiempo exacto de inicio y la duración de cada vocalización en el audio.

## Proceso de Ingeniería y Preprocesamiento

### 1. Extracción Estructurada
Dado que cargar y procesar múltiples archivos de audio comprimido es costoso en memoria, se desarrolló una función de extracción que escanea y abre los miles de archivos `.svl` utilizando `xml.etree.ElementTree`. Se extrajeron las métricas más importantes (frame de inicio, duración, rango de frecuencias, y nombre del archivo original) generando un dataset maestro estructurado, serializado eficientemente en Pandas como `df_master.pkl`. 

### 2. Análisis Estadístico y Padding Dinámico (EDA)
Debido a la naturaleza aleatoria del canto (algunos duran 1 segundo, otros 10 segundos), estandarizamos el tamaño de las entradas bajo estas asunciones:
* Mediante un gráfico de barras horizontales basado en **cuantiles**, se logró determinar que la abrumadora mayoría de los cantos cabían en una ventana deslizable de **6.0 segundos**.
* Teniendo en cuenta que las Redes Neuronales requieren formas dimensionales (tensores) perfectamente estandarizadas. Recortamos o "Paddeamos" los audios para encajarlos forzosamente en esta ventana.

### 3. Evitando el Sobrentrenamiento (La Estrategia de Ruido)
Se implementó un diseño de *Overlapping Bounding Box*. En los recortes donde el pájaro cantaba muy pocos segundos, no se rellenó la matriz con "silencio absoluto digital o ceros puros". De haberlo hecho, el modelo habría creado fuerte dependencia a buscar bloques negros, siendo ineficaz en audios salvajes reales.
La estrategia aplicada tomó **ruido ambiental o de fondo originario de la misma pista acústica** ("Ambient Padding") para rellenar los 6 segundos estandarizados, creando así un modelo tremendamente más robusto.

### 4. Transformación de Audio a Imagen
Una vez logrado el arreglo unidimensional exacto de 6.0 segundos en la memoria RAM, se transformó este audio (utilizando la librería científica `librosa`) a un **Espectrograma Mel** evaluado en Decibeles y utilizando 128 bandas (`n_mels=128`). Esto simula auditivamente cómo las aves perciben naturalmente las altas y bajas frecuencias, devolviendo matrices puras de aspecto `[128 x N]`.

## Exportación para el Equipo de ML
Para facilitar la logística técnica con quienes continúan en la etapa de modeladop de arquitectura (*Trainning*):
* Se dividieron lógicamente las muestras entre Training y Validation respetando estrictamente las carpetas de los investigadores base (utilizando máscaras booleanas con un array `splits`).
* Se ejecutó el pre-procesamiento del "Pipeline", escupiendo archivos de clasificación categorizados nativos u organizados en forma de carpetas visuales directas.

Todo este código se encuentra ordenadamente implementado y disponible para replicarse paso a paso dentro del cuaderno principal: **`primera.ipynb`**.
