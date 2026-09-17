# Identificación automatizada de minerales mediante espectros Raman y técnicas de machine learning & deep learning

Proyecto para la clasificación de minerales a partir de espectros Raman de la base de datos RRUFF. Se comparan modelos clásicos de machine learning y redes neuronales convolucionales unidimensionales (CNN-1D), aplicando una metodología de evaluación diseñada para evitar la fuga de información entre muestras procedentes de una misma fuente.

> **Estado:** Prototipo de clasificación - prueba de concepto API - Flask.

## Objetivo

Desarrollar un flujo reproducible capaz de recibir un espectro Raman y devolver una lista ordenada de minerales candidatos. El proyecto abarca:

- auditoría y limpieza de datos;
- armonización de los espectros sobre una rejilla común de 512 puntos;
- partición por fuente independiente (`source_id`);
- aumento de datos aplicado únicamente al entrenamiento de las CNN;
- comparación de SVM, Random Forest y dos arquitecturas CNN-1D;
- análisis de estabilidad con cinco semillas para las CNN;
- interpretación de predicciones mediante SHAP;
- serialización del modelo ganador y exposición mediante una API Flask.

## Datos

Los espectros proceden de [RRUFF - Comprehensive Database of Mineral Data](https://www.rruff.net/about/download-data/) y se distribuyen en siete dominios de calidad y orientación:

1. Excellent Oriented
2. Excellent Unoriented
3. Fair Oriented
4. Fair Unoriented
5. Poor Unoriented
6. Unrated Oriented
7. Unrated Unoriented

Los archivos originales no se redistribuyen en este repositorio. Deben descargarse desde la fuente oficial y ubicarse en el directorio de datos indicado por los notebooks.

## Decisiones metodológicas principales

### Auditoría por fuente

La exploración inicial mostró que distintos espectros podían proceder de una misma muestra mineral. Una división aleatoria por observaciones habría permitido que mediciones relacionadas aparecieran simultáneamente en entrenamiento y prueba, produciendo métricas artificialmente optimistas. Para evitarlo, la partición definitiva se realizó por `source_id`.

### Preprocesamiento

Los espectros se ordenaron, se promediaron posiciones Raman duplicadas, se interpolaron sobre una rejilla común de 512 puntos y se normalizaron por su intensidad máxima absoluta. No se aplicaron correcciones intensivas de línea base o rayos cósmicos, con el propósito de permitir que las CNN aprendieran directamente los patrones espectrales.

### Aumento de datos

El *data augmentation* se aplicó exclusivamente al conjunto de entrenamiento mediante pequeños desplazamientos horizontales y ruido gaussiano. Los experimentos preliminares mostraron que esta estrategia contribuyó a mejorar el rendimiento y la capacidad de generalización de las CNN. Los conjuntos de validación y prueba permanecieron sin modificaciones.

## Cronología del proyecto

| Etapa | Desarrollo | Aprendizaje obtenido |
|---|---|---|
| 1 | CNN inicial sobre Excellent Unoriented | Una división por espectros puede producir resultados excesivamente optimistas. |
| 2 | Auditoría de `source_id` y etiquetas | Se identificaron fuentes ausentes, duplicados y asociaciones contradictorias. |
| 3 | Partición por fuentes independientes | Se evitó compartir una misma fuente entre entrenamiento, validación y prueba. |
| 4 | Aumento de datos en entrenamiento | Las variaciones espectrales plausibles mejoraron el desempeño de las CNN. |
| 5 | Incorporación progresiva de dominios | Aumentar la diversidad hizo el problema más difícil, pero más representativo. |
| 6 | Integración de los siete dominios | Se estableció una partición definitiva y congelada para comparar los modelos. |
| 7 | SVM y Random Forest | Se construyeron modelos clásicos de referencia mediante *Grid Search*. |
| 8 | MLROD CNN y Reference CNN | Las dos arquitecturas se entrenaron con cinco semillas sobre la misma partición. |
| 9 | Evaluación y análisis de errores | Se compararon métricas globales, equilibrio entre clases, calibración y estabilidad. |
| 10 | Interpretabilidad SHAP | Se localizaron regiones Raman que favorecen o reducen cada predicción. |
| 11 | Serialización y API Flask | El modelo ganador se expuso como un servicio local que devuelve resultados Top-1 y Top-5. |

## Modelos evaluados

- **SVM:** modelo clásico ajustado mediante *Grid Search*.
- **Random Forest:** ensamble de árboles ajustado mediante *Grid Search*.
- **MLROD CNN:** arquitectura adaptada de Berlanga et al. (2022).
- **Reference CNN:** arquitectura adaptada de Liu et al. (2017).

Los cuatro modelos utilizaron la misma partición de prueba de la base completa, formada por 2.488 espectros y 255 clases minerales.

## Resultados finales sobre Whole Database

| Modelo | Top-1 | Top-5 | Macro F1 | Weighted F1 | Entrenamiento registrado |
|---|---:|---:|---:|---:|---:|
| **Reference CNN** | **0,8514 ± 0,0096** | **0,9366 ± 0,0051** | **0,7351 ± 0,0083** | **0,8434 ± 0,0095** | 10,76 ± 1,36 min por semilla |
| MLROD CNN | 0,8314 ± 0,0046 | 0,9254 ± 0,0080 | 0,7120 ± 0,0075 | 0,8231 ± 0,0065 | 9,14 ± 1,38 min por semilla |
| SVM | 0,7050 | 0,8428 | 0,4442 | 0,6722 | 26,38 min |
| Random Forest | 0,6781 | 0,8694 | 0,5189 | 0,6717 | 221,13 min |

Las métricas de las CNN corresponden a la media y desviación estándar de cinco semillas. SVM y Random Forest se ajustaron con una configuración reproducible y se evaluaron una sola vez, por lo que sus resultados no incluyen desviación estándar.

La **Reference CNN** fue seleccionada como modelo ganador por combinar el mejor rendimiento global, baja variabilidad entre semillas y una exactitud Top-5 del 93,66 %. En un experimento separado con espectros Excellent Unoriented alcanzó un Top-1 de 0,8989 ± 0,0071, lo que evidencia la importancia de la calidad de adquisición.

## Interpretabilidad mediante SHAP

SHAP (*SHapley Additive exPlanations*) permite identificar qué regiones del espectro favorecen o reducen la puntuación asignada a una clase. El análisis incluyó predicciones correctas, casos de baja confianza y confusiones mineralógicas relevantes.

En 16 casos representativos, ocultar las regiones destacadas por SHAP redujo la probabilidad predicha en una media de 0,3095, frente a 0,0230 al ocultar regiones aleatorias. Este resultado respalda la relevancia de las zonas identificadas para el comportamiento de la CNN, aunque no demuestra por sí solo una relación química causal.

## Prototipo Flask

El modelo ganador se serializó junto con:

- la rejilla Raman de 512 puntos;
- el orden de las 255 clases;
- las reglas de interpolación y normalización;
- el *checkpoint* seleccionado.

La API Flask se ejecuta localmente en el puerto `8081`. Recibe las posiciones Raman y sus intensidades, reproduce el preprocesamiento y devuelve una respuesta JSON con el mineral más probable y las cinco clases principales con sus probabilidades estimadas.

Ejemplo conceptual de respuesta:

```json
{
  "prediction": "Quartz",
  "confidence": 0.94,
  "top_5": [
    {"mineral": "Quartz", "probability": 0.94},
    {"mineral": "Cristobalite", "probability": 0.02},
    {"mineral": "Tridymite", "probability": 0.01},
    {"mineral": "Coesite", "probability": 0.01},
    {"mineral": "Stishovite", "probability": 0.01}
  ]
}
```

Los valores anteriores son únicamente ilustrativos y no corresponden a una ejecución concreta.


## Instalación

### Requisitos del entorno

El proyecto fue ejecutado con **Python 3.11**. El análisis de interpretabilidad se comprobó con **TensorFlow 2.21.0** y **SHAP 0.51.0**. No se requiere Java ni H2O para reproducir la implementación final.

Para ejecutar todos los notebooks, el análisis SHAP y el prototipo Flask se necesitan los siguientes paquetes:

| Paquete | Uso dentro del proyecto |
|---|---|
| `numpy` | Vectores, matrices, rejilla Raman y operaciones numéricas. |
| `pandas` | Metadatos, manifiestos de partición, tablas y archivos CSV. |
| `scipy` | Operaciones científicas y correlación de Spearman del análisis SHAP. |
| `scikit-learn` | SVM, Random Forest, Grid Search, preprocesamiento y métricas. |
| `joblib` | Serialización y utilidades empleadas por los modelos clásicos. |
| `tensorflow` | Construcción, entrenamiento, evaluación y serialización de las CNN-1D. Keras está incluido como `tensorflow.keras`. |
| `matplotlib` | Curvas de aprendizaje, matrices de confusión, tablas y figuras. |
| `seaborn` | Visualizaciones estadísticas y matrices de confusión. |
| `tqdm` | Barras de progreso durante la carga y el procesamiento. |
| `shap` | Explicación de las regiones Raman que influyen en las predicciones. |
| `flask` | API local para servir el modelo serializado. |
| `werkzeug` | Servidor utilizado para iniciar y detener Flask desde Jupyter. |
| `requests` | Peticiones HTTP realizadas desde el notebook cliente. |
| `jupyterlab`, `notebook` e `ipykernel` | Ejecución de los notebooks en el entorno `raman-cnn`. |

Los siguientes módulos también aparecen en el código, pero forman parte de la biblioteca estándar de Python y **no requieren instalación independiente**: `pathlib`, `os`, `sys`, `json`, `zipfile`, `re`, `io`, `hashlib`, `shutil`, `time`, `random`, `warnings`, `threading`, `collections`, `subprocess`, `importlib` y `platform`.

### Creación del entorno con Conda

Se recomienda crear un entorno aislado para evitar conflictos con otras instalaciones:

```bash
conda create -n raman-cnn python=3.11
conda activate raman-cnn
python -m pip install --upgrade pip setuptools wheel
```

Instalar todas las dependencias utilizadas:

```bash
python -m pip install \
  "numpy>=1.26,<3" \
  "pandas>=2.0,<3" \
  "scipy>=1.11,<2" \
  "scikit-learn>=1.3,<2" \
  "joblib>=1.3,<2" \
  "tensorflow==2.21.0" \
  "matplotlib>=3.8,<4" \
  "seaborn>=0.13,<1" \
  "tqdm>=4.66,<5" \
  "shap==0.51.0" \
  "flask>=3.0,<4" \
  "werkzeug>=3.0,<4" \
  "requests>=2.31,<3" \
  "jupyterlab>=4,<5" \
  "notebook>=7,<8" \
  "ipykernel>=6,<7"
```

Registrar el entorno para poder seleccionarlo desde Jupyter:

```bash
python -m ipykernel install --user --name raman-cnn --display-name "Python (raman-cnn)"
```

Después se puede iniciar Jupyter con:

```bash
jupyter notebook
```

Dentro de Jupyter debe seleccionarse el kernel **Python (raman-cnn)**.

### Instalación mediante `requirements.txt`

Cuando el repositorio incluya `requirements.txt`, las mismas dependencias podrán instalarse mediante:

```bash
conda create -n raman-cnn python=3.11
conda activate raman-cnn
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m ipykernel install --user --name raman-cnn --display-name "Python (raman-cnn)"
```

### Verificación de la instalación

Ejecutar esta celda antes de iniciar los experimentos:

```python
from importlib.metadata import version

packages = [
    "numpy", "pandas", "scipy", "scikit-learn", "joblib",
    "tensorflow", "matplotlib", "seaborn", "tqdm", "shap",
    "flask", "werkzeug", "requests", "jupyterlab", "notebook",
    "ipykernel",
]

for package in packages:
    print(f"{package:15s} {version(package)}")
```

Finalmente, verificar que no existan dependencias incompatibles:

```bash
python -m pip check
```

Para ejecutar la API, el puerto `8081` debe estar disponible. El servidor y el notebook cliente deben utilizar la misma dirección:

```text
http://127.0.0.1:8081
```

## Orden de ejecución

Las instrucciones de ejecución se encuentran descritas de manera explicita en el siguiente enlace:

[Ver el orden de lectura de los notebooks](notebooks/README.md)

Los notebooks históricos se conservan para documentar la evolución metodológica del proyecto.

## Reproducibilidad

- La asignación de fuentes a entrenamiento, validación y prueba permanece congelada.
- Las CNN se evalúan con cinco semillas y reportan media y desviación estándar.
- El aumento de datos se aplica únicamente durante el entrenamiento.
- La selección de hiperparámetros utiliza validación; el conjunto de prueba permanece aislado.
- SVM y Random Forest corresponden a evaluaciones puntuales reproducibles.
- Los artefactos de producción incluyen verificaciones de integridad mediante hashes.

## Alcances

- El modelo solo puede elegir entre las 255 clases conocidas.
- Las probabilidades de la capa *softmax* no deben interpretarse como certeza.
- Algunas clases poseen pocas fuentes independientes.
- El rendimiento disminuye en espectros de menor calidad.
- Aún se requiere validación externa con instrumentos y muestras diferentes de RRUFF.
- SHAP explica el comportamiento estadístico del modelo, no una causalidad química.

## Líneas futuras

- incorporar más fuentes independientes y equilibrar las clases;
- validar con espectros obtenidos mediante otros instrumentos;
- calibrar las probabilidades y detectar muestras fuera de distribución;
- estudiar cuantitativamente la similitud entre minerales confundidos;
- evaluar un ensamble ponderado de Reference CNN y MLROD CNN;
- añadir autenticación, monitorización y trazabilidad a la API;
- desarrollar una interfaz destinada a técnicos y especialistas.

## Referencias principales

- Berlanga, G., Williams, Q., & Temiquel, N. (2022). Convolutional neural networks as a tool for Raman spectral mineral classification under low signal, dusty Mars conditions. *Earth and Space Science, 9*, e2021EA002125. 
- Lafuente, B., Downs, R. T., Yang, H., & Stone, N. (2015). The power of databases: The RRUFF project. En T. Armbruster & R. M. Danisi (Eds.), *Highlights in mineralogical crystallography* (pp. 1–30). De Gruyter. 
- Liu, J., Osadchy, M., Ashton, L., Foster, M., Solomon, C. J., & Gibson, S. J. (2017). Deep convolutional neural networks for Raman spectrum recognition: A unified solution. *Analyst, 142*(21), 4067–4074. 
- Liu, Y., Wu, Y., Wang, J., Qi, J., Zhou, C., & Xue, Y. (2026). Recent advances in Raman spectral classification with machine learning. *Sensors, 26*(1), 341. 
- RRUFF. (s. f.). *Download files*. Recuperado el 13 de septiembre de 2026, de https://www.rruff.net/about/download-data/
- Smith, E., & Dent, G. (2005). Modern Raman spectroscopy: A practical approach. John
Wiley & Sons.

## Autoría y herramientas

Proyecto desarrollado por **Nicolás González Villarreal** utilizando Python. Durante el desarrollo se empleó asistencia de modelos de lenguaje, incluidos Codex y Sol, para apoyar tareas de programación, revisión y documentación. Las decisiones metodológicas, ejecuciones, validaciones e interpretación de los resultados permanecieron bajo responsabilidad del autor.

