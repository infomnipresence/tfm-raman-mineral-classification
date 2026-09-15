Nota Previa: Se recomienda descargar todos los notebooks y almacenarlos en una misma y única carpeta raíz
para efectos de que los archivos generados en algunos notebooks queden disponibles para su posterior
uso en algún otro notebook que lo requiera para su correcta ejecución futura. Adicionalmente, se recomienda
ejecutar los notebooks en orden, para que la generación de archivos sea coherente y no genere errores.

# Dataset 

La base de datos se encuentra en el siguiente link: 

(Elegir la opción Raman y descargar todas las carpetas).

https://www.rruff.net/about/download-data/

# Notebooks

Esta carpeta contiene el análisis exploratorio y la auditoría de los datos junto con
los experimentos históricos, y los modelos definitivos del proyecto.

Para una comprensión de la cronología del proyecto se recomienda leer los notebooks en el siguiente orden:

## 1. Análisis exploratorio

El análisis exploratorio revela la composición y la estructura de la base de datos, así como la 
auditoría de los datos propuesta con el fin de asegurar la independencia de las fuentes junto con
las comprensiones sobre el pre-procesamiento de los datos previo a la ejecución de los modelos. 

[Ver notebook de análisis exploratorio](./00_analisis_exploratorio.ipynb)

## 2. Experimentos históricos

A continuación se lista una serie de notebooks que fueron ejecutados de manera progresiva y experimental,
con el fin de comprender las exigencias que presenta procesamiento de la base de datos a la luz de obtener los mejores
resultados en las redes convolucionales.

Esto permitió llevar el proyecto a un refinamiento gradual previo a la ejecución de los modelos candidatos.

### 2.1 Primera aproximación (Base de datos sin auditar):
   Este notebook ejecuta una red neuronal genérica, con el fin de observar su desempeño
  en la base de datos sin auditoría de independencia de fuentes.
  
  La ejecución se hace únicamente sobre los espectros Excellent - Unoriented.
   
  [Ver notebook Primera aproximación](./01_primera_aproximacion.ipynb)

### 2.2 Segunda aproximación (Auditoría de independencia): 
Este notebook tiene como finalidad ejecutar la auditoría de datos por fuentes
  independientes propuestas en el EDA, con el fin de demostrar que un modelo que no garantice la independencia
  de fuentes es susceptible de presentar resultados con un sesgo muy optimista.

  La ejecución se hace únicamente sobre los espectros Excellent - Unoriented.
  
  Se utilizó la misma CNN genérica de la primera aproximación para tener una referencia comparativa sólida.

  [Ver notebook Segunda aproximación](./02_segunda_aproximacion.ipynb)
  
  Después de este notebook, todos los siguientes tienen en cuenta la auditoría de independencia de fuentes.

### 2.3 Tercera aproximación (auditoría + augmentation): 
Este notebook explora el efecto de aplicar técnicas de
  data augmentation en el pipeline de la CNN genérica

  La ejecución se hace únicamente sobre los espectros Excellent - Unoriented.
  
  Se utilizó la misma CNN genérica de la primera aproximación para tener una referencia comparativa sólida.

  [Ver notebook Tercera aproximación](./03_tercera_aproximacion.ipynb)

  Después de este notebook, todos los siguientes trabajan sobre datos audítados y con augmentation.

### 2.4 Cuarta aproximación (auditoría + augmentation + whole DB): 
Este notebook pone a prueba la hipótesis de que contar con una
  base de datos más grande puede mejorar el poder de generalización de la base de datos aunque el rendimiento se vea un poco
  afectado debido a la presencia de datos de menor calidad. Esto puede sonar un poco contra-intuitivo pues puede conllevar a una
  disminución en las métricas de rendimiento, sin embargo es necesario hacerlo pues es lo que más se acerca a la recolección
  real de las muestras: obtención en situaciones distintas no ideales fuera del laboratorio.
  
  La ejecución se hace únicamente sobre los 7 dominios de la base de datos.
  
  Se utilizó la misma CNN genérica de la primera aproximación para tener una referencia comparativa sólida.

  [Ver notebook Cuarta aproximación](./04_cuarta_aproximacion.ipynb)

