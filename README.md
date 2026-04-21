# Identificación de Especies de Aves mediante Análisis de Espectrogramas

## 🔍 Sobre el Proyecto
Este proyecto aborda la clasificación automatizada de cantos de aves de enorme similitud biológica mediante técnicas de Inteligencia Artificial. Reformulamos este desafío tradicional de procesamiento de señales bioacústicas para resolverlo como un problema de Visión por Computadora (*Computer Vision*). 

Para lograrlo, señales acústicas unidimensionales son convertidas matemáticamente en matrices bidimensionales (Espectrogramas Mel). Posteriormente, entrenamos y comparamos múltiples arquitecturas reconocidas de Redes Neuronales Convolucionales (ResNet-18, MobileNetV3-small, SqueezeNet y YAMNet) para determinar estratégicamente cuál posee mejor capacidad de *Dominio de Adaptación*, permitiendo asimilar el conocimiento de patrones fotográficos pre-existentes sobre tensores frecuenciales sonoros.

## 🦅 Origen de los Datos (El Dataset Principal)
Nuestra ingesta se apalanca en el trabajo científico originalmente ensamblado por Loris Jeantet y Emmanuel Dufourq (2023). Estos autores se dedicaron a investigar minuciosamente en [**Xeno-Canto**](https://xeno-canto.org/) (la plataforma colaborativa de mapeo sonoro ecológico mundial), descargando solo cantos con grado "Calidad A" y etiquetando a mano las ocurrencias exactas sobre el milisegundo de inicio y fin en un registro `.csv`.

Para este repositorio y experimento en específico, nosotros leímos sus registros en formato CSV, pero **filtramos agresivamente los metadatos**. Aislamos y reconstruimos un conjunto equilibrado de **13 especies específicas** (Paseriformes), y adicionalmente fabricamos una clase negativa sintética llamada `Sin_Pajaro`.

🔗 **Repositorio original de descarga (Zenodo):** [Manually labeled Bird song dataset of 22 species from Xeno-canto](https://zenodo.org/records/7828148)


## 🗂️ Estructura del Repositorio
Para un recorrido ordenado, el proyecto está completamente fragmentado en las siguientes ramas operativas:

* 📁 **`EDA_y_generación_espectrogramas/`**
  Alberga el corazón del pre-procesamiento del equipo. Aquí reside el código en Python y el pipeline con `librosa` encargado de limpiar estadísticamente la metadata original, seccionar los audios largos de la naturaleza en ventanas unificadas de 6.0 segundos (usando técnicas de re-centrado/Jittering) y exportar definitivamente todos los `.png` matemáticos.

* 📁 **`Modelo_Resnet_18/`** 
  Contiene los scripts de entrenamiento y clasificación de nuestra red campeona (Baseline de máxima capacidad). A través del *Fine-Tuning* profundo (entrenando todos sus millones de pesos), alcanzó un Accuracy espectacular del 96%.

* 📁 **`MovilNetV3small/`**
  Dedicado integralmente al despliegue de *MobileNetV3*. Muestra cómo un modelo drásticamente recortado en cantidad de parámetros aún se las arregla para memorizar espectrogramas a un gran nivel ($0.91$ Macro FB).

* 📁 **`YAMNet/`**
  Documenta el proceso alrededor del modelo YAMNet. Este fue nuestro "Grupo de Control", ya que, en teoría, YAMNet ya estaba pre-entrenado para sonidos y no fotos, pero su evaluación en transferencia superficial como puro extractor lo frenó y demostró la brutalidad física de la adaptación de dominios acústicos.
