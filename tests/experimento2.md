¿Fue necesario modificar las métricas para que los resultados fueran
comparables? ¿Por qué?
Sí, pues el quantum era demasiado pequeño como para que iobench mostrara la cantidad de operaciones por tick. En el quantum de 10000 mostraba 0 operaciones por tick, lo que nos llevó a multiplicar la métrica por 100 en iobench, de manera que mostrara la cantidad de operaciones IO cada 100 ticks y así tuviesemos más información para comparar. Lo mismo ocurrió en el quantum de 1000: tuvimos que multiplicar por 1000 para obtener la cantidad de operacione IO cada 1000 ticks. En cpubench también multiplicamos por 10 la métrica para obtener cantidad de operaciones cpu cada 10 ticks, pues el número se hizo muy pequeño.
En resumen:
-Quantum de 10000: multiplicamos métrica de iobench * 100, obteniendo operaciones IO cada 100 ticks.
-Quantum de 1000: métrica de iobench * 1000 (operaciones IO cada 1000 ticks), metrica de cpubench * 10 (kilo operaciones de CPU cada 10 ticks).
Como el iobench 20 demoraba mucho en el quantum de 1000, decidimos repetir los experimentos pero con un N menor. Así, optamos por usar un N=4. También modificamos las métricas iniciales: en vez de cantidad de operaciones por tick, ahora mediremos cantidad de operaciones cada 1000 ticks tanto en cpubench como iobench.

