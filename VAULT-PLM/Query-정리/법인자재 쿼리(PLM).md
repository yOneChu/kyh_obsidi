---
tags:
  - PLM
---
---


| 쿼리
```
SELECT A.md$number AS PARTNO,  
       A.md$desc AS PART_NAME,  
       A.BLOCKNO_NUMBER AS BLOCKNO, A.spec, A.g_l_code GL_CODE,  
       A.nation,  
       CODN(A.NATION) AS NA,  
       cod(A.uom) AS UOM,  
       cod(A.origin_div) origin_div,  
       cod(A.spt) spt,  
       A.PART_STATUS,  
       CODN(A.part_status) part_status,  
       A.old_code, A.old_code2, A.old_code3, old_code4  
FROM normalpart$vf A, normalpart$id B  
WHERE vf$ouid=id$last  
  AND LENGTH (md$number)=11 and md$status='RLS' and part_status=2466425004 AND NATION = 2803457356  
AND MD$NUMBER = 'C121P013545';  
-- part_status -> 2466425004: 활성  
-- NATION -> 2803457356:중국법인,
```