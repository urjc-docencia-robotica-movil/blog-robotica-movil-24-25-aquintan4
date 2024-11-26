# P4 Global Navigation
## Álvaro Quintana
## 1. Introducción
En esta práctica se implementa un **sistema de navegación global** para un coche simulado en el entorno Gazebo. 
Este enfoque se fundamenta en la disponibilidad de un **mapa del entorno**, que actúa como una representación precisa del espacio en el que opera el vehículo. 
Además, se conoce la posición del coche en todo momento, mediante la función `HAL.getPose3d()`, y también la posición de destino:
la cual es seleccionada por el usuario utilizando la interfaz gráfica y consultada a través de `GUI.getTargetPose()` .

Como ya hemos mencionado utilizamos un **mapa de ocupación** (previamente construido). En este mapa,
las áreas libres están representadas en blanco (valor 255) y los obstáculos en negro (valor 0). A partir de esta información, el sistema no busca una ruta óptima,
sino que *emplea una estrategia basada en el descenso de gradiente*. Este enfoque permite al coche avanzar hacia el objetivo adaptándose dinámicamente a cambios en el entorno o nuevas condiciones.

El algoritmo implementado combina varias técnicas: primero, utiliza una **búsqueda en anchura (BFS)** para generar un mapa de costos que define la "distancia acumulada" desde cada punto del mapa hasta el destino.
Luego, se aplica un proceso de inflación de obstáculos, que expande las zonas cercanas a las paredes u obstaculos del mapa y las hace menos transitables aumentando su coste, creando una zona de seguridad para evitar colisiones.
Finalmente, el coche selecciona su siguiente movimiento hacia el destino siguiendo la dirección de menor coste en el mapa, lo que constituye la esencia del descenso de gradiente.
Esta combinación de técnicas proporciona una navegación robusta y flexible, permitiendo que el vehículo responda de manera más dinámica y segura a las condiciones del entorno.

<div align="center">
  <img src="https://github.com/user-attachments/assets/c8ba20d9-7f2a-487b-8771-7f19cc83f8d9" alt="Descripción de la imagen">
</div>

En las siguientes secciones se explicará el diseño y la implementación de cada componente, junto con las decisiones técnicas adoptadas para obtener un comportamiento eficiente y seguro del vehículo. 
## 2. Parte Deliverativa
He propuesto mejorar la eficiencia del proceso deliberativo (cálculo del mapa de costos e inflación de obstáculos), con el objetivo de reducir al máximo el tiempo de cómputo, de modo que la espera desde que se marca el destino hasta que 
el vehículo comienza a avanzar sea lo más corta posible. Aunque podría haber logrado una mayor velocidad eliminando las visualizaciones, he preferido mantenerlas para que se pueda observar todo el proceso de forma clara.

Comenzando con la primera parte de este proceso deliberativo, que consiste en generar un mapa de costos, el objetivo es obtener la distancia acumulada desde el destino hasta el vehículo. Para ello, he utilizado el algoritmo de 
**BFS (Breadth-First Search)**. Un detalle importante en esta implementación es la utilización de un conjunto (`set`) para almacenar los nodos ya explorados, ya que proporciona una complejidad de acceso **O(1)**, a diferencia de un array, 
que vería su rendimiento degradado a medida que aumentara el número de nodos explorados. Además, he creado un array denominado **walls** donde se almacenan los nodos que contienen obstáculos, ya que esta información será necesaria más
adelante en el proceso.

Toda la matriz de costos está inicialmente llena de **infinito**. Esto se debe a que las áreas no exploradas y las zonas con obstáculos no podrán ser visitadas por el vehículo bajo ninguna circunstancia. A medida que la onda de BFS avanza 
y explora el mapa, las áreas accesibles se van actualizando con los valores correspondientes a la distancia desde el destino. En cuanto a la visualización, es necesario que las zonas de coste se normalicen, ya que cualquier valor superior a
255, que es el límite máximo para representar valores en imágenes, se ajustará proporcionalmente para mantenerse dentro de este rango. Además, para asegurar una correcta visualización, es fundamental convertir la matriz de costos al tipo 
e dato **`uint8`**, que es el formato adecuado para mostrar imágenes en sistemas de visualización como **GUI.showNumpy()**.

El algoritmo de BFS comienza a propagar la onda desde el punto de partida, actualizando el mapa de costos con la distancia recorrida. En presencia de un obstáculo, la propagación de la onda se detiene en esa dirección, lo que nos 
permite obtener el mapa de costos completo desde el destino hacia el vehículo, priorizando las áreas de mayor costo cercanas al vehículo y las de menor costo cerca del destino.

El siguiente paso consiste en expandir un área más grande alrededor del **target**, ya que necesitamos conocer un espacio más amplio en todas las direcciones para poder maniobrar de forma segura.
Esto es importante porque, en caso de que sea necesario, no podemos avanzar hacia atrás. Para lograr esto, he utilizado nuevamente **BFS**. 

En este caso, he vuelto a inicializar la frontera, manteniendo los nodos que ya fueron expandidos en el paso anteriro, pero ahora el nuevo nodo a expandir será el propio **target**. La condición de parada también cambia: en lugar de 
detenerse al alcanzar un único punto, ahora el proceso se detiene cuando se logran expandir un número de nodos concretos (marcados por una constante) o se vacía la frontera,
lo que permite flexibilidad en la maniobra ya que conocemos más espacio por el que podemos desplazarnos con seguridad.

A continuación, implementamos la **inflation layer**, cuyo propósito es expandir los obstáculos almacenados en la variable `walls` para que tengan un área de influencia. Esto nos permite evitar que el vehículo roce o toque estos
obstáculos durante su navegación.

Este proceso también se lleva a cabo utilizando **BFS** sobre cada uno de los nodos almacenados en `walls`, realizando un número finito de expansiones (definido en una constante). El coste asociado a cada celda de
la **inflation layer** se calcula de la siguiente forma:

        `cell_cost = (1 / euclidian_dist_node2obstacle) * scale_factor`

El resultado de la parte Deliverativa:

[Screencast from 2024-11-26 23-24-22.webm](https://github.com/user-attachments/assets/effdf072-988e-4d36-ada6-03085e2c369e)

