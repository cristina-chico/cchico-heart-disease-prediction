# cchico-heart-disease-prediction
Proyecto Heart disease prevalence prediction comparando Linear regression, Random Forest and Gradient Boosting regression.



## Tools: Linear Regression, Random Forest, Gradient Boosting Regressor.

* Linear Regression: modela la relación entre las variables predictoras (X) y 
  el target (y) como una combinación lineal: y = β₀ + β₁·x₁ + β₂·x₂ + ... + βₙ·xₙ. 
  Es el modelo más simple e interpretable: cada coeficiente β indica cuánto cambia 
  el target cuando esa variable aumenta en una unidad (con las demás constantes). 
  Funciona bien cuando la relación entre X e y es realmente lineal. Sus puntos 
  débiles: es sensible a outliers, asume que las variables no están muy 
  correlacionadas entre sí (de ahí la importancia del VIF para detectar 
  multicolinealidad), y no captura relaciones no lineales ni interacciones entre 
  variables. En este proyecto sirve como modelo base (baseline) para comparar 
  contra modelos más complejos.

* Random Forest: entrena muchos árboles en paralelo sobre muestras aleatorias 
  de los datos y promedia sus predicciones. Es difícil de sobreajustar.

* Gradient Boosting: entrena árboles en secuencia, cada uno corrigiendo los 
  errores del anterior. Suele dar mejor precisión pero es más sensible a 
  hiperparámetros y tarda más en entrenar.

## Observaciones sobre los modelos 
* GB (MAE 0.95, R² 0.74) obtiene mejor resultado por poco sobre 
RF (MAE 0.98, R² 0.72).

GB suele dar mejor resultado, es un patrón típico: 
* GB es un pelín mejor en precisión en cambio RF es casi igual pero aporta 
  menos riesgo. Es decir, RF obtiene menos probabilidad de que el modelo 
  falle o se comporte peor de lo esperado.

## Comparación de modelos

1. Menos riesgo de overfitting

* Random Forest entrena cada árbol con una muestra aleatoria distinta de 
  los datos y las predicciones se promedian. Ese promedio "suaviza" errores 
  individuales por lo tanto es muy difícil que se sobreajuste aunque uses 
  los hiperparámetros por defecto.

* Gradient Boosting entrena árboles en secuencia, cada uno corrigiendo los 
  errores del anterior. Si se deja entrenar demasiado o con learning_rate 
  mal calibrado, puede memorizar el ruido del train set y dar R² alto en 
  train pero bajo en test. Para este modelo es importante configurar de 
  manera adecuada los hiperparámetros.

  Hiperparámetros clave de GB (los tres interactúan entre sí):
  - n_estimators: cuántos árboles entrena en secuencia (default: 100). 
    Muchos árboles = más riesgo de sobreajuste.
  - learning_rate: cuánto "peso" tiene cada árbol nuevo en la corrección 
    (default: 0.1). Más alto = aprende rápido pero ruidoso.
  - max_depth: profundidad máxima de cada árbol (default: 3). Más profundo 
    = cada árbol captura más patrones, pero también más ruido.
  
  La dificultad está en que se compensan: si bajas learning_rate a 0.01 
  necesitas más n_estimators (ej. 1000) para llegar al mismo resultado. 
  En RF esto no pasa tanto: con n_estimators=100 y el resto por defecto 
  suele funcionar bien.

2. Menos riesgo de que cambien los resultados al re-entrenar

* RF es más estable. Si mañana añades 50 filas nuevas al dataset y 
  reentrenas, las predicciones cambian muy poco.
* GB es más sensible. Pequeños cambios en los datos pueden mover más 
  las predicciones, porque cada árbol depende del anterior en cadena.

3. Menos riesgo en producción (tiempo y recursos)

* RF puedes paralelizar (n_jobs=-1) para entrenar rápido el modelo.
* GB no se puede paralelizar bien porque cada árbol necesita al anterior. 
  En datasets grandes tarda más, y si tienes que reentrenarlo semanalmente 
  en producción eso importa.

## Conclusión 
* En un caso práctico real optaría por escoger RF o dedicaría 
más tiempo al sobreajuste de GB. En este proyecto concreto, la diferencia 
mínima entre RF y GB probablemente viene porque el dataset es mediano, 
contiene outliers que se decidió mantener por ser datos médicos, y tiene 
features correlacionadas. Ese contexto favorece que RF se acerque mucho 
a GB sin necesidad de configuración adicional.

## Anotaciones sobre los modelos

### Cómo influyen los datos en la elección del modelo:

* Cantidad de datos:
  - Pocos datos (<1000 filas): GB sobreajusta más fácil porque cada árbol 
    se aferra a patrones específicos. RF resiste mejor.
  - Datos medianos (1000-10000 filas): RF y GB dan resultados parecidos. 
    Este dataset tiene 3140 filas, lo que explica que RF y GB se queden 
    tan cerca (0.72 vs 0.74).
  - Muchos datos (>50.000 filas): GB suele ganar claramente porque tiene 
    "material" suficiente para aprender patrones reales sin memorizar ruido.

* Calidad de los datos:
  - Ruido y outliers extremos: GB los persigue (intenta corregirlos árbol 
    tras árbol) → sobreajuste. RF los ignora mejor porque promedia.
  - Relaciones no lineales complejas: GB brilla, captura interacciones 
    sutiles mejor que RF.
  - Features muy correlacionadas (multicolinealidad): ambos aguantan, pero 
    RF es algo más estable. Linear Regression es la que más sufre.

### Hiperparámetro n_jobs:

¿Qué es n_jobs y por qué acelera RF?
  n_jobs indica a sklearn cuántos núcleos (cores) de la CPU puede usar a 
  la vez. Por defecto usa solo uno y el resto están parados.
  - n_jobs=1  → usa 1 núcleo (default)
  - n_jobs=4  → usa 4 núcleos en paralelo
  - n_jobs=-1 → usa TODOS los núcleos disponibles
  
  Funciona en RF porque sus 100 árboles son independientes entre sí: se 
  pueden repartir entre núcleos y entrenarse a la vez. Con 4 núcleos el 
  entrenamiento tarda aproximadamente 1/4 del tiempo.
  
  No funciona en GB porque cada árbol depende del anterior (árbol 2 
  necesita los errores del árbol 1, etc.). Aunque se ponga n_jobs=-1, 
  sklearn no puede paralelizar esa parte.
  
  Analogía: RF = pintar 100 paredes independientes con 8 pintores a la 
  vez. GB = aplicar 100 capas de pintura sobre la misma pared, cada capa 
  tiene que secar antes de la siguiente.
  
  Nota: n_jobs=-1 también acelera cross_val_score y GridSearchCV, que 
  entrenan varios modelos a la vez.

     
