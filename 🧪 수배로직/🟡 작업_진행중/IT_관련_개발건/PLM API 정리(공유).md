---
작성일: 2026-07-22
---
---



## 01. 최초설계완료 호기 조회
```
- https://plmpro.hdel.co.kr/jsp/help/gethogilistByBlockopt.jsp?searchdate=타겟일자

[사용방법]
searchdate 파라미터의 '타겟일자'에 조회하려는 날짜(YYYYMMDD)를 입력하시면 됩니다.
  
```

## 02. BOM 조회
```
[API 설명]
- 제품(현장,호기)의 BOM

[API URL]
- https://vault-in.hdel.co.kr:8070/api/findProductInfo?key=subae&productNo=N28431L01

[파라미터 정보]
- productNo : 현장(호기) 번호
```

## 03. 영업사양 정보 조회
```
[API 설명]
- 제품(현장,호기)의 BOM

[API URL]
- https://vault-in.hdel.co.kr:8070/api/findElvSearch?key=subae&productNo=N28431L01

[파라미터 정보]
- productNo : 현장(호기) 번호
  
```

## PLM 영업사양 정보
https://plmpro.hdel.co.kr/jsp/help/salesInfoFromProductViewJson.jsp?productNumber=N28431L01

## 04. 품번으로 속성정보 조회

```
[API 설명]
- 제품(현장,호기)의 BOM

[API URL]
- https://vault-in.hdel.co.kr:8070/api/findPartOneWithPartNo?key=subae&partNo=32610139G07TQ

[파라미터 정보]
- partNo : 품번,자재번호
  
[결과 컬럼]
PARTNO : 자재번호
PARTNAME : 자재명,파트명
BLOCKNO
SPEC : SPEC
PARTSIZE : SIZE
PART_STATUS : 활성 상태
NATION : 자재코드 OWNERSHIP
DESIGN_USE : 설계 사용
COST_USE : 견적 사용
ORIGIN_DIV : 최초구분

  
```

## 05. 특성코드 리스트 추출
```
[API URL]
- https://vault-in.hdel.co.kr:8070/api/getCodeList


[결과 컬럼]
code : 사양  
codeName : 사양명  
typeName : 특성명  
typeVal : 특성값  
name : name
```

