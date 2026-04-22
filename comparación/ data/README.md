## Correcciones e información del procesamiento de la data:

* LEAKAGE: Solución de error en Split X e y. En el dataset existe más de una variable que aporta información de la predicción.
En este caso son tres columnas que dan información de la respuesta y es data que aunque tuvieramos en una prueba real no queremos usarla en el modelo de entrenamiento.

  * Ejemplo del proyecto: Las columnas sospechosas que podrían seguir en df son; Heart disease_Upper 95% CI (límite superior del intervalo de confianza del target) y Heart disease_Lower 95% CI (límite inferior del mismo intervalo).

  * Por qué esto es leakage: Un intervalo de confianza se calcula a partir del propio valor del target. No se puede tener el "intervalo de confianza de heart disease" si todavía no sabemos el valor de heart disease. Es como predecir usando como feature un valor aproximado del target.
 
  * Si Heart disease_prevalence = 10.5, entonces típicamente: Heart disease_Upper 95% CI ≈ 11.2 y Heart disease_Lower 95% CI ≈ 9.8 pero el modelo estaría aprendiendo que el resultado del target es la media de esas dos variables, realmente no estaría prediciendo nada.


