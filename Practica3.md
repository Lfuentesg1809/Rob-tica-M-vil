Práctica 2 - Robótica Móvil
1. Introducción

En esta práctica se ha desarrollado un algoritmo de navegación autónoma para un robot móvil capaz de desplazarse hacia una serie de objetivos evitando los obstáculos que encuentra durante el recorrido.

El objetivo principal es conseguir que el robot alcance los puntos de destino de forma autónoma, modificando continuamente su dirección y velocidad en función de la posición del objetivo y de los obstáculos detectados mediante el sensor láser.

Para ello, se utiliza un algoritmo basado en el método de campos de fuerzas virtuales (VFF, Virtual Force Field). Este método combina dos tipos de fuerzas:

Fuerza de atracción: orienta al robot hacia el objetivo que debe alcanzar.
Fuerza de repulsión: permite al robot alejarse de los obstáculos para evitar posibles colisiones.

A partir de la combinación de ambas fuerzas, se calcula una dirección de movimiento que permite al robot avanzar hacia su destino mientras intenta mantener una distancia de seguridad respecto a los obstáculos.

El comportamiento del robot se basa principalmente en cuatro acciones:

Seguir el objetivo: el robot calcula la posición relativa del objetivo y genera una fuerza de atracción hacia él.
Evitar obstáculos: utiliza las lecturas del láser para calcular una fuerza de repulsión lateral.
Regular la velocidad: adapta la velocidad lineal según la distancia a los obstáculos y la intensidad del giro.
Alcanzar el destino: cuando el robot se encuentra suficientemente cerca del objetivo, lo marca como alcanzado y detiene el movimiento.

De esta manera, el robot puede navegar por un entorno con obstáculos sin necesidad de conocer previamente una trayectoria completa.

2. Parámetros

Al principio del programa se definen los parámetros utilizados para controlar el movimiento y el comportamiento del algoritmo VFF:

MAX_SPEED = 3.0: velocidad lineal máxima del robot cuando dispone de espacio suficiente para avanzar.
MAX_TURN = 2.0: límite establecido para la velocidad angular o corrección de giro.
SAFE_DISTANCE = 3.0: distancia de seguridad a partir de la cual comienzan a tenerse en cuenta los obstáculos para calcular la fuerza de repulsión.
DANGER_DISTANCE = 1.2: distancia que determina cuándo un obstáculo se encuentra demasiado cerca y es necesario aplicar una estrategia adicional de evasión.
TARGET_DISTANCE = 1.0: distancia mínima al objetivo para considerarlo alcanzado.
K_TARGET = 1.0: constante que regula la influencia de la fuerza de atracción hacia el objetivo.
K_OBSTACLE = 1.0: constante que regula la influencia de la fuerza de repulsión de los obstáculos.
ALPHA = 0.25: coeficiente utilizado para suavizar la velocidad angular y evitar cambios bruscos en la dirección.
previous_w = 0.0: variable que almacena la velocidad angular de la iteración anterior.

Estos parámetros permiten ajustar el comportamiento del robot, equilibrando la capacidad de avanzar hacia el objetivo con la necesidad de evitar los obstáculos.

Durante el desarrollo de la práctica se ha dado prioridad a conseguir una evasión fiable de los obstáculos frente a alcanzar una velocidad elevada. Por este motivo, se ha establecido una velocidad máxima de 3.0, buscando un comportamiento más seguro durante la navegación.

3. Cálculo de la posición relativa del objetivo

Una de las primeras operaciones que realiza el algoritmo es obtener la posición actual del robot y la del siguiente objetivo.

Para ello, se utilizan las funciones HAL.getPose3d() y WebGUI.getNextTarget(). La primera proporciona la posición y orientación del robot, mientras que la segunda permite obtener el siguiente objetivo establecido en el entorno de simulación.

El objetivo se encuentra expresado en coordenadas absolutas, por lo que es necesario transformar su posición al sistema de coordenadas relativo del robot.

