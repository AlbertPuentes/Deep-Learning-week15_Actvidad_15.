# Deep-Learning-week15_Actvidad_15.
Semana 15 – Actividad 15: Trabajo Cooperativo sobre Data Journey, Acceso y Manipulación de Datos, Monitoreo y Logging, y Model Serving con Weights &amp; Biases 


## Conclusiones 

### Conclusiones tecnicas
- **Aprendizajes**: El ciclo de vida del modelo va más allá del entrenamiento. La trazabilidad con W&B permite comparar experimentos, detectar overfitting y auditar decisiones.
- **Dificultades**: Sincronizar métricas en tiempo real requiere estructura clara de logging. El manejo de escalado y serialización exige atención a versiones de dependencias.
- **Oportunidades**: Implementar CI/CD para ML, integrar monitoreo de drift (Evidently AI, WhyLabs), y automatizar reentrenamiento con pipelines (Airflow, MLflow).
- **Trabajo en equipo**: Dividir roles (ingeniería de datos, modelado, MLOps, documentación) acelera el desarrollo y mejora la calidad del entregable.

## Análisis Breve de Métricas
Con base en los datos registrados y visualizados en el dashboard de Weights & Biases, el comportamiento del modelo Perceptrón Multicapa MLP es altamente satisfactorio:
* **Convergencia y Estabilidad:** Las curvas de pérdida train_loss y val_loss muestran un descenso continuo y suave, estabilizándose en valores cercanos a cero. Esto indica que la tasa de aprendizaje (0.001) y el optimizador Adam fueron elecciones adecuadas para el dataset.
* **Prevención de Sobreajuste:** El modelo alcanza una precisión en validación de aproximadamente 97% - 98%, manteniendo una estrecha correlación con el rendimiento de entrenamiento. La brecha mínima entre las métricas de entrenamiento y validación confirma que la inclusión de la capa Dropout(0.3) actuó efectivamente como regularizador.
* **Eficacia del Early Stopping:** El monitoreo evidenció la correcta ejecución del mecanismo de parada temprana.

## Explicación de una Estrategia Básica de Model Serving
El Model Serving es la fase donde el modelo entrenado se expone para realizar inferencias sobre datos nuevos en un entorno de producción. Para este proyecto, la estrategia óptima y ligera consiste en una arquitectura de microservicios usando FastAPI.

* **Exposición mediante API REST:** Se crea una aplicación con FastAPI que define un endpoint ejemplo POST /predict.
* **Carga de Artefactos:** Al iniciar el servidor, se cargan en memoria los dos artefactos guardados en el ciclo anterior: el transformador de datos scaler.pkl y los pesos del modelo neuronal model_weights.pth.
* **Flujo de Inferencia (Entrada):** El usuario o sistema cliente envía un payload JSON con las 30 características.
* **Flujo de Inferencia (Procesamiento):** El servicio aplica el scaler.transform() a los datos de entrada para garantizar que estén en la misma escala que los datos de entrenamiento. Luego, el tensor resultante pasa por el modelo de PyTorch y se aplica una función sigmoide a la salida para obtener una probabilidad entre 0 y 1.
* **Respuesta:** La API devuelve un JSON con la clasificación final (Benigno/Maligno) y el nivel de confianza de la predicción.


## Conclusiones del Equipo
* **Gestión Rigurosa del Data Journey:** Garantizar el orden correcto de las operaciones permite evitar el data leakage. Ajustar el StandardScaler exclusivamente con el conjunto de entrenamiento asegura que las métricas obtenidas sean un reflejo real de la capacidad de generalización del modelo.
* **Valor del Monitoreo Activo:** La integración con Weights & Biases transformó un script de código estático en un entorno auditable. El registro de hiperparámetros y métricas por epoch permite la reproducibilidad total del experimento y facilita la toma de decisiones informadas para el despliegue.
* **Viabilidad de la Arquitectura:** Para un conjunto de datos tabular y estructurado como el Breast Cancer Wisconsin, una arquitectura MLP profunda (64-32-1) resulta suficiente, logrando capturar los patrones no lineales de las células con alta precisión.
 
## Socialización Breve de los Resultados
> **"Predicción de Cáncer de Mama mediante Redes Neuronales y MLOps"**

Hemos desarrollado y monitoreado un modelo de Inteligencia Artificial capaz de clasificar tumores (benignos o malignos) utilizando 569 muestras clínicas del dataset Breast Cancer Wisconsin. Mediante una red neuronal construida en PyTorch, con lo que el modelo aprende patrones médicos complejos, y tambien alcanza una buen precisión.

Más allá de la precisión, nuestro mayor logro es la confiabilidad del proceso: implementamos un pipeline de Machine Learning Operations* (MLOps) utilizando Weights & Biases. Esto nos permite monitorizar el entrenamiento en tiempo real, evitar el sobreajuste y guardar de forma segura los transformadores de datos y pesos del modelo. Realizando este proceso se puede obtener auditable que permite ser integrado en una aplicación de diagnósticos
 
##  Recomendaciones
* **Mejora de Estabilidad Numérica:** se evidencia que las iteraciones en PyTorch, es viable eliminar la capa nn.Sigmoid() al final de la red y sustituir la función de pérdida nn.BCELoss  por la funcion nn.BCEWithLogitsLoss, lo que permitiria mejorar la estabilidad durante el cálculo de los gradientes.
* **Validación Cruzada por Volumen de Datos:** Dado que el dataset es pequeño (569 muestras), el conjunto de prueba es de apenas 85 muestras.
* **Monitoreo en Producción:** Una vez que el modelo esté en serving, se debe implementar un sistema de monitoreo para el Data Drift (deriva de datos). 
Si se cambiase la forma de medir los tumores, las estadísticas de los datos de entrada cambiarán, y el modelo deberá ser reentrenado.
