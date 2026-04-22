## Correcciones e información del procesamiento de la data:

* LEAKAGE: Solución de error en Split X e y. En el dataset existe más de una variable que aporta información de la predicción.
En este caso son tres columnas que dan información de la respuesta y es data que aunque tuvieramos en una prueba real no queremos usarla en el modelo de entrenamiento.

  * Ejemplo del proyecto: Las columnas sospechosas que podrían seguir en df son; Heart disease_Upper 95% CI (límite superior del intervalo de confianza del target) y Heart disease_Lower 95% CI (límite inferior del mismo intervalo).

  * Por qué esto es leakage: Un intervalo de confianza se calcula a partir del propio valor del target. No se puede tener el "intervalo de confianza de heart disease" si todavía no sabemos el valor de heart disease prevalence. Es como predecir usando como feature un valor aproximado del target.
 
  * Si Heart disease_prevalence = 10.5, entonces típicamente: Heart disease_Upper 95% CI ≈ 11.2 y Heart disease_Lower 95% CI ≈ 9.8 pero el modelo estaría aprendiendo que el resultado del target es la media de esas dos variables, realmente no estaría prediciendo nada y puede dar fallos en producción.
    * En un caso práctico de producción, si queremos predecir la prevalencia de enfermedad cardíaca y no tenemos el intervalo de confianza de la misma porque todavía no se ha medido. Entonces el modelo no puede usar esas columnas en producción y sin ellas, el R² real se desploma. Por eso las tres columnas (prevalence, Upper 95% CI, Lower 95% CI) tienen que salir de X.

* Importancia del VIF: En el dataset como hemos visto con el target aparecen otras columnas con indicadores de datos cercanos o similares a ellos, es decir, existe multicolinealidad entre algunas variables predictoras, no solo entre predictoras(X) y la variable target(y).
   * Ejemplo del proyecto: Las columnas age 60-79 pct % (porcentaje de población entre 60-79 años) y Percent of Population Aged 60+ (porcentaje de población mayor de 60) miden casi lo mismo. Si una sube, la otra sube casi igual. El modelo puede interpretarlo como información a añadir y no como información duplicada.
 
   * Función VIF
 
     ```
     from sklearn.linear_model import LinearRegression

def calculate_vif(X):
    vif_data = pd.DataFrame()
    vif_data["feature"] = X.columns
    vif_values = []
    
    for col in X.columns:
        X_other = X.drop(col, axis=1)
        y_target = X[col]
        
        model = LinearRegression()
        model.fit(X_other, y_target)
        r2 = model.score(X_other, y_target)
        
        # Fix para colinealidad perfecta
        if r2 >= 1.0:
            vif_values.append(float("inf"))
        else:
            vif_values.append(1 / (1 - r2))
    
    vif_data["VIF"] = vif_values
    return vif_data.sort_values("VIF", ascending=False)
     ´´´
