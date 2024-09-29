# P1. Vacum Cleaner
### Álvaro Quintana
# 1. Algoritmo:

Inicialmente pensé en tratar de hacer espirales cuadradas hacia dentro e ir apuntando los sectores que iba barriendo, de esa forma podría recorrer facilmente todo el mapa, evitando repeticiones y completar la tarea en un tiempo ínfimo. El problema de este enfoque, en este problema, es **no poder autolocalizarnos**. Esto resulta en que nos es imposible realizar acciones tan sencillas como recorrer una distancia determinada o girar un ángulo específico. Esta incapacidad se debe a la naturaleza del enunciado: Nuestra aspiradora es un **producto robótico**, por lo que debe tener un precio lo más bajo posible para poder ser comercializado. Por lo tanto, la aspiradora, **debe de ser lo más sencilla posible** para mantener los costes de producción y mantenimiento lo más bajos posibles. Como es posible limpiar una habitación sin autolocalizarnos (auque resultaría muy útil) debemos sacrificar esta ventaja para mantener los cóstes al mínimo.<br>

La otra necesidad de un **producto robótico** es la **robustez**. Para que mi código sea lo más robusto posible he decidido simplificarlo lo máximo posible: *he seguido un comportamiento de Bump&Go y cuando se dan las condiciones apropiadas, el robot realiza una espiral para poder cubrir la mayor superficie posible*. Al usar este enfoque puramente reactivo, he prestado especial atención a la aleatoriedad, he tratado de añadir toda la aliatoriedad posible al sistema. En tiempos de cada estado, direcciones de giro y velocidades. Para asegurar que en algún momento saldré de situaciones complicadas como habitaciones cerradas o lugares estrechos.

*Este es el diagrama de estados de mi FSM:*

![Screenshot from 2024-09-29 22-11-20](https://github.com/user-attachments/assets/423e667a-749d-4791-8c3f-8ab2d06676b5)
