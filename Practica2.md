# Práctica 2 - Robótica Móvil

## 1. Introducción

En esta práctica se ha desarrollado un algoritmo de navegación autónoma para un robot móvil capaz de seguir una línea roja utilizando una cámara.

El objetivo principal es conseguir que el robot siga el recorrido marcado en el suelo de forma autónoma, ajustando continuamente su dirección y su velocidad en función de la posición de la línea detectada en la imagen.

Para ello, se utiliza la información proporcionada por la cámara del robot. El algoritmo procesa las imágenes identificando el color rojo y calculando la posición aproximada de la línea respecto al centro de la imagen.

A partir de esta información, se calcula un error que representa la desviación entre el centro de la línea y el centro de la imagen. Este error se utiliza para controlar el movimiento del robot mediante un controlador PD.

El comportamiento del robot se basa principalmente en tres acciones:

- **Seguir la línea:** el robot detecta la línea roja y calcula la dirección que debe tomar para seguirla.
- **Regular la velocidad:** el robot avanza más rápido cuando la línea está centrada y reduce su velocidad cuando la pierde o se descentra.
- **Buscar la línea:** si el robot deja de detectar la línea roja, baja la velocidad e intenta recuperarla.

De esta manera, el robot puede corregir continuamente su trayectoria sin necesidad de conocer previamente el circuito.

## 2. Parámetros

Al principio del programa se definen los parámetros utilizados para controlar el movimiento del robot:

- `Kp = 0.004`: constante proporcional del controlador PD. Determina cuánto influye el error actual en la corrección de la dirección.
- `Kd = 0.020`: constante derivativa del controlador PD. Permite tener en cuenta los cambios del error entre iteraciones para ajustar la respuesta del robot.
- `V_MAX = 10`: velocidad lineal máxima cuando el error es cero (línea centrada).
- `V_MIN = 3`: velocidad lineal mínima cuando el robot se desvía o pierde la línea.
- `error_anterior = 0`: variable que guarda el error para obtener la derivada.
- `perdida = 0`: contador que registra las iteraciones en las que no se detecta correctamente la línea.
- `V = 3`: velocidad lineal inicial del robot.
- `W = 0`: velocidad angular inicial del robot.

## 3. Detección de la línea mediante visión artificial

La detección de la línea roja es una de las partes fundamentales del algoritmo.

En cada iteración, el robot obtiene una imagen de la cámara. Si la imagen no está disponible, el programa continúa con la siguiente iteración sin realizar el procesamiento.

Una vez obtenida la imagen, se calculan sus dimensiones y se convierte del espacio de color BGR al espacio HSV.

El espacio HSV permite identificar colores mediante sus componentes de tono, saturación y luminosidad, lo que facilita la segmentación del color rojo.

Como el rojo puede aparecer en dos intervalos diferentes del tono HSV, se crean dos máscaras:

- **Primera máscara:** detecta los tonos rojos comprendidos entre los valores HSV `(0, 100, 100)` y `(10, 255, 255)`.
- **Segunda máscara:** detecta los tonos rojos comprendidos entre `(160, 100, 100)` y `(180, 255, 255)`.

Después, ambas máscaras se combinan, obteniendo una única imagen en la que se representan las zonas identificadas como rojas.

Finalmente, se utiliza `cv2.findNonZero()` para comprobar si existen suficientes píxeles detectados. Si se encuentran más de 10 puntos, el algoritmo considera que dispone de información suficiente para calcular la posición de la línea.

## 4. Cálculo del error y control de dirección

Una vez detectada la línea, el algoritmo calcula su centro utilizando los momentos de la imagen binaria.

El centro horizontal se obtiene a partir de los momentos de la máscara. Si el momento es cero, se utiliza el centro de la imagen como valor alternativo para evitar una división entre cero.

Luego, se calcula el error comparando la posición del centro detectado con el centro de la imagen. Este error permite conocer la desviación lateral que debe corregir el robot.

### Controlador PD

Para controlar la dirección se utiliza un controlador proporcional-derivativo (PD), que combina el error actual con la variación del error respecto a la iteración anterior.

## 5. Control de la velocidad lineal

Además de controlar la dirección, el algoritmo adapta la velocidad lineal del robot en función de la desviación de la línea.

