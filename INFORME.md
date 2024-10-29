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
Quantum es una medida que indica la cantidad de tiempo que se ejecuta un proceso. En el código aparece en start.c, en la función timerinit(), medido como un intervalo de ciclos de reloj (1000000 por quantum). El quantum en xv6 es de una décima de segundo o 100 ms.

4. ¿En qué parte del código ocurre el cambio de contexto en xv6-riscv? ¿En qué funciones un proceso deja de ser ejecutado? ¿En qué funciones se elige el nuevo proceso a ejecutar?
El cambio de contexto ocurre en la función swtch(), que se encuentra aplicada en las funciones scheduler() y sched(). En scheduler, se busca constantemente programas que estén en estado RUNNABLE. Al encontrar uno, se cambia su estado a RUNNING y se hace un context switch con swtch(), luego de esta función parece ser que el proceso ya no está ejecutando. En sched() también se da un cambio de contexto, y esta función parece ser llamada cuando un programa cambia su estado de RUNNING a otro estado, ya sea por esperar I/O, por esperar la finalización de un evento u otra razón.

Funciones en las que un proceso deja de ser ejecutado:

    yield(): cambia el estado del proceso en ejecución a RUNNABLE y llama a otro proceso con sched().

    sleep(void *chan, struct spinlock *lk): pone un proceso en espera, cambiando su estado a SLEEPING. Libera los recursos acaparados por el proceso que está SLEEPING y llama a otro proceso usando sched().Podemos observar que sched() se usa para un cambio de contexto entre un programa que debe dejar de ejecutarse y un programa que puede ejecutarse.

    exit(int status): sale del proceso, convirtiéndolo en ZOMBIE. Deja la información en la tabla de procesos para que el padre la recoja. Se hace wakeup() en la función exit() para despertar a un padre que esté en espera (wait), y así recopile la información.

5. ¿El cambio de contexto consume tiempo de un quantum?
No, al terminar el quantum, el programa se interrumpe y ocurre un cambio de contexto, luego del cual se le da otro quantum al nuevo proceso. El quantum es el tiempo de acceso de un proceso al CPU, mientras que el cambio de contexto es el tiempo que toma el sistema en guardar registros del proceso actual y cargar registros del proceso a ejecutar, entre otras cosas.

## Segunda Parte: Medir operaciones de cómputo y de entrada/salida  

### Primera aproximación

Con el quantum original los experimentos arrojan los siguientes resultados:  

`$ iobench 20 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 4   | [iobench]        | 51      | 27         | 20             |
| 4   | [iobench]        | 53      | 47         | 19             |
| 4   | [iobench]        | 53      | 66         | 19             |
| 4   | [iobench]        | 53      | 85         | 19             |
| 4   | [iobench]        | 53      | 104        | 19             |
| 4   | [iobench]        | 53      | 123        | 19             |
| 4   | [iobench]        | 53      | 142        | 19             |
| 4   | [iobench]        | 53      | 161        | 19             |
| 4   | [iobench]        | 53      | 180        | 19             |
| 4   | [iobench]        | 53      | 199        | 19             |
| 4   | [iobench]        | 53      | 218        | 19             |
| 4   | [iobench]        | 53      | 237        | 19             |
| 4   | [iobench]        | 53      | 256        | 19             |
| 4   | [iobench]        | 53      | 275        | 19             |
| 4   | [iobench]        | 53      | 294        | 19             |
| 4   | [iobench]        | 53      | 313        | 19             |
| 4   | [iobench]        | 51      | 332        | 20             |
| 4   | [iobench]        | 56      | 352        | 18             |
| 4   | [iobench]        | 53      | 370        | 19             |
| 4   | [iobench]        | 53      | 389        | 19             |

`$ iobench 20 &; iobench 20 &; iobench 20 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 9   | [iobench]        | 146     | 569        | 17             |
| 2   | [iobench]        | 146     | 569        | 7              |
| 11  | [iobench]        | 146     | 569        | 7              |
| 12  | [iobench]        | 146     | 576        | 7              |
| 9   | [iobench]        | 146     | 576        | 7              |
| 11  | [iobench]        | 146     | 576        | 7              |
| 12  | [iobench]        | 146     | 583        | 7              |
| 9   | [iobench]        | 146     | 583        | 7              |
| 11  | [iobench]        | 146     | 583        | 7              |
| 12  | [iobench]        | 146     | 590        | 7              |
| 9   | [iobench]        | 146     | 590        | 7              |
| 11  | [iobench]        | 146     | 590        | 7              |
| 12  | [iobench]        | 146     | 597        | 7              |
| 9   | [iobench]        | 146     | 597        | 7              |
| 11  | [iobench]        | 146     | 597        | 7              |
| 12  | [iobench]        | 146     | 604        | 7              |
| 9   | [iobench]        | 146     | 604        | 7              |
| 11  | [iobench]        | 146     | 604        | 7              |
| 12  | [iobench]        | 170     | 611        | 6              |
| 9   | [iobench]        | 170     | 611        | 6              |
| 11  | [iobench]        | 146     | 611        | 7              |
| 12  | [iobench]        | 146     | 617        | 7              |
| 9   | [iobench]        | 146     | 617        | 7              |
| 11  | [iobench]        | 146     | 618        | 7              |
| 12  | [iobench]        | 146     | 624        | 7              |
| 9   | [iobench]        | 128     | 624        | 8              |
| 11  | [iobench]        | 146     | 625        | 7              |
| 12  | [iobench]        | 128     | 631        | 8              |
| 9   | [iobench]        | 146     | 632        | 7              |
| 11  | [iobench]        | 146     | 632        | 7              |
| 12  | [iobench]        | 146     | 639        | 7              |
| 9   | [iobench]        | 128     | 639        | 8              |
| 11  | [iobench]        | 128     | 639        | 8              |
| 12  | [iobench]        | 146     | 646        | 7              |
| 11  | [iobench]        | 146     | 647        | 7              |
| 9   | [iobench]        | 146     | 647        | 7              |
| 12  | [iobench]        | 146     | 653        | 7              |
| 11  | [iobench]        | 170     | 654        | 6              |
| 9   | [iobench]        | 146     | 654        | 7              |
| 12  | [iobench]        | 146     | 660        | 7              |
| 11  | [iobench]        | 128     | 660        | 8              |
| 9   | [iobench]        | 146     | 661        | 7              |
| 12  | [iobench]        | 146     | 668        | 7              |
| 11  | [iobench]        | 146     | 668        | 7              |
| 9   | [iobench]        | 146     | 668        | 7              |
| 12  | [iobench]        | 146     | 675        | 7              |
| 11  | [iobench]        | 146     | 675        | 7              |
| 9   | [iobench]        | 128     | 675        | 8              |
| 12  | [iobench]        | 146     | 682        | 7              |
| 11  | [iobench]        | 146     | 682        | 7              |
| 9   | [iobench]        | 146     | 683        | 7              |
|  12   | [iobench]        |   stdout mezclado de 11 y 12    |         |      8      |
|  11   | [iobench]        |  idem     |         |      8      |
| 9   | [iobench]        | 128     | 690        | 8              |
| 12  | [iobench]        | 146     | 697        | 7              |
| 11  | [iobench]        | 146     | 697        | 7              |
| 9   | [iobench]        | 146     | 698        | 7              |
| 12  | [iobench]        | 146     | 704        | 7              |
| 11  | [iobench]        | 146     | 704        | 7              |
| 9   | [iobench]        | 128     | 705        | 8              |