Para realizar esta transformación se utiliza la función absolute2relative(), que recibe las coordenadas del objetivo, la posición del robot y su orientación (yaw).

Primero se calcula la diferencia entre las posiciones:

dx: diferencia entre las coordenadas horizontales.
dy: diferencia entre las coordenadas verticales.

Después, se aplica una transformación de coordenadas utilizando el ángulo de orientación del robot:

xr: posición del objetivo respecto al eje delantero del robot.
yr: posición del objetivo respecto al eje lateral del robot.

Una vez obtenidas estas coordenadas, se calcula la distancia al objetivo mediante la función math.hypot().

Esta transformación es importante porque permite que el algoritmo trabaje con la posición del objetivo desde la perspectiva del propio robot, independientemente de su orientación en el entorno.

4. Fuerza de atracción hacia el objetivo

Una vez calculada la posición relativa del objetivo, el algoritmo obtiene el vector de atracción que indica la dirección en la que debe avanzar el robot.

Para ello, se normalizan las coordenadas relativas dividiéndolas entre la distancia al objetivo:

target_fx: componente horizontal de la fuerza de atracción.
target_fy: componente lateral de la fuerza de atracción.

De esta manera, se obtiene un vector unitario que apunta directamente hacia el objetivo.

En el caso de que la distancia al objetivo sea inferior a TARGET_DISTANCE, es decir, menor que 1 metro, el algoritmo considera que el destino ha sido alcanzado.

Cuando esto sucede, se ejecutan las siguientes acciones:

Se marca el objetivo como alcanzado mediante target.setReached(True).
Se establece la velocidad lineal en cero.
Se establece la velocidad angular en cero.
Se reinicia la variable previous_w.
Se espera al siguiente ciclo de ejecución.

Así, el robot puede completar los objetivos de forma progresiva y continuar con el siguiente cuando corresponda.

5. Detección de obstáculos mediante el sensor láser

La detección de obstáculos se realiza utilizando el sensor láser del robot.

En cada iteración, se obtienen las lecturas mediante HAL.getLaserData().values. Estas lecturas proporcionan las distancias a los objetos detectados en diferentes direcciones alrededor del robot.

El programa utiliza como referencia el índice 90 del sensor, que representa la dirección frontal del robot.

Para cada lectura se calcula su ángulo mediante:

angle = math.radians(i - 90)

De esta forma, cada índice del láser queda asociado a un ángulo respecto a la dirección de avance.

Antes de utilizar una lectura, se comprueba que sea válida. Se descartan los valores que no sean finitos y las distancias inferiores o iguales a 0.05 metros.

División del entorno en sectores

Para mejorar la detección de obstáculos, el algoritmo divide las lecturas del láser en tres zonas principales:

Sector frontal: comprende los índices entre 70 y 110. Se utiliza para detectar obstáculos situados directamente delante del robot.
Sector izquierdo: comprende los índices entre 91 y 140.
Sector derecho: comprende los índices entre 40 y 89.

Para cada sector se almacena la distancia mínima detectada:

front_min: distancia mínima de los obstáculos frontales.
left_min: distancia mínima de los obstáculos situados a la izquierda.
right_min: distancia mínima de los obstáculos situados a la derecha.

Inicialmente, estas variables toman el valor de SAFE_DISTANCE, es decir, 3 metros.

La información obtenida permite conocer si existe un obstáculo delante del robot y determinar qué lado presenta más espacio disponible para realizar una maniobra de evasión.

6. Cálculo de la fuerza de repulsión

Una vez detectados los obstáculos, el algoritmo calcula la fuerza de repulsión mediante el método VFF.

En este caso, se utiliza únicamente la componente lateral de la fuerza, almacenada en la variable obstacle_y.

Para cada lectura válida cuya distancia sea inferior a SAFE_DISTANCE, se calcula la intensidad de repulsión mediante la siguiente expresión:

strength = 1.0 / d - 1.0 / SAFE_DISTANCE

