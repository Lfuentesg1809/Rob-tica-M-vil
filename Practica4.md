# Práctica 3 - Navegación Global

## 1. Introducción

En esta práctica se ha desarrollado un sistema de navegación para un coche autónomo que debe desplazarse por una ciudad hasta un destino seleccionado en el mapa.

Para ello, se utiliza el algoritmo **Gradient Path Planning**, que genera un campo de costes teniendo en cuenta la distancia al destino y la proximidad a los obstáculos. El coche sigue las zonas de menor coste para avanzar hacia su objetivo.

## 2. Parámetros

Se han ajustado varios parámetros para controlar el movimiento del coche:

- **Velocidad máxima (12.0):** permite avanzar más rápido en los tramos favorables.
- **Velocidad de giro (2.5):** reduce el avance cuando es necesario girar mucho.
- **Ganancia de giro (2.0):** regula las correcciones de dirección.
- **Distancia al objetivo (3.0):** indica cuándo se considera alcanzado el destino.
- **Anticipación (8):** permite seguir un punto adelantado de la trayectoria.
- **Radio e intensidad de penalización (4 y 5.0):** favorecen rutas alejadas de los obstáculos.

## 3. Planificación de la ruta

Primero, el programa carga el mapa de la ciudad y distingue las zonas libres de los obstáculos.

Después, cuando el usuario selecciona un destino, calcula un campo de costes desde ese punto hacia las zonas transitables. Las celdas cercanas a los obstáculos reciben una penalización, por lo que se favorecen recorridos más despejados.

El coche obtiene su trayectoria siguiendo las celdas cuyo coste disminuye hasta llegar al objetivo.

## 4. Control del coche

Durante el recorrido, el coche consulta continuamente su posición y selecciona un punto adelantado de la trayectoria como referencia.

A partir de ese punto, ajusta su dirección y velocidad. Avanza más rápido cuando está bien orientado y reduce la velocidad al girar o acercarse al destino.

Cuando llega al objetivo, se detiene y espera a que el usuario seleccione otro destino.

## 5. Visualización

El programa muestra en WebGUI el campo de costes y la trayectoria calculada. Esto permite observar el recorrido previsto y cómo se tienen en cuenta los obstáculos del mapa.

## 6. Resultados y conclusión

El programa integra la planificación global de rutas con el control reactivo de las velocidades del coche. Permite seleccionar destinos en el mapa, calcular un recorrido y seguirlo de forma autónoma.

Como posible mejora, se podría perfeccionar el seguimiento de las curvas y aumentar la seguridad al pasar cerca de los obstáculos. El funcionamiento real y la ausencia de colisiones deben comprobarse mediante pruebas en el simulador.