`$ cpubench 20 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 15  | [cpubench]       | 76690   | 1031       | 7              |
| 15  | [cpubench]       | 59648   | 1038       | 9              |
| 15  | [cpubench]       | 67104   | 1047       | 8              |
| 15  | [cpubench]       | 89472   | 1056       | 6              |
| 15  | [cpubench]       | 76690   | 1062       | 7              |
| 15  | [cpubench]       | 76690   | 1069       | 7              |
| 15  | [cpubench]       | 89472   | 1076       | 6              |
| 15  | [cpubench]       | 76690   | 1082       | 7              |
| 15  | [cpubench]       | 76690   | 1089       | 7              |
| 15  | [cpubench]       | 89472   | 1096       | 6              |
| 15  | [cpubench]       | 76690   | 1102       | 7              |
| 15  | [cpubench]       | 76690   | 1109       | 7              |
| 15  | [cpubench]       | 89472   | 1116       | 6              |
| 15  | [cpubench]       | 76690   | 1122       | 7              |
| 15  | [cpubench]       | 76690   | 1129       | 7              |
| 15  | [cpubench]       | 89472   | 1136       | 6              |
| 15  | [cpubench]       | 76690   | 1142       | 7              |
| 15  | [cpubench]       | 76690   | 1149       | 7              |
| 15  | [cpubench]       | 76690   | 1156       | 7              |
| 15  | [cpubench]       | 89472   | 1163       | 6              |

`$ cpubench 20 &; cpubench 20 &; cpubench 20 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 19  | [cpubench]       | stdout mezclado   |        | 7              |
| 21  | [cpubench]       | stdout mezclado  |        | 7              |
| 22  | [cpubench]       | 67104   | 1343       | 8              |
| 19  | [cpubench]       | 76690   | 1350       | 7              |
| 21  | [cpubench]       | 76690   | 1350       | 7              |
| 22  | [cpubench]       | 76690   | 1351       | 7              |
| 19  | [cpubench]       | 89472   | 1357       | 6              |
| 21  | [cpubench]       | 89472   | 1357       | 6              |
| 22  | [cpubench]       | 76690   | 1358       | 7              |
| 21  | [cpubench]       | 89472   | 1364       | 6              |
| 22  | [cpubench]       | 76690   | 1365       | 7              |
| 19  | [cpubench]       | 59648   | 1363       | 9              |
| 21  | [cpubench]       | 76690   | 1370       | 7              |
| 22  | [cpubench]       | 89472   | 1372       | 6              |
| 21  | [cpubench]       | 89472   | 1377       | 6              |
| 19  | [cpubench]       | 48802   | 1372       | 11             |
| 22  | [cpubench]       | 76690   | 1378       | 7              |
| 21  | [cpubench]       | 76690   | 1383       | 7              |
| 22  | [cpubench]       | 76690   | 1385       | 7              |
| 19  | [cpubench]       | 44736   | 1384       | 12             |
| 21  | [cpubench]       | 76690   | 1390       | 7              |
| 22  | [cpubench]       | 89472   | 1392       | 6              |
| 19  | [cpubench]       | 89472   | 1396       | 6              |
| 21  | [cpubench]       | 89472   | 1397       | 6              |
| 22  | [cpubench]       | 76690   | 1398       | 7              |
| 19  | [cpubench]       | 76690   | 1402       | 7              |
| 21  | [cpubench]       | 76690   | 1403       | 7              |
| 22  | [cpubench]       | 76690   | 1405       | 7              |
| 19  | [cpubench]       | 76690   | 1409       | 7              |
| 21  | [cpubench]       | 76690   | 1410       | 7              |
| 22  | [cpubench]       | 67104   | 1412       | 8              |
| 19  | [cpubench]       | 89472   | 1416       | 6              |
| 21  | [cpubench]       | 89472   | 1417       | 6              |
| 22  | [cpubench]       | 76690   | 1420       | 7              |
| 19  | [cpubench]       | 76690   | 1422       | 7              |
| 21  | [cpubench]       | 76690   | 1423       | 7              |
| 22  | [cpubench]       | 76690   | 1427       | 7              |
| 19  | [cpubench]       | 76690   | 1429       | 7              |
| 21  | [cpubench]       | 76690   | 1430       | 7              |
| 22  | [cpubench]       | 89472   | 1434       | 6              |
| 19  | [cpubench]       | 89472   | 1436       | 6              |
| 21  | [cpubench]       | 89472   | 1437       | 6              |
| 22  | [cpubench]       | 76690   | 1440       | 7              |
| 19  | [cpubench]       | 76690   | 1442       | 7              |
| 21  | [cpubench]       | 76690   | 1443       | 7              |
| 22  | [cpubench]       | 76690   | 1447       | 7              |
| 21  | [cpubench]       | 76690   | 1450       | 7              |
| 19  | [cpubench]       | 67104   | 1449       | 8              |
| 22  | [cpubench]       | 76690   | 1454       | 7              |
| 21  | [cpubench]       | 89472   | 1457       | 6              |
| 19  | [cpubench]       | 89472   | 1457       | 6              |
| 22  | [cpubench]       | 89472   | 1461       | 6              |
| 21  | [cpubench]       | 76690   | 1463       | 7              |
| 19  | [cpubench]       | 76690   | 1463       | 7              |
| 22  | [cpubench]       | 76690   | 1467       | 7              |
| 21  | [cpubench]       | 89472   | 1470       | 6              |
| 19  | [cpubench]       | 76690   | 1470       | 7              |
| 22  | [cpubench]       | 89472   | 1474       | 6              |
| 19  | [cpubench]       | 89472   | 1477       | 6              |
| 19  | [cpubench]       | 76690   | 1483       | 7              |

