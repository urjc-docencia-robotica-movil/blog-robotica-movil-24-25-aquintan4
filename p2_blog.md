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
  # Recortar la imagen utilizando slicing de NumPy
  cropped_image = image[rango columnas, rango filas]
  ```
  Haciendo pruebas compruebo que la parte superior de la imagen, no aporta ningúna información ya que la linea del horizonte se encuentra a mitad de pantalla, de todos modos cuando haya velocidades altas dejaré un
  margen de la parte superior por si hay desniveles y para evitar errores. En las velocidades bajas también recortaré la parte de abajo ya que no me aporta información relevante y es tiempo extra de cálculo. No he recortado el ancho de la imagen por el momento por que no sé como reaccionará a las curvas.

He marcado los límites que utilizaré dependiendo de la velocidad (Región naranja cuando velocidad es alta), Región azul (Cuando velocidad es más baja):
![image](https://github.com/user-attachments/assets/39931016-e467-448b-9a74-3f8ab6a2e652)

# 1. Centro de la linea (Referencia para controlar el coche)
Ahora tengo una imagen, con la linea filtrada y quiero obtener el **Centroide** de la imagen, para saber si esta centrado o hay algún error que corregir. Para ello he tenido que rebuscar buscar bastante información y el cálculo del centrómero está basado en la información de la siguiente [página](https://www.geeksforgeeks.org/python-opencv-find-center-of-contour/). Voy a explicarlo paso por paso:

El **Centroide** representa el "centro de masas" o más bien "centro de puntos" de una forma geométrica, donde hay una mayor densidad de puntos. En nuestro caso **representará el centro de la linea**. Para ello tenemos que trabajar con imágenes binarias. La máscara nos indicaba de si cada pixel de nuestra imagen ha pasado el filtro de color (pixel en blanco) o no (pixel en negro), por lo tanto es una imagen binaria que podemos utilizar para calcular el centroide:

Ahora utilizaré el método de la biblioteca CV2 `findContours()`:
```python3
contours, hierarchy = cv.findContours(image, mode, method)
```

