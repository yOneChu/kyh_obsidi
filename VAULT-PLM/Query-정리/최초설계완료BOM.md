
---

| 기존거에서 개선한 쿼리
```SQL
WITH PRODUCT_BOM AS(  
      SELECT B.MD$NUMBER  , B.VF$VERSION , B.MD$DESC , C.MD$NUMBER PART , C.MD$DESC PART_DESC, C.SPEC,  
             C.BLOCKNO_NUMBER,cod(E.block_opt) block_opt, A.QTY,  
             D.UCHECK, A.CDATE, A.CMT  
      FROM PARTOFEBOM A  
      INNER JOIN PRODUCT$VF B ON    A.PRODUCTOUID = B.VF$OUID  
      INNER JOIN NORMALPART$VF C ON    A.PARTOUID = C.VF$oUID  
      LEFT OUTER JOIN VARIABLEPART_NEW D ON   A.ASSOOUID = D.ASSOOUID    AND A.PRODUCTOUID = D.PRODUCTOUID  
      INNER JOIN blockno$sf E ON  'blockno$sf@'||lower(dectohex(E.sf$ouid)) =  c.blockno  
      WHERE B.MD$NUMBER = '211936L20'  
      ),  
    PRODUCT_BOM_NUM AS (  
    SELECT A.*,  
           TO_NUMBER(A.VF$VERSION) AS VF_VERSION_NUM  
    FROM PRODUCT_BOM A  
    WHERE REGEXP_LIKE(A.VF$VERSION, '^[0-9]+$')  
),  
FIRST_BOM_VF AS (  
    SELECT BLOCK_OPT,  
           MIN(VF_VERSION_NUM) FIRST_VF  
    FROM PRODUCT_BOM_NUM  
    GROUP BY BLOCK_OPT  
),  
FIRST_PRODUCT_BOM AS (  
    SELECT *  
    FROM (  
        SELECT A.*,  
               ROW_NUMBER() OVER (  
                   PARTITION BY A.MD$NUMBER, A.PART, A.CDATE  
                   ORDER BY A.VF_VERSION_NUM  
               ) RN  
        FROM PRODUCT_BOM_NUM A  
        INNER JOIN FIRST_BOM_VF B  
            ON A.BLOCK_OPT = B.BLOCK_OPT  
           AND A.VF_VERSION_NUM = B.FIRST_VF  
    )  
    WHERE RN = 1  
)  
SELECT MD$NUMBER,  
       VF$VERSION,  
       MD$DESC,  
       PART,  
       PART_DESC,  
       SPEC,  
       BLOCKNO_NUMBER,  
       BLOCK_OPT,  
       QTY,  
       UCHECK,  
       CDATE,  
       CMT  
FROM FIRST_PRODUCT_BOM  
WHERE QTY > 0  
   OR (QTY = 0 AND UCHECK = '1')
```

| 기존
```sql
WITH PRODUCT_BOM AS(  
      SELECT B.MD$NUMBER  , B.VF$VERSION , B.MD$DESC , C.MD$NUMBER PART , C.MD$DESC PART_DESC, C.SPEC,  
             C.BLOCKNO_NUMBER,cod(E.block_opt) block_opt, A.QTY,  
             D.UCHECK, A.CDATE, A.CMT  
      FROM PARTOFEBOM A  
      INNER JOIN PRODUCT$VF B ON    A.PRODUCTOUID = B.VF$OUID  
      INNER JOIN NORMALPART$VF C ON    A.PARTOUID = C.VF$oUID  
      LEFT OUTER JOIN VARIABLEPART_NEW D ON   A.ASSOOUID = D.ASSOOUID    AND A.PRODUCTOUID = D.PRODUCTOUID  
      INNER JOIN blockno$sf E ON  'blockno$sf@'||lower(dectohex(E.sf$ouid)) =  c.blockno  
      WHERE B.MD$NUMBER = '211936L20'  
      ), FIRST_BOM_VF AS  
      (  
          SELECT BLOCK_OPT, MIN(TO_NUMBER(VF$VERSION)) FIRST_VF  
          FROM PRODUCT_BOM  
          GROUP BY  BLOCK_OPT  
      ),  
     FIRST_PRODUCT_BOM AS  
      (  
          SELECT * FROM (  
             SELECT A.*, ROW_NUMBER() OVER (PARTITION BY MD$NUMBER, PART, CDATE ORDER BY VF$VERSION) RN  
              FROM PRODUCT_BOM A  
             INNER JOIN FIRST_BOM_VF B ON A.BLOCK_OPT = B.BLOCK_OPT AND A.VF$VERSION = B.FIRST_VF) WHERE RN=1  
      )  
          SELECT * FROM FIRST_PRODUCT_BOM WHERE QTY>0 or (QTY=0 AND UCHECK =1)
```