`iobench 20 &; cpubench 20 &; cpubench 20 &; cpubench 20 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 10  | [cpubench]       | 89472   | 17         | 6              |
| 7   | [cpubench]       | 67104   | 16         | 8              |
| 9   | [cpubench]       | 67104   | 16         | 8              |
| 10  | [cpubench]       | 76690   | 23         | 7              |
| 7   | [cpubench]       | 89472   | 24         | 6              |
| 9   | [cpubench]       | 76690   | 24         | 7              |
| 10  | [cpubench]       | 76690   | 30         | 7              |
| 7   | [cpubench]       | 76690   | 30         | 7              |
| 9   | [cpubench]       | 89472   | 31         | 6              |
| 10  | [cpubench]       | 89472   | 37         | 6              |
| 7   | [cpubench]       | 76690   | 37         | 7              |
| 9   | [cpubench]       | 76690   | 37         | 7              |
| 7   | [cpubench]       | 76690   | 44         | 7              |
| 10  | [cpubench]       | 67104   | 43         | 8              |
| 9   | [cpubench]       | 76690   | 44         | 7              |
| 7   | [cpubench]       | 89472   | 51         | 6              |
| 10  | [cpubench]       | 89472   | 51         | 6              |
| 9   | [cpubench]       | 89472   | 51         | 6              |
| 7   | [cpubench]       | 76690   | 57         | 7              |
| 10  | [cpubench]       | 76690   | 57         | 7              |
| 9   | [cpubench]       | 89472   | 58         | 6              |
| 7   | [cpubench]       | 89472   | 64         | 6              |
| 10  | [cpubench]       | 76690   | 64         | 7              |
| 9   | [cpubench]       | 76690   | 64         | 7              |
| 7   | [cpubench]       | 89472   | 71         | 6              |
| 9   | [cpubench]       | 89472   | 71         | 6              |
| 10  | [cpubench]       | 89472   | 71         | 6              |
| 7   | [cpubench]       | 76690   | 77         | 7              |
| 9   | [cpubench]       | 76690   | 77         | 7              |
| 10  | [cpubench]       | 76690   | 77         | 7              |
| 7   | [cpubench]       | 76690   | 84         | 7              |
| 10  | [cpubench]       | 76690   | 84         | 7              |
| 9   | [cpubench]       | 67104   | 84         | 8              |
| 7   | [cpubench]       | 89472   | 91         | 6              |
| 10  | [cpubench]       | 89472   | 91         | 6              |
| 9   | [cpubench]       | 76690   | 92         | 7              |
| 7   | [cpubench]       | 76690   | 97         | 7              |
| 10  | [cpubench]       | 76690   | 97         | 7              |
| 9   | [cpubench]       | 89472   | 99         | 6              |
| 10  | [cpubench]       | 76690   | 104        | 7              |
| 7   | [cpubench]       | 76690   | 104        | 7              |
| 9   | [cpubench]       | 76690   | 105        | 7              |
| 10  | [cpubench]       | 89472   | 111        | 6              |
| 7   | [cpubench]       | 89472   | 111        | 6              |
| 9   | [cpubench]       | 76690   | 112        | 7              |
| 10  | [cpubench]       | 76690   | 117        | 7              |
| 7   | [cpubench]       | 76690   | 117        | 7              |
| 9   | [cpubench]       | 89472   | 119        | 6              |
| 10  | [cpubench]       | 89472   | 124        | 6              |
| 7   | [cpubench]       | 76690   | 124        | 7              |
| 9   | [cpubench]       | 76690   | 125        | 7              |
| 10  | [cpubench]       | 76690   | 130        | 7              |
| 7   | [cpubench]       | 89472   | 131        | 6              |
| 9   | [cpubench]       | 76690   | 132        | 7              |
| 10  | [cpubench]       | 76690   | 137        | 7              |
| 7   | [cpubench]       | 76690   | 137        | 7              |
| 9   | [cpubench]       | 89472   | 139        | 6              |
| 10  | [cpubench]       | 89472   | 144        | 6              |
| 7   | [cpubench]       | 76690   | 144        | 7              |
| 9   | [cpubench]       | 76690   | 145        | 7              |
| 5   | [iobench]        | 6       | 16         | 153            |
| 5   | [iobench]        | 51      | 169        | 20             |
| 5   | [iobench]        | 53      | 189        | 19             |
| 5   | [iobench]        | 51      | 208        | 20             |
| 5   | [iobench]        | 53      | 228        | 19             |
| 5   | [iobench]        | 53      | 247        | 19             |
| 5   | [iobench]        | 51      | 266        | 20             |
| 5   | [iobench]        | 53      | 286        | 19             |
| 5   | [iobench]        | 53      | 305        | 19             |
| 5   | [iobench]        | 51      | 324        | 20             |
| 5   | [iobench]        | 53      | 344        | 19             |
| 5   | [iobench]        | 51      | 363        | 20             |
| 5   | [iobench]        | 53      | 383        | 19             |
| 5   | [iobench]        | 51      | 402        | 20             |
| 5   | [iobench]        | 53      | 422        | 19             |
| 5   | [iobench]        | 53      | 441        | 19             |
| 5   | [iobench]        | 51      | 460        | 20             |
| 5   | [iobench]        | 53      | 480        | 19             |
| 5   | [iobench]        | 51      | 499        | 20             |
| 5   | [iobench]        | 53      | 519        | 19             |

`cpubench 20 &; iobench 20 &; iobench 20 &; iobench 20 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 14  | [cpubench]      | 76690   | 2173       | 7              |
| 14  | [cpubench]      | 76690   | 2180       | 7              |
| 19  | [iobench]       | 64      | 2173       | 16             |
| 16  | [iobench]       | 53      | 2173       | 19             |
| 14  | [cpubench]      | 76690   | 2187       | 7              |
| 14  | [cpubench]      | 76690   | 2194       | 7              |
| 19  | [iobench]       | 68      | 2189       | 15             |
| 14  | [cpubench]      | 76690   | 2201       | 7              |
| 16  | [iobench]       | 51      | 2192       | 20             |
| 18  | [iobench]       | 24      | 2173       | 41             |
| 14  | [cpubench]      | 76690   | 2208       | 7              |
| 14  | [cpubench]      | 76690   | 2215       | 7              |
| 19  | [iobench]       | 56      | 2204       | 18             |
| 14  | [cpubench]      | 76690   | 2222       | 7              |
| 16  | [iobench]       | 60      | 2212       | 17             |
| 14  | [cpubench]      | 76690   | 2229       | 7              |
| 19  | [iobench]       | 73      | 2222       | 14             |
| 14  | [cpubench]      | 76690   | 2236       | 7              |
| 18  | [iobench]       | 35      | 2214       | 29             |
| 16  | [iobench]       | 73      | 2230       | 14             |
| 14  | [cpubench]      | 76690   | 2243       | 7              |
| 19  | [iobench]       | 68      | 2236       | 15             |
| 14  | [cpubench]      | 76690   | 2250       | 7              |
| 16  | [iobench]       | 64      | 2244       | 16             |
| 14  | [cpubench]      | 76690   | 2257       | 7              |
| 19  | [iobench]       | 78      | 2251       | 13             |
| 14  | [cpubench]      | 76690   | 2264       | 7              |
| 18  | [iobench]       | 34      | 2243       | 30             |
| 14  | [cpubench]      | 89472   | 2271       | 6              |
| 16  | [iobench]       | 51      | 2260       | 20             |
| 14  | [cpubench]      | 89472   | 2278       | 6              |
| 19  | [iobench]       | 48      | 2264       | 21             |
| 14  | [cpubench]      | 89472   | 2285       | 6              |
| 16  | [iobench]       | 68      | 2280       | 15             |
| 19  | [iobench]       | 85      | 2285       | 12             |
| 14  | [cpubench]      | 76690   | 2291       | 7              |
| 14  | [cpubench]      | 76690   | 2298       | 7              |
| 14  | [cpubench]      | 76690   | 2305       | 7              |
| 16  | [iobench]       | 53      | 2295       | 19             |
| 18  | [iobench]       | 24      | 2273       | 42             |
| 19  | [iobench]       | 53      | 2297       | 19             |
| 16  | [iobench]       | 113     | 2314       | 9              |
| 18  | [iobench]       | 113     | 2315       | 9              |
| 19  | [iobench]       | 113     | 2316       | 9              |
| 16  | [iobench]       | 113     | 2323       | 9              |
| 18  | [iobench]       | 113     | 2324       | 9              |
| 19  | [iobench]       | 113     | 2325       | 9              |
| 16  | [iobench]       | 102     | 2332       | 10             |
| 18  | [iobench]       | 93      | 2333       | 11             |
| 19  | [iobench]       | 93      | 2334       | 11             |
| 16  | [iobench]       | 113     | 2342       | 9              |
| 18  | [iobench]       | 113     | 2344       | 9              |
| 19  | [iobench]       | 113     | 2345       | 9              |
| 16  | [iobench]       | 102     | 2351       | 10             |
| 18  | [iobench]       | 113     | 2354       | 9              |
| 19  | [iobench]       | 102     | 2354       | 10             |
| 16  | [iobench]       | 102     | 2361       | 10             |
| 18  | [iobench]       | 113     | 2364       | 9              |
| 19  | [iobench]       | 113     | 2364       | 9              |
| 16  | [iobench]       | 102     | 2371       | 10             |
| 18  | [iobench]       | 102     | 2373       | 10             |
| 19  | [iobench]       | 102     | 2373       | 10             |
| 16  | [iobench]       | 113     | 2381       | 9              |
| 18  | [iobench]       | 128     | 2383       | 8              |
| 19  | [iobench]       | 113     | 2383       | 9              |
| 16  | [iobench]       | 113     | 2390       | 9              |
| 18  | [iobench]       | 113     | 2391       | 9              |
| 19  | [iobench]       | 128     | 2392       | 8              |
| 16  | [iobench]       | 113     | 2399       | 9              |
| 19  | [iobench]       | 113     | 2400       | 9              |
| 18  | [iobench]       | 113     | 2400       | 9              |
| 16  | [iobench]       | 128     | 2408       | 8              |
| 19  | [iobench]       | 128     | 2409       | 8              |
| 18  | [iobench]       | 128     | 2409       | 8              |
| 16  | [iobench]       | 56      | 2416       | 18             |
| 18  | [iobench]       | 56      | 2417       | 18             |
| 18  | [iobench]       | 53      | 2435       | 19             |
| 18  | [iobench]       | 51      | 2454       | 20             |
| 18  | [iobench]       | 53      | 2474       | 19             |
| 18  | [iobench]       | 54      | 2494       | 18             |
| 19  | [iobench]       | 34      | 2515       | 30             |
| 19  | [iobench]       | 34      | 2537       | 30             |
| 19  | [iobench]       | 34      | 2558       | 30             |

