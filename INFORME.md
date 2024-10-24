1. ¿Qué política de planificación utiliza xv6-riscv para elegir el próximo proceso a ejecutarse?
Round Robin: tiene switch e interrupciones. Se puede ver el código de scheduler en proc.c. Además, en trap.c aparece la función clockintr, que maneja las interrupciones del temporizador. Pocas políticas interrumpen un proceso a mitad de camino: RR, STCF y MLFQ. Si fuese STCF debería tener una forma de contar cuánto dura cada proceso, pero no lo hace. También sé que no es un multilevel feedback queue, pues no hay división por prioridad, sino que hay una sola lista donde están todos (struct proc proc[NPROC]).

2. ¿Cuáles son los estados en los que un proceso puede permanecer en xv6-riscv y qué los hace cambiar de estado?
Estados:
UNUSED, USED, SLEEPING, RUNNABLE, RUNNING, ZOMBIE.

    UNUSED: al inicializar la tabla de procesos, cada uno se inicializa con estado UNUSED.

    USED: la función allocproc() busca procesos con estado UNUSED y les asigna un PID. Así inicializa el estado requerido para correr en el kernel, y su estado cambia a USED.

    RUNNABLE: el primer proceso de usuario inicializado pasa a RUNNABLE por la función procinit(). Ese primer proceso puede generar otros procesos mediante llamadas a fork(), los cuales se generarán con el estado RUNNABLE. Además, si un proceso estaba en estado SLEEPING por esperar algún recurso o evento, la función wakeup() lo reactiva y actualiza su estado a RUNNABLE.

    RUNNING: el scheduler revisa la tabla de procesos, y al encontrar uno en estado RUNNABLE, adquiere el lock y cambia su estado a RUNNING, guardando el contexto del proceso a ejecutar, luego de lo cual se ejecuta.

    ZOMBIE: la función exit() se usa cuando un proceso ha terminado su ejecución. El proceso pasa al estado ZOMBIE, y la información del mismo sigue existiendo en la tabla de procesos, pudiendo ser recogida por el padre, el cual puede estar esperando gracias a la función wait() a que el hijo termine. Los hijos de este proceso se reasignan al primer proceso inicializado.

3. ¿Qué es un quantum? ¿Dónde se define en el código? ¿Cuánto dura un quantum en xv6-riscv?
Quantum es una medida que indica la cantidad de tiempo que se ejecuta un proceso. En el código aparece en trap.c, en la función clockintr(). El quantum en xv6 es de una décima de segundo o 100 ms.

4. ¿En qué parte del código ocurre el cambio de contexto en xv6-riscv? ¿En qué funciones un proceso deja de ser ejecutado? ¿En qué funciones se elige el nuevo proceso a ejecutar?
El cambio de contexto ocurre en la función swtch(), que se encuentra aplicada en las funciones scheduler() y sched(). En scheduler, se busca constantemente programas que estén en estado RUNNABLE. Al encontrar uno, se cambia su estado a RUNNING y se hace un context switch con swtch(), luego de esta función parece ser que el proceso ya no está ejecutando. En sched() también se da un cambio de contexto, y esta función parece ser llamada cuando un programa cambia su estado de RUNNING a otro estado, ya sea por esperar I/O, por esperar la finalización de un evento u otra razón.

Funciones en las que un proceso deja de ser ejecutado:

    yield(): cambia el estado del proceso en ejecución a RUNNABLE y llama a otro proceso con sched().

    sleep(void *chan, struct spinlock *lk): pone un proceso en espera, cambiando su estado a SLEEPING. Libera los recursos acaparados por el proceso que está SLEEPING y llama a otro proceso usando sched().Podemos observar que sched() se usa para un cambio de contexto entre un programa que debe dejar de ejecutarse y un programa que puede ejecutarse.

    exit(int status): sale del proceso, convirtiéndolo en ZOMBIE. Deja la información en la tabla de procesos para que el padre la recoja. Se hace wakeup() en la función exit() para despertar a un padre que esté en espera (wait), y así recopile la información.

5. ¿El cambio de contexto consume tiempo de un quantum?
No, al terminar el quantum, el programa se interrumpe y ocurre un cambio de contexto, luego del cual se le da otro quantum al nuevo proceso. El quantum es el tiempo de acceso de un proceso al CPU, mientras que el cambio de contexto es el tiempo que toma el sistema en guardar registros del proceso actual y cargar registros del proceso a ejecutar, entre otras cosas.
