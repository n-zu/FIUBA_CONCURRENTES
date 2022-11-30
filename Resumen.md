# Técnicas de Programación Concurrentes

## Definiciones

### Concurrencia

- **Programa**: conjunto de datos e instrucciones, que se ejecutan secuencialmente en un procesador y acceden a datos almacenados en memoria

- **Programa concurrente**: conjunto de programas secuenciales que pueden ejecutarse en paralelo.

  > Ejecución del programa concurrente: resulta al ejecutar una secuencia de instrucciones atómicas que se obtiene de intercalar arbitrariamente las instrucciones atómicas de los procesos que lo componen

- **Proceso**: programa secuencial que forman parte de uno concurrente.

- **Sistema paralelo**: sistema compuesto por varios programas que se ejecutan **simultáneamente** en procesadores distintos.

- **Multitasking**: ejecución de múltiples procesos concurrentemente.

  > El scheduler coordina el acceso a los procesadores.

- **Multithreading**: [construcción](# "provista por lenguajes de programación") que permite la ejecución concurrente de threads dentro del mismo programa.

- **Desafíos** de la concurrencia:

  - **Sincronización**: coordinación temporal entre distintos
    procesos
  - **Comunicación**: datos que necesitan compartir los procesos
    para cumplir la función del programa

- **Sección Crítica**: sección de código que puede ser ejecutada por un solo proceso a la vez.

### Corrección

Se definen 2 tipos de propiedades de corrección

#### Safety

> "bad things don't happen". debe ser verdadera siempre

- **Exclusión mutua**: dos procesos no deben intercalar ciertas (sub)secuencias de instrucciones.
  > Ejemplo: incremento de variable global.
- **Ausencia de deadlock**: un sistema que aún no finalizó debe poder continuar realizando su tarea, es decir, avanzar productivamente.

#### Liveness

> debe volverse verdadera eventualmente

- **Ausencia de starvation**: todo proceso que esté listo para utilizar un recurso debe recibir dicho recurso eventualmente
- **Fairness**: equidad
  > Un escenario es (débilmente) fair, si en algún estado en el escenario, una instrucción que está continuamente habilitada, eventualmente aparece en el escenario.

## Modelos de concurrencia

### Estado mutable compartido

#### Locks

- Realizar **exclusión mutua**
- Métodos: `lock` y `unlock`
- Requieren soporte del hardware y SO

#### Semáforos

- Mecanismos de sincronismo
- Compuesto por:
  - `L`: cantidad de "recursos" disponibles
  - `V`: set de procesos esperando por un recurso
- Operaciones (atómicas):
  - `wait` : espera por un recurso
  - `signal`: libera un recurso
- Problema clásico: **productor-consumidor**

#### Barreras

- Permiten sincronizar varios threads en puntos determinados de un
  cálculo o algoritmo
- Métodos
  - `new(n)` : crea una barrera con `n` "participantes"
  - `wait`: espera a que todos los "participantes" lleguen a la barrera
  - `is_leader`: devuelve `true` si el thread que invoca es el primero en llegar a la barrera
- Las barreras son reutilizables automáticamente

#### Condvars

- Métodos:
  - `waitC`: espera a que se cumpla una condición
  - `signalC`: notifica a un thread que está esperando por una condición
  - `empty`: devuelve `true` si no hay threads esperando por una condición

#### Monitores

- Herramientas de sincronización que permite a los hilos tener exclusión mutua y la posibilidad de esperar (block) por que una condición se vuelva falsa.
- Tienen un mecanismo para señalizar otros hilos cuando su condición se cumple.

##### Synchronized

- Bloques synchronized: mecanismo propio de Java
- Bloque de código que se ejecuta de forma atómica
- Cada objeto tiene un lock asociado, que se utiliza para sincronizar el acceso a los métodos sincrónicos de la clase

### Paralelismo fork-join

el cómputo (task) es partido en sub-cómputos menores (subtasks).
Los resultados de estos se unen (join) para construir la solución al cómputo inicial.

- Las sub-tareas se pueden crear en cualquier momento de la ejecución de la tarea.
- Las tareas no deben bloquearse, excepto para esperar el final de las subtareas.
- Las tareas son independientes entre sí.

#### Robo de trabajo

- Los worker threads inactivos roban trabajo a threads ocupados, para realizar balanceo de carga
- Implementación
  - Cada "worker" tiene un [deque](# lista de doble entrada) de tareas
  - Las subtareas creadas x una tarea se colocan al final
  - Toma tareas del final de su cola
  - Roba tareas del principio de la cola de otro worker
- Detalles
  - Estadísticamente, se roban tareas mas grandes, lo cual rebalancea las tareas minimizando la necesidad de robo
  - Como toman de diferentes lados se minimiza el conflicto -> necesidad de locks
- Ventajas:
  - Los worker threads se comunican solamente cuando lo necesitan → menor necesidad de sincronización.
  - La implementación de la cola deque agrega bajo overhead de sincronización.

### Programación Asincrónica

> En Rust

- **Tareas** asincrónicas

  - Se pueden intercalar tareas en un único thread o en un pool de threads.
  - Son mucho más livianas que los threads.
  - Más rápidas de crear, más eficiente de cambiar de estado.
  - Menor overhead de memoria.

- **Futures**

  - operación sobre la que se puede consultar si se completó
  - `poll` no bloquea, devuelve Pending / Ready
  - cada vez que es polleado, avanza todo lo que puede
  - almacena lo necesario para realizar el pedido hecho por la invocación
  - rust llama poll solo cuando puede avanzar o retornar

- `async` / `await`

  - una función `async` retorna un future inmediatamente
  - Al ejecutar poll por primera vez sobre el retorno, se ejecuta el cuerpo de la función hasta el primer await

- **Executors**:

  - `block_on`
    - una función sincrónica que produce el valor final de la función asincrónica
    - No debe usarse en una función async (bloquea a todo el thread)
  - `spawn_local`
    - recibe un future y lo agrega a un pool que realizará polling en el block_on

- Cuando usar
  - Muchas tareas bloqueantes
  - No usar para computo pesado pq bloquea el thread hasta resolver

### Canales / Mensajes

### Actores

## Sistemas distribuidos

### Exclusion mutua

### Elección de líder

### Transacciones

### Ambientes Distribuidos