### 2.5 CNN Mejorada (auditoría + augmentation + whole DB + Enhanced CNN): 
En esta fase se implementa toda la línea previa
  de mejoras hasta la cuarta aproximación, modificando la arquitectura de la red convolucional mediante cambios en sus
  parámetros, regularizers, etc... con el fin de comprobar que una vez se ha optimizado el pre-procesamiento de los datos
  de la mayor forma posible, buscar una arquitectura adecuada puede mejorar el rendimiento de los modelos.

  Se utilizó una CNN distinta de la primera aproximación, procurando mejorar las características de la arquitectura.

  Se aplicó análisis multisemilla.

  [Ver notebook CNN mejorada](./05_cnn_mejorada.ipynb)

  ## 3. Modelos Candidatos

  Teniendo en cuenta los aprendizajes obtenidos durante los experimentos históricos,
  esta sección implementa los modelos candidatos descritos por la literatura científica (SVM, Random Forest, CNNs - 1D) con el fin de
  comparar su rendimiento y elegir el modelo más justo después de una amplia exploración.

  ### 3.1 Modelos Machine Learning
  
  En esta fase se ajustan técnicas de grid search sobre modelos SVM & Random forest con el objetivo de explorar
  su capacidad de clasificación sobre la base de datos completa y auditada mediante fuentes independientes.

  
  [Ver notebook SVM](./06_SVM.ipynb)

  
  [Ver notebook Random Forest](./07_random_forest.ipynb)

  
  ### 3.2 Modelos Deep Learning
  
  En esta sección se ajustan técnicas de exploración multi-semilla sobre dos arquitecturas de redes convolucionales (Reference & CNN)
  descritas por la literatura de referencia con el objetivo de explorar su capacidad de clasificación sobre la base de datos completa,        aumentada y auditada mediante fuentes independientes.

  [Ver notebook MLROD CNN](./08_mlrod_cnn.ipynb)

  [Ver notebook Reference CNN](./09_reference_cnn.ipynb)

  Adicionalmente, se adjunta el modelo Reference CNN entrenado únicamente sobre la base de datos Excellent Unoriented, con el objetivo
  de mostrar que datos de alta calidad presentan mayor performance de clasificación.

  No obstante, usar datos tomados bajo distintas condiciones es una aproximación más honesta con la realidad del trabajo fuera del            laboratorio y el modelo ganador logra hacerlo con un buen desempeño a pesar de poder contar con una mayor cantidad de fuentes      independientes.

  [Ver notebook Reference CNN Excellent Unoriented DB](./10_reference_cnn_excellent.ipynb)

  ### 3.3 Modelo Ganador

  Se adjunta el notebook del modelo ganador listo para convertirlo en checkpoint que posibilite su serialización y puesta en producción.
  
  [Ver notebook checkpoint modelo ganador](./11_checkpoint_ganador.ipynb)

  ## 4. Análisis SHAP

  Para mejorar la interpretabilidad del modelo ganador se adjunta un notebook explicando cómo con la herramienta SHAP,
  es posible comprender de manera gráfica la forma en la que la red convolusional ganadora comprende estadísticamente las características
  que hacen que un espectro sea distinguible de los demás de forma única.

  [Ver notebook análisis SHAP](./12_SHAP.ipynb)

  ## 5. Implementación FLASK

  Para la ejecución de estos notebooks es necesario haber ejecutado previamente el notebook de la sección 3.3 para obtener el checkpoint del modelo ganador.

  Esta sección presenta una API prototípica del modelo ganador serializado que recibe como petición la instrucción de
  clasificar un espectro cuya entrada se proporciona en formato JSON y devuelve como respuesta el top 5 espectros más probables
  junto con su fiabilidad porcentual.

  En este notebook se encapsula el checkpoint en un protocolo FLASK listo para recibir peticiones externas:

  [Ver notebook API (servidor flask)](./13_flask_api_protocol.ipynb)

  En este notebook se hace una prueba de contexto con espectros sintéticos para constatar el buen funcionamiento del protocolo:

  [Ver notebook cliente externo flask](./14_flask_external_client.ipynb). Es necesario ejecutar el notebook del servidor 
  y tenerlo abierto para poder comunicar las peticiones.
  

 
 
  
  
