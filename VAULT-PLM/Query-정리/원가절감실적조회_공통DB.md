
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
