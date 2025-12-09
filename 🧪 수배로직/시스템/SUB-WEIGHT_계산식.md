




Q. 최적의 해(개수)를 찾아라!
```
최적화된 SUBWEIGHT 를 구하는 문제야. 

자재1의 무게는 X kg, 두께는 A mm 

자재2의 무게는 Y kg, 두께는 B mm 

자재1의 수량은 C, 자재 2의 수량은 D라고 했을때 

최적화된 C와 D를 구하는 문제야.(단, C가 가장 많이 들어가는) 

단, 필요무게는 E 로 XC + YD 가 E 이상이 되어야하고(단 , XC+YD가 최대한 E에 가까운) 

필요두께는 F로 AC + BD 는 F 보다 작아야해 (단, AC+BD는 F에 최대한 가까운) 

X, A, Y, B , E, F는 상수로 주어질 거야 

예시로 X = 30, A = 33 Y = 40, B = 38 E = 3000kg F = 2800mm 구할 수 있는 JAVA 코딩해줘
```

```
INPUT : 

SUB_WT_1        1st 자재의 개당 무게
SUB_H_1          1st 자재의 개당 두께
SUB_P_1          1st 자재의 개당 가격
SUB_WT_2     2nd 자재의 개당 무게
SUB_H_2        2nd 자재의 개당 두께
SUB_P_2          2nd  자재의 개당 가격
SUB_NEED_WT   SUBWEIGHT 최소 필요 무게
SUB_MAX_LOAD_H      SUBWEIGHT 최대 적재가능 높이

OUTPUT:
SUB_BEST_Q1        1st 자재의 최적 갯수
SUB_BEST_Q2       2nd 자재의 최적 갯수
```

