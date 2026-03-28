
---



| 공통DB에서 조회 시
```SQL
  
--생산-BOM수신  
SELECT A.*  
FROM ZPPAT1010 A  
WHERE  
    A.CRDAT = '20260327'  
;  
  
--생산-BOM수신2 -> 이 쿼리가 원가절감실적조회랑 유사함
--원가절감 조회시 자재번호의 파라미터는 'A.IDNRK' 로 조회된다.
--2레벨을 조회하려면 모품번컬럼(MATNR)에 해당 자재번호를 넣으면된다.
SELECT  
    A.MATNR, A.IDNRK,  
    A.*  
FROM ZPPAT1011 A  
--WHERE TO_CHAR(A.CRDAT, 'YYYYMMDD') = '20260327'  
WHERE  
    A.CRDAT = '20260327'  
  --AND A.MATNR = '10110300G090A'  
--AND A.WOKNUM = 'M20623NC200-M20623L02'  
AND A.IDNRK = '10110300G090A' --자재번호  
;  
  
--생산-BOM주석수신  
SELECT A.* FROM COMPRD.ZPPAT1012 A  
WHERE  
    A.CRDAT = '20260327'  
AND A.IDNRK = '10110300G090A';  
  
--comment on column ZPPAT1011.MATNR is '모품목코드      '--comment on column ZPPAT1011.IDNRK is '자품목코드     '
```


| 2레벨 이하 조회
- 2레벨 이하만 조회하려면 BOM_LEVEL 조건으로 `1` 제외 하면 된다.
```SQL
--생산-BOM수신2 - 2레벨 이하 조회  
SELECT  
    A.WOKNUM, --제품번호  
    A.WOKVER, --제품버전  
    A.MATNR, --모품번  
    A.ITEM_SEQ,  
    A.IDNRK, --자품번  
    A.BOM_LEVEL,  
    A.MENGE,  
    A.MATKL,  
    A.ZPART, --품목  
    A.*  
FROM ZPPAT1011 A  
--WHERE TO_CHAR(A.CRDAT, 'YYYYMMDD') = '20260327'  
WHERE  
    A.CRDAT = '20260327'  
    AND A.MATNR = '10110300G090A' --모품목코드  
    AND A.WOKNUM = 'M20623NC200-M20623L02'  
    --AND A.MATKL = 'R31'  
;
```