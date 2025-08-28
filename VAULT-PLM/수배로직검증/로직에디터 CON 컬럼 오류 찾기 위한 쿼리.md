---
aliases:
---
---



```
SELECT h.pid, D.NO, NVL(D.REMARKS, '-') AS REMARKS,  
         NVL(D.CON1, '-') AS CON1,  
          NVL(D.CON2, '-') AS CON2,  
          NVL(D.CON3, '-') AS CON3,  
          NVL(D.CON4, '-') AS CON4,  
          NVL(D.CON5, '-') AS CON5,  
          NVL(D.CON6, '-') AS CON6,  
          NVL(D.CON7, '-') AS CON7,  
          NVL(D.CON8, '-') AS CON8,  
          NVL(D.CON9, '-') AS CON9,  
         NVL(D.CON10, '-') AS CON10,  
         NVL(D.CON11, '-') AS CON11,  
         NVL(D.CON12, '-') AS CON12,  
         NVL(D.CON13, '-') AS CON13,  
         NVL(D.CON14, '-') AS CON14,  
         NVL(D.CON15, '-') AS CON15,  
         NVL(D.CON16, '-') AS CON16,  
         NVL(D.CON17, '-') AS CON17,  
         NVL(D.CON18, '-') AS CON18,  
         NVL(D.CON19, '-') AS CON19,  
         NVL(D.CON20, '-') AS CON20  
        FROM variant_d d, variant_h h, variant_id id  
        WHERE h.HOUID = id.LAST_HOUID  
        AND h.HOUID =d.HOUID  
AND (  
             D.CON1 LIKE '< %' OR D.CON2 LIKE '< %'  
                 OR D.CON3 LIKE '< %' OR D.CON4 LIKE '< %'  
                 OR D.CON5 LIKE '< %' OR D.CON6 LIKE '< %'  
                 OR D.CON7 LIKE '< %' OR D.CON18 LIKE '< %'  
                 OR D.CON9 LIKE '< %' OR D.CON10 LIKE '< %'  
                 OR D.CON11 LIKE '< %' OR D.CON12 LIKE '< %'  
                 OR D.CON13 LIKE '< %' OR D.CON14 LIKE '< %'  
                 OR D.CON15 LIKE '< %' OR D.CON16 LIKE '< %'  
                 OR D.CON17 LIKE '< %' OR D.CON18 LIKE '< %'  
                 OR D.CON19 LIKE '< %' OR D.CON20 LIKE '< %'  
             )
```
