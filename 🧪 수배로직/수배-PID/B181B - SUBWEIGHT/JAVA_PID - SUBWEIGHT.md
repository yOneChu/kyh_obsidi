
---

## SUBWEIGHT 수량 산출 JAVA 코드
```java
 public VariantMap FUNCTION_CALSUBWEIGHT(Map elvEnt, List<Map> floorMasterList, Map partInfo)  
          throws Exception {

      VariantMap result = new VariantMap();

      // 입력값 읽기  
      String Xs  = StringUtil.NVL(elvEnt.get("SUB_WT_1"), "");  
      String As  = StringUtil.NVL(elvEnt.get("SUB_H_1"), "");  
      String P1s = StringUtil.NVL(elvEnt.get("SUB_P_1"), "");

      String Ys  = StringUtil.NVL(elvEnt.get("SUB_WT_2"), "");  
      String Bs  = StringUtil.NVL(elvEnt.get("SUB_H_2"), "");  
      String P2s = StringUtil.NVL(elvEnt.get("SUB_P_2"), "");

      String AA_s = StringUtil.NVL(elvEnt.get("SUB_NEED_WT"), "");  
      String BB_s = StringUtil.NVL(elvEnt.get("SUB_MAX_LOAD_H"), "");

      // ★ 숫자 체크 (정수만 허용)  
      if (!isInteger(Xs) || !isInteger(As) || !isInteger(P1s)  
              || !isInteger(Ys) || !isInteger(Bs) || !isInteger(P2s)  
              || !isInteger(AA_s) || !isInteger(BB_s)) {

          result.put("SUB_BEST_Q1", "9999");  
          result.put("SUB_BEST_Q2", "9999");  
          return result;  
      }

      // 파싱  
      int X  = Integer.parseInt(Xs);  
      int A  = Integer.parseInt(As);  
      int p1 = Integer.parseInt(P1s);

      int Y  = Integer.parseInt(Ys);  
      int B  = Integer.parseInt(Bs);  
      int p2 = Integer.parseInt(P2s);

      int AA = Integer.parseInt(AA_s);  // 최소 무게  
      int BB = Integer.parseInt(BB_s);  // 최대 두께

      int max = 200;

      boolean found = false;  
      int bestC = -1, bestD = -1;

      int bestCost = Integer.MAX_VALUE;  
      int bestDiff1 = Integer.MAX_VALUE;  // (sum1 - AA)  
      int bestDiff2 = Integer.MAX_VALUE;  // (BB - sum2)

      for (int C = 0; C <= max; C++) {  
          for (int D = 0; D <= max; D++) {

              int sum1 = X * C + Y * D;  
              int sum2 = A * C + B * D;

              if (sum1 < AA) continue;   // 무게 조건  
              if (sum2 > BB) continue;   // 두께 조건 (≤ BB 허용)

              int cost = p1 * C + p2 * D;

              int diff1 = sum1 - AA; // 최소 초과량  
              int diff2 = BB - sum2; // 최대한 근접하도록

              if (cost < bestCost  
                      || (cost == bestCost && (diff1 < bestDiff1  
                      || (diff1 == bestDiff1 && diff2 < bestDiff2)))) {

                  bestCost = cost;  
                  bestDiff1 = diff1;  
                  bestDiff2 = diff2;  
                  bestC = C;  
                  bestD = D;  
                  found = true;  
              }  
          }  
      }

      if (!found) {  
          result.put("SUB_BEST_Q1", "9999");  
          result.put("SUB_BEST_Q2", "9999");  
      } else {  
          result.put("SUB_BEST_Q1", String.valueOf(bestC));  
          result.put("SUB_BEST_Q2", String.valueOf(bestD));  
      }

    result.put("FUNCTION_CALSUBWEIGHT", "Y");  
      return result;  
  }

  /**  
     * 숫자인지 판별하는 메소드 (소수점 포함)  
     * 빈 값("")이나 문자열이 들어오면 false를 반환  
     */  
  private boolean isInteger(String str) {  
      if (str == null || str.trim().isEmpty()) return false;  
      return str.matches("-?\\d+"); // 음수/양수 정수만 허용  
  }
```

## JAVA 로직 설명

## 한 줄 요약

**두 종류의 서브 웨이트 블록(1번, 2번)을 각각 몇 개씩 쌓아야 하는지 계산하는 함수**입니다. "필요 무게 이상 + 최대 적재 높이 미만"을 만족하는 조합 중 **가장 싼 조합**을 찾고, 그 개수를 `SUB_BEST_Q1`, `SUB_BEST_Q2`로 반환합니다. 방식은 0~200개 범위의 모든 조합을 다 대입해 보는 전수 탐색입니다.

## 1. 입력값

`elvEnt`에서 8개 값을 읽습니다.

