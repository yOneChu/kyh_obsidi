---
tags:
---
---

부품에 연결된 ECO 조회
```
WITH PARTOID AS (  
    select A.vf$ouid from NORMALPART$vf A, NORMALPART$id B  
    where A.vf$identity = B.id$ouid and A.vf$ouid = B.id$wip  
    and (  
        md$number in (  
        'C103P002778',  
        'C103P002779',  
        'C103P002780',  
        'C103P002781',  
        'C103P002782',  
        'C103P002783',  
        'C103P002784',  
        'C103P002785',  
        'C103P002786',  
        'C103P002787'  
            )  
        )  
)  
select ECO.MD$NUMBER,  
       ECO.MD$DESC,  
       (SELECT B.MD$NUMBER FROM NORMALPART$vf B WHERE B.VF$OUID = A.AS$END2) AS PARTNO  
from CHANGEORDER$SF ECO, ECOANDPART$AS A  
WHERE  
    ECO.SF$OUID = A.AS$END1  
AND A.AS$END2 IN (SELECT * FROM PARTOID);
```