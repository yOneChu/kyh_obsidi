---
작성일: 2025.09.30
수정일:
---
---




### | 버퍼
버퍼의 NEX?모델은 영업에서 이미 어떤 버퍼를 사용하라고 정해서 PLM으로 넘어온다.
그러다보니 PLAN-C랑 꼬여서 그렇다.
이런거들은 최초 기계설계 일자를 3/13이전으로해서 처리해라.

---
### | 도면
![[Pasted image 20260518142341.png]]
![[Pasted image 20260518142357.png]]

### | 참고사항
중저속은 CAR와 CWT 버퍼블럭킹의 높이가 다르다.
하지만 고속일 경우에는 높이가 동일하다.

![[Pasted image 20250930164432.png]]

### | 버퍼

| EA  | 2   | 3   | 4   | 6   | 8   |
| --- | --- | --- | --- | --- | --- |
| CWT | 1   | 1   | 2   | 2   | 2/2 |
| CAR | 1   | 2   | 2   | 4   | 4   |

### | 길이 계산(기본)
```


EL_ECBUFBH = [$ round( {EL_EHP}-{P_TH}-{S_TH}-{B_TH}-{CAR_RB}-{B183AA_BH}-{FOOTING_TH}+{HS_TH}, -1) $]

0) EL_EHP = 승강로 PIT
1) P_TH = PLATFORM_TH + EL_BFTH(◎ 바닥 두께)
2) S_TH = SAFETY_TH
3) B_TH = BOLSTER_TH
4) HS_TH = HS_SPARE
5) B183AA_BH
6) FOOTING_TH
   

```



