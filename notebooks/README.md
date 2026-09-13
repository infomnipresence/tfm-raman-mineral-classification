# Notebooks

Esta carpeta contiene el análisis exploratorio y la auditoría de los datos junto con
los experimentos históricos, y los modelos definitivos del proyecto.

Para una comprensión de la cronología del proyecto se recomienda leer los notebooks en el siguiente orden:

## 1. Análisis exploratorio

El análisis exploratorio revela la composición y la estructura de la base de datos, así como la 
auditoría de los datos propuesta con el fin de asegurar la independencia de las fuentes junto con
las comprensiones sobre el pre-procesamiento de los datos previo a la ejecución de los modelos. 

[Ver notebook de análisis exploratorio](./00_analisis_exploratorio.ipynb)

## 2.1 Experimentos históricos

A continuación se lista una serie de notebooks que fueron ejecutados de manera progresiva y experimental,
con el fin de comprender las exigencias que presenta la base de datos a la luz de obtener los mejores
resultados, respetando su estructura y alcances.

Esto permitió llevar el proyecto a un refinamiento gradual previo a la ejecución de los modelos candidatos.

- Primera aproximación (Base de datos sin auditar): Este notebook ejecuta una red neuronal genérica, con el fin de observar su desempeño
  en la base de datos sin auditoría de independencia de fuentes. 
  [Ver notebook Primera aproximación](./01 Primera Aproximación.ipynb)

- Segunda aproximación (Auditoría de independencia): Este notebook tiene como finalidad ejecutar la auditoría de datos por fuentes
  independientes propuestas en el EDA, con el fin de demostrar que un modelo que no garantice la independencia
  de fuentes es susceptible de presentar resultados con un sesgo muy optimista.

  [Ver notebook Segunda aproximación](./02 Segunda Aproximación - Source Audited_Group_Aware - CNN.ipynb)
  
  Después de este notebook, todos los siguientes tienen en cuenta la auditoría de independencia de fuentes.
  
  
  
