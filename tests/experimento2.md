1. ¿Fue necesario modificar las métricas para que los resultados fueran
comparables? ¿Por qué?
Sí, pues el quantum era demasiado pequeño como para que iobench mostrara la cantidad de operaciones por tick. En el quantum de 10000 mostraba 0 operaciones por tick, lo que nos llevó a multiplicar la métrica por 100 en iobench, de manera que mostrara la cantidad de operaciones IO cada 100 ticks y así tuviesemos más información para comparar. Lo mismo ocurrió en el quantum de 1000: tuvimos que multiplicar por 1000 para obtener la cantidad de operacione IO cada 1000 ticks. En cpubench también multiplicamos por 10 la métrica para obtener cantidad de operaciones cpu cada 10 ticks, pues el número se hizo muy pequeño.
En resumen:
-Quantum de 10000: multiplicamos métrica de iobench * 100, obteniendo operaciones IO cada 100 ticks.
-Quantum de 1000: métrica de iobench * 1000 (operaciones IO cada 1000 ticks), metrica de cpubench * 10 (kilo operaciones de CPU cada 10 ticks).
Como el iobench 20 demoraba mucho en el quantum de 1000, decidimos repetir los experimentos pero con un N menor. Así, optamos por usar un N=4. También modificamos las métricas iniciales: en vez de cantidad de operaciones por tick, ahora mediremos cantidad de operaciones cada 1000 ticks tanto en cpubench como iobench.

## Hardware 1: CPU Ryzen 5 3600, RAM 32GB 3200Mhz
### Quantum a 10000
me$ iobench 20 &
$|4	| [iobench]	|46	 |  4903  |    2190  |
| 4 | [iobench]	49	7096	   2064  
| 4 | [| obench]	4 | 	9 |163	2  |    2061  
  4 | [iobench] | 50 |	1122  |     	203 | |8
| 4 | [iobench] | 49 |	1326  |     	20 |7 |9
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
| 
    | || |   &; iobench 20 &; iobench 20 &
