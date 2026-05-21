---
작성일: 2026-05-20
---
---


**| 특정 문구 어느 PID에 걸려있는 조회(모든 버전 및 PID)**
```SQL
  
SELECT h.pid, H.VERSION, D.NO, NVL(D.REMARKS, '-') AS REMARKS,  
        NVL(D.SPEC1, '-') AS SPEC1, NVL(D.CON1, '-') AS CON1,  
        NVL(D.SPEC2, '-') AS SPEC2,  NVL(D.CON2, '-') AS CON2,  
        NVL(D.SPEC3, '-') AS SPEC3,  NVL(D.CON3, '-') AS CON3,  
        NVL(D.SPEC4, '-') AS SPEC4,  NVL(D.CON4, '-') AS CON4,  
        NVL(D.SPEC5, '-') AS SPEC5,  NVL(D.CON5, '-') AS CON5,  
        NVL(D.SPEC6, '-') AS SPEC6,  NVL(D.CON6, '-') AS CON6,  
        NVL(D.SPEC7, '-') AS SPEC7,  NVL(D.CON7, '-') AS CON7,  
        NVL(D.SPEC8, '-') AS SPEC8,  NVL(D.CON8, '-') AS CON8,  
        NVL(D.SPEC9, '-') AS SPEC9,  NVL(D.CON9, '-') AS CON9,  
        NVL(D.KEY1, '-') AS KEY1,  NVL(D.VAL1, '-') AS VAL1,  
        NVL(D.KEY2, '-') AS KEY2,  NVL(D.VAL2, '-') AS VAL2,  
        NVL(D.KEY3, '-') AS KEY3,  NVL(D.VAL3, '-') AS VAL3,  
        NVL(D.KEY4, '-') AS KEY4,  NVL(D.VAL4, '-') AS VAL4,  
        NVL(D.KEY5, '-') AS KEY5,  NVL(D.VAL5, '-') AS VAL5,  
        NVL(D.KEY6, '-') AS KEY6,  NVL(D.VAL6, '-') AS VAL6,  
        NVL(D.KEY7, '-') AS KEY7,  NVL(D.VAL7, '-') AS VAL7,  
        NVL(D.KEY8, '-') AS KEY8,  NVL(D.VAL8, '-') AS VAL8,  
        NVL(D.KEY9, '-') AS KEY9,  NVL(D.VAL9, '-') AS VAL9  
        FROM variant_d d, variant_h h --, variant_id id  
        WHERE  
            --h.HOUID = id.LAST_HOUID  
        h.HOUID =d.HOUID  
        AND H.PID = 'AUTO_EL_ZERR_M3'  
AND (  
       D.VAL1 LIKE '%공기청정기 2개 적용 유무 확인 할 것%'  
    OR D.VAL2 LIKE '%공기청정기 2개 적용 유무 확인 할 것%'  
    OR D.VAL3 LIKE '%공기청정기 2개 적용 유무 확인 할 것%'  
    )  
ORDER BY H.VERSION DESC
```