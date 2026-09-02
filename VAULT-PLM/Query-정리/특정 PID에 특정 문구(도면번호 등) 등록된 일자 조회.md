---
작성일: 2026-06-16
수정일: 2026-09-02
---
#최초등록PID 

---


### 특정 문구(도면번호 등)가 최초 수배된 PID 조회
```SQL
SELECT d.NO, H.*  
FROM variant_d d,  
     variant_h h  
WHERE h.HOUID = d.HOUID  
  AND h.PID = 'EL_PB186A01'  
  AND h.REG_DATE > TO_DATE('20250101', 'YYYYMMDD')  
  AND (D.VAL1 LIKE '%18600209%' 
  OR D.VAL2 LIKE '%18600209%' 
  OR D.VAL3 LIKE '%18600209%' 
  OR D.VAL4 LIKE '%18600209%'  
  OR D.VAL5 LIKE '%18600209%' 
  OR D.VAL6 LIKE '%18600209%'  
  OR D.VAL7 LIKE '%18600209%' 
  OR D.VAL8 LIKE '%18600209%' 
  OR D.VAL9 LIKE '%18600209%')  
  AND H.VERSION != '-1'
```

### 쿼리 개선
```SQL
WITH TARGET_DATA AS (
    SELECT d.NO,
           H.pid,
           H.name,
           H.REG_DATE,
           H.VERSION,
           H.REMARKS,
           -- 버전을 오름차순으로 정렬하여 순위를 매김 (1위가 제일 낮은 버전)
           RANK() OVER (ORDER BY H.VERSION ASC) AS rnk
    FROM variant_d d,
         variant_h h
    WHERE h.HOUID = d.HOUID
      AND h.PID = 'EL_PB183D01'
      AND h.REG_DATE < TO_DATE('20250101', 'YYYYMMDD')
      AND (d.VAL1 LIKE '%?%'
        OR d.VAL2 LIKE '%?%'
        OR d.VAL3 LIKE '%?%'
        OR d.VAL4 LIKE '%?%'
      )
      AND H.VERSION != '-1'
)
SELECT NO,
       pid,
       name,
       REG_DATE,
       VERSION,
       REMARKS
FROM TARGET_DATA
WHERE rnk = 1;
```