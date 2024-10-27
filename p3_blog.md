# P3 Obstacle Avoidance
## 1.Observaciones iniciales
Lo primero en lo que me centraré está práctica es en laser, creo que es lo que marca la diferencia entre obtener un buen VFF o no. Lo primero que noto es que el rango de 0 a 180 grados es muy grande, no podemos obtener información de calidad con este rango ya que no todos los valores son igual de relevantes como hemos visto en clase. La parte de delante del vehículo es importante, mientras que los laterales del mismo no es tan relevante (en distancias suficientemente grandes, a distancias cortas nuestro coche debería reaccionar).

![Screenshot from 2024-10-27 16-38-39](https://github.com/user-attachments/assets/4a734684-4b87-4f60-b476-db2ab4733bcc)


 ¿Como pretendo filtrar esta información? **Utilizaré solo la información dentro de un rectangulo de la siguiente forma:**

 
![Screenshot from 2024-10-27 16-38-39](https://github.com/user-attachments/assets/dc7c3d1f-30f4-493f-988d-8d6bb4fff7af)

Matemáticamente: **Si la proyección en el eje horizontal de la medida del laser supera un máximo, descarto esa medida para la formación del vector repulsivo (tendrá peso de cero en la media ponderada).**
He creado una constante y he visualizado cual sería el rango dentro del circuito:

![image](https://github.com/user-attachments/assets/84301fa7-57a2-4726-aaf0-aef1a5e673d7)
![Screenshot from 2024-10-27 17-17-14](https://github.com/user-attachments/assets/3d368389-d900-4d9e-ab98-14d488f96e93)

Ahora lo siguiente que haré será hacer la media ponderada de todos los obstáculos que detecte el laser (dentro del rango antes mencionado). Como también tengo que aumentar la frecuencia evitando operaciones costosas (sobretodo la librería Math) he hecho un filtro más sencillo, antes del anteriormente mencionado: la distancia del obstaculo sin importar la dirección de detección, si no supone un peligro paso a comprobar el siguiente medida. Luego tenemos el filtro del cuadrado y finalmente tenemos la media ponderada. Para la ponderación he pensado que a una distancia más corta al obstáculo, es más relevante que una más lejana. La distancia y la repulsión son inversamente proporcionales, pero quiero que sea más brusco por lo que esta distancia irá al cuadrado. Es decir dentro de la media lo que importa cada vector viene dado por:

```python3
        sqr_laser_inv = (1/(laser_data.values[i] * laser_data.values[i]))
```
El resultado es bueno: la dirección y sentido es correcta. Reacciona bien ante la presencia de varios obstáculos cambiantes. Por otro lado, debo escalar el módulo, para que reaccione más exagerado. Esto ya estaba previsto y es un grado de libertad importante en el VFF llamaré a este escalar **beta**.


