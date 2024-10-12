# P2 Follow Line
## 1. Observaciones inciales
Esta practica tendrá un problema adicional y se trata del rendimiento, tengo que optimizar al máximo el filtrado de la imagen y 
el cálculo del control para hacer que pueda ir a una mayor frecuencia. Tal como hablamos en clase la curva entre la frecuencia de la aplicación
y la calidad del resultado.

**¿Como propongo aumentar la frecuencia?**

* *Evitar el uso de asignaciones inecesarias en cada iteración:* por ejemplo el array que contiene los límites
del Hue(Tono) para el color que queremos filtrar.

* *Recortar la imagen antes del filtrado de color:* Una imagen como hemos visto en clase es una matriz de `n_filas * n_columnas * 3 canales` lo que hace que sea un tamaño
  muy grande y nos lleve mucho tiempo de cómputo, para ello he pensado que puedo recortar regiones de interes atendiendo a la velocidad, a velocidades bajas obtendré regiónes mas cercanas al coche
  y a velocidades altas obtendré las regiones más alejadas del coche.

  ![Screenshot from 2024-10-12 10-45-27](https://github.com/user-attachments/assets/040fecc9-37a6-4644-aa13-7537d7707453)

  No recortaré exactamente las partes marcadas en el esquema, al recortar hará que se tenga que procesar menos información y por tanto será más rápido.
  La herramienta para recortar en cv:
  ```python3
  # Definir las coordenadas
  # x e y son las coordenadas del punto superior izquierdo
  x, y, width, height = 100, 100, 300, 300
  # Recortar la imagen utilizando slicing de NumPy
  cropped_image = image[y:y+height, x:x+width]
  ```