Donde:

d representa la distancia al obstáculo.
SAFE_DISTANCE establece el alcance de influencia de la fuerza de repulsión.
strength representa la intensidad de la fuerza generada por el obstáculo.

Cuanto más cerca se encuentra el obstáculo, mayor es la intensidad de la repulsión. Por el contrario, cuando se encuentra a una distancia igual o superior a 3 metros, no genera ninguna fuerza.

Posteriormente, se calcula la componente lateral de la fuerza utilizando el seno del ángulo de cada lectura:

obstacle_y -= math.sin(angle) * strength

De esta manera, los obstáculos situados a ambos lados generan fuerzas laterales que permiten modificar la dirección del robot para alejarlo de ellos.

Es importante destacar que el algoritmo no calcula una componente longitudinal de repulsión. Por tanto, la fuerza de los obstáculos se utiliza exclusivamente para modificar la dirección lateral del movimiento.

Estrategia adicional para obstáculos frontales

Puede ocurrir que el robot detecte un obstáculo directamente delante, pero que la fuerza lateral calculada sea prácticamente nula.

Para solucionar este problema, el programa incorpora una estrategia adicional.

Si se cumple que:

La distancia frontal es inferior a DANGER_DISTANCE.
El valor absoluto de obstacle_y es inferior a 0.05.

Entonces, el robot compara el espacio disponible a ambos lados.

Si el lado izquierdo presenta más espacio que el derecho, establece una fuerza de repulsión positiva de 0.5.

En caso contrario, establece una fuerza negativa de -0.5 para orientar el movimiento hacia la derecha.

Esta estrategia permite evitar que el robot continúe avanzando directamente hacia un obstáculo cuando la fuerza de repulsión calculada inicialmente no es suficiente para generar un giro.

Finalmente, se limita la fuerza lateral al intervalo comprendido entre -1.5 y 1.5 mediante la función clamp().

Esto evita que la fuerza de repulsión alcance valores excesivamente elevados y provoque cambios de dirección demasiado bruscos.

7. Cálculo del vector resultante VFF y control de dirección

Una vez calculadas la fuerza de atracción y la fuerza de repulsión, el algoritmo combina ambas para obtener el vector resultante que determina la dirección de movimiento del robot.

La componente horizontal se calcula mediante:

direction_x = max(0.5, target_fx)

De esta forma, se establece un valor mínimo de 0.5 para evitar que la componente horizontal sea demasiado pequeña.

La componente lateral se obtiene combinando las dos fuerzas:

direction_y = K_TARGET * target_fy + K_OBSTACLE * obstacle_y

En esta expresión intervienen:

La fuerza de atracción lateral hacia el objetivo.
La fuerza de repulsión lateral de los obstáculos.
Las constantes K_TARGET y K_OBSTACLE, que regulan la influencia de cada fuerza.

Como ambas constantes tienen inicialmente el valor 1.0, las dos fuerzas tienen la misma importancia en el cálculo del vector resultante.

Cálculo de la velocidad angular

Una vez obtenido el vector resultante, se calcula el ángulo de dirección deseado utilizando la función math.atan2():

desired_w = math.atan2(direction_y, direction_x)

Este ángulo indica cuánto debe girar el robot para seguir la dirección resultante de las fuerzas.

Para evitar pequeñas correcciones innecesarias, se establece una zona muerta:

Si el valor absoluto de desired_w es inferior a 0.025, se establece directamente en cero.

Después, se limita el valor del giro al intervalo comprendido entre -MAX_TURN y MAX_TURN.

Suavizado del giro

Para conseguir un movimiento más estable, el algoritmo no utiliza directamente el ángulo calculado como velocidad angular.

En su lugar, aplica un suavizado utilizando la variable previous_w:

angular_speed = ALPHA * desired_w + (1.0 - ALPHA) * previous_w

Este cálculo combina el giro deseado en la iteración actual con la velocidad angular anterior.