Con el quantum 10 veces más chico que el original los experimentos arrojan los siguientes resultados:  

`$ iobench 20 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 4	  | [iobench]	| 5 |	454	 |  176 |
| 4	  | [iobench]	| 5 |	630	 |  175 |
| 4	  | [iobench]	| 5 |	806	 |  176 |
| 4	  | [iobench]	| 5 |	983	 |  174 |
| 4	  | [iobench]	| 5 |	1157 |  	176 |
| 4	  | [iobench]	| 5 |	1333 |  	177 |
| 4	  | [iobench]	| 5 |	1510 |  	176 |
| 4	  | [iobench]	| 5 |	1687 |  	175 |
| 4	  | [iobench]	| 5 |	1863 |  	175 |
| 4	  | [iobench]	| 5 |	2039 |  	175 |
| 4	  | [iobench]	| 5 |	2214 |  	175 |
| 4	  | [iobench]	| 5 |	2390 |  	177 |
| 4	  | [iobench]	| 5 |	2567 |  	177 |
| 4	  | [iobench]	| 5 |	2744 |  	177 |
| 4	  | [iobench]	| 5 |	2921 |  	175 |
| 4	  | [iobench]	| 5 |	3096 |  	175 |
| 4	  | [iobench]	| 5 |	3271 |  	177 |
| 4	  | [iobench]	| 5 |	3448 |  	176 |
| 4	  | [iobench]	| 5 |	3625 |  	176 |
| 4	  | [iobench]	| 5 |	3801 |  	176 |

`$ iobench 20 &; iobench 20 &; iobench 20 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 8	| [iobench]	| 13	| 6001 |	74 |
| 10 | 	[iobench] | 	13 | 	6001| 	74|
| 11 | 	[iobench] | 	13 | 	6001| 	76|
| 8	 | [iobench]	 | 13	 | 6075	| 74 |
| 11 | 	[iobench] | 	13 | 	6077| 	74|
| 10 | 	[iobench] | 	13 | 	6076| 	75|
| 8	 | [iobench]	 | 15	 | 6150	| 68 |
| 10 | 	[iobench] | 	14 | 	6151| 	71|
| 11 | 	[iobench] | 	14 | 	6151| 	71|
| 8	 | [iobench]	 | 14	 | 6219	| 73 |
| 11 | 	[iobench] | 	14 | 	6222| 	71|
| 10 | 	[iobench] | 	14 | 	6222| 	72|
| 8	 | [iobench]	 | 13	 | 6292	| 74|
| 11 | 	[iobench] | 	13 | 	6293| 	75|
| 10 | 	[iobench] | 	13 | 	6295| 	74|
| 8	 | [iobench]	 | 15	 | 6367	| 68|
| 11 | 	[iobench] | 	13 | 	6368| 	74|
| 10 | 	[iobench] | 	14 | 	6369| 	73|
| 8	 | [iobench]	 | 14	 | 6436	| 72|
| 11 | 	[iobench] | 	13 | 	6442| 	76|
| 10 | 	[iobench] | 	13 | 	6443| 	77|
| 8	 | [iobench]	 | 13	 | 6508	| 78|
| 11 | 	[iobench] | 	13 | 	6519| 	76|
| 10 | 	[iobench] | 	12 | 	6520| 	81|
| 8	 | [iobench]	 | 13	 | 6587	| 74|
| 11 | 	[iobench] | 	13 | 	6595| 	74|
| 10 | 	[iobench] | 	14 | 	6601| 	71|
| 8	 | [iobench]	 | 13	 | 6661	| 76|
| 11 | 	[iobench] | 	13 | 	6669| 	75|
| 10 | 	[iobench] | 	13 | 	6672| 	76|
| 8	 | [iobench]	 | 12	 | 6737	| 79|
| 11 | 	[iobench] | 	13 | 	6744| 	76|
| 10 | 	[iobench] | 	13 | 	6748| 	74|
| 8	 | [iobench]	 | 14	 | 6816	| 71|
| 11 | 	[iobench] | 	14 | 	6820| 	73|
| 10 | 	[iobench] | 	14 | 	6823| 	72|
| 8	 | [iobench]	 | 13	 | 6888	| 76|
| 11 | 	[iobench] | 	13 | 	6893| 	76|
| 10 | 	[iobench] | 	13 | 	6895| 	74|
| 8	 | [iobench]	 | 14	 | 6964	| 70|
| 10 | 	[iobench] | 	15 | 	6970| 	67|
| 11 | 	[iobench] | 	14 | 	6969| 	69|
| 8	 | [iobench]	 | 14	 | 7034	| 71|
| 10 | 	[iobench] | 	14 | 	7038| 	69|
| 11 | 	[iobench] | 	14 | 	7038| 	70|
| 8	 | [iobench]	 | 14	 | 7105	| 69|
| 10 | 	[iobench] | 	14 | 	7107| 	69|
| 11 | 	[iobench] | 	15 | 	7109| 	68|
| 8	 | [iobench]	 | 15	 | 7174	| 67|
| 11 | 	[iobench] | 	15 | 	7178| 	67|
| 10 | 	[iobench] | 	14 | 	7177| 	69|
| 8	 | [iobench]	 | 14	 | 7241	| 69|
| 11 | 	[iobench] | 	14 | 	7245| 	70|
| 10 | 	[iobench] | 	14 | 	7246| 	71|
| 8	 | [iobench]	 | 13	 | 7310	| 77|
| 11 | 	[iobench] | 	12 | 	7315| 	79|
| 10 | 	[iobench] | 	12 | 	7317| 	85|
| 8	 | [iobench]	 | 12	 | 7388	| 81|
| 11 | 	[iobench] | 	13 | 	7394| 	78|
| 10 | 	[iobench] | 	12 | 	7403| 	79|

