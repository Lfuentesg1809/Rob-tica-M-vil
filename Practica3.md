# Práctica 2 - Robótica Móvil

## 1. Intro

En esta práctica se ha desarrollado un algoritmo de navegación autónoma para un robot móvil, en este caso un Fórmula 1, capaz de desplazarse por un circuito evitando los obstáculos que encuentra durante el recorrido.

El objetivo principal es conseguir que el veículo alcance la meta de forma autónoma, modificando continuamente su dirección y velocidad en función del mapa y de los obstáculos detectados mediante el sensor láser.

Para ello, se utiliza un algoritmo basado en el método de campos de fuerzas virtuales (VFF, *Virtual Force Field*). Este método combina dos tipos de fuerzas:

- **Fuerza de atracción:** orienta al robot hacia el objetivo que debe alcanzar.
- **Fuerza de repulsión:** permite al robot alejarse de los obstáculos para evitar posibles colisiones.

A partir de la combinación de ambas fuerzas, se calcula una dirección de movimiento que permite al robot avanzar hacia su destino mientras intenta mantener una distancia de seguridad respecto a los obstáculos.

El comportamiento del robot se basa principalmente en cuatro acciones:

- **Seguir el objetivo:** el coche calcula la posición relativa del objetivo y genera una fuerza de atracción hacia él.
- **Evitar obstáculos:** utiliza las lecturas del láser para calcular una fuerza de repulsión lateral.
- **Regular la velocidad:** adapta la velocidad según la distancia a los obstáculos y la intensidad del giro.
- **Alcanzar el destino:** cuando el robot se encuentra suficientemente cerca del objetivo, lo marca como alcanzado y detiene el movimiento.

De esta manera, el robot puede navegar por un entorno con obstáculos sin necesidad de conocer previamente una trayectoria completa.

## 2. Parámetros

- `MAX_SPEED = 3.0`: velocidad lineal máxima del F1 cuando dispone de espacio suficiente para avanzar.
- `MAX_TURN = 2.0`: límite establecido para la velocidad angular o corrección de giro.
- `SAFE_DISTANCE = 3.0`: distancia de seguridad a partir de la cual comienzan a tenerse en cuenta los obstáculos para calcular la fuerza de repulsión.
- `DANGER_DISTANCE = 1.2`: distancia que determina cuándo un obstáculo se encuentra demasiado cerca y es necesario aplicar una estrategia adicional de evasión.
- `TARGET_DISTANCE = 1.0`: distancia mínima al objetivo para considerarlo alcanzado.
- `K_TARGET = 1.0`: constante que regula la influencia de la fuerza de atracción hacia el objetivo.
- `K_OBSTACLE = 1.0`: constante que regula la influencia de la fuerza de repulsión de los obstáculos.
- `ALPHA = 0.25`: coeficiente utilizado para suavizar la velocidad angular y evitar cambios bruscos en la dirección.
- `previous_w = 0.0`: variable que almacena la velocidad angular de la iteración anterior.

Estos parámetros permiten ajustar el comportamiento del robot, equilibrando la capacidad de avanzar hacia el objetivo con la necesidad de evitar los obstáculos.

Durante el desarrollo de la práctica se ha dado prioridad a conseguir una evasión fiable de los obstáculos frente a alcanzar una velocidad elevada. Por este motivo, se ha establecido una velocidad máxima de 3.0, buscando un comportamiento más seguro durante la navegación.

## 3. Cálculo de la posición relativa del objetivo

Una de las primeras operaciones que realiza el algoritmo es obtener la posición actual del móvil y la del siguiente objetivo.

Para ello, se utilizan las funciones `HAL.getPose3d()` y `WebGUI.getNextTarget()`. La primera proporciona la posición y orientación del robot, mientras que la segunda permite obtener el siguiente objetivo establecido en el entorno de simulación.

## 4. Fuerza de atracción hacia el objetivo

Una vez calculada la posición relativa del objetivo, el algoritmo obtiene el vector de atracción que indica la dirección en la que debe avanzar.

Para ello, se normalizan las coordenadas relativas dividiéndolas entre la distancia al objetivo:

- `target_fx`: componente horizontal de la fuerza de atracción.
- `target_fy`: componente lateral de la fuerza de atracción.

De esta manera, se obtiene un vector unitario que apunta directamente hacia el objetivo.

En el caso de que la distancia al objetivo sea inferior a `TARGET_DISTANCE`, es decir, menor que 1 metro, el algoritmo considera que el destino ha sido alcanzado.

Cuando esto sucede, se ejecutan las siguientes acciones:

