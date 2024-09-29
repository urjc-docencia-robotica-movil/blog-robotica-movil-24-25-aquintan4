# P1. Vacum Cleaner
### Álvaro Quintana
# 1. Algoritmo:

Inicialmente pensé en tratar de hacer espirales cuadradas hacia dentro e ir apuntando los sectores que iba barriendo, de esa forma podría recorrer facilmente todo el mapa, evitando repeticiones y completar la tarea en un tiempo ínfimo. El problema de este enfoque, en este problema, es **no poder autolocalizarnos**. Esto resulta en que nos es imposible realizar acciones tan sencillas como recorrer una distancia determinada o girar un ángulo específico. Esta incapacidad se debe a la naturaleza del enunciado: Nuestra aspiradora es un **producto robótico**, por lo que debe tener un precio lo más bajo posible para poder ser comercializado. Por lo tanto, la aspiradora, **debe de ser lo más sencilla posible** para mantener los costes de producción y mantenimiento lo más bajos posibles. Como es posible limpiar una habitación sin autolocalizarnos (auque resultaría muy útil) debemos sacrificar esta ventaja para mantener los cóstes al mínimo.<br>

La otra necesidad de un **producto robótico** es la **robustez**. Para que mi código sea lo más robusto posible he decidido simplificarlo lo máximo posible: *he seguido un comportamiento de Bump&Go y cuando se dan las condiciones apropiadas, el robot realiza una espiral para poder cubrir la mayor superficie posible*. Al usar este enfoque puramente reactivo, he prestado especial atención a la aleatoriedad, he tratado de añadir toda la aliatoriedad posible al sistema. En tiempos de cada estado, direcciones de giro y velocidades. Para asegurar que en algún momento saldré de situaciones complicadas como habitaciones cerradas o lugares estrechos.

*Este es el diagrama de estados de mi FSM:*

![Screenshot from 2024-09-29 22-11-20](https://github.com/user-attachments/assets/423e667a-749d-4791-8c3f-8ab2d06676b5)
# 2. Python
Otro de mis problemas a lo largo de la practica ha sido enfrentarme a la sintaxis tan permisiva de Python acostumbrado a otros lenguajes. He intentado mantener mi código lo más limpio y organizado posible. Para lograr esto, he desarrollado las siguientes funciones:
```python3
# Función para manejar las transiciones de estado en la FSM
def go_state(new_state: FSM_States, duration: float = -1, min_state_duration: float = 0, max_state_duration: float = 0, new_turn_direction: int = 0)

# Funciones para la gestión de temporizadores
def create_timer(duration: float = 0, random_duration: bool = False, min_duration: float = 0, max_duration: float = 0)
def has_timer_expired()
def reset_timer()
def timer_remaining()

# Verifica si hay suficiente espacio para realizar un espiral utilizando datos de láser
def can_do_spiral()
```

Gracias a estas sencillas funciones, he logrado gestionar toda mi máquina de estados finita (FSM) de manera efectiva. Además, el diseño modular que he implementado me permite añadir nuevas funcionalidades con facilidad en caso de que sea necesario gracias al uso de
1. **Uso de Enums:**
   ```python3
   from enum import Enum
  
    class FSM_States(Enum):
        FORWARDING = 1
        BACKING = 2
        TURNING = 3
        DOING_SPIRAL = 4
        STOP = 5
    
    class State_Duration(Enum):
        ENDLESS_DURATION = -1
        RANDOM_DURATION = -2
    ```
   
2. **Funciones con valores por defecto:** Un ejemplo de esto es la función *go_state*, la cual **me ha permitido reutilizar una misma función para diferentes propósitos** sin añadir demasiada complejidad. Por ejemplo, al desarrollar el comportamiento *Bump & Go*, puedo utilizar esta función para manejar tres estados distintos: uno en el que quiero que el estado FORWARDING dure indefinidamente (hasta que ocurra un evento), otro con una duración fija (BACKING), y un tercero con una duración aleatoria (TURNING). ¡Todo esto con una sola función!
Otra cosa que me ha resultado muy útil es la posibilidad de declarar los tipos de dato en los argumentos de las funciones añadiendo :. Esto me ha ayudado significativamente a mejorar la legibilidad del código, ya que hace más evidente qué tipo de valor espera cada parámetro y facilita la comprensión general del flujo del programa.
   ```python3
   def go_state(new_state:FSM_States, duration:float=-1, min_state_duration:float=0, max_state_duration:float=0, new_turn_direction:int=0):
   ```

3. **Encapsulamiento de lógica:** He agrupado la lógica relacionada en funciones específicas, lo que ayuda a mantener el código organizado y facilita su comprensión.
La lógica de mi máquina de estados se puede resumir en el siguiente fragmento del código principal. Siendo conciso, claro y funcional:
```python3
while True:

    if (state == FSM_States.FORWARDING):
        if (HAL.getBumperData().state == 1):
            go_state(FSM_States.BACKING, State_Duration.RANDOM_DURATION, 0.5, 1)
        if (can_do_spiral()):
            go_state(FSM_States.DOING_SPIRAL, 15)
        HAL.setV(1)
        HAL.setW(0)
    
    elif (state == FSM_States.BACKING):
        if (has_timer_expired()):
            go_state(FSM_States.TURNING, State_Duration.RANDOM_DURATION, 1, 3)
        HAL.setV(-1)
        HAL.setW(0)

    elif (state == FSM_States.TURNING):
        if (has_timer_expired()):
            go_state(FSM_States.FORWARDING, State_Duration.ENDLESS_DURATION)
        HAL.setV(0)
        HAL.setW(turn_direction * random.uniform(0.5, 2))

    elif (state == FSM_States.DOING_SPIRAL):
        # V = r * w, r = t * SCALING_FACTOR
        v_speed = W_SPEED * (time.time() - state_init_time) * SCALING_FACTOR
        # clamp V
        if (v_speed > MAX_V):
            v_speed = MAX_V
            
        HAL.setV(v_speed)
        HAL.setW(W_SPEED * turn_direction)

        if (HAL.getBumperData().state == 1):
            go_state(FSM_States.BACKING, State_Duration.RANDOM_DURATION, 0.5, 1)
        if (has_timer_expired()):
            go_state(FSM_States.FORWARDING, State_Duration.ENDLESS_DURATION)

    elif (state == FSM_States.STOP):
        HAL.setV(0)
        HAL.setW(0)
```