| PID | Tipo de Procesos | Métrica | Start Tick | Elapsed Ticks |
|-----|------------------|---------|------------|----------------|
$| 8 |	[iobench] | 44	|80086	  | 22 84|
|10| 	[iobench] |	41	|80090	  | 24 52|
|11| 	[iobench] |	41	|80090	  | 24 86|
 8	   | [iobenc	   |44	|82381	  | 23 07|
|10| 	[iobench] |	41	|82554	  | 24 78|
|11| 	[iobench] |	36	|82582	  | 27 68|
|10| 	[iobench] |	48	|85040	  | 20 91|
 8	   | [iobenc	   |39	|84695	  | 25 93|
|11| 	[iobench] |	45	|85356	  | 22 30|
|10| 	[iobench] |	44	|87136	  | 23 06|
 8	   | [iobenc]	  |44	|87297	  | 22 99|
|11| 	[iobench] |	44	|87592	  | 23 02|
 8	   | [iobenc	   |4 2	|898  |   	|23 52|
|11| |[iobench] |	  4|	899  |   2052 	 |
|10| 	[iobench] |	48	|9184 7 |	2092 |
 8	   | [iobenc]	  |45	|9184 9 |	2255 |
|11| 	[iobench] |	43	|9216 8 |	2356 |
|10| 	[iobench] |	45	|9394 6 |	2243 |
 8	   | [iobenc]	  |42	|9410 9 |	2393 |
|11| 	[iobench] |	41	|9453 3 |	2448 |
|10| 	[iobench] |	44	|9619 9 |	2300 |
 8	   | [iobenc]	  |42	|9650 7 |	2435 |
|11| 	[iobench] |	45	|9698 8 |	2248 |
|10| 	[iobench] |	44	|9850 7 |	2321 |
|1   	[iobench] | 45  |98942  |   2247 |
|8 |1| 	[iench] |h]	4|99242  |49	255||4|7
|10| 	[iobench] |	44	|100835 |	2283 |
|11| 	[iobench] |	51	|101498 |	2006 |
 8	   | [iobenc 	  |45	|101496 |	2251 |
|10| 	[iobench] |	45	|103124 |	2251 |
 8	   | [iobenc]	  |49	|103754 |	2060 |
|11| 	[iobench] |	39	|103525 |	2616 |
|10| 	[iobench] |	44	|105385 |	2312 |
 8	   | [iobenc]	  |39	|105822 |	2610 |
|11| 	[iobench] |	42	|106148 |	2424 |
|10| 	[iobench] |	44	|107706 |	2317 |
 8	   | [iobenc]	  |43	|108438 |	2362 |
|11| 	[iobench] |	40	|108577 |	2536 |
|10| 	[iobench] |	41	|110032 |	2476 |
 8	   | [iobenc]	  |40	|110809 |	2560 |
|11| 	[iobench] |	40	|111120 |	2499 |
|10| 	[iobench] |	43	|112515 |	2358 |
 8	   | [iobenc]	  |44	|113375 |	2326 |
|11| 	[iobench] |	42	|113626 |	2390 |
|10| 	[iobench] |	39	|114880 |	2594 |
 8	   | [iobenc]	  |39	|115707 |	2611 |
|11| 	[iobench] |	38	|116022 |	2654 |
|10| 	[iobench] |	39	|117482 |	2577 |
 8	   | [iobenc]	  |40	|118324 |	2551 |
|11| 	[iobench] |	37	|118684 |	2749 |
|10| 	[iobench] |	43	|120067 |	2363 |
 8	   | [iobenc]	  |42	|120882 |	2405 |
|11| 	[iobench] |	43	|121439 |	2332 |
|10| 	[iobench] |	43	|122437 |	2331 |
 8    | [iobenc]	  |44	|123293 |	2293 |
|11| 	[iobench] |	39	|123778 |	2566 |
|10| 	[iobench] |	49	|124775 |	2076 |
 8	   | [iobenc]	  |42	|125592 |	2389 |
|11| 	[iobench] |	47	|126351 |	2171 |


$ cpubench 20 &
$ |14 |	[cpubench] |	409	|   200096 |	1311|
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

$ cpubench 20 &; cpubench 20 &; cpubench 20 &
$ |20|	[cpubench] |	238	|   249160 |    2250|
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
|21 |	[cpubench] |	606 |	294680 |	885
|
$ iobench 20 &; cpubench 20 &; cpubench 20 &; cpubench 20 &
$ |29|	[cpubench]|	230	|337775|	23 25|
|30 |	[cpubench]| 221	|337788 |	24 24|
|29 |	[cpubench]| 230	|340109 |	23 32|
|30 |	[cpubench]| 225	|340224 |	23 83|
|27 |	[cpubench]| 108	|337774 |	49 48|
|30 |	[cpubench]| 217	|342619 |	24 70|
|29 |	[cpubench]| 201	|342450 |	26 65|
|29 |	[cpubench]| 231	|345127 |	23 19|
|30 |	[cpubench]| 223	|345101 |	24 03|
|27 |	[cpubench]| 107	|342749 |	49 78|
|25 |	[iobench] | 9	|337777 |	10311|
|29 |	[cpubench]| 216	|347458 |	24 85|
|30 |	[cpubench]| 211	|347516 |	25 33|
|29 |	[cpubench]| 235	|349955 |	22 84|
|30 |	[cpubench]| 227	|350061 |	23 59|
|27 |	[cpubench]| 113	|347743 |	47 29|
|29 |	[cpubench]| 230	|352251 |	23 32|
|30 |	[cpubench]| 206	|352432 |	25 99|
|29 |	[cpubench]| 214	|354592 |	25 00|
|27 |	[cpubench]| 107	|352496 |	50 05|
|30 |	[cpubench]| 200	|355043 |	26 71|
|25 |	[iobench] | 10	|348100 |	99 42|
|29 |	[cpubench]| 221	|357104 |	24 26|
|30 |	[cpubench]| 221	|357733 |	24 28|
|29 |	[cpubench]| 223	|359542 |	23 98|
|27 |	[cpubench]| 115	|357516 |	46 30|
|30 |	[cpubench]| 204	|360173 |	26 19|
|29 |	[cpubench]| 233	|361952 |	23 04|
|30 |	[cpubench]| 226	|362804 |	23 71|
|29 |	[cpubench]| 233	|364268 |	23 04|
|27 |	[cpubench]| 105	|362170 |	50 94|
|30 |	[cpubench]| 220	|365187 |	24 38|
|25 |	[iobench] | 10	|358054 |	99 22|
|29 |	[cpubench]| 217	|366584 |	24 67|
|30 |	[cpubench]| 216	|367640 |	24 74|
|29 |	[cpubench]| 232	|369063 |	23 13|
|27 |	[cpubench]| 114	|367288 |	47 02|
|30 |	[cpubench]| 225	|370126 |	23 82|
|29 |	[cpubench]| 231	|371388 |	23 22|
|30 |	[cpubench]| 227	|372520 |	23 64|
|29 |	[cpubench]| 234	|373722 |	22 86|
|27 |	[cpubench]| 106	|372014 |	50 46|
|30 |	[cpubench]| 215	|374897 |	24 89|
|25 |	[iobench] | 10	|367988 |	99 26|
|29 |	[cpubench]| 219	|376020 |	24 46|
|30 |	[cpubench]| 215	|377398 |	24 93|
|29 |	[cpubench]| 233	|378478 |	22 97|
|27 |	[cpubench]| 116	|377084 |	46 24|
|30 |	[cpubench]| 226	|379903 |	23 73|
|29 |	[cpubench]| 234	|380787 |	22 92|
|30 |	[cpubench]| 223	|382288 |	24 05|
|29 |	[cpubench]| 226	|383091 |	23 75|
|27 |	[cpubench]| 117	|381732 |	45 87|
|30 |	[cpubench]| 291	|384705 |	18 41|
|25 |	[iobench] | 11	|377926 |	89 68|
|27 |	[cpubench]| 285	|386335 |	18 82|
|27 |	[cpubench]| 284	|388228 |	18 89|
|25 |	[iobench] | 30	|386902 |	34 00|
|27 |	[cpubench]| 295	|390124 |	18 15|
|25 |	[iobench] | 30	|390310 |	33 82|
|27 |	[cpubench]| 292	|391949 |	18 33|
|27 |	[cpubench]| 288	|393791 |	18 58|
|25 |	[iobench] | 30	|393698 |	34 04|
|27 |	[cpubench]| 293	|395659 |	18 29|
|27 |	[cpubench]| 290	|397498 |	18 49|
|25 |	[iobench] | 30	|397108 |	33 88|
|27 |	[cpubench]| 299	|399359 |	17 92|
|27 |	[cpubench]| 288	|401163 |	18 60|
|25 |	[iobench] | 30	|400504 |	33 90|
|27 |	[cpubench]| 299	|403034 |	17 91|
|25 |	[iobench] | 42	|403902 |	24 31|
|25 |	[iobench] | 50	|406337 |	20 08|
|25 |	[iobench] | 50	|408348 |	20 20|
|25 |	[iobench] | 50	|410372 |	20 36|
|25 |	[iobench] | 49	|412411 |	20 75|
|25 |	[iobench] | 49	|414490 |	20 56|
|25 |	[iobench] | 50	|416552 |	20 26|
|25 |	[iobench] | 49	|418581 |	20 67|
|25 |	[iobench] | 48	|420653 |	21 00|
|25 |	[iobench] | 49	|422757 |	20 88|

$ cpubench 20 &; iobench 20 &; iobench 20 &; iobench 20 &
$ 34	[cpubench]	286	455295	1871
34	[cpubench]	280	457176	1916
36	[iobench]	24	455309	4165
34	[cpubench]	248	459101	2163
39	[iobench]	17	455311	5983
38	[iobench]	16	455310	6119
36	[iobench]	29	459485	3488
34	[cpubench]	262	461270	2045
34	[cpubench]	266	463324	2017
38	[iobench]	23	461444	4283
36	[iobench]	26	462979	3839
34	[cpubench]	268	465347	1998
39	[iobench]	16	461303	6359
34	[cpubench]	287	467353	1867
36	[iobench]	27	466824	3692
34	[cpubench]	258	469230	2073
38	[iobench]	16	465733	6144
34	[cpubench]	276	471312	1945
39	[iobench]	16	467672	6361
36	[iobench]	26	470527	3849
34	[cpubench]	266	473266	2015
38	[iobench]	19	471887	5307
34	[cpubench]	272	475292	1968
36	[iobench]	30	474384	3411
34	[cpubench]	258	477269	2078
39	[iobench]	17	474047	5762
34	[cpubench]	295	479353	1815
36	[iobench]	26	477801	3795
38	[iobench]	20	477209	4942
34	[cpubench]	263	481183	2040
34	[cpubench]	260	483234	2061
36	[iobench]	26	481608	3840
39	[iobench]	17	479823	5966
34	[cpubench]	286	485307	1877
38	[iobench]	18	482166	5668
36	[iobench]	28	485456	3592
34	[cpubench]	247	487193	2170
34	[cpubench]	280	489372	1915
39	[iobench]	17	485799	5775
36	[io38	[iobbench]	3ench]	244	489054	487849	4	3011
217
34	[cpubench]	255	491293	2104
36	[iobench]	32	492074	3130
34	[cpubench]	246	493405	2179
38	[iobench]	28	492076	3635
39	[iobench]	20	491585	5033
36	[iobench]	42	495210	2398
38	[iobench]	43	495720	2336
39	[iobench]	38	496623	2682
36	[iobe38	[iobench]nch]	48	4	4098065	21	4906
7616	2554
39	[iobench]	44	499312	2304
36	[iobench]	52	500182	1965
38	[iobench]	42	500177	2418
39	[iobench]	38	501622	2640
36	[iobench]	43	502153	2336
38	[iobench]	40	502601	2554
36	[iobench]	46	504499	2202
39	[iobench]	40	504274	2502
38	[iobench]	43	505162	2374
39	[iobench]	48	506786	2133
36	[iobench]	43	506707	2378
38	[iobench]	45	507543	2233
36	[iobench]	45	509091	2230
39	[iobench]	39	508936	2568
38	[iobench]	43	509785	2344
39	[iobench]	47	511512	2151
36	[iobench]	42	511327	2412
38	[iobench]	46	512139	2193
39	[iobench]	44	513674	2276
36	[iobench]	41	513747	2470
38	[iobench]	44	514339	2283
39	[iobench]	48	515958	2106
38	[iobench]	55	516629	1846
39	[iobench]	53	518070	1908
38	[iobench]	51	518483	2007
39	[iobench]	48	519986	2107
38	[iobench]	50	520500	2039
39	[iobench]	46	522099	2220
39	[iobench]	49	524323	2070


### Quantum a 1000

$ iobench 20 &
4	[iobench]	480	5700	2133
4	[iobench]	501	7837	2043
4	[iobench]	501	9884	2041
4	[iobench]	496	11928	2062
4	[iobench]	499	13994	2050
4	[iobench]	500	16047	2048
4	[iobench]	508	18099	2012
4	[iobench]	512	20114	1997
4	[iobench]	503	22114	2034
4	[iobench]	496	24151	2062
4	[iobench]	503	26216	2034
4	[iobench]	489	28254	2090
4	[iobench]	496	30347	2064
4	[iobench]	504	32415	2030
4	[iobench]	501	34449	2043
4	[iobench]	504	36495	2029
4	[iobench]	506	38528	2020
4	[iobench]	506	40551	2021
4	[iobench]	497	42575	2060
4	[iobench]	502	44640	2037

$ iobench 20 &; iobench 20 &; iobench 20 &
$ 10	[iobench]	496	81240	2063
11	[iobench]	471	81243	2174
8	[iobench]	389	81239	2627
11	[iobench]	452	83426	2265
10	[iobench]	406	83318	2521
8	[iobench]	416	83873	2458
10	[iobench]	455	85845	2247
11	[iobench]	414	85698	2470
8	[iobench]	448	86337	2285
10	[iobench]	466	88105	2195
11	[iobench]	411	88176	2489
8	[iobench]	428	88630	2388
10	[iobench]	395	90306	2589
11	[iobench]	410	90671	2494
8	[iobench]	434	91024	2356
10	[iobench]	474	92905	2160
11	[iobench]	448	93175	2282
8	[iobench]	427	93385	2397
10	[iobench]	440	95073	2326
11	[iobench]	458	95464	2233
8	[iobench]	418	95787	2446
10	[iobench]	411	97409	2488
11	[iobench]	399	97704	2561
8	[iobench]	377	98238	2716
10	[iobench]	412	99906	2483
11	[iobench]	353	100271	2895
8	[iobench]	387	100960	2643
10	[iobench]	408	102397	2505
11	[iobench]	415	103173	2465
8	[iobench]	358	103609	2855
10	[iobench]	430	104910	2380
11	[iobench]	430	105644	2381
8	[iobench]	425	106470	2407
10	[iobench]	458	107297	2234
11	[iobench]	445	108032	2297
8	[iobench]	412	108883	2482
10	[iobench]	408	109538	2505
11	[iobench]	415	110336	2463
8	[iobench]	386	111371	2646
10	[iobench]	389	112050	2626
11	[iobench]	398	112805	2572
8	[iobench]	414	114023	2471
10	[iobench]	408	114683	2504
11	[iobench]	398	115384	2568
8	[iobench]	434	116500	2357
10	[iobench]	460	117194	2222
11	[iobench]	401	117959	2553
8	[iobench]	423	118872	2418
10	[iobench]	421	119424	2427
11	[iobench]	408	120518	2505
8	[iobench]	399	121296	2562
10	[iobench]	414	121858	2468
11	[iobench]	395	123032	2587
8	[iobench]	409	123866	2502
10	[iobench]	397	124334	2577
11	[iobench]	414	125626	2469
8	[iobench]	373	126374	2743
10	[iobench]	433	126917	2362
11	[iobench]	414	128102	2469
8	[iobench]	510	129132	2007

$ cpubench 20 &
$ 14	[cpubench]	6660	156526	806
14	[cpubench]	6308	157336	851
14	[cpubench]	6086	158191	882
14	[cpubench]	6135	159077	875
14	[cpubench]	4858	159957	1105
14	[cpubench]	3751	161066	1431
14	[cpubench]	5054	162501	1062
14	[cpubench]	6170	163567	870
14	[cpubench]	6778	164441	792
14	[cpubench]	6769	165237	793
14	[cpubench]	5809	166034	924
14	[cpubench]	5803	166962	925
14	[cpubench]	5931	167891	905
14	[cpubench]	6360	168799	844
14	[cpubench]	6677	169647	804
14	[cpubench]	5879	170455	913
14	[cpubench]	6735	171372	797
14	[cpubench]	6761	172173	794
14	[cpubench]	5803	172971	925
14	[cpubench]	6562	173899	818

$ cpubench 20 &; cpubench 20 &; cpubench 20 &
$ 26	[cpubench]	2378	310160	2257
28	[cpubench]	2373	310168	2262
29	[cpubench]	2270	310175	2364
26	[cpubench]	2373	312429	2262
28	[cpubench]	2366	312439	2268
29	[cpubench]	2256	312551	2379
26	[cpubench]	2360	314703	2274
28	[cpubench]	2360	314719	2274
29	[cpubench]	2248	314939	2388
26	[cpubench]	2360	316989	2274
28	[cpubench]	2360	317002	2274
29	[cpubench]	2155	317336	2490
26	[cpubench]	2348	3219275	2288	[cpube6
nch]	2351	319285	2283
29	[cpubench]	2176	319838	2466
26	[cpubench]	23628	[cpu0	321573	bench]	232274
63	321580	2271
29	[cpubench]	2262	322313	2373
26	[cpubench]	237028	[cpub	323856	2ench]	236265
6	323860	2268
29	[cpubench]	2256	324698	2379
26	[cpubench28	[]	2366	32cpubench]6133	2268	2366	326
137	2268
29	[cpubench]	2259	327089	2376
26	[cpub28	[ench]	236cpubench]3	328413		2366	3282271
417	2268
29	[cpubench]	2250	329477	2385
26	28	[[cpubenccpubench]h]	2360	3	2363	3330693	2270697	2274
1
29	[cpubench]	2242	331871	2394
28	[cpubench]	2363	332980	2271
26	[cpubench]	2339	332979	2295
29	[cpubench]	2236	334277	2400
28	[cpubench]	2345	335263	2289
26	[cpubench]	2345	335286	2289
29	[cpubench]	2245	336686	2391
28	[cpubench]	2348	337564	2286
26	[cpubench]	2351	337587	2283
29	[cpubench]	2217	339092	2421
28	[cpubench]	2339	339859	2295
26	[cpubench]	2339	339879	2295
29	[cpubench]	2239	341522	2397
28	[cpubench]	2348	342166	2286
26	[cpubench]	2348	342186	2286
29	[cpubench]	2228	343934	2409
28	[cpubench]	2342	344464	2292
26	[cpubench]	2339	344484	2295
29	[cpubench]	2222	346355	2415
28	[cpubench]	2339	346768	2295
26	[cpubench]	2339	346788	2295
29	[cpubench]	2231	348779	2406
28	[cpubench]	2342	349075	2292
26	[cpubench]	2336	349095	2298
29	[cpubench]	2236	351197	2400
28	[cpubench]	2348	351379	2286
26	[cpubench]	2336	351405	2298
28	[cpubench]	2351	353677	2283
26	[cpubench]	2359	353715	2275
29	[cpubench]	2248	353609	2388
29	[cpubench]	6693	356003	802

cpubench 20 &; iobench 20 &; iobench 20 &; iobench 20 &
$ 42	[cpubench]	2379	572174	2256
46	[iobench]	211	569864	4841
51	[cpubench]	2037	573993	2635
47	[iobench]	139	569344	7332
42	[cpubench]	2200	574442	2440
51	[cpubench]	2500	576634	2147
42	[cpubench]	2152	576890	2494
51	[cpubench]	2241	578789	2395
44	[iobench]	123	573156	8287
42	[cpubench]	2146	579400	2501
51	[cpubench]	2447	581190	2193
42	[cpubench]	2365	581907	2269
51	[cpubench]	2223	583397	2414
46	[iobench]	91	574723	11219
53	[iobench]	83	574003	12245
42	[cpubench]	2173	584190	2470
55	[iobench]	76	574021	13459
56	[iobench]	71	574486	14300
51	[cpubench]	1757	585822	3055
42	[cpubench]	2383	586668	2252
51	[cpubench]	2388	588895	2248
42	[cpubench]	2255	588926	2380
44	[iobench]	94	581454	10851
51	[cpubench]	2137	591151	2511
42	[cpubench]	2126	591314	2525
47	[iobench]	55	576692	18403
51	[cpub42	[cpenchubench]	]	212267	59311	5847	23689366825
42
53	[iobench]	93	586261	10996
46	[iobench]	85	585960	11907
55	[iobench]	90	587456	[iobe95	113nch]	10171
	588805	10076
42	[cpubench]	1874	596225	2864
51	[cpubench]	1838	596227	2920
51	[cpubench]	2782	599156	1929
44	[iobench]	114	592319	8930
51	[cpubench]	2580	60109353	[iobe	2080
nch]	173	597267	5915
47	[iobench]	104	595113	9783
51	[cpubench]	2513	603184	2136
46	[iobench]	131	597879	7777
55	[iobench]	136	598887	7501
51	[cpubench]	2680	605326	2003
56	[iobench]	118	598892	8655
53	[iobench]	190	603190	5370
51	[cpubench]	2566	607335	2092
44	[iobench]	103	601259	9896
51	[cpubench]	2730	609441	1966
55	[iobench]	173	606401	5886
51	[cpubench]	2547	611422	2107
47	[iobench]	113	604905	8992
46	[iobench]	118	605671	8632
53	[iobench]	150	608579	6794
51	[cpubench]	2473	613546	2170
56	[iobench]	124	607559	8238
51	[cpubench]	2699	615724	1989
55	[iobench]	159	612298	6407
44	[iobench]	127	611172	8016
53	[iobench]	227	615388	4498
51	[cpubench]	2281	617723	2353
46	[iobench]	154	614321	6614
47	[iobench]	142	613914	7188
56	[iobench]	162	615808	6298
44	[iobench]	244	619203	4192
55	[iobench]	214	618722	4764
53	[iobench]	248	619893	4120
47	[iobench]	275	621112	3713
46	[iobench]	239	620943	4267
56	[iobench]	255	622116	4005
44	[iobench]	235	623407	4342
53	[iobench]	231	624019	4431
55	[iobench]	198	623495	5169
47	[iobench]	223	624835	4588
46	[iobench]	225	625217	4535
56	[iobench]	237	626129	4303
44	[iobench]	228	627761	4488
53	[iobench]	239	628462	4279
47	[iobench]	293	629433	3492
55	[iobench]	225	628673	4546
46	[iobench]	234	629762	4365
56	[iobench]	228	630439	4489
44	[iobench]	262	632255	3907
53	[iobench]	233	632752	4387
47	[iobench]	241	632935	4242
55	[iobench]	215	633227	4761
46	[iobench]	240	634140	4255
56	[iobench]	227	634934	4510
44	[iobench]	263	636171	3879
47	[iobench]	266	637183	3840
53	[iobench]	238	637153	4298
55	[iobench]	242	637994	4221
46	[iobench]	234	638402	4373
56	[iobench]	238	639450	4298
44	[iobench]	276	640056	3707
47	[iobench]	263	641032	3882
53	[iobench]	254	641462	4028
46	[iobench]	260	642785	3924
55	[iobench]	226	642225	4527
44	[iobench]	254	643773	4024
56	[iobench]	252	643756	4060
47	[iobench]	243	644921	4201
53	[iobench]	256	645498	3997
55	[iobench]	264	646760	3872
46	[iobench]	259	646721	3945
56	[iobench]	259	647823	3952
44	[iobench]	240	647805	4266
47	[iobench]	256	649132	3990
53	[iobench]	239	649505	4280
46	[iobench]	271	650675	3771
55	[iobench]	230	650640	4440
44	[iobench]	249	652080	4111
56	[iobench]	232	651795	4410
47	[iobench]	239	653136	4283
53	[iobench]	221	653796	4633
46	[iobench]	243	654453	4213
55	[iobench]	229	655086	4461
44	[iobench]	242	656200	4226
56	[iobench]	228	656214	4491
47	[iobench]	259	657425	3939
46	[iobench]	279	658676	3664
53	[iobench]	238	658440	4290
55	[iobench]	271	659554	3770
56	[iobench]	260	660714	3930
47	[iobench]	290	661372	3529
46	[iobench]	308	662348	3323
55	[iobench]	345	663337	2961
53	[iobench]	281	662741	3644
56	[iobench]	340	664654	3008
47	[iobench]	324	664909	3155
53	[iobench]	386	666394	2651
55	[iobench]	358	666306	2855
56	[iobench]	377	667669	2713
53	[iobench]	464	669062	2206
55	[iobench]	403	669168	2535
56	[iobench]	399	670388	2563
53	[i55	[ioobenchbench]	]	3434	6717166	2	2358
671274	2795
56	[iobench]	436	672958	2346
55	[iobench]	513	674076	1994
56	[iobench]	492	675309	2078





## Hardware 2:CPU Ryzen 3 3200G 3.6Ghz, RAM 32GB 3200Mhz
### Quantum a 10000

iobench 20 &
|4 |	[iobench] |	34 |	4271 |  2997|
|4 |	[iobench] |	35 |	7273 |  2916|
|4 |	[iobench] |	35 |	10193| 	2898|
|4 |	[iobench] |	34 |	13096| 	2952|
|4 |	[iobench] |	34 |	16054| 	2960|
|4 |	[iobench] |	34 |	19018| 	2973|
|4 |	[iobench] |	34 |	21996| 	2975|
|4 |	[iobench] |	34 |	24976| 	2980|
|4 |	[iobench] |	34 |	27961| 	2988|
|4 |	[iobench] |	35 |	30953| 	2917|
|4 |	[iobench] |	34 |	33875| 	3010|
|4 |	[iobench] |	34 |	36889| 	2986|
|4 |	[iobench] |	34 |	39880| 	2956|
|4 |	[iobench] |	34 |	42841| 	2948|
|4 |	[iobench] |	34 |	45795| 	3005|
|4 |	[iobench] |	34 |	48804| 	2988|
|4 |	[iobench] |	34 |	51797| 	2993|
|4 |	[iobench] |	34 |	54795| 	2982|
|4 |	[iobench] |	34 |	57781| 	2951|
|4 |	[iobench] |	34 |	60738| 	2988|

iobench 20 &; iobench 20 &; iobench 20 &
|10	|[iobench]|	28|	322112|	3547|
|9	|[iobench]|	27|	322108|	3668|
|7	|[iobench]|	26|	322107|	3898|
|7	|[iobench]|	35|	326015|	2866|
|10	|[iobench]|	30|	325673|	3380|
|9	|[iobench]|	30|	325784|	3360|
|9	|[iobench]|	38|	329154|	2693|
|10 |[iobench]|	30|	328895|	3330|	
|7  |[iobench]| 32| 329064| 3160|
|9	|[iobench]|	32|	331861|	3141|
|7	|[iobench]|	31|	332240|	3222|
|10	|[iobench]|	30|	332241|	3327|
|9	|[iobench]|	32|	335018|	3134|
|7	|[iobench]|	35|	335472|	2887|
|10	|[iobench]|	32|	335583|	3186|
|9	|[iobench]|	33|	338160|	3081|
|7	|[iobench]|	32|	338367|	3191|
|10	|[iobench]|	30|	338776|	3397|
|9	|[iobench]|	31|	341250|	3253|
|7	|[iobench]|	29|	341567|	3418|
|10	|[iobench]|	30|	342185|	3365|
|9	|[iobench]|	30|	344517|	3337|
|7	|[iobench]|	30|	344998|	3305|
|10	|[iobench]|	34|	345560|	2972|
|9	|[iobench]|	30|	347866|	3376|
|7	|[iobench]|	32|	348313|	3138|
|10	|[iobench]|	29|	348544|	3423|
|9	|[iobench]|	31|	351252|	3297|
|7	|[iobench]|	31|	351460|	3248|
|10	|[iobench]|	27|	351981|	3768|
|9	|[iobench]|	31|	354563|	3261|
|7	|[iobench]|	28|	354719|	3595|
|10	|[iobench]|	29|	355756|	3447|
|9	|[iobench]|	32|	357836|	3177|
|7	|[iobench]|	33|	358324|	3098|
|10	|[iobench]|	31|	359216|	3237|
|9	|[iobench]|	34|	361022|	3010|
|7	|[iobench]|	34|	361434|	2952|
|10	|[iobench]|	32|	362464|	3156|
|9	|[iobench]|	32|	364042|	3125|
|7	|[iobench]|	32|	364400|	3132|
|10	|[iobench]|	30|	365632|	3383|
|9	|[iobench]|	36|	367177|	2789|
|7	|[iobench]|	36|	367543|	2798|
|10	|[iobench]|	31|	369031|	3282|
|9	|[iobench]|	32|	369974|	3198|
|7	|[iobench]|	30|	370354|	3329|
|10	|[iobench]|	30|	372323|	3368|
|9	|[iobench]|	30|	373187|	3367|
|7	|[iobench]|	30|	373693|	3333|
|10	|[iobench]|	30|	375705|	3392|
|9	|[iobench]|	32|	376563|	3158|
|7	|[iobench]|	29|	377030|	3491|
|10	|[iobench]|	27|	379107|	3686|
|9	|[iobench]|	27|	379734|	3701|
|7	|[iobench]|	30|	380534|	3365|
|10	|[iobench]|	32|	382804|	3118|
|9	|[iobench]|	32|	383445|	3131|
|7	|[iobench]|	35|	383907|	2925|
|10	|[iobench]|	32|	385933|	3124|

cpubench 20 &
$ |12	|[cpubench]	|319|	413631|	1682|
|12|	[cpubench]|	320|	415320|	1676|
|12|	[cpubench]|	313|	417001|	1715|
|12|	[cpubench]|	313|	418722|	1711|
|12|	[cpubench]|	321|	420439|	1668|
|12|	[cpubench]|	316|	422107|	1694|
|12|	[cpubench]|	325|	423806|	1651|
|12|	[cpubench]|	324|	425462|	1653|
|12|	[cpubench]|	315|	427120|	1701|
|12|	[cpubench]|	315|	428827|	1704|
|12|	[cpubench]|	324|	430536|	1655|
|12|	[cpubench]|	317|	432197|	1692|
|12|	[cpubench]|	319|	433894|	1678|
|12|	[cpubench]|	321|	435578|	1671|
|12|	[cpubench]|	325|	437255|	1651|
|12|	[cpubench]|	317|	438911|	1690|
|12|	[cpubench]|	316|	440606|	1697|
|12|	[cpubench]|	317|	442308|	1692|
|12|	[cpubench]|	315|	444005|	1704|
|12|	[cpubench]|	317|	445714|	1689|

cpubench 20 &; cpubench 20 &; cpubench 20 &
$ 
$ 27	[cpubench29	[c]	110	1pubenc249898h]	110	4	12847
49908	4841
|30|	[cpubench]|	104	|1249915|	5129|
|27|	[cpubench]|	112	|1254763|	4779|
|29|	[cpubench]|	111	|1254770|	4800|
|30|	[cpubench]|	106	|1255056|	5064|
|29|	[27	[cpubenccpubenh]	110ch]	110	12	159259582	560	4874855
9
|30|	[cpubench]|	104|	1260135|	5142|
|29|	[cpubench]|	112|	1264452|	4764|
|27|	[cpubench]|	112|	1264454|	4791|
|30|	[cpubench]|	106|	1265295|	5050|
|27|	[cpubench]|	113|	1269260|	4710|
|29|	[cpubench]|	112|	1269231|	4777|
|30|	[cpubench]|	107|	1270360|	5014|
|29|	[cpubench]|	113|	1274023|	4726|
|27|	[cpubench]|	112|	1273983|	4789|
|30|	[cpubench]|	105|	1275392|	5098|
|27|	[cpubench]|	115|	1278787|	4660|
|29|	[cpubench]|	113|	1278761|	4711|
|30|	[cpubench]|	106|	1280493|	5023|
|27|	[cpubench]|	111|	1283462|	4800|
|29|	[cpubench]|	111|	1283487|	4794|
|30|	[cpubench]|	105|	1285531|	5091|
|27|	[cpubench]|	111|	1288277|	4797|
|29|	[cpubench]|	111|	1288296|	4797|
|30|	[cpubench]|	104|	1290637|	5156|
|29|	[cpubench]|	113|	1293108|	4728|
|27|	[cpubench]|	112|	1293089|	4776|
|30|	[cpubench]|	105|	1295808|	5098|
|29|	[cpubench]|	111|	1297848|	4824|
|27|	[cpubench]|	111|	1297880|	4812|
|30|	[cpubench]|	105|	1300921|	5103|
27	[cp29	ubench[cpube]	111	nch]	130270110	1307	4821
2681	4848
|30|	[cpubench]|	104|	1306045|	5121|
|27|	[cpubench]|	111|	1307543|	4796|
|29|	[cpubench]|	111|	1307544|	4826|
|30|	[cpubench]|	104|	1311184|	5142|
|27|	[cpubench]|	111|	1312354|	4828|
|29|	[cpubench]|	110|	1312385|	4852|
|30|	[cpubench]|	105|	1316347|	5102|
|27|	[cpubench]|	112|	1317197|	4766|
|29|	[cpubench]|	112|	1317252|	4787|
|30|	[cpubench]|	105|	1321467|	5106|
|27|	[cpubench]|	112|	1321981|	4761|
|29|	[cpubench]|	111|	1322060|	4797|
|27|	[cpubench]|	112|	1326757|	4773|
|29|	[cpubench]|	112|	1326872|	4773|
|30|	[cpubench]|	105|	1326591|	5076|
|27|	[cpubench]|	112|	1331548|	4755|
|29|	[cpubench]|	111|	1331660|	4812|
|30|	[cpubench]|	105|	1331682|	5112|
|27|	[cpubench]|	113|	1336318|	4728|
|29|	[cpubench]|	113|	1336487|	4741|
|30|	[cpubench]|	106|	1336812|	5026|
|27|	[cpubench]|	112|	1341061|	4781|
|29|	[cpubench]|	113|	1341243|	4733|
|30|	[cpubench]|	120|	1341844|	4449|
|30|	[cpubench]|	319|	1346300|	1682|

iobench 20 &; cpubench 20 &; cpubench 20 &; cpubench 20 &
$ 38	[cpubench]	106	1394934	5048
39	[cpubench]	104	1394959	5158
38	[cpubench]	108	1399994	4952
39	[cpubench]	104	1400136	5125
34	[iobench]	9	1394948	11174
36	[cpubench]	42	1394937	12534
38	[cpubench]	104	1404965	5134
39	[cpubench]	101	1405274	5277
38	[cpubench]	106	1410115	5060
39	[cpubench]	102	1410570	5218
34	[iobench]	9	1406138	10997
36	[cpubench]	43	1407517	12365
38	[cpubench]	104	1415191	5117
39	[cpubench]	100	1415803	5337
38	[cpubench]	104	1420323	5132
39	[cpubench]	101	1421162	5286
34	[iobench]	9	1417159	10869
38	[cpubench]	103	1425471	5189
39	[cpubench]	101	1426467	5297
36	[cpubench]	42	1419920	12649
38	[cpubench]	106	1430672	5025
39	[cpubench]	103	1431783	5162
34	[iobench]	9	1428044	10842
38	[cpubench]	104	1435712	5129
39	[cpubench]	101	1436961	5306
36	[cpubench]	43	1432617	12466
38	[cpubench]	108	1440856	4947
39	[cpubench]	102	1442282	5240
34	[iobench]	9	1438906	10904
38	[cpubench]	104	1445812	5136
39	[cpubench]	101	1447540	5306
38	[cpubench]	107	1450963	4976
39	[cpubench]	103	1452861	5209
36	[cpubench]	41	1445115	13070
34	[iobench]	9	1449830	10785
38	[cpubench]	104	1455954	5160
39	[cpubench]	99	1458088	5389
38	[cpubench]	106	1461129	5060
39	[cpubench]	102	1463492	5220
36	[cpubench]	44	1458218	12167
38	[cpubench]	102	1466204	5229
34	[iobench]	9	1460631	11059
39	[cpubench]	101	1468730	5301
38	[cpubench]	107	1471452	5011
39	[cpubench]	102	1474046	5213
38	[cpubench]	104	1476478	5129
36	[cpubench]	44	1470431	11934
34	[iobench]	9	1471710	10935
39	[cpubench]	100	1479278	5329
38	[cpubench]	103	1481622	5174
39	[cpubench]	102	1484625	5234
38	[cpubench]	106	1486811	5034
34	[iobench]	9	1482665	11003
36	[cpubench]	43	1482388	12305
39	[cpubench]	100	1489878	5326
38	[cpubench]	105	1491860	5098
39	[cpubench]	131	1495223	4085
34	[iobench]	13	1493696	7357
36	[cpubench]	75	1494742	7081
34	[iobench]	23	1501063	4428
36	[cpubench]	130	1501843	4112
34	[iobench]	23	1505501	4370
36	[cpubench]	129	1505964	4158
34	[iobench]	23	1509881	4391
36	[cpubench]	126	1510142	4232
34	[iobench]	23	1514282	4335
36	[cpubench]	125	1514396	4273
34	[iobench]	23	1518629	4333
36	[cpubench]	123	1518681	4362
36	[cpubench]	127	1523057	4217
34	[iobench]	23	1522970	4364
36	[cpubench]	128	1527285	4166
34	[iobench]	23	1527344	4361
36	[cpubench]	126	1531462	4244
34	[iobench]	23	1531715	4409
36	[cpubench]	129	1535715	4149
34	[iobench]	22	1536134	4509
36	[cpubench]	130	1539877	4121
34	[iobench]	22	1540653	4510
36	[cpubench]	233	1544013	2304


cpubench 20 &; iobench 20 &; iobench 20 &; iobench 20 &
42	[cpubench]	112	2018734	4760
44	[iobench]	19	2018749	5377
46	[iobench]	17	2018754	5834
47	[iobench]	15	2018757	6497
42	[cpubench]	112	2023513	4759
44	[iobench]	19	2024141	5380
46	[iobench]	17	2024650	5854
47	[iobench]	15	2025281	6683
42	[cpubench]	113	2028285	4712
44	[iobench]	24	2029530	4119
46	[iobench]	18	2030518	5472
42	[cpubench]	111	2033014	4818
47	[iobench]	16	2031993	4461	[iobench]12	22	20
33658	4469
46	[iobench]	19	2036001	5240
44	[iobench]	22	2038137	4510
42	[cpubench]	105	2037853	5065
47	[iobench]	17	2038134	5746
46	[iobench]	19	2041254	5187
42	[cpubench]	114	2042934	4689
44	[iobench]	19	2042658	5239
47	[iobench]	17	2043898	5961
46	[iobench]	19	2046455	5297
42	[cpubench]	115	2047642	4636
44	[iobench]	20	2047908	4993
46	[io47	[benchioben]	ch]	15	204987722		649209
51775	4600
42	[cpubench]	102	2052287	5251
44	[iobench]	21	2052911	4752
46	[iobench]	18	2056402	5541
47	[iobench]	18	2056394	5621
44	[iobench]	21	2057677	4759
42	[cpubench]	106	2057572	5033
46	[iobench]	18	2061962	5395
44	[iobench]	19	2062448	5162
42	[cpubench]	106	2062620	5036
47	[iobench]	16	2062041	6098
42	[cpubench]	116	2067667	4611
46	[iobench]	19	2067375	5295
44	[iobench]	18	2067631	5625
47	[iobench]	15	2068161	6401
42	[cpubench]	116	2072302	4611
44	[iobench]	22	2073265	4533
46	[iobench]	18	2072678	5588
47	[iobench]	18	2074583	5633
42	[cpubench]	113	2076930	4729
44	[iobench]	20	2077810	5017
46	[iobench]	18	2078285	5613
47	[iobench]	18	2080252	5684
42	[cpubench]	120	2081674	4473
44	[iobench]	20	2082839	5086
46	[iobench]	19	2083908	5307
42	[cpubench]	113	2086168	4723
47	[iobench]	16	2085956	6028
44	[iobench]	21	2087943	4697
46	[iobench]	20	2089228	4999
42	[cpubench]	113	2090909	4732
44	[iobench]	24	2092654	4191
47	[iobench]	17	2092019	5841
46	[iobench]	21	2094239	4866
42	[cpubench]	110	2095653	4851
44	[iobench]	20	2096855	4976
47	[iobench]	18	2097879	5459
46	[iobench]	21	2099120	4766
42	[cpubench]	110	2100518	4874
44	[iobench]	21	2101843	4837
46	[iobench]	21	2103920	4826
47	[iobench]	16	2103346	6178
42	[cpubench]	117	2105409	4567
44	[iobench]	24	2106692	4255
46	[iobench]	22	2108777	4570
47	[iobench]	19	2109546	5219
42	[cpubench]	110	2109991	4840
44	[iobench]	24	2110955	4133
46	[iobench]	28	2113355	3649
47	[iobench]	34	2114806	2947
46	[iobench]	36	2117021	2841
47	[iobench]	38	2117766	2637
47	[iobench]	34	2120407	2931
47	[iobench]	34	2123343	2988


### Quantum a 1000