1. Se marca el objetivo como alcanzado mediante `target.setReached(True)`.
2. Se establece la velocidad lineal en cero.
3. Se establece la velocidad angular en cero.
4. Se reinicia la variable `previous_w`.
5. Se espera al siguiente ciclo de ejecución.

Así, el veículo puede completar los objetivos de forma progresiva y continuar con el siguiente cuando corresponda.

## 5. Detección de obstáculos mediante el sensor láser

La detección de obstáculos se realiza utilizando el sensor láser del F1.

En cada iteración, se obtienen las lecturas mediante `HAL.getLaserData().values`. Estas lecturas proporcionan las distancias a los objetos detectados en diferentes direcciones alrededor del robot.

El programa utiliza como referencia el índice 90 del sensor, que representa la dirección frontal del robot.

Para cada lectura se calcula su ángulo mediante:

`angle = math.radians(i - 90)`

De esta forma, cada índice del láser queda asociado a un ángulo respecto a la dirección de avance.

Antes de utilizar una lectura, se comprueba que sea válida. Se descartan los valores que no sean finitos y las distancias inferiores o iguales a 0.05 metros.

### División del entorno en sectores

Para mejorar la detección de obstáculos, el algoritmo divide las lecturas del láser en tres zonas principales:

- **Sector frontal:** comprende los índices entre 70 y 110. Se utiliza para detectar obstáculos situados directamente delante del robot.
- **Sector izquierdo:** comprende los índices entre 91 y 140.
- **Sector derecho:** comprende los índices entre 40 y 89.

Para cada sector se almacena la distancia mínima detectada:

- `front_min`: distancia mínima de los obstáculos frontales.
- `left_min`: distancia mínima de los obstáculos situados a la izquierda.
- `right_min`: distancia mínima de los obstáculos situados a la derecha.

Inicialmente, estas variables toman el valor de `SAFE_DISTANCE`, es decir, 3 metros.

La información obtenida permite conocer si existe un obstáculo delante del robot y determinar qué lado presenta más espacio disponible para realizar una maniobra de evasión.

## 6. Cálculo del vector resultante VFF y control de dirección

Una vez calculadas la fuerza de atracción y la fuerza de repulsión, el algoritmo combina ambas para obtener el vector resultante que determina la dirección de movimiento del robot.

La componente horizontal se calcula mediante:

`direction_x = max(0.5, target_fx)`

De esta forma, se establece un valor mínimo de 0.5 para evitar que la componente horizontal sea demasiado pequeña.

La componente lateral se obtiene combinando las dos fuerzas:

`direction_y = K_TARGET * target_fy + K_OBSTACLE * obstacle_y`

En esta expresión intervienen:

- La fuerza de atracción lateral hacia el objetivo.
- La fuerza de repulsión lateral de los obstáculos.
- Las constantes `K_TARGET` y `K_OBSTACLE`, que regulan la influencia de cada fuerza.

Como ambas constantes tienen inicialmente el valor 1.0, las dos fuerzas tienen la misma importancia en el cálculo del vector resultante.

### Cálculo de la velocidad angular

Una vez obtenido el vector resultante, se calcula el ángulo de dirección deseado utilizando la función `math.atan2()`.

Este ángulo indica cuánto debe girar el coche para seguir la dirección resultante de las fuerzas.

Para evitar pequeñas correcciones innecesarias, se establece una zona muerta.

Si el valor absoluto de `desired_w` es inferior a 0.025, se establece directamente en cero.

Después, se limita el valor del giro al intervalo comprendido entre `-MAX_TURN` y `MAX_TURN`.

### Suavizado del giro

Para conseguir un movimiento más estable, el algoritmo no utiliza directamente el ángulo calculado como velocidad angular.

En su lugar, aplica un suavizado utilizando la variable `previous_w`.

De esta manera, se evitan cambios bruscos en la velocidad angular y se consigue una trayectoria más suave.

Finalmente, se actualiza `previous_w` con el valor de `angular_speed` para utilizarlo en la siguiente iteración.

## 7. Visualización de las fuerzas y ejecución del algoritmo

Para facilitar la comprensión del comportamiento del móvil, el programa utiliza la función `WebGUI.showForces()`.

Esta función permite representar gráficamente los tres vectores principales utilizados en el algoritmo.

La visualización permite observar cómo se combinan las fuerzas y cómo cambia la dirección de movimiento cuando aparecen obstáculos en el entorno.

### Funcionamiento general

El programa funciona mediante un bucle infinito que ejecuta continuamente las siguientes operaciones:

