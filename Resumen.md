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

<!-- pagebreak -->

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

Do not communicate by sharing memory; instead, share memory by communicating.

#### Modelos de comunicación

- Comunicación
  - Sincrónica
  - Asincrónica (Buffer)
- Direccionamiento . Cómo se determina a quién dirigir un mensaje?
  - Simétrico
  - Asimétrico
  - Sin direccionamiento (matcheo por estructura del mensaje)
- Flujo de datos
  - Unidireccional
  - Bidireccional

#### Otros

- **Selective Input**: Permite escuchar en varios canales de forma bloqueante y desbloquearse con el primero que recibe un mensaje

### Actores

- **Actor**
  - Primitiva principal
  - Liviano
  - Encapsula comportamiento y estado
  - Compuesto por: **Address** y **Mailbox**
  - Es aislado (no comparte memoria), su estado es privado
  - Procesa un mensaje a la vez
- **Mensaje**
  - via de comunicación entre actores
  - procesados de forma asincrona

#### En Rust

- Cada actor se ejecuta dentro de un arbitrer
- Se ejecutan en un contexto de ejecución

<!-- pagebreak -->

## Sistemas distribuidos

### Exclusion mutua

#### Algoritmo Centralizado

1. Un proceso es elegido coordinador
2. Cuando un proceso quiere entrar a la SC, envía un mensaje al coordinador
3. Si no hay ningún proceso en la SC, el coordinador envía OK; si hay, el coordinador no envía respuesta hasta que se libere la SC

#### Algoritmo Distribuido

Cuando un proceso quiere entrar en una sección crítica, envía un [mensaje](# "con el nombre de la sección crítica, el número de proceso y el timestamp") a todos

1. Si no está en la CS y no quiere entrar, responde OK
2. Si está en la CS, no responde y encola el mensaje. Cuando sale de la SC, responde OK
3. Si quiere entrar en la CS, compara el timestamp y gana el menor

#### Algoritmo Token Ring

- Se conforma un anillo mediante conexiones punto a punto
- Al inicializar, el proceso 0 recibe un token que va circulando por el anillo
- Sólo el proceso que tiene el token puede entrar a la SC
- Cuando el proceso sale de la SC, continua circulando el token
- El proceso no puede entrar a otra SC con el mismo token

### Elección de líder

Varios algoritmos requieren de un coordinador con un rol especial (ej: algoritmos de exclusión mutua distribuida).

#### Algoritmo Bully

Cuando un proceso P nota que el coordinador no responde, inicia el proceso de elección:

1. P envía el mensaje ELECTION a todos los procesos que tengan número mayor
2. Si nadie responde, P gana la elección y es el nuevo coordinador
3. Si contesta algún proceso con número mayor, éste continúa con el proceso y P finaliza
4. El nuevo coordinador se anuncia con un mensaje COORDINATOR

Siempre gana el proceso con mayor número.

#### Algoritmo Ring

1. Los procesos están ordenados lógicamente; cada uno conoce a su sucesor
2. Cuando un proceso nota que el coordinador falló, arma un mensaje ELECTION que contiene su número de proceso y lo envía al sucesor
3. El proceso que recibe el mensaje, agrega su número de proceso a la lista dentro del mensaje y lo envía al sucesor
4. Cuando el proceso original recibe el mensaje, lo cambia a COORDINATOR y lo envía. El nuevo coordinador es el proceso de mayor número de la lista. La lista se mantiene para informar el nuevo anillo
5. Cuando este mensaje finaliza la circulación, se elimina del anillo

### Transacciones

Sistema está conformado por un conjunto de procesos independientes; cada uno puede fallar aleatoriamente.

#### Propiedades ACID

- **Atomicity**: La transacción se ejecuta completamente o no se ejecuta en absoluto
- **Consistency**: La transacción debe dejar el sistema en un estado consistente
- **Isolation**: Las transacciones concurrentes no deben interferir entre sí
- **Durability**: Los efectos de una transacción deben ser permanentes

#### Implementación

##### Private Workspace

- Al iniciar una transacción, el proceso recibe una copia de todos los archivos a los cuales tiene acceso
- Hasta que hace commit, el proceso trabaja con la copia
- Al hacer commit, se persisten los cambios
- Desventaja: extremadamente costoso salvo por optimizaciones

##### Write-ahead Log

- Los archivos se modifican in place, pero se mantiene una lista de los cambios aplicados (primero se escribe la lista y luego se modifica el archivo)
- Al commitear la transacción, se escribe un registro commit en el log
- Si la transacción se aborta, se lee el log de atrás hacia adelante para deshacer los cambios (rollback)

##### Commit en dos fases

- El coordinador es aquel proceso que ejecuta la transacción
- Fase 1: **PREPARE**
  1. El coordinador escribe prepare en su log y envía el mensaje prepare al resto de los procesos
  2. Los procesos que reciben el mensaje, escriben **READY** en el log y envían ready al coordinador
- Fase 2: **COMMIT**
  1. El coordinador hace los cambios y envía el mensaje commit al resto de los procesos
  2. Los procesos que reciben el mensaje, escriben commit en el log y envían finished al coordinador

#### Control de Concurrencia

##### Lockeo en 2 fases

- Fase de expansión: se toman todos los locks a usar
- Fase de contracción: se liberan todos los locks (no se pueden tomar nuevos locks)
- Garantiza propiedad serializable para las transacciones
- Pueden ocurrir deadlocks
- Strict two-phase locking: la contracción ocurre después del commit

##### Concurrencia Optimista

- Se asume que no habrá conflictos
- Al finalizar la transacción, se chequea si hubo conflictos

> Libre de deadlocks, pero costoso si hay conflictos

##### Timestamps

- Existen timestamps únicos globales para garantizar orden (ver algoritmo de relojes de Lamport)
- Cada archivo tiene dos timestamps: lectura y escritura y qué transacción hizo la última operación en cada caso
- Cada transacción al iniciarse recibe un timestamp
- Se compara el timestamp de la transacción con los timestamps del archivo:
  - Si es mayor, la transacción está en orden y se procede con la operación
  - Si es menor, la transacción se aborta
- Al commitear se actualizan los timestamps del archivo

#### Detección de Deadlocks

##### Algoritmo centralizado

- El proceso coordinador mantiene el grafo de uso de recursos
- Los procesos envían mensajes al coordinador cuando obtienen / liberan un recurso y el coordinador actualiza el grafo
- Si se genera un ciclo, hay deadlock
- Problema: los mensajes pueden llegar llegar desordenados y generar falsos deadlocks
- Posible solución: utilizar timestamps globales para ordenar los mensajes (algoritmo de Lamport)

##### Algoritmo distribuido

- Cuando un proceso debe esperar por un recurso, envía un probe message al proceso que tiene el recurso.
  - El mensaje contiene: id del proceso que se bloquea, id del proceso que envía el mensaje y id del proceso destinatario
- Al recibir el mensaje, el proceso actualiza el id del proceso que envía y el id del destinatario y lo envía a los procesos que tienen el recurso que necesita
- Si el mensaje llega al proceso original, tenemos un ciclo en el grafo

#### Prevención de Deadlocks

##### wait-die

- Se asigna un timestamp único a cada transacción
- Al querer tomar un recurso que posee otro proceso, si es:
  - Más viejo: espera
  - Más reciente: aborta la transacción

##### Wound-wait

- Se asigna un timestamp único a cada transacción
- Al querer tomar un recurso que posee otro proceso, si es:
  - Más viejo: aborta la transacción del más reciente y toma el recurso
  - Más reciente: espera

<!-- pagebreak -->

### Ambientes Distribuidos

#### Entidades

- **Entidad**: Es la unidad de cómputo de ambiente informático distribuido
  > Puede ser un proceso, un procesador, etc.
  - **Capacidades**
    - Lectura y escritura de memoria local
    - Procesamiento local
    - [Comunicación](# preparación, transmisión y recepción de mensajes)
    - Setear y resetear un reloj local
  - **Reactiva**: solo responde a [eventos externos](# mensaje/activación de reloj/impulso espontaneo)
- **Acción**: Secuencia finita e indivisible de operaciones (atómica)
- **Regla**: [estado × evento → acción](# Es la relación entre el evento que ocurre y el estado en el que se encuentra la entidad cuando ocurre dicho evento)
- **Comportamiento**: Es el conjunto de todas las reglas que obedece una entidad
  > Para cada posible evento y estado debe existir una única regla
  - Comportamiento Homogéneo: todas las entidades tienen el mismo comportamiento
- **Comunicación**
  - Una entidad se comunica con otras entidades mediante mensajes
  - Puede ocurrir que una entidad sólo pueda comunicarse con un subconjunto del resto de las entidades

#### Axiomas

- **Axiomas**
  - [Delays de comunicación finitos](# En ausencia de fallas los delays en la comunicación tienen una duración finita)
  - [Orientación local](# Una entidad puede distinguir entre sus vecinos NOUT y entre sus vecinos NIN)
- Restricciones de confiabilidad
  - Entrega garantizada
  - Confiabilidad parcial: no ocurrirán fallas
  - Confiabilidad total: no han ocurrido ni ocurrirán fallas
- Restricciones temporales
  - Delays de comunicación acotados
  - Delays de comunicación unitarios
  - Relojes sincronizados

#### Costo y Complejidad

Son las medidas de comparación de los algoritmos distribuidos

- Cantidad de actividades de comunicación
  - Cantidad de transmisiones o costo de mensajes, M
  - Carga de trabajo por entidad y carga de transmisión
- Tiempo
  - Tiempo total de ejecución del protocolo
  - Tiempo ideal de ejecución

#### Conocimiento

Conocimiento local: contenido de la memoria local de x y la información que se deriva
En ausencia de fallas, el conocimiento no puede perderse