`$ cpubench 20 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 14 |	[cpubench] |	7780 |	8990	| 69 |
| 14 |	[cpubench] |	8012 |	9059	| 67 |
| 14 |	[cpubench] |	8012 |	9127	| 67 |
| 14 |	[cpubench] |	8012 |	9194	| 67 |
| 14 |	[cpubench] |	8012 |	9262	| 67 |
| 14 |	[cpubench] |	8012 |	9329	| 67 |
| 14 |	[cpubench] |	8012 |	9396	| 67 |
| 14 |	[cpubench] |	8012 |	9464	| 67 |
| 14 |	[cpubench] |	8012 |	9531	| 67 |
| 14 |	[cpubench] |	8012 |	9598	| 67 |
| 14 |	[cpubench] |	8012 |	9666	| 67 |
| 14 |	[cpubench] |	8012 |	9733	| 67 |
| 14 |	[cpubench] |	8012 |	9800	| 67 |
| 14 |	[cpubench] |	8012 |	9868	| 67 |
| 14 |	[cpubench] |	8012 |	9935	| 67 |
| 14 |	[cpubench] |	8012 |	10002| 	67 |
| 14 |	[cpubench] |	8133 |	10070| 	66 |
| 14 |	[cpubench] |	8012 |	10137| 	67 |
| 14 |	[cpubench] |	8012 |	10204| 	67 |
| 14 |	[cpubench] |	8012 |	10271| 	67 |

`$ cpubench 20 &; cpubench 20 &; cpubench 20 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 21 |	[cpubench] |	8012 |	11901 |	67 |
| 20 |	[cpubench] |	8133 |	11968 |	66 |
| 18 |	[cpubench] |	8012 |	11968 |	67 |
| 21 |	[cpubench] |	7894 |	11969 |	68 |
| 20 |	[cpubench] |	8012 |	12034 |	67 |
| 18 |	[cpubench] |	8012 |	12035 |	67 |
| 21 |	[cpubench] |	8133 |	12038 |	66 |
| 20 |	[cpubench] |	8133 |	12101 |	66 |
| 18 |	[cpubench] |	8012 |	12102 |	67 |
| 21 |	[cpubench] |	8012 |	12104 |	67 |
| 20 |	[cpubench] |	8012 |	12167 |	67 |
| 18 |	[cpubench] |	8012 |	12169 |	67 |
| 21 |	[cpubench] |	8133 |	12171 |	66 |
| 20 |	[cpubench] |	8133 |	12234 |	66 |
| 18 |	[cpubench] |	8012 |	12236 |	67 |
| 21 |	[cpubench] |	8012 |	12237 |	67 |
| 20 |	[cpubench] |	8133 |	12300 |	66 |
| 21 |	[cpubench] |	8133 |	12304 |	66 |
| 18 |	[cpubench] |	5710 |	12304 |	94 |
| 20 |	[cpubench] |	8133 |	12367 |	66 |
| 21 |	[cpubench] |	8012 |	12370 |	67 |
| 18 |	[cpubench] |	8133 |	12399 |	66 |
| 20 |	[cpubench] |	8012 |	12434 |	67 |
| 21 |	[cpubench] |	8133 |	12437 |	66 |
| 18 |	[cpubench] |	8133 |	12465 |	66 |
| 20 |	[cpubench] |	8012 |	12501 |	67 |
| 21 |	[cpubench] |	8133 |	12504 |	66 |
| 18 |	[cpubench] |	8133 |	12532 |	66 |
| 20 |	[cpubench] |	8012 |	12568 |	67 |
| 21 |	[cpubench] |	8133 |	12570 |	66 |
| 18 |	[cpubench] |	8133 |	12598 |	66 |
| 20 |	[cpubench] |	8012 |	12635 |	67 |
| 21 |	[cpubench] |	8133 |	12637 |	66 |
| 18 |	[cpubench] |	8133 |	12665 |	66 |
| 18 |	[cpubench] |	8133 |	12731 |	66 |
| 21 |	[cpubench] |	8133 |	12770 |	66 |
| 20 |	[cpubench] |	8012 |	12770 |	67 |
| 18 |	[cpubench] |	7254 |	12798 |	74 |
| 21 |	[cpubench] |	8012 |	12836 |	67 |
| 20 |	[cpubench] |	8012 |	12837 |	67 |
| 18 |	[cpubench] |	8012 |	12872 |	67 |
| 21 |	[cpubench] |	8133 |	12903 |	66 |
| 20 |	[cpubench] |	8012 |	12904 |	67 |
| 18 |	[cpubench] |	6971 |	12939 |	77 |
| 21 |	[cpubench] |	8133 |	12970 |	66 |
| 20 |	[cpubench] |	8012 |	12971 |	67 |
| 18 |	[cpubench] |	8133 |	13016 |	66 |
| 21 |	[cpubench] |	8133 |	13036 |	66 |
| 20 |	[cpubench] |	8012 |	13038 |	67 |
| 18 |	[cpubench] |	8012 |	13082 |	67 |
| 21 |	[cpubench] |	8133 |	13103 |	66 |
| 20 |	[cpubench] |	8012 |	13105 |	67 |
| 18 |	[cpubench] |	8133 |	13149 |	66 |
| 21 |	[cpubench] |	7561 |	13169 |	71 |
| 20 |	[cpubench] |	7669 |	13173 |	70 |
| 18 |	[cpubench] |	8133 |	13216 |	66 |