1. Se obtiene la posición y orientación actual.
2. Se obtiene el siguiente objetivo y su posición.
3. Se transforma la posición absoluta del objetivo a coordenadas relativas.
4. Se calcula la distancia al objetivo.
5. Se comprueba si el objetivo ha sido alcanzado.
6. Si todavía no se ha alcanzado, se calcula el vector de atracción.
7. Se obtienen las lecturas del sensor láser.
8. Se descartan las lecturas inválidas.
9. Se identifican los obstáculos frontales, izquierdos y derechos.
10. Se calcula la fuerza de repulsión lateral.
11. Si existe un obstáculo frontal peligroso, se selecciona el lado con más espacio disponible.
12. Se combinan las fuerzas de atracción y repulsión para obtener el vector resultante.
13. Se calcula y suaviza la velocidad angular.
14. Se establece la velocidad lineal según la distancia frontal y la intensidad del giro.
15. Se envían las velocidades.
16. Se muestran gráficamente las fuerzas utilizadas.
17. Se espera al siguiente ciclo mediante `Frequency.tick(50)`.
    
## 8. Resultados

Durante las pruebas realizadas, el algoritmo ha conseguido completar el circuito entero y esquivar correctamente los obstáculos que encuentra durante el recorrido.

El comportamiento general del robot es satisfactorio en cuanto a navegación, ya que consigue alcanzar los objetivos establecidos sin chocar con los obstáculos del entorno.

Sin embargo, durante las pruebas también se han observado algunos problemas relacionados con la estabilidad y la velocidad de desplazamiento.

Uno de los principales inconvenientes es que el robot presenta un movimiento bastante oscilante o de tambaleo. Esto se debe a las continuas correcciones de dirección que realiza al combinar la fuerza de atracción hacia el objetivo con la fuerza de repulsión de los obstáculos. Aunque se ha incorporado un suavizado de la velocidad angular mediante el parámetro `ALPHA`, las correcciones siguen provocando cambios de dirección que hacen que el desplazamiento no sea completamente estable.

Por otra parte, el robot también se desplaza relativamente despacio, especialmente cuando encuentra obstáculos o necesita realizar giros para evitarlos.

Durante el ajuste de los parámetros se intentó reducir el tambaleo y aumentar la velocidad de desplazamiento. Sin embargo, al realizar estas modificaciones, el robot dejaba de esquivar los obstáculos con la misma eficacia. Al intentar conseguir un movimiento más rápido y estable, se perdía parte de la capacidad de reacción necesaria para evitar correctamente los objetos del entorno.

Por este motivo, se decidió priorizar la evasión de obstáculos frente a la velocidad y la estabilidad del movimiento. Se estableció una velocidad máxima de 3.0 y se mantuvieron unos parámetros que permiten completar el circuito esquivando correctamente los obstáculos, aunque el movimiento presente ciertas oscilaciones y el desplazamiento sea más lento de lo deseado.

Por tanto, se considera que el algoritmo consigue cumplir su objetivo principal, que es completar el recorrido de forma autónoma evitando los obstáculos, aunque todavía presenta margen de mejora en cuanto a suavidad y rapidez.

## 11. Conclusión

En esta práctica se ha desarrollado un algoritmo de navegación autónoma para un robot móvil utilizando el método de campos de fuerzas virtuales (VFF).

El sistema combina una fuerza de atracción hacia el objetivo con una fuerza de repulsión generada a partir de las lecturas del sensor láser. A partir de estas dos fuerzas, se calcula un vector resultante que determina la dirección de movimiento del robot.

Durante las pruebas se ha conseguido que el robot complete el circuito entero y esquive correctamente los obstáculos. No obstante, se han observado dos inconvenientes principales: el movimiento presenta bastante tambaleo y la velocidad de desplazamiento es relativamente baja.

Se intentó mejorar estos aspectos aumentando la velocidad y reduciendo las oscilaciones, pero estas modificaciones afectaban negativamente a la capacidad del robot para esquivar los obstáculos. Por ello, finalmente se decidió priorizar la evasión de obstáculos frente a la rapidez y la estabilidad del movimiento.

Como posibles mejoras, sería interesante conseguir un equilibrio mejor entre la estabilidad, la velocidad y la capacidad de evasión.

En definitiva, aunque el algoritmo todavía presenta problemas de oscilación y velocidad, se ha conseguido cumplir el objetivo principal de la práctica: desarrollar un robot capaz de navegar de forma autónoma, completar el circuito y evitar correctamente los obstáculos del entorno.
