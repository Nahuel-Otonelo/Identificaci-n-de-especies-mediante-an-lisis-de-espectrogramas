# Reconocimiento de Especies de Aves a través del Análisis de Espectrogramas de Canto

Este repositorio documenta el **Análisis Exploratorio de Datos (EDA)** y la construcción del *Pipeline End-to-End* para el preprocesamiento de características acústicas (*Feature Extraction*) orientado a problemas de Visión Computacional aplicado al sonido. 

El fin último de este sistema es aislar, purgar y homogeneizar vocalizaciones de aves sudamericanas obtenidas de metadatos de campo, preparándolas para entregar **Tensores 3D matemáticamente perfectos** a redes neuronales convolucionales.

---

## 1. Naturaleza del Dataset
El proyecto procesa datos "salvajes" extraídos originalmente de la iniciativa **Xeno-canto**, estructurados en tres ejes fundamentales:

1. **`Metadatos Estructurales (.csv)`**: Un catálogo tabular brindando contexto geográfico, climático y logístico del audio.
2. **`Audios Crudos (.mp3)`**: Archivos altamente comprimidos capturados en naturaleza real. Esto expone al dataset a altos niveles de *Ruido Ambiental* como viento, agua, e interferencia antropogénica.
3. **`Etiquetas de la Verdad Fundamental (.svl)`**: Archivos XML (Sonic Visualiser Layer) generados colaborativamente por ornitólogos, informando la posición de la Bounding Box exacta donde una especie canta dentro de la línea de tiempo.

---

## 2. Desarrollo e Ingeniería del Pipeline de Features

A lo largo del proyecto, la extracción cruda arrojó severos desequilibrios en la naturaleza empírica de las grabaciones de campo. Se diseñó el siguiente flujo de procesamiento robusto para combatir inconsistencias geométricas y biológicas:

### A. Auditoría Estadística (Prevención de Overfitting)
Debido al desequilibrio natural en la distribución biológica del volumen de audios anotados por especie, procesar todas las aves arrojaba un ecosistema de predicciones altamente sesgado hacia clases con mucha muestra.
Se implementó un umbral poblacional duro (*Threshold = 100*). Toda especie minoritaria que no logre dicha representación exclusiva en el grupo de Entrenamiento (Train) es destruida de la matriz de compilación.

### B. Homogeneización Forzada de Sample Rates (Upsampling)
Un error oculto generalizado de los datasets crowdsourceados como Xeno-canto es la asimetría de los micrófonos de los grabadores. Mezclar audios en frecuencias de *DVD (48000 Hz), CD (44100 Hz)* o *Teléfonos (22050 Hz)* provocaba que las matrices finales difirieran sutilmente en cantidad de pixeles longitudinales, provocando fallos por colisión geométrica en Numpy (*inhomogeneous shape*).
Se resolvió implementando un re-muestreo matemático por Interpolación de Señal en el nodo de origen. Al inyectar una lectura forzada de `sr=44100`, las imágenes de baja calidad son artificialmente infladas por Librosa, produciendo un Tensor unificado maestro.

### C. Estrategia de Padding por Centrado Continuo Orgánico
La red convolucional exige entradas matemáticamente idénticas. Se definió un tamaño hiper-paramétrico base de **6.0 segundos** para encapsular cualquier llamada.
* **El Problema del Padding Convencional**: Utilizar relleno de "ceros silenciosos" (Zero-Padding) viciaba a la IA enseñándole "bloques negros" en lugar de naturaleza. Concatenar recortes ajenos de ruido provocaba cortes perceptuales drásticos (rayas de fase anómalas).
* **La Solución (Continuous Centering)**: Para un canto de ej: 3.5 segundos, el algoritmo calcula su centro numérico y expande analíticamente +/- 3 segundos hacia cada lado *dentro de la continuidad del archivo maestro*. El ave queda permanentemente en el cenit de la matriz, rodeado de ruido natural del bosque con el que fluía armónicamente.

### D. Reducción Espectral y Nicho Acústico
Finalmente, el recorte orgánico sufre una *Short-Time Fourier Transform (STFT)* hacia el dominio visual en escala Mel evaluada bajo Decibelios de potencia (`n_mels=128`).
Apoyado en la **Hipótesis del Nicho Acústico**, el clasificador fuerza un techo y descarte del espectro en Frecuencia a **10.000 Hz** (`fmax=10000`). Entendiendo fisiológicamente a los pájaros paseriformes como vocalizadores de media banda (1.5 kHz a 8 kHz), esta eliminación ahorra poder computacional y desecha el ruido de fondo innecesario que emiten grandes enjambres de insectos, logrando una estilización muy aguda del contorno de las vocalizaciones aviares.

---