`iobench 20 &; cpubench 20 &; cpubench 20 &; cpubench 20 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 30 |	[cpubench] |	8133 |	16532 |	66 |
| 27 |	[cpubench] |	7894 |	16533 |	68 |
| 29 |	[cpubench] |	6882 |	16533 |	78 |
| 30 |	[cpubench] |	8012 |	16599 |	67 |
| 27 |	[cpubench] |	7894 |	16602 |	68 |
| 29 |	[cpubench] |	7780 |	16611 |	69 |
| 30 |	[cpubench] |	8012 |	16666 |	67 |
| 27 |	[cpubench] |	7780 |	16670 |	69 |
| 29 |	[cpubench] |	7561 |	16680 |	71 |
| 30 |	[cpubench] |	8012 |	16733 |	67 |
| 27 |	[cpubench] |	7894 |	16740 |	68 |
| 29 |	[cpubench] |	7561 |	16751 |	71 |
| 30 |	[cpubench] |	8012 |	16801 |	67 |
| 27 |	[cpubench] |	7780 |	16808 |	69 |
| 29 |	[cpubench] |	7780 |	16823 |	69 |
| 30 |	[cpubench] |	7894 |	16869 |	68 |
| 27 |	[cpubench] |	7780 |	16878 |	69 |
| 29 |	[cpubench] |	7780 |	16892 |	69 |
| 30 |	[cpubench] |	7561 |	16937 |	71 |
| 27 |	[cpubench] |	7780 |	16947 |	69 |
| 29 |	[cpubench] |	7561 |	16961 |	71 |
| 30 |	[cpubench] |	7063 |	17008 |	76 |
| 27 |	[cpubench] |	7669 |	17016 |	70 |
| 29 |	[cpubench] |	7780 |	17033 |	69 |
| 30 |	[cpubench] |	7780 |	17085 |	69 |
| 27 |	[cpubench] |	7894 |	17087 |	68 |
| 29 |	[cpubench] |	7561 |	17102 |	71 |
| 30 |	[cpubench] |	7780 |	17154 |	69 |
| 27 |	[cpubench] |	7669 |	17155 |	70 |
| 29 |	[cpubench] |	7456 |	17173 |	72 |
| 30 |	[cpubench] |	8012 |	17223 |	67 |
| 27 |	[cpubench] |	7780 |	17225 |	69 |
| 29 |	[cpubench] |	7669 |	17245 |	70 |
| 30 |	[cpubench] |	7894 |	17291 |	68 |
| 27 |	[cpubench] |	7780 |	17294 |	69 |
| 29 |	[cpubench] |	7780 |	17315 |	69 |
| 30 |	[cpubench] |	7894 |	17359 |	68 |
| 27 |	[cpubench] |	7669 |	17363 |	70 |
| 29 |	[cpubench] |	7669 |	17385 |	70 |
| 30 |	[cpubench] |	7669 |	17427 |	70 |
| 27 |	[cpubench] |	7669 |	17433 |	70 |
| 29 |	[cpubench] |	7780 |	17455 |	69 |
| 30 |	[cpubench] |	8012 |	17497 |	67 |
| 27 |	[cpubench] |	7780 |	17504 |	69 |
| 29 |	[cpubench] |	7780 |	17525 |	69 |
| 30 |	[cpubench] |	7894 |	17565 |	68 |
| 27 |	[cpubench] |	7561 |	17573 |	71 |
| 29 |	[cpubench] |	7780 |	17594 |	69 |
| 30 |	[cpubench] |	7780 |	17633 |	69 |
| 27 |	[cpubench] |	7894 |	17645 |	68 |
| 29 |	[cpubench] |	7561 |	17664 |	71 |
| 30 |	[cpubench] |	7669 |	17702 |	70 |
| 27 |	[cpubench] |	7669 |	17714 |	70 |
| 29 |	[cpubench] |	7669 |	17736 |	70 |
| 30 |	[cpubench] |	7780 |	17773 |	69 |
| 27 |	[cpubench] |	7669 |	17784 |	70 |
| 29 |	[cpubench] |	7456 |	17806 |	72 |
| 25 |	[iobench]	|0	| 164 |64	148 |7 |
| 25 |	[iobench]	|5	| 179 |51	182 |
| 25 |	[iobench]	|5	| 181 |34	181 |
| 25 |	[iobench]	|5	| 183 |15	179 |
| 25 |	[iobench]	|5	| 184 |95	174 |
| 25 |	[iobench]	|5	| 186 |69	179 |
| 25 |	[iobench]	|5	| 188 |48	173 |
| 25 |	[iobench]	|5	| 190 |22	172 |
| 25 |	[iobench]	|5	| 191 |94	174 |
| 25 |	[iobench]	|5	| 193 |68	174 |
| 25 |	[iobench]	|5	| 195 |43	177 |
| 25 |	[iobench]	|5	| 197 |20	187 |
| 25 |	[iobench]	|5	| 199 |07	176 |
| 25 |	[iobench]	|5	| 200 |83	179 |
| 25 |	[iobench]	|5	| 202 |63	184 |
| 25 |	[iobench]	|5	| 204 |47	184 |
| 25 |	[iobench]	|5	| 206 |31	181 |
| 25 |	[iobench]	|5	| 208 |12	184 |
| 25 |	[iobench]	|5	| 209 |96	186 |
| 25 |	[iobench]	|5	| 211 |82	193 |

` |cpubench 20 &; iobench 20 &; iobench 20 &; iobench 20 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
|34	| [cpubench] |	7456 |	24063 |	72 |
| 34 |	[cpubench]	 | 7669	|24136|	70|
| 39 |	[iobench]	| 5 | 	24064	|187|
| 36 |	[iobench]	| 5 | 	24064	|192|
| 34 |	[cpubench] 	 | 7456|	24206|	72|
| 34 |	[cpubench] 	 | 7669|	24278|	70|
| 39 |	[iobench]	| 7 | 	24251	|139|
| 36 |	[iobench]	| 6 | 	24256	|159|
| 34 |	[cpubench] 	 | 7561	|24348|	71|
| 34 |	[cpubench] 	 | 7669	|24419|	70|
| 38 |	[iobench]	| 2 | 	24064|	427|
| 39 |	[iobench]	| 7 | 	24390|	135|
| 34 |	[cpubench] 	 | 7561	|24490|	71|
| 36 |	[iobench]	| 6 | 	24416|	170|
| 34 |	[cpubench] 	 | 7456	|24561|	72|
| 39 |	[iobench]	| 7 | 	24526|	138|
| 38 |	[iobench]	| 5 | 	24492	|196|
| 34 |	[cpubench] 	 | 7456	|24633|	72|
| 36 |	[iobench]	| 6 | 	24586|	169|
| 34 |	[cpubench] 	 | 7669	|24706|	70|
| 39 |	[iobench]	| 6 | 	24665|	160|
| 34 |	[cpubench] 	 | 7669	|24776|	70|
| 36 |	[iobench]	| 7 | 	24755|	138|
| 34 |	[cpubench] 	 | 7669|	24846|	70|
| 39 |	[iobench]	| 6 | 	24825|	148|
| 38 |	[iobench]	| 3 | 	24688|	289|
| 34 |	[cpubench] 	 | 7561	|24916|	71|
| 34 |	[cpubench] 	 | 7669	|24988|	70|
| 36 |	[iobench]	| 5 | 	24893	194
| 34 |	[cpubench] 	 | 7669	|25058|	70|
| 39 |	[iobench]	| 5 | 	24973|	174|
| 34 |	[cpubench] | 7669	| 25129	|70|
| 36 |	[iobench]	| 7 | 	25087|	129|
| 38 |	[iobench]	| 4 | 	24977|	240|
| 34 |	[cpubench] 	 | 7456	|25199|	72|
| 39 |	[iobench]	| 6 | 	25147|	149|
| 34 |	[cpubench] 	 | 7669 |	25271 |	70 |
| 34 |	[cpubench] 	 | 7780 |	25342 |	69 |
| 36 |	[iobench]	| 5 | 	25216|	202 |
| 39 |	[iobench]	| 7 | 	25297|	135 |
| 34 |	[cpubench]  | 7669	| 25412 |	70|
| 38 |	[iobench]	| 3 | 	25217 |	285 |
| 36 |	[iobench]	| 8 | 	25418 |	123 |
| 39 |	[iobench]	| 8 | 	25432 |	117 |
| 38 |	[iobench]	| 12 |	25502 |	83 |
| 36 |	[iobench]	| 12 |	25542 |	85 |
| 39 |	[iobench]	| 12 |	25549 |	83 |
| 38 |	[iobench]	| 11 |	25585 |	87 |
| 36 |	[iobench]	| 11 |	25628 |	89 |
| 39 |	[iobench]	| 10 |	25632 |	94 |
| 38 |	[iobench]	| 11 |	25672 |	91 |
| 36 |	[iobench]	| 12 |	25718 |	84 |
| 39 |	[iobench]	| 12 |	25726 |	80 |
| 38 |	[iobench]	| 11 |	25763 |	86 |
| 36 |	[iobench]	| 11 |	25802 |	89 |
| 39 |	[iobench]	| 11 |	25806 |	90 |
| 38 |	[iobench]	| 11 |	25850 |	88 |
| 36 |	[iobench]	| 11 |	25891 |	90 |
| 39 |	[iobench]	| 12 |	25897 |	84 |
| 38 |	[iobench]	| 12 |	25938 |	80 |
| 36 |	[iobench]	| 13 |	25981 |	77 |
| 39 |	[iobench]	| 13 |	25982 |	77 |
| 38 |	[iobench]	| 12 |	26018 |	80 |
| 36 |	[iobench]	| 12 |	26058 |	85 |
| 39 |	[iobench]	| 12 |	26059 |	85 |
| 38 |	[iobench]	| 12 |	26098 |	84 |
| 36 |	[iobench]	| 13 |	26143 |	78 |
| 39 |	[iobench]	| 12 |	26144 |	80 |
| 38 |	[iobench]	| 12 |	26183 |	85 |
| 36 |	[iobench]	| 11 |	26222 |	87 |
| 39 |	[iobench]	| 11 |	26225 |	87 |
| 38 |	[iobench]	| 11 |	26269 |	90 |
| 36 |	[iobench]	| 12 |	26310 |	85 |
| 39 |	[iobench]	| 11| 	26312 |	89 |
| 38 |	[iobench]	| 8 | 	26359 |	126 |
| 36 |	[iobench]	| 7 | 	26396 |	144 |
| 38 |	[iobench]	| 6 | 	26485 |	157 |
| 38 |	[iobench]	| 5 | 	26642 |	176 |
| 38 |	[iobench]	| 5 | 	26818 |	174 |
| 38 |	[iobench]	| 5 | 	26993 |	173 |