|변수|키|의미|
|---|---|---|
|X|SUB_WT_1|1번 블록 1개 무게|
|A|SUB_H_1|1번 블록 1개 높이(두께)|
|p1|SUB_P_1|1번 블록 1개 단가|
|Y|SUB_WT_2|2번 블록 1개 무게|
|B|SUB_H_2|2번 블록 1개 높이(두께)|
|p2|SUB_P_2|2번 블록 1개 단가|
|AA|SUB_NEED_WT|필요한 최소 총 무게|
|BB|SUB_MAX_LOAD_H|최대 적재 높이|

`floorMasterList`, `partInfo`는 인자로 받지만 **함수 안에서 사용하지 않습니다.**

## 2. 입력 검증

8개 중 하나라도 `isPositiveNumber()`가 false면 Q1, Q2 모두 `"ERROR"`를 넣고 즉시 반환합니다.

주석에는 "정수만 허용"이라고 되어 있지만, 실제로 정수만 통과시키는지, 0을 허용하는지는 `isPositiveNumber()` 구현에 달려 있습니다. 이 코드만으로는 확인할 수 없습니다. 파싱은 `Double.parseDouble`로 하기 때문에, 검증을 통과하면 소수도 계산은 됩니다.

## 3. 핵심 로직: 전수 탐색

C(1번 개수)와 D(2번 개수)를 각각 0~200까지 돌려서 **201 × 201 = 40,401개 조합**을 모두 검사합니다.

각 조합마다 아래를 계산합니다.

- 총 무게 `sum1 = X·C + Y·D`
- 총 높이 `sum2 = A·C + B·D`
- 총 비용 `cost = p1·C + p2·D`

**조건 필터**는 두 가지입니다.

- `sum1 < AA`이면 제외합니다. 즉 무게가 필요 무게 **이상**이어야 합니다.
- `sum2 >= BB`이면 제외합니다. 즉 높이가 최대 높이 **미만**이어야 합니다.

**최적 선정 기준**은 아래 순서로 적용됩니다.

1. 비용(cost)이 가장 작은 조합
2. 비용이 같으면, 초과 무게 `diff1 = sum1 − AA`가 가장 작은 조합
3. 그것도 같으면, 남는 높이 `diff2 = BB − sum2`가 가장 작은 조합. 즉 높이가 BB에 더 가까운(더 높게 쌓이는) 조합입니다.
4. 세 기준이 모두 같으면, 비교가 `<`(엄격 비교)이므로 **먼저 발견된 조합이 유지**됩니다. 루프 순서상 C가 작은 쪽, 그다음 D가 작은 쪽입니다.

## 4. 출력

- 조건을 만족하는 조합이 하나도 없으면 Q1, Q2 모두 `"ERROR"`를 반환합니다.
- 있으면 `SUB_BEST_Q1 = C`, `SUB_BEST_Q2 = D`를 문자열로 반환합니다.
- `FUNCTION_CALSUBWEIGHT = "Y"`는 탐색 결과가 ERROR여도 항상 들어갑니다. 단, 입력 검증에서 걸려 조기 반환된 경우에는 들어가지 않습니다.

## 5. 예시

X=25, A=40, p1=100 / Y=15, B=30, p2=70 / AA=100, BB=200인 경우입니다.

|C|D|무게|높이|비용|판정|
|---|---|---|---|---|---|
|4|0|100|160|400|통과 (최저가)|
|3|2|105|180|440|통과|
|1|5|100|190|450|통과|
|2|4|110|200|480|제외 (높이 = BB)|
|0|7|105|210|490|제외 (높이 초과)|

결과는 **Q1 = 4, Q2 = 0**입니다.

## 6. 짚어볼 점

1. **주석과 코드가 다릅니다.** `if (sum2 >= BB) continue; // 두께 조건 (≤ BB 허용)`에서 주석은 BB와 같아도 허용한다고 되어 있지만, 코드는 **같으면 제외**합니다. 위 예시의 (2, 4) 조합이 그 경우입니다. 어느 쪽이 설계 의도인지 확인이 필요합니다. 허용이 맞다면 `sum2 > BB`로 바꿔야 합니다.
2. **최대 개수가 200으로 고정되어 있습니다.** 한쪽 블록을 200개 넘게 써야만 답이 나오는 경우에는 ERROR가 됩니다. 실제 사양상 문제가 없는 범위인지 확인하면 좋습니다.
3. **double을 `==`로 비교합니다.** 입력이 정수면 문제없습니다. 소수 단가나 소수 무게가 들어오면 부동소수점 오차 때문에 동률 판정(2·3순위 기준)이 의도대로 동작하지 않을 수 있습니다.
4. **2순위와 3순위 기준의 방향이 다릅니다.** 2순위는 무게를 필요량에 최대한 가깝게(적게) 맞추는 반면, 3순위는 높이를 최대한 높게 쌓는 쪽을 고릅니다. 의도된 것이라면 괜찮지만, 업무 규칙과 맞는지 한 번 확인해 볼 만합니다.

성능은 4만 회 단순 연산이라 실무상 문제없는 수준입니다.