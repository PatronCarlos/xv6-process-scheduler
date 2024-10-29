¿Fue necesario modificar las métricas para que los resultados fueran
comparables? ¿Por qué?
Sí, pues el quantum era demasiado pequeño como para que iobench mostrara la cantidad de operaciones por tick. En el quantum de 10000 mostraba 0 operaciones por tick, lo que nos llevó a multiplicar la métrica por 100 en iobench, de manera que mostrara la cantidad de operaciones IO cada 100 ticks y así tuviesemos más información para comparar. Lo mismo ocurrió en el quantum de 1000: tuvimos que multiplicar por 1000 para obtener la cantidad de operacione IO cada 1000 ticks. En cpubench también multiplicamos por 10 la métrica para obtener cantidad de operaciones cpu cada 10 ticks, pues el número se hizo muy pequeño.
En resumen:
-Quantum de 10000: multiplicamos métrica de iobench * 100, obteniendo operaciones IO cada 100 ticks.
-Quantum de 1000: métrica de iobench * 1000 (operaciones IO cada 1000 ticks), metrica de cpubench * 10 (kilo operaciones de CPU cada 10 ticks).
Como el iobench 20 demoraba mucho en el quantum de 1000, decidimos repetir los experimentos pero con un N menor. Así, optamos por usar un N=4. También modificamos las métricas iniciales: en vez de cantidad de operaciones por tick, ahora mediremos cantidad de operaciones cada 1000 ticks tanto en cpubench como iobench.

## Hardware 1: CPU Ryzen 5 3600, RAM 32GB 3200Mhz
### Quantum a 10000

`$ iobench 4 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 4 | [iobench] | 476 | 4491 | 2150 |
| 4 | [iobench] | 483 | 6646 | 2116 |
| 4 | [iobench] | 475 | 8768 | 2153 |
| 4 | [iobench] | 477 | 10927| 2145 |

`$ iobench 4 &; iobench 4 &; iobench 4 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 7 | [iobench] | 491 | 856 | 2085 |
| 5 | [iobench] | 474 | 852 | 2157 |
| 8 | [iobench] | 423 | 858 | 2419 |
| 7 | [iobench] | 433 | 2952 | 2360 |
| 5 | [iobench] | 438 | 3019 | 2334 |
| 8 | [iobench] | 408 | 3290 | 2506 |
| 7 | [iobench] | 474 | 5321 | 2158 |
| 5 | [iobench] | 459 | 5364 | 2229 |
| 8 | [iobench] | 454 | 5806 | 2251 |
| 7 | [iobench] | 439 | 7487 | 2331 |
| 5 | [iobench] | 435 | 7605 | 2354 |
| 8 | [iobench] | 482 | 8066 | 2121 |

`$ cpubench 4 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 14 | [cpubench] | 66439 | 51782 | 808 |
| 14 | [cpubench] | 64756 | 52596 | 829 |
| 14 | [cpubench] | 64678 | 53431 | 830 |
| 14 | [cpubench] | 67272 | 54267 | 798 |

`$ cpubench 4 &; cpubench 4 &; cpubench 4 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 21 | [cpubench] | 22340 | 61190 | 2403 |
| 21 | [cpubench] | 22146 | 63611 | 2424 |
| 20 | [cpubench] | 23421 | 65806 | 2292 |
| 18 | [cpubench] | 23269 | 65811 | 2307 |
| 21 | [cpubench] | 22312 | 66050 | 2406 |
| 20 | [cpubench] | 23545 | 68113 | 2280 |
| 18 | [cpubench] | 23483 | 68133 | 2286 |
| 21 | [cpubench] | 25358 | 68471 | 2117 |

`$ iobench 4 &; cpubench 4 &; cpubench 4 &; cpubench 4 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 29 | [cpubench] | 23300 | 98185 | 2304 |
| 30 | [cpubench] | 22219 | 98194 | 2416 |
| 29 | [cpubench] | 23249 | 100504 | 2309 |
| 30 | [cpubench] | 22128 | 100628 | 2426 |
| 27 | [cpubench] | 10632 | 98180 | 5049 |
| 29 | [cpubench] | 23421 | 102828 | 2292 |
| 30 | [cpubench] | 22340 | 103069 | 2403 |
| 29 | [cpubench] | 23401 | 105135 | 2294 |
| 30 | [cpubench] | 23774 | 105487 | 2258 |
| 27 | [cpubench] | 11567 | 103265 | 4641 |
| 25 | [iobench]  | 103   | 98187 | 9850 |
| 27 | [cpubench] | 28738 | 107917 | 1868 |
| 25 | [iobench]  | 298   | 108047 | 3425 |
| 27 | [cpubench] | 29080 | 109799 | 1846 |
| 25 | [iobench]  | 475   | 111482 | 2155 |
| 25 | [iobench]  | 483   | 113643 | 2120 |

`$ cpubench 4 &; iobench 4 &; iobench 4 &; iobench 4 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 34 | [cpubench] | 28135| 151187 | 1908 |
| 34 | [cpubench] | 26072| 153114 | 2059 |
| 36 | [iobench]  | 240  | 151188 | 4260 |
| 39 | [iobench]  | 187  | 151201 | 5456 |
| 34 | [cpubench] | 27236| 155190 | 1971 |
| 38 | [iobench]  | 170  | 151192 | 6002 |
| 36 | [iobench]  | 266  | 155458 | 3845 |
| 34 | [cpubench] | 25144| 157184 | 2135 |
| 39 | [iobench]  | 268 | 156681 | 3807 |
| 38 | [iobench]  | 298 | 157204 | 3429 |
| 36 | [iobench]  | 413 | 159314 | 2475 |
| 39 | [iobench]  | 420 | 160497 | 2436 |
| 38 | [iobench]  | 386 | 160644 | 2650 |
| 36 | [iobench]  | 432 | 161799 | 2367 |
| 38 | [iobench]  | 518 | 163306 | 1976 |
| 39 | [iobench]  | 437 | 162947 | 2340 |

