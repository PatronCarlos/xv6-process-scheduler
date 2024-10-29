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

1. Describa los parámetros de los programas cpubench e iobench para este experimento (o sea, los define al principio y el valor de N. Tener en cuenta que podrían cambiar en experimentos futuros, pero que si lo hacen los resultados ya no serán comparables).
2. ¿Los procesos se ejecutan en paralelo? ¿En promedio, qué proceso o procesos se ejecutan primero? Hacer una observación cualitativa.
3. ¿Cambia el rendimiento de los procesos iobound con respecto a la cantidad y tipo de procesos que se estén ejecutando en paralelo? ¿Por qué?
4. ¿Cambia el rendimiento de los procesos cpubound con respecto a la cantidad y tipo de procesos que se estén ejecutando en paralelo? ¿Por qué?
5. ¿Es adecuado comparar la cantidad de operaciones de cpu con la cantidad de operaciones iobound?
