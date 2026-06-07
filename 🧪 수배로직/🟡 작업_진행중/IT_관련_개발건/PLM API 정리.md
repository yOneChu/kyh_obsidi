---
작성일: 2026-06-05
---
---



## 01. PID 시뮬레이터 연계

| EL_PB186A01 - ROPE SHACKE  > 성공
```

https://plmpro.hdel.co.kr/plmetc/vault/pidExecute?PID=EL_PB186A01&hogi=208223L01&testVersion=&isfloor=N&floor=


```

결과
```JSON
{
  "ADD_CLIP_CWT": "N",
  "BABBIT_YN": "N",
  "EL_PB186A01_1": "18600182G0500",
  "SHACKLE_TYPE_CAR": "WEDGE",
  "EL_PB186A01_4": "",
  "EL_PB186A01_5": "",
  "EL_PB186A01_2": "18600182G0500",
  "EL_PB186A01_3": "",
  "SHACKLE_TYPE_CWT": "WEDGE",
  "SHACKLE_PLANC_D10": "Y",
  "ADD_CLIP_CAR": "N",
  "EL_PB186A01_6": ""
}
```

| 서브웨이트

아래 api에서 PID는 pid명이고 hogi는 제품의 명이야.
부품 : 서브웨이트
PID: EL_PB181B
제품명: 210521L13

이 정보를 바탕으로 201763L22 현장 서브웨이트 뭐가 수배되야하는지 분석해줘

[추가정보]
V_SUB_H_1 : 수량 1
V_SUB_H_2 : 수량 2
V_SUB_H_3 : 수량 3
V_SUB_PARTNO_1 : 자재번호 1
V_SUB_PARTNO_2 : 자재번호 2
V_SUB_PARTNO_3 : 자재번호 3
V_SUB_P_1 : 자재 1번 금액
V_SUB_P_2 : 자재 2번 금액
V_SUB_P_3 : 자재 3번 금액


```
https://plmpro.hdel.co.kr/plmetc/vault/pidExecute?PID=EL_PB181B&hogi=210521L13&testVersion=&isfloor=N&floor=
```


| 벨런스웨이트 > 성공

아래 api에서 PID는 pid명이고 hogi는 제품의 명이야.
PID: EL_PB120B03
제품명: 208223L01


```
https://plmpro.hdel.co.kr/plmetc/vault/pidExecute?PID=EL_PB120B03&hogi=202801L01&testVersion=&isfloor=N&floor=
```

## 02. 설계변경에서 수정된거

```
https://plmpro.hdel.co.kr/plmetc/vault/findModParts?productNo=N26185L01
```





| BOM 1레벨 추출
```
https://plmpro.hdel.co.kr/jsp/help/searchProductOneLevelJson.jsp?productNumber=204201L11&checkOption01=false
```


| BOM 1레벨 추출 : 테스트 완료 > 엑셀로 정리 완료
```api
https://plmpro.hdel.co.kr/plmetc/vault/getProductOneList?hogiNumber=204201L11
```


| 영업사양 정보 추출

```
https://plmpro.hdel.co.kr/plmetc/vault/findSalesLogic?proudtNo=210521L13
```