### Quantum a 1000

`iobench 4 &` 

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 4 | [iobench] | 38 | 9180 | 26519 |
| 4 | [iobench] | 38 | 35807 | 26317 |
| 4 | [iobench] | 39 | 62209 | 25916 |
| 4 | [iobench] | 38 | 88230 | 26313 |

`iobench 4 &; iobench 4 &; iobench 4 &` 

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 10 | [iobench] | 36 | 149284 | 28293 |
| 8 | [iobench]  | 34 | 149295 | 29525 |
| 11 | [iobench] | 33 | 149444 | 30300 |
| 10 | [iobench] | 34 | 177791 | 29260 |
| 8 | [iobench]  | 33 | 179066 | 30211 |
| 11 | [iobench] | 33 | 179977 | 30136 |
| 10 | [iobench] | 36 | 207249 | 28392 |
| 8 | [iobench]  | 37 | 209497 | 27300 |
| 11 | [iobench] | 36 | 210270 | 28030 |
| 10 | [iobench] | 36 | 235843 | 27754 |
| 8 | [iobench]  | 35 | 237006 | 28506 |
| 11 | [iobench] | 37 | 238461 | 27483 |

`$ cpubench 4 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 14 | [cpubench] | 1267 | 429500 | 42370 |
| 14 | [cpubench] | 1363 | 471992 | 39382 |
| 14 | [cpubench] | 1208 | 511488 | 44409 |
| 14 | [cpubench] | 1217 | 555995 | 44102 |

`$ cpubench 4 &; cpubench 4 &; cpubench 4 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 18 | [cpubench] | 1462 | 694313 | 36698 |
| 20 | [cpubench] | 1452 | 694410 | 36967 |
| 18 | [cpubench] | 1408 | 731177 | 38118 |
| 20 | [cpubench] | 1412 | 731585 | 38016 |
| 18 | [cpubench] | 1441 | 769435 | 37241 |
| 20 | [cpubench] | 1433 | 769774 | 37444 |
| 21 | [cpubench] | 460 | 694601 | 116454 |
| 18 | [cpubench] | 1342 | 806820 | 39986 |
| 20 | [cpubench] | 1352 | 807413 | 39703 |
| 21 | [cpubench] | 812 | 811379 | 66047 |
| 21 | [cpubench] | 1222 | 877523 | 43922 |
| 21 | [cpubench] | 1382 | 921560 | 38826 |

`$ iobench 4 &; cpubench 4 &; cpubench 4 &; cpubench 4 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 25 | [iobench]  | 17 | 1054466 | 59919 |
| 27 | [cpubench] | 883 | 1054375 | 607|91  |
|29  | [cpubench] | 882 | 1054472 | 60856 |
| 25 | [iobench]  | 17 | 1114594 | 59787 |
| 27 | [cpubench] | 906 | 1115396 | 59230 |
| 29 | [cpubench] | 877 | 1115657 | 61152 |
| 30 | [cpubench] | 406 | 1054706 | 131970 |
| 25 | [iobench]  | 16 | 1174549 | 61133 |
| 27 | [cpubench] | 867 | 1174939 | 61908 |
| 29 | [cpubench] | 855 | 1177046 | 62785 |
| 25 | [iobench]  | 17 | 1235851 | 59935 |
| 27 | [cpubench] | 895 | 1237050 | 59976 |
| 29 | [cpubench] | 917 | 1240092 | 58493 |
| 30 | [cpubench] | 451 | 1187011 | 118901 |
| 30 | [cpubench] | 1261 | 1306027 | 42554 |
| 30 | [cpubench] | 1317 | 1348689 | 40745 |

`$ cpubench 4 &; iobench 4 &; iobench 4 &; iobench 4 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 36 | [iobench] | 26 | 1480471 | 38411 |
| 38 | [iobench] | 25 | 1480509 | 40186 |
| 39 | [iobench] | 22 | 1480763 | 44985 |
| 34 | [cpubench] | 791 | 1480525 | 67806 |
| 36 | [iobench] | 24 | 1519140 | 42404 |
| 38 | [iobench] | 24 | 1520970 | 41745 |
| 39 | [iobench] | 22 | 1525990 | 45871 |
| 36 | [iobench] | 26 | 1561779 | 38342 |
| 38 | [iobench] | 26 | 1562959 | 38889 |
| 39 | [iobench] | 24 | 1572122 | 42260 |
| 34 | [cpubench] | 751 | 1548508 | 71425 |
| 36 | [iobench] | 25 | 1600375 | 39414 |
| 38 | [iobench] | 26 | 1602052 | 38459 |
| 39 | [iobench] | 26 | 1614655 | 38613 |
| 34 | [cpubench] | 959 | 1620099 | 55962 |
| 34 | [cpubench] | 1331 | 1676169 | 40330 |

