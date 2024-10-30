1. ¿Qué política de planificación utiliza xv6-riscv para elegir el próximo proceso a ejecutarse?
Round Robin: tiene switch e interrupciones. Se puede ver el código de scheduler en proc.c. Además, en trap.c aparece la función clockintr, que maneja las interrupciones del temporizador. Pocas políticas interrumpen un proceso a mitad de camino: RR, STCF y MLFQ. Si fuese STCF debería tener una forma de contar cuánto dura cada proceso, pero no lo hace. También sé que no es un multilevel feedback queue, pues no hay división por prioridad, sino que hay una sola lista donde están todos (struct proc proc[NPROC]).

2. ¿Cuáles son los estados en los que un proceso puede permanecer en xv6-riscv y qué los hace cambiar de estado?
Estados:
UNUSED, USED, SLEEPING, RUNNABLE, RUNNING, ZOMBIE.

    UNUSED: al inicializar la tabla de procesos, cada uno se inicializa con estado UNUSED.

    USED: la función allocproc() busca procesos con estado UNUSED y les asigna un PID. Así inicializa el estado requerido para correr en el kernel, y su estado cambia a USED.

     RUNNABLE: la función procinit() cambia el estado del primer proceso de usuario inicializado a RUNNABLE. Ese primer proceso puede generar otros procesos mediante llamadas a fork(), los cuales se generarán con el estado RUNNABLE. Además, si un proceso estaba en estado SLEEPING por esperar algún recurso o evento, la función wakeup() lo reactiva y actualiza su estado a RUNNABLE.

    RUNNING: el scheduler revisa la tabla de procesos, y al encontrar uno en estado RUNNABLE, adquiere el lock y cambia su estado a RUNNING, guardando el contexto del proceso a ejecutar, luego de lo cual se ejecuta.
    
    SLEEPING: la función sleep(void *chan, struct spinlock *lk) cambia el estado de RUNNABLE a SLEEPING. Es usada
cuando se espera por eventos específicos.

    ZOMBIE: la función exit() se usa cuando un proceso ha terminado su ejecución. El proceso pasa al estado ZOMBIE, y la información del mismo sigue existiendo en la tabla de procesos, pudiendo ser recogida por el padre, el cual puede estar esperando gracias a la función wait() a que el hijo termine. Los hijos de este proceso se reasignan al primer proceso inicializado.

3. ¿Qué es un quantum? ¿Dónde se define en el código? ¿Cuánto dura un quantum en xv6-riscv?
Quantum es el tiempo fijo que tiene asignado un proceso para usar el cpu. Está definido en start.c, en la función timerinit(), como 1000000 de ciclos de reloj, lo que equivale a 100ms en el simulador. 

4. ¿En qué parte del código ocurre el cambio de contexto en xv6-riscv? ¿En qué funciones un proceso deja de ser ejecutado? ¿En qué funciones se elige el nuevo proceso a ejecutar?
El cambio de contexto ocurre en la función swtch(), que se encuentra aplicada en las funciones scheduler() y sched(). En scheduler, se busca constantemente programas que estén en estado RUNNABLE. Al encontrar uno, se cambia su estado a RUNNING y se hace un context switch con swtch(), luego de esta función parece ser que el proceso ya no está ejecutando. En sched() también se da un cambio de contexto, y esta función parece ser llamada cuando un programa cambia su estado de RUNNING a otro estado, ya sea por esperar I/O, por esperar la finalización de un evento u otra razón.

Funciones en las que un proceso deja de ser ejecutado:

    yield(): cambia el estado del proceso en ejecución a RUNNABLE y llama a otro proceso con sched().

    sleep(void *chan, struct spinlock *lk): pone un proceso en espera, cambiando su estado a SLEEPING. Libera los recursos acaparados por el proceso que está SLEEPING y llama a otro proceso usando sched().Podemos observar que sched() se usa para un cambio de contexto entre un programa que debe dejar de ejecutarse y un programa que puede ejecutarse.

    exit(int status): sale del proceso, convirtiéndolo en ZOMBIE. Deja la información en la tabla de procesos para que el padre la recoja. Se hace wakeup() en la función exit() para despertar a un padre que esté en espera (wait), y así recopile la información.

5. ¿El cambio de contexto consume tiempo de un quantum?
Sí, el cambio de contexto consume una parte del quantum, por lo tanto mientras menor sea el quantum, más porcentaje de él será usado para el cambio de contexto.

## Segunda Parte: Medir operaciones de cómputo y de entrada/salida  

### Primera aproximación

[Tablas de Datos de Experimento 1](./tests/experimento1.md)

1. Describa los parámetros de los programas cpubench e iobench para este experimento (o sea, los define al principio y el valor de N. Tener en cuenta que podrían cambiar en experimentos futuros, pero que si lo hacen los resultados ya no serán comparables).

Métrica de Cpubench = total_cpu_kops / elapsed_ticks. N = 20.  
Métrica de IObench = total_iops / elapsed_ticks. N = 20.  

2. ¿Los procesos se ejecutan en paralelo? ¿En promedio, qué proceso o procesos se ejecutan primero? Hacer una observación cualitativa.

Al ejecutar xv6 en un único procesador mediante el comando make CPU=1 qemu, no hay paralelismo en el sentido de que los procesos no pueden ejecutarse simultáneamente; un solo CPU puede manejar únicamente un proceso a la vez. Sin embargo, el planificador de tipo Round Robin (RR) permite que la ejecución de estos procesos se multiplexe en el tiempo. Esto significa que el CPU alterna entre los procesos, asignando a cada uno un quantum de tiempo igual. Como el CPU se distribuye entre varios procesos, da la impresión de que varios procesos están activos al mismo tiempo, es decir que son paralelos.

Los procesos se ejecutan por orden de llegada. Esto puede verse en el start tick de cada proceso en los tests. 
Al ejecutar cpubench e iobench en segundo plano, el iobench queda en espera -en estado sleeping- hasta que cpubench termina su ejecucion. Además, si hay otros procesos iobench en segundo plano, como el archivo de lectura y escritura es el mismo para cada llamada iobench N &, deben esperar a que el primero libere dicho archivo para ejecutarse. Esto se evidencia en los tests, ya que el primer proceso iobound planificado se queda esperando la disponibilidad del recurso IO, por lo que devuelve un número alto de interrupciones (representado por todos los demás procesos que fueron planificados después de que comenzó la ejecución del proceso IO de marras).

3. ¿Cambia el rendimiento de los procesos iobound con respecto a la cantidad y tipo de procesos que se estén ejecutando en paralelo? ¿Por qué?

Según los tests realizados, los procesos iobound aumentan su rendimiento cuando están en simultáneo a otros procesos iobound, no son afectados por la concurrencia de procesos cpubound.

4. ¿Cambia el rendimiento de los procesos cpubound con respecto a la cantidad y tipo de procesos que se estén ejecutando en paralelo? ¿Por qué?

Al contrario que los procesos iobound, no se ven afectados practicamente por la concurrencia con otros procesos cpubound, no hay diferencias notables.

Sin embargo sí puede percibirse que hay un aumento del rendimiento en los procesos cpu-bound cuando están intercalados con procesos iobound. Probablemente porque logran un mejor equilibrio en el sistema en cuánto al tiempo de uso del CPU y los I/O.  

5. ¿Es adecuado comparar la cantidad de operaciones de cpu con la cantidad de operaciones iobound?

Probablemente haya una manera mejor. Con esta métrica se puede tener una visión general del planificador. 

## todo

Probar otras métricas como el turnaround_time, response_time y otras a pensar.
