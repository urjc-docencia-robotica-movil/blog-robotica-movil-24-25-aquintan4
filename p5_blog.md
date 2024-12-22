# Practica 5: Algoritmo de Montecarlo
Esta práctica se compone de cuatro pasos claramente definidos. El primer paso consiste en muestrear el mapa con las partículas de forma aleatoria. Esto significa distribuir un número determinado de partículas en el mapa para realizar comparaciones posteriores. Es importante destacar que las partículas no pueden estar directamente ligadas a las coordenadas del mapa, ya que un mapa es una representación discretizada del espacio. Si, por ejemplo, cada píxel del mapa representara una gran distancia (como kilómetros), las partículas no podrían capturar de manera precisa la posición real del robot, y el proceso de localización sería inviable, por eso utilizamos las coordenadas del mundo para las
las partículas.

En nuestro caso, utilicé la función `init_particles`, que recibe como argumento el número de partículas a inicializar. Esta función convierte los límites del mapa en coordenadas globales y distribuye las partículas de forma uniforme dentro de esos límites mediante np.random.uniform. De esta forma las partículas estén bien repartidas en el espacio, que se encuentren dentro de los límites del mapa no significa que estén en lugares accesibles para el robot!

*Ejemplo de inicialización de 300 partículas:*
![image](https://github.com/user-attachments/assets/d56f476b-f7b5-48f1-abcb-63d07d10f4a2)


El siguiente paso consiste en aplicar el modelo probabilístico de movimiento. Si una partícula representa la posición del robot y sabemos cómo se está moviendo, esa partícula debe desplazarse de manera similar, pero añadiendo un ruido que modele posibles errores como imprecisiones en los sensores, derrapes, resbalones o fallos en los actuadores.

Para implementar esto, utilizo los datos de odometría proporcionados por `HAL.getOdom()` . **Comparo la odometría en el instante t con la del instante anterior t−1** para calcular los incrementos en las coordenadas x, y y en el ángulo yaw. Estas diferencias se aplican al movimiento de las partículas, junto con un modelo de ruido que introduce pequeñas variaciones aleatorias en las posiciones y orientaciones, permitiendo que las partículas reflejen más realista el desplazamiento del robot, teniendo en cuenta las incertidumbres del sistema.

*Ejemplo de aplicación del modelo probabilístico*
![motion_particles](https://github.com/user-attachments/assets/e6921a57-1ea8-485a-9ace-14570b208f62)