Para ello, primero se normaliza el valor absoluto del error. Este cálculo transforma la desviación en un valor comprendido entre 0 y 1.

- **Error normalizado igual a 0:** la línea está centrada.
- **Error normalizado próximo a 1:** la línea presenta una desviación elevada respecto al centro.

A partir de este valor, se calcula la velocidad. De esta manera, la velocidad varía entre 10 y 3, según los valores establecidos en el programa.

Cuando el error es cero, el robot utiliza la velocidad máxima. A medida que aumenta la desviación, la velocidad disminuye progresivamente hasta alcanzar el valor mínimo de 3.

## 6. Comportamiento cuando se pierde la línea

Puede ocurrir que el robot deje de detectar la línea roja, por ejemplo, cuando se encuentra ante una curva pronunciada, una interrupción del recorrido o una imagen en la que no se identifican suficientes píxeles rojos.

Cuando esto sucede, el algoritmo ejecuta una estrategia de recuperación.

En primer lugar, incrementa el contador que indica cuánto tiempo se ha perdido la línea. Después, establece la velocidad lineal en su valor mínimo. De esta forma, el robot continúa avanzando lentamente mientras intenta recuperar la línea.

El objetivo es orientar el robot hacia la zona en la que podría encontrarse nuevamente la línea.

Sin embargo, esta estrategia tiene un problema: si la línea se pierde y el error anterior es cero, el robot siempre selecciona el mismo sentido de giro.

Además, el contador `perdida` no se utiliza para establecer un tiempo máximo de búsqueda ni para detener el robot si no consigue recuperar la línea.

Por tanto, permite intentar recuperar la trayectoria, pero no asegura que el robot consiga encontrarla siempre.

## 7. Ejecución del algoritmo

El programa funciona mediante un bucle que realiza las siguientes operaciones:

1. Se obtiene una imagen de la cámara.
2. Se comprueba que la imagen esté disponible.
3. Se convierte la imagen al espacio de color HSV.
4. Se detectan los píxeles rojos mediante las dos máscaras de color.
5. Se analiza la zona inferior de la imagen.
6. Se comprueba si existen suficientes puntos detectados.
7. Si la línea está presente, se calcula su centro y el error respecto al centro de la imagen.
8. Se aplica el controlador PD.
9. Se calcula la velocidad lineal en función del error normalizado.
10. Si no se detecta la línea, se reduce la velocidad y se inicia la búsqueda mediante un giro.
11. Se muestra la imagen procesada.

La función de frecuencia establece una frecuencia de ejecución objetivo de 20 Hz.

## 8. Resultados

Durante las pruebas, el objetivo del algoritmo es conseguir que el robot siga la línea roja ajustando su trayectoria de forma autónoma.

El comportamiento esperado es el descrito a lo largo del documento. No obstante, en una situación real, el comportamiento dependería de las condiciones de iluminación, del contraste entre la línea y el suelo, de la posición de la cámara y de los valores seleccionados para las constantes del controlador.

Además, el algoritmo no resuelve igual de bien todos los circuitos disponibles en Unibotics. En algunos circuitos con curvas muy pronunciadas, no consigue restablecer el recorrido una vez perdida la línea roja.

Del mismo modo, el programa pierde eficiencia cuando se aumenta considerablemente la velocidad del robot móvil.

## 9. Conclusión

En esta práctica se ha desarrollado un algoritmo de seguimiento de línea para un robot móvil utilizando visión artificial y un controlador PD.

El sistema procesa las imágenes de la cámara para detectar el color rojo, calcula el centro de la línea y determina el error respecto al centro de la imagen. A partir de este error, el controlador ajusta la velocidad angular del robot para corregir su dirección.

En general, el algoritmo tiene los principales elementos necesarios para realizar un seguimiento autónomo de una trayectoria mediante una cámara: adquisición de imágenes, segmentación por color, cálculo del error, control de dirección y adaptación de la velocidad.

Como posibles mejoras, sería interesante incorporar un límite a la velocidad angular, suavizar la derivada del error y mejorar la estrategia de recuperación cuando se pierde la línea.

De esta forma, se podría conseguir un movimiento más estable y preciso, reduciendo las oscilaciones y aumentando la capacidad del robot para seguir recorridos con curvas pronunciadas, incluso a mayor velocidad.
