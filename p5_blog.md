# Practica 5: Algoritmo de Montecarlo
Esta práctica se compone de cuatro pasos claramente definidos. El primer paso consiste en muestrear el mapa con las partículas de forma aleatoria. Esto significa distribuir un número determinado de partículas en el mapa para realizar comparaciones posteriores. Es importante destacar que las partículas no pueden estar directamente ligadas a las coordenadas del mapa, ya que un mapa es una representación discretizada del espacio. Si, por ejemplo, cada píxel del mapa representara una gran distancia (como kilómetros), las partículas no podrían capturar de manera precisa la posición real del robot, y el proceso de localización sería inviable, por eso utilizamos las coordenadas del mundo para las
las partículas.

En nuestro caso, utilicé la función `init_particles`, que recibe como argumento el número de partículas a inicializar. Esta función convierte los límites del mapa en coordenadas globales y distribuye las partículas de forma uniforme dentro de esos límites mediante np.random.uniform. De esta forma las partículas estén bien repartidas en el espacio, que se encuentren dentro de los límites del mapa no significa que estén en lugares accesibles para el robot!

*Ejemplo de inicialización de 300 partículas:*
![image](https://github.com/user-attachments/assets/d56f476b-f7b5-48f1-abcb-63d07d10f4a2)


El siguiente paso consiste en aplicar el modelo probabilístico de movimiento. Si una partícula representa la posición del robot y sabemos cómo se está moviendo, esa partícula debe desplazarse de manera similar, pero añadiendo un ruido que modele posibles errores como imprecisiones en los sensores, derrapes, resbalones o fallos en los actuadores.

Para implementar esto, utilizo los datos de odometría proporcionados por `HAL.getOdom()` . **Comparo la odometría en el instante t con la del instante anterior t−1** para calcular los incrementos en las coordenadas x, y y en el ángulo yaw. Estas diferencias se aplican al movimiento de las partículas, junto con un modelo de ruido que introduce pequeñas variaciones aleatorias en las posiciones y orientaciones, permitiendo que las partículas reflejen más realista el desplazamiento del robot, teniendo en cuenta las incertidumbres del sistema.

*Ejemplo de aplicación del modelo probabilístico de movimiento*<br>
![motion_particles](https://github.com/user-attachments/assets/e6921a57-1ea8-485a-9ace-14570b208f62)

En este punto, ya tenemos las partículas distribuidas aleatoriamente en el mundo (dentro de la zona de nuestro mapa) y siguiendo el movimiento del robot. Ahora necesitamos asignar un peso (`weights` en el código) a cada partícula para determinar qué tan probable es que esa partícula represente la posición real del robot. 

¿Cómo sabemos si una partícula representa la posición del robot? Comparando:

1. **Percepción sensorial real** del robot (a la que tenemos acceso directamente, en este caso es el láser real del robot).
2. **Percepción teórica o virtual** que tendría el robot si estuviera en la posición de esa partícula.

Para calcular esta percepción teórica, he implementado un algoritmo de ray tracing basado en el algoritmo **DDA** (Digital Differential Analyzer). Este algoritmo genera un rayo desde la posición de la partícula y lo avanza hasta que encuentra un obstáculo en el mapa o alcanza la distancia máxima medible por el sensor láser (en nuestro caso, esta distancia es 10, obtenida haciendo un `print(HAL.getLaserData().max_dist)`).

El cálculo de la percepción láser teórica es computacionalmente costoso, ya que implica trazar múltiples rayos para todas las partículas en cada ciclo de ejecución. Para acelerar este proceso, he implementado varias optimizaciones:

1. **Ejecución en paralelo con multiprocessing**: Utilizo varios hilos de ejecución distribuidos en todos los núcleos disponibles de la CPU. Esto permite que los cálculos de los láseres teóricos de todas las partículas se realicen en paralelo, reduciendo significativamente el tiempo necesario.

2. **Reducción del número de haces del láser**: En lugar de trabajar con los 180 haces que genera el láser por defecto, utilizo la constante `LASER_NUM_BEAMS` para limitar el número de haces analizados. Con solo 5 haces es suficiente para obtener información representativa del entorno, lo que reduce considerablemente la carga computacional.

3. **Saltos en el ray tracing**: Para evitar pintar un láser perfecto píxel a píxel en el mapa, empleo la constante `RAYTRACING_SKIP_STEPS`, que permite avanzar en saltos durante el trazado del rayo. Esto optimiza el cálculo al obtener prácticamente la misma información pero en menos tiempo.

Estas estrategias combinadas hacen que el cálculo de la percepción teórica sea más eficiente y rápida sacrificando exactitud y precisión.

Una vez obtenida la percepción real y la percepción teórica, calculo qué tan similares son utilizando la fórmula vista en clase:

![image](https://github.com/user-attachments/assets/b8467978-a312-401e-9387-6b0da849d611)


La fórmula refleja la intuición de que, cuanto más similares sean las percepciones sensoriales (real y teórica), mayor será la probabilidad de que esa partícula represente la posición del robot. 

Donde:

* Distancia_manhattan: es la suma de las diferencias absolutas entre los valores de cada vector láser (real y teórico).
* e : es el número de Euler.
1. **Mayor parecido**: Si la distancia Manhattan es 0 (es decir, los vectores son idénticos), el exponente será \( e^0 \), lo que resulta en un peso de 1, indicando máxima probabilidad.
2. **Menor parecido**: A medida que la distancia Manhattan aumenta, el exponente negativo hace que el peso tienda a 0, lo que refleja una probabilidad nula.

También hago una normalización para que la suma de los pesos de las partículas sea 1.

### Resample
#### Remuestreo de partículas

El último paso en el proceso consiste en tomar nuevas muestras basadas en las partículas anteriores, priorizando aquellas que tienen un mayor peso. Las partículas con mayor peso serán los candidatos ideales para ser remuestreadas, y para evitar que todas las partículas se ubiquen exactamente en las mismas posiciones, se les añade un ruido, basado en el **algoritmo de la ruleta**.

Para implementar este remuestreo de manera sencilla, utilizo la función `np.random.choice` de NumPy, donde se elige una nueva muestra de partículas de la colección original, con la probabilidad de selección (argumento `p=`) determinada por los pesos calculados previamente. Es decir, las partículas de la siguiente generación estarán distribuidas con mayor probabilidad cerca de las partículas con mayor peso de la generación anterior. Esto permite que el algoritmo afine gradualmente la estimación de la posición del robot.

Adicionalmente, para introducir variabilidad y evitar la sobreconfianza en una sola ubicación, añado ruido a las nuevas partículas. Este ruido se genera siguiendo una desviación estándar definida en las constantes `RESAMPLE_XY_NOISE_STD` y `RESAMPLE_ANGLE_NOISE_STD`, lo que permite un ajuste realista a la localización.