1. Describa los parámetros de los programas cpubench e iobench para este experimento (o sea, los define al principio y el valor de N. Tener en cuenta que podrían cambiar en experimentos futuros, pero que si lo hacen los resultados ya no serán comparables).

Métrica de Cpubench = total_cpu_kops / elapsed_ticks. N = 20.  
Métrica de IObench = total_iops / elapsed_ticks. N = 20.  

2. ¿Los procesos se ejecutan en paralelo? ¿En promedio, qué proceso o procesos se ejecutan primero? Hacer una observación cualitativa.

Al ejecutar xv6 en un único procesador mediante el comando make CPU=1 qemu, no hay paralelismo en el sentido de que los procesos no pueden ejecutarse simultáneamente; un solo CPU puede manejar únicamente un proceso a la vez. Sin embargo, el planificador de tipo Round Robin (RR) permite que la ejecución de estos procesos se multiplexe en el tiempo. Esto significa que el CPU alterna entre los procesos, asignando a cada uno un quantum de tiempo igual. Como el CPU se distribuye entre varios procesos, da la impresión de que varios procesos están activos al mismo tiempo, es decir que son paralelos.

Los procesos se ejecutan por orden de llegada. Esto puede verse en el start tick de cada proceso en los tests. 
Al ejecutar cpubench e iobench en segundo plano, el iobench queda en espera -en estado sleeping- hasta que cpubench termina su ejecucion. Además, si hay otros procesos iobench en segundo plano, como el archivo de lectura y escritura es el mismo para cada llamada iobench N &, deben esperar a que el primero libere dicho archivo para ejecutarse. Esto se evidencia en los tests, ya que el primer proceso iobound planificado se queda esperando la disponibilidad del recurso IO, por lo que devuelve un número alto de interrupciones (representado por todos los demás procesos que fueron planificados después de que comenzó la ejecución del proceso IO de marras).