## Hardware 1: CPU Ryzen 5 3600, RAM 32GB 3200Mhz
### Quantum a 10000
`$ iobench 20 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 4	| [iobench]	|46	 |  4903  |    2190  |
| 4 | [iobench]	|49  |	7096  |	   2064  |
| 4 | [iobench]	|49  |  1632  |    2061  |
| 4 | [iobench] | 50 |	1122  |     203 | 
| 4 | [iobench] | 49 |	1326  |    	2079 |
| 4 | [iobench] | 50 |	15350 | 	2015 |
| 4 | [iobench] | 49 |	17368 | 	2062 |
| 4 | [iobench] | 49 |	19433 | 	2086 |
| 4 | [iobench] | 49 |	21522 | 	2052 |
| 4 | [iobench] | 50 |	23577 | 	2041 |
| 4 | [iobench] | 49 |	25621 | 	2051 |
| 4 | [iobench] | 50 |	27676 | 	2020 |
| 4 | [iobench] | 49 |	29699 | 	2073 |
| 4 | [iobench] | 48 |	31776 | 	2111 |
| 4 | [iobench] | 50 |	33891 | 	2042 |
| 4 | [iobench] | 50 |	35936 | 	2039 |
| 4 | [iobench] | 49 |	37979 | 	2071 |
| 4 | [iobench] | 50 |	40054 | 	2038 |
| 4 | [iobench] | 49 |	42095 | 	2087 |
| 4 | [iobench] | 49 |	44185 | 	2050 |

`&; iobench 20 &; iobench 20 &`  

| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
| 8 |	[iobench] | 44	|80086	|   2284 |
|10| 	[iobench] |	41	|80090	|   2452 |
|11| 	[iobench] |	41	|80090	|   2486 |
|8 |    [iobench] | 44	|82381	|   2307 |
|10| 	[iobench] |	41	|82554	|   2478 |
|11| 	[iobench] |	36	|82582	|   2768 |
|10| 	[iobench] |	48	|85040	|   2091 |
|8 |    [iobench] | 39	|84695	|   2593 |
|11| 	[iobench] |	45	|85356	|   2230 |
|10| 	[iobench] |	44	|87136	|   2306 |
|8 |    [iobench] | 44	|87297	|   2299 |
|11| 	[iobench] |	44	|87592	|   2302 |
|8 |    [iobench] | 42  |89894  |   2396 |
|11| 	[iobench] |	45  |89905  |   2052 |
|10| 	[iobench] |	48	|91847  |	2092 |
|8 |    [iobench] | 45	|91849  |	2255 |
|11| 	[iobench] |	43	|92168  |	2356 |
|10| 	[iobench] |	45	|93946  |	2243 |
|8 |    [iobench] | 42	|94109  |	2393 |
|11| 	[iobench] |	41	|94533  |	2448 |
|10| 	[iobench] |	44	|96199  |	2300 |
|8 |    [iobench] | 42	|96507  |	2435 |
|11| 	[iobench] |	45	|96988  |	2248 |
|10| 	[iobench] |	44	|98507  |	2321 |
|11|   	[iobench] | 45  |98942  |   2247 |
|8 | 	[iobench] |	40	|99242  |	2541 |		
|10| 	[iobench] |	44	|100835 |	2283 |
|11| 	[iobench] |	51	|101498 |	2006 |
|8 |    [iobench] | 45	|101496 |	2251 |
|10| 	[iobench] |	45	|103124 |	2251 |
|8 |    [iobench] | 49	|103754 |	2060 |
|11| 	[iobench] |	39	|103525 |	2616 |
|10| 	[iobench] |	44	|105385 |	2312 |
|8 |    [iobench] | 39	|105822 |	2610 |
|11| 	[iobench] |	42	|106148 |	2424 |
|10| 	[iobench] |	44	|107706 |	2317 |
|8 |    [iobench] | 43	|108438 |	2362 |
|11| 	[iobench] |	40	|108577 |	2536 |
|10| 	[iobench] |	41	|110032 |	2476 |
|8 |    [iobench] | 40	|110809 |	2560 |
|11| 	[iobench] |	40	|111120 |	2499 |
|10| 	[iobench] |	43	|112515 |	2358 |
|8 |    [iobench] | 44	|113375 |	2326 |
|11| 	[iobench] |	42	|113626 |	2390 |
|10| 	[iobench] |	39	|114880 |	2594 |
|8 |    [iobench] | 39	|115707 |	2611 |
|11| 	[iobench] |	38	|116022 |	2654 |
|10| 	[iobench] |	39	|117482 |	2577 |
|8 |    [iobench] | 40	|118324 |	2551 |
|11| 	[iobench] |	37	|118684 |	2749 |
|10| 	[iobench] |	43	|120067 |	2363 |
|8 |    [iobench] | 42	|120882 |	2405 |
|11| 	[iobench] |	43	|121439 |	2332 |
|10| 	[iobench] |	43	|122437 |	2331 |
|8 |    [iobench] | 44	|123293 |	2293 |
|11| 	[iobench] |	39	|123778 |	2566 |
|10| 	[iobench] |	49	|124775 |	2076 |
|8 |    [iobench] | 42	|125592 |	2389 |
|11| 	[iobench] |	47	|126351 |	2171 |


`$ cpubench 20 &`  

|14  |	[cpubench] |	409	|   200096 |	1311|
| 14 |	[cpubench] |	647 |	201412 |	829 |
| 14 |	[cpubench] |	589 |	202245 |	911 |
| 14 |	[cpubench] |	620 |	203160 |	865 |
| 14 |	[cpubench] |	675 |	204028 |	795 |
| 14 |	[cpubench] |	676 |	204827 |	793 |
| 14 |	[cpubench] |	584 |	205624 |	918 |
| 14 |	[cpubench] |	565 |	206546 |	950 |
| 14 |	[cpubench] |	384 |	207500 |	1398|
| 14 |	[cpubench] |	488 |	208902 |	1099|
| 14 |	[cpubench] |	607 |	210005 |	884 |
| 14 |	[cpubench] |	640 |	210892 |	838 |
| 14 |	[cpubench] |	589 |	211733 |	910 |
| 14 |	[cpubench] |	680 |	212647 |	789 |
| 14 |	[cpubench] |	682 |	213439 |	786 |
| 14 |	[cpubench] |	560 |	214228 |	958 |
| 14 |	[cpubench] |	675 |	215190 |	795 |
| 14 |	[cpubench] |	580 |	215988 |	925 |
| 14 |	[cpubench] |	677 |	216916 |	792 |
| 14 |	[cpubench] |	681 |	217712 |	788 |

`$ cpubench 20 &; cpubench 20 &; cpubench 20 &`  

|20|	[cpubench] |	238	|   249160 |    2250|
|18 |	[cpubench] |	236 |	249156 |	2271|
|21 |	[cpubench] |	226 |	249173 |	2367|
|20 |	[cpubench] |	237 |	251422 |	2256|
|18 |	[cpubench] |	237 |	251436 |	2256|
|21 |	[cpubench] |	227 |	251552 |	2364|
|20 |	[cpubench] |	238 |	253687 |	2250|
|18 |	[cpubench] |	238 |	253704 |	2250|
|21 |	[cpubench] |	227 |	253928 |	2361|
|20 |	[cpubench] |	238 |	255946 |	2250|
|18 |	[cpubench] |	232 |	255963 |	2313|
|21 |	[cpubench] |	227 |	256304 |	2355|
|20 |	[cpubench] |	238 |	258208 |	2250|
|18 |	[cpubench] |	237 |	258285 |	2256|
|21 |	[cpubench] |	227 |	258671 |	2361|
|20 |	[cpubench] |	237 |	260470 |	2256|
|18 |	[cpubench] |	237 |	260550 |	2262|
|21 |	[cpubench] |	225 |	261041 |	2379|
|20 |	[cpubench] |	238 |	262735 |	2247|
|18 |	[cpubench] |	238 |	262821 |	2250|
|21 |	[cpubench] |	225 |	263432 |	2382|
|20 |	[cpubench] |	235 |	264994 |	2277|
|18 |	[cpubench] |	237 |	265080 |	2259|
|21 |	[cpubench] |	223 |	265826 |	2400|
|20 |	[cpubench] |	236 |	267283 |	2268|
|18 |	[cpubench] |	234 |	267348 |	2292|
|21 |	[cpubench] |	225 |	268238 |	2385|
|20 |	[cpubench] |	236 |	269563 |	2271|
|18 |	[cpubench] |	236 |	269652 |	2274|
|21 |	[cpubench] |	226 |	270635 |	2373|
|20 |	[cpubench] |	236 |	271843 |	2271|
|18 |	[cpubench] |	236 |	271935 |	2271|
|21 |	[cpubench] |	220 |	273020 |	2436|
|20 |	[cpubench] |	234 |	274123 |	2286|
|18 |	[cpubench] |	234 |	274215 |	2289|
|21 |	[cpubench] |	224 |	275465 |	2391|
|20 |	[cpubench] |	237 |	276421 |	2262|
|18 |	[cpubench] |	236 |	276516 |	2274|
|21 |	[cpubench] |	226 |	277865 |	2373|
|20 |	[cpubench] |	237 |	278692 |	2262|
|18 |	[cpubench] |	235 |	278799 |	2277|
|21 |	[cpubench] |	226 |	280247 |	2373|
|20 |	[cpubench] |	236 |	280966 |	2271|
|18 |	[cpubench] |	230 |	281088 |	2334|
|21 |	[cpubench] |	226 |	282632 |	2373|
|20 |	[cpubench] |	237 |	283246 |	2256|
|18 |	[cpubench] |	237 |	283431 |	2259|
|21 |	[cpubench] |	222 |	285017 |	2412|
|20 |	[cpubench] |	233 |	285514 |	2295|
|18 |	[cpubench] |	234 |	285702 |	2289|
|21 |	[cpubench] |	223 |	287441 |	2403|
|20 |	[cpubench] |	234 |	287818 |	2289|
|18 |	[cpubench] |	235 |	288003 |	2283|
|21 |	[cpubench] |	220 |	289856 |	2430|
|20 |	[cpubench] |	229 |	290116 |	2337|
|18 |	[cpubench] |	230 |	290298 |	2334|
|21 |	[cpubench] |	226 |	292298 |	2370|
|20 |	[cpubench] |	237 |	292465 |	2256|
|18 |	[cpubench] |	243 |	292644 |	2201|
|21 |	[cpubench] |	606 |	294680 |	885 |


`$ iobench 20 &; cpubench 20 &; cpubench 20 &; cpubench 20 &`  

|29 |	[cpubench]|	230	|337775 |	2325 |
|30 |	[cpubench]| 221	|337788 |	2424 |
|29 |	[cpubench]| 230	|340109 |	2332 |
|30 |	[cpubench]| 225	|340224 |	2383 |
|27 |	[cpubench]| 108	|337774 |	4948 |
|30 |	[cpubench]| 217	|342619 |	2470 |
|29 |	[cpubench]| 201	|342450 |	2665 |
|29 |	[cpubench]| 231	|345127 |	2319 |
|30 |	[cpubench]| 223	|345101 |	2403 |
|27 |	[cpubench]| 107	|342749 |	4978 |
|25 |	[iobench] | 9	|337777 |	10311|
|29 |	[cpubench]| 216	|347458 |	2485 |
|30 |	[cpubench]| 211	|347516 |	2533 |
|29 |	[cpubench]| 235	|349955 |	2284 |
|30 |	[cpubench]| 227	|350061 |	2359 |
|27 |	[cpubench]| 113	|347743 |	4729 |
|29 |	[cpubench]| 230	|352251 |	2332 |
|30 |	[cpubench]| 206	|352432 |	2599 |
|29 |	[cpubench]| 214	|354592 |	2500 |
|27 |	[cpubench]| 107	|352496 |	5005 |
|30 |	[cpubench]| 200	|355043 |	2671 |
|25 |	[iobench] | 10	|348100 |	9942 |
|29 |	[cpubench]| 221	|357104 |	2426 |
|30 |	[cpubench]| 221	|357733 |	2428 |
|29 |	[cpubench]| 223	|359542 |	2398 |
|27 |	[cpubench]| 115	|357516 |	4630 |
|30 |	[cpubench]| 204	|360173 |	2619 |
|29 |	[cpubench]| 233	|361952 |	2304 |
|30 |	[cpubench]| 226	|362804 |	2371 |
|29 |	[cpubench]| 233	|364268 |	2304 |
|27 |	[cpubench]| 105	|362170 |	5094 |
|30 |	[cpubench]| 220	|365187 |	2438 |
|25 |	[iobench] | 10	|358054 |	9922 |
|29 |	[cpubench]| 217	|366584 |	2467 |
|30 |	[cpubench]| 216	|367640 |	2474 |
|29 |	[cpubench]| 232	|369063 |	2313 |
|27 |	[cpubench]| 114	|367288 |	4702 |
|30 |	[cpubench]| 225	|370126 |	2382 |
|29 |	[cpubench]| 231	|371388 |	2322 |
|30 |	[cpubench]| 227	|372520 |	2364 |
|29 |	[cpubench]| 234	|373722 |	2286 |
|27 |	[cpubench]| 106	|372014 |	5046 |
|30 |	[cpubench]| 215	|374897 |	2489 |
|25 |	[iobench] | 10	|367988 |	9926 |
|29 |	[cpubench]| 219	|376020 |	2446 |
|30 |	[cpubench]| 215	|377398 |	2493 |
|29 |	[cpubench]| 233	|378478 |	2297 |
|27 |	[cpubench]| 116	|377084 |	4624 |
|30 |	[cpubench]| 226	|379903 |	2373 |
|29 |	[cpubench]| 234	|380787 |	2292 |
|30 |	[cpubench]| 223	|382288 |	2405 |
|29 |	[cpubench]| 226	|383091 |	2375 |
|27 |	[cpubench]| 117	|381732 |	4587 |
|30 |	[cpubench]| 291	|384705 |	1841 |
|25 |	[iobench] | 11	|377926 |	8968 |
|27 |	[cpubench]| 285	|386335 |	1882 |
|27 |	[cpubench]| 284	|388228 |	1889 |
|25 |	[iobench] | 30	|386902 |	3400 |
|27 |	[cpubench]| 295	|390124 |	1815 |
|25 |	[iobench] | 30	|390310 |	3382 |
|27 |	[cpubench]| 292	|391949 |	1833 |
|27 |	[cpubench]| 288	|393791 |	1858 |
|25 |	[iobench] | 30	|393698 |	3404 |
|27 |	[cpubench]| 293	|395659 |	1829 |
|27 |	[cpubench]| 290	|397498 |	1849 |
|25 |	[iobench] | 30	|397108 |	3388 |
|27 |	[cpubench]| 299	|399359 |	1792 |
|27 |	[cpubench]| 288	|401163 |	1860 |
|25 |	[iobench] | 30	|400504 |	3390 |
|27 |	[cpubench]| 299	|403034 |	1791 |
|25 |	[iobench] | 42	|403902 |	2431 |
|25 |	[iobench] | 50	|406337 |	2008 |
|25 |	[iobench] | 50	|408348 |	2020 |
|25 |	[iobench] | 50	|410372 |	2036 |
|25 |	[iobench] | 49	|412411 |	2075 |
|25 |	[iobench] | 49	|414490 |	2056 |
|25 |	[iobench] | 50	|416552 |	2026 |
|25 |	[iobench] | 49	|418581 |	2067 |
|25 |	[iobench] | 48	|420653 |	2100 |
|25 |	[iobench] | 49	|422757 |	2088 |
