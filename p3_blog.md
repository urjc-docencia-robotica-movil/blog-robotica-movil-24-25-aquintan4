# P3 Obstacle Avoidance
## 1.Observaciones iniciales
Lo primero en lo que me centraré está práctica es en laser, creo que es lo que marca la diferencia entre obtener un buen VFF o no. Lo primero que noto es que el rango de 0 a 180 grados es muy grande, no podemos obtener información de calidad con este rango ya que no todos los valores son igual de relevantes como hemos visto en clase. La parte de delante del vehículo es importante, mientras que los laterales del mismo no es tan relevante (en distancias suficientemente grandes, a distancias cortas nuestro coche debería reaccionar). ¿Como pretendo filtrar esta información?, utilizaré solo la información dentro de un rectangulo de la siguiente forma:

![Screenshot from 2024-10-27 16-38-39](https://github.com/user-attachments/assets/4a734684-4b87-4f60-b476-db2ab4733bcc)

![Screenshot from 2024-10-27 16-38-39](https://github.com/user-attachments/assets/dc7c3d1f-30f4-493f-988d-8d6bb4fff7af)