Con ALPHA = 0.25, el algoritmo da un 25 % de importancia al nuevo giro y un 75 % al giro anterior.

De esta manera, se evitan cambios bruscos en la velocidad angular y se consigue una trayectoria más suave.

Finalmente, se actualiza previous_w con el valor de angular_speed para utilizarlo en la siguiente iteración.

8. Control de la velocidad lineal

Además de controlar la dirección, el algoritmo adapta la velocidad lineal en función de la distancia a los obstáculos detectados en el sector frontal.

Para ello, se utilizan tres situaciones diferentes:

Espacio libre: si front_min es superior a 2.0 metros, el robot utiliza la velocidad máxima de 3.0.
Obstáculo a distancia intermedia: si front_min es superior a 1.2 metros, pero no supera los 2.0 metros, la velocidad se reduce al 80 % de la velocidad máxima, es decir, 2.4.
Obstáculo cercano: si front_min es inferior o igual a 1.2 metros, la velocidad se reduce al 55 % de la velocidad máxima, es decir, 1.65.

De esta forma, el robot puede avanzar a mayor velocidad cuando tiene espacio suficiente y reducir su velocidad cuando detecta obstáculos cercanos.

Reducción de velocidad durante los giros

El programa también tiene en cuenta la velocidad angular para adaptar la velocidad lineal durante las maniobras.

Si el valor absoluto de angular_speed es superior a 1.0, se limita la velocidad lineal al 80 % de la velocidad máxima, es decir, 2.4.

Esta estrategia permite reducir la velocidad durante los giros pronunciados y mejorar la estabilidad del movimiento.

La decisión de utilizar una velocidad máxima de 3.0 responde a las pruebas realizadas durante el desarrollo. Se comprobó que, al intentar aumentar considerablemente la velocidad, el robot tenía más dificultades para esquivar correctamente los obstáculos. Por este motivo, se decidió mantener una velocidad más reducida para priorizar la seguridad y la capacidad de evasión.

9. Visualización de las fuerzas y ejecución del algoritmo

Para facilitar la comprensión del comportamiento del robot, el programa utiliza la función WebGUI.showForces().

Esta función permite representar gráficamente los tres vectores principales utilizados en el algoritmo:

Fuerza de atracción: [target_fx, target_fy]. Representa la dirección hacia el objetivo.
Fuerza de repulsión: [0.0, obstacle_y]. Representa la influencia lateral de los obstáculos.
Fuerza resultante: [direction_x, direction_y]. Representa la dirección final que debe seguir el robot.

La visualización permite observar cómo se combinan las fuerzas y cómo cambia la dirección de movimiento cuando aparecen obstáculos en el entorno.

Funcionamiento general

El programa funciona mediante un bucle infinito que ejecuta continuamente las siguientes operaciones:

Se obtiene la posición y orientación actual del robot.
Se obtiene el siguiente objetivo y su posición.
Se transforma la posición absoluta del objetivo a coordenadas relativas del robot.
Se calcula la distancia al objetivo.
Se comprueba si el objetivo ha sido alcanzado.
Si todavía no se ha alcanzado, se calcula el vector de atracción.
Se obtienen las lecturas del sensor láser.
Se descartan las lecturas inválidas.
Se identifican los obstáculos frontales, izquierdos y derechos.
Se calcula la fuerza de repulsión lateral.
Si existe un obstáculo frontal peligroso, se selecciona el lado con más espacio disponible.
Se combinan las fuerzas de atracción y repulsión para obtener el vector resultante.
Se calcula y suaviza la velocidad angular.
Se establece la velocidad lineal según la distancia frontal y la intensidad del giro.
Se envían las velocidades al robot.
Se muestran gráficamente las fuerzas utilizadas.
Se espera al siguiente ciclo mediante Frequency.tick(50).

La función Frequency.tick(50) establece una frecuencia de ejecución objetivo de 50 Hz, permitiendo actualizar continuamente la navegación del robot.

10. Resultados

