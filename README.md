# Reconocimiento de Especies de Aves a través del Análisis de Espectrogramas de Canto

Este repositorio documenta el **Análisis Exploratorio de Datos (EDA)** y la construcción integral del *Pipeline End-to-End* para el preprocesamiento de características acústicas (*Feature Extraction*) orientado a problemas de Deep Learning.

El fin último de este sistema es extraer, purgar y homogeneizar vocalizaciones de aves obtenidas de grabaciones de campo, preparándolas para entregar **Tensores 2D estandarizados (256x256 pixeles)** a las Redes Neuronales Convolucionales (CNN) más modernas.

---

## 1. Naturaleza del Dataset
El proyecto arranca procesando datos "salvajes" extraídos colaborativamente a través de la iniciativa **Xeno-canto**, presentados bajo tres grandes pilares:

1. **`Metadatos Estructurales (.csv)`**: Contexto posicional y natural general.
2. **`Audios Crudos (.mp3)`**: Archivos altamente comprimidos capturados en la intemperie (con viento, lluvia y animales de fondo).
3. **`Archivos de Anotación (.svl)`**: Capas de la Verdad Fundamental (Ground Truth) en formato XML brindadas por expertos. **Como pilar analítico de este proyecto, desarrollamos un parseo programático de este XML**, consolidando el "segundo exacto de inicio", la duración, las frecuencias de corte y la especie en un único mega-dataframe cruzado (`df_master`).

---

## 2. Ingeniería Convolutiva del Pipeline Maestro (V6)

Ante la inmensa desestructuración de audios de longitudes caóticas y calidades de compresión variadas, desarrollamos la siguiente arquitectura de software paso por paso:

### A. Ventaneo y Normalización Matemática
Basados en el EDA de las longitudes de los cantos de aves sudamericanas, definimos una caja limitadora estricta de **6.0 segundos** como el contenedor ideal para capturar la esencia frecuencial de cualquier especie.

### B. Cuadratura Directa al 256x256 (Downsampling Mágico)
Varios modelos icónicos del *Computer Vision* rinden mejor ingiriendo imágenes cuadradas exactas, ahorrando las destructivas funciones de re-escalado (`cv2.resize`):
* Al forzar el **remuestreo de todo el audio crudo a exactamente `21845 Hz`**.
* Ese *Sample Rate* interactuando durante 6.0 segundos produce una longitud de bytes tal, que la STFT divide exactamente en **`256` fotogramas/eje X**.
* Configuramos el Hiperparámetro Vertical `N_Mels = 256`. 
* **Resultado:** Salida nativa, paramétrica e inalterada de matrices en **256x256**.

### C. Data Augmentation por Autenticidad (Jitter Posicional)
En arquitecturas tempranas forzábamos a que los cantos (ej: cantos chicos de 2s) estuvieran estrictamente encuadrados en el centro geométrico del espectrograma, causándole a la Red Neuronal un sesgo grave de hiper-fijación espacial.
**La Solución:** Integramos un "Padding Estocástico Aleatorio". Para cajas vocales chicas el código ya no toma el "centro", sino que escupe el canto de forma tirada a la izquierda, o a la derecha, sumándole ruido orgánico original del MP3 alrededor. Le estamos enseñando empíricamente **invarianza traslacional** a la inteligencia artificial.

### D. Supresión del "Spectral Leakage" (Ventana de Hann)
Corroboramos internamente cómo la Transformada de Fourier recuenta infinitos al truncar ondas abruptamente en segmentos de 6 segundos causando "Falsa fuga de frecuencias" arriba y abajo del pájaro. Constatamos analíticamente que la función `librosa.feature.melspectrogram` inyecta nativamente la suavización en campana del legendario **Filtro de Hann**, librando a nuestro output de "Cliks" de ruido en los bordes. 

### E. Ecosistema de la Clase Negativa (`Sin_Pajaro`)
Detectamos que si a la Red Neuronal solo le enviamos grabaciones de aves, al conectarla al mundo salvaje terminará clasificando a "un tractor de fondo" como alguna de nuestras aves con 99% de confianza de manera forzada. 
Instanciamos el generador automatizado ecológico para solucionar esto a través de dos compuertas de seguridad masivas:
1. Pone a prueba 1500 donantes eligiendo espacios de 6.0seg que según el SVL **nunca colisionan** con la especie.
2. Como los humanos que etiquetaron a veces fallan (sub-etiquetado de aves menores gesticulando al fondo), el compilador evalúa físicamente al audio usando **`librosa.feature.spectral_flatness`**. Si el audio contiene oscilaciones "demasiado perfectas o puras" (planitud cercana a 0.0), el programa delata que hay un "pájaro cantando de fondo escondido" y destruye al archivo automáticamente, tomando solo aquellas ventanas que simulan verdadero ruido abiótico natural.

### F. Purga de Anomalías
Cerramos el diseño con un *Guardiamarina Estadístico*: cualquier especie sumamente extraña (con `< 100 cantos representativos`) es purgada del Tensor en las sombras para evitar que nuestra matriz de pesos de una futura capa *Softmax* quede crónicamente desbalanceada.

---
El resultado final es un empaquetado de tensores pesados (`X, y`) listo para ser consumido instantáneamente por ecosistemas de PyTorch o TensorFlow.