3. ¿Cambia el rendimiento de los procesos iobound con respecto a la cantidad y tipo de procesos que se estén ejecutando en paralelo? ¿Por qué?

Según los tests realizados, los procesos iobound aumentan directamente que toman solamente cuando están en simultáneo a otros procesos iobound, no son afectados por la concurrencia de procesos cpubound.

Sucede por una cuestión de qué recursos son los que utiliza más. Los procesos iobound compiten entre ellos por el disco (aunque en xv6, en este modo de prueba, el "disco" está en la ram), y los cpubound por el CPU.  

4. ¿Cambia el rendimiento de los procesos cpubound con respecto a la cantidad y tipo de procesos que se estén ejecutando en paralelo? ¿Por qué?

Al contrario que los procesos iobound, no se ven afectados practicamente por la concurrencia con otros procesos cpubound, no hay diferencias notables.

Sin embargo sí puede percibirse que hay un aumento del rendimiento en los procesos cpu-bound cuando están intercalados con procesos iobound. Probablemente porque logran un mejor equilibrio en el sistema en cuánto al tiempo de uso del CPU y los I/O.  

5. ¿Es adecuado comparar la cantidad de operaciones de cpu con la cantidad de operaciones iobound?

Probablemente haya una manera mejor. Con esta métrica se puede tener una visión general del planificador. 

## todo

Probar otras métricas como el turnaround_time, response_time y otras a pensar.