Durante las pruebas realizadas, el algoritmo ha conseguido completar el circuito entero y esquivar correctamente los obstáculos que encuentra durante el recorrido.

El comportamiento general del robot es satisfactorio en cuanto a navegación, ya que consigue alcanzar los objetivos establecidos sin chocar con los obstáculos del entorno.

Sin embargo, durante las pruebas también se han observado algunos problemas relacionados con la estabilidad y la velocidad de desplazamiento.

Uno de los principales inconvenientes es que el robot presenta un movimiento bastante oscilante o de tambaleo. Esto se debe a las continuas correcciones de dirección que realiza al combinar la fuerza de atracción hacia el objetivo con la fuerza de repulsión de los obstáculos. Aunque se ha incorporado un suavizado de la velocidad angular mediante el parámetro ALPHA, las correcciones siguen provocando cambios de dirección que hacen que el desplazamiento no sea completamente estable.

Por otra parte, el robot también se desplaza relativamente despacio, especialmente cuando encuentra obstáculos o necesita realizar giros para evitarlos.

Durante el ajuste de los parámetros se intentó reducir el tambaleo y aumentar la velocidad de desplazamiento. Sin embargo, al realizar estas modificaciones, el robot dejaba de esquivar los obstáculos con la misma eficacia. Al intentar conseguir un movimiento más rápido y estable, se perdía parte de la capacidad de reacción necesaria para evitar correctamente los objetos del entorno.

Por este motivo, se decidió priorizar la evasión de obstáculos frente a la velocidad y la estabilidad del movimiento. Se estableció una velocidad máxima de 3.0 y se mantuvieron unos parámetros que permiten completar el circuito esquivando correctamente los obstáculos, aunque el movimiento presente ciertas oscilaciones y el desplazamiento sea más lento de lo deseado.

Por tanto, se considera que el algoritmo consigue cumplir su objetivo principal, que es completar el recorrido de forma autónoma evitando los obstáculos, aunque todavía presenta margen de mejora en cuanto a suavidad y rapidez.

11. Conclusión

En esta práctica se ha desarrollado un algoritmo de navegación autónoma para un robot móvil utilizando el método de campos de fuerzas virtuales (VFF).

El sistema combina una fuerza de atracción hacia el objetivo con una fuerza de repulsión generada a partir de las lecturas del sensor láser. A partir de estas dos fuerzas, se calcula un vector resultante que determina la dirección de movimiento del robot.

Además, se ha incorporado un sistema de suavizado de la velocidad angular para evitar cambios bruscos de dirección y un control de velocidad lineal que permite reducir la velocidad cuando aparecen obstáculos cercanos o cuando el robot realiza giros pronunciados.

Durante las pruebas se ha conseguido que el robot complete el circuito entero y esquive correctamente los obstáculos. No obstante, se han observado dos inconvenientes principales: el movimiento presenta bastante tambaleo y la velocidad de desplazamiento es relativamente baja.

Se intentó mejorar estos aspectos aumentando la velocidad y reduciendo las oscilaciones, pero estas modificaciones afectaban negativamente a la capacidad del robot para esquivar los obstáculos. Por ello, finalmente se decidió priorizar la evasión de obstáculos frente a la rapidez y la estabilidad del movimiento, estableciendo una velocidad máxima de 3.0.

Como posibles mejoras, sería interesante conseguir un equilibrio mejor entre la estabilidad, la velocidad y la capacidad de evasión. Para ello, se podría experimentar con el coeficiente ALPHA, las constantes de atracción y repulsión y la forma de calcular la fuerza resultante.

También podría incorporarse una componente longitudinal de repulsión y mejorar la estrategia de selección de rutas de evasión.

En definitiva, aunque el algoritmo todavía presenta problemas de oscilación y velocidad, se ha conseguido cumplir el objetivo principal de la práctica: desarrollar un robot capaz de navegar de forma autónoma, completar el circuito y evitar correctamente los obstáculos del entorno.
