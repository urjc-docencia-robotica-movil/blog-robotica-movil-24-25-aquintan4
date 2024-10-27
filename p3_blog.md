# P3 Obstacle Avoidance
## 1.Observaciones iniciales
Lo primero en lo que me centraré está práctica es en laser, creo que es lo que marca la diferencia entre obtener un buen VFF o no. Lo primero que noto es que el rango de 0 a 180 grados es muy grande, no podemos obtener información de calidad con este rango ya que no todos los valores son igual de relevantes como hemos visto en clase. La parte de delante del vehículo es importante, mientras que los laterales del mismo no es tan relevante (en distancias suficientemente grandes, a distancias cortas nuestro coche debería reaccionar).

![Screenshot from 2024-10-27 16-38-39](https://github.com/user-attachments/assets/4a734684-4b87-4f60-b476-db2ab4733bcc)


 ¿Como pretendo filtrar esta información? **Utilizaré solo la información dentro de un rectangulo de la siguiente forma:**

 
![Screenshot from 2024-10-27 16-38-39](https://github.com/user-attachments/assets/dc7c3d1f-30f4-493f-988d-8d6bb4fff7af)

Matemáticamente: **Si la proyección en el eje horizontal de la distancia medida por el laser, supera un máximo, descarto esa medida para la formación del vector repulsivo (tendrá peso de cero en la media ponderada).**
He creado una constante y he visualizado cual sería el rango dentro del circuito:

![image](https://github.com/user-attachments/assets/84301fa7-57a2-4726-aaf0-aef1a5e673d7)
![Screenshot from 2024-10-27 17-17-14](https://github.com/user-attachments/assets/3d368389-d900-4d9e-ab98-14d488f96e93)

Ahora lo siguiente que haré será **hacer la media ponderada de todos los obstáculos que detecte el laser** (dentro del rango antes mencionado). Como también tengo que aumentar la frecuencia evitando operaciones costosas (sobretodo la librería Math) he hecho un filtro más sencillo, antes del anteriormente mencionado: la distancia del obstaculo sin importar la dirección de detección, si no supone un peligro paso a comprobar el siguiente medida. Luego tenemos el filtro del cuadrado y finalmente tenemos la media ponderada. Para la ponderación he pensado que a una distancia más corta al obstáculo, es más relevante que una más lejana. La distancia y la repulsión son inversamente proporcionales, pero quiero que sea más brusco por lo que esta distancia irá al cuadrado. Es decir dentro de la media lo que importa cada vector viene dado por:

```python3
        sqr_laser_inv = (1/(laser_data.values[i] * laser_data.values[i]))
```
El resultado es bueno: la dirección y sentido es correcta. Reacciona bien ante la presencia de varios obstáculos cambiantes. Por otro lado, debo escalar el módulo, para que reaccione más exagerado. Esto ya estaba previsto y es un grado de libertad importante en el VFF llamaré a este escalar **beta**.

## 3. El vector atractivo
Sabemos las posiciones a las que tenemos que llegar con un método, estas coordenadas son globales, es decir están en coordenadas del mapa. Necesitamos hacer un cambio de coordenadas globales a relativas. Tenemos el ejemplo hecho pero desglosandolo:
 1. Primero obtenemos las coordenadas tanto del robot como del target, invocando a `HAL.getPose3d()` y a `target.getPose()`.
 2. Restamos obtenemos la distancia entre el target y el robot (destination - origin).
 3. Finalmente, necesitamos saber la orientación: para ello proyectamos el yaw del robot (la orientación en el plano horizontal) en el vector distancia, esto nos permite obtener como debe ajustar su orientación para alinearse con el target.
Una vez sabemos donde está el target respecto a nuestro robot, tenemos la dirección y el sentido del vector atractivo.

Ahora tenemos que ajustar el módulo ya que a distancias grandes la atracción sería tan alta que eclipsaría la repulsión por completo. Para ello tenemos que saturar el módulo del vector atractivo a un valor razonable, también debemos limitar el vector a la baja, ya que cuando se acerque al target o esté justo en el la atracción sera 0 y el coche se dentendrá.
Para ello, he comenzado calculando el módulo y calcularía un factor de escala en caso de que no estuviera en el rango con la fórmula `scale_factor = max_magnitude / current_magnitude`. Computacionalmente hay soluciones más óptimas, a si que voy a limitar las componentes x e y del vector atractivo, esto hará que el vector atractivo se mantenga dentro de una circunferencia de radio ATTR_MAX sin mucho coste computacional.

## 3. Vector Resultante
Una vez calculadas las fuerzas repulsiva y atractiva, obtenemos el vector resultante sumando sus componentes. Esto nos da un solo vector que apunta en la dirección óptima hacia donde debemos movernos.
En el código, calculamos W como el ángulo de este vector resultante usando la función `math.atan2`, lo que proporciona una transición suave en la dirección. A la vez, V representa el módulo de este vector resultante, indicando la velocidad. Tanto W como V están limitados con un clamp para mantenerlos dentro de los límites definidos, lo que permite que el movimiento sea controlado y evite los cambios bruscos.

Finalmente, tenemos que ir probando y ajustando los valores de `ALPHA` y `BETA` que hagan que el coche tenga el comportamiento adecuado, no siendo ni muy "osado" ni muy "medioso", sobretodo evitar el equilibrio de fuerzas siempre tiene que haber alguna mayor a la otra

El resultado podemos encontrarlo [aquí](https://www.dropbox.com/scl/fi/zevwzjpga9kszei98l7hb/Screencast-from-2024-10-27-23-42-35.webm?rlkey=jvd57u6v6nc1rry6yaqkq2wdp&st=gq4pgvsj&dl=0), cabe destacar que el grabar pantalla ha hecho que se realentice la ejecución, aún así el resultado es bueno y cumple con lo esperado.
