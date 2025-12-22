
---


| 파트  조회
```sql
with ouid as  
                  ( select A.vf$ouid from NORMALPART$vf A, NORMALPART$id B  
                    where A.vf$identity = B.id$ouid and A.vf$ouid = B.id$wip  
                    -- AND ( md$number in ( 'C182P014647') )  
                    AND SUBSTR(A.MD$CDATE, 0, 4) IN( '2022' )  
                  )  
                  SELECT A.MD$NUMBER AS PARTNO,  
                         A.MD$DESC AS PARTNAME, -- 파트명  
                         A.VF$VERSION AS VERSION,  
                         A.ENAME,  
                         A.CNAME,  
                         A.UOM,  
                         CODN(A.UOM),  
                         A.PART_DIVISION,  
                         A.SAVEUSER,  
                         A.DISUSE_YN, -- 폐기여부  
                         A.ORIGIN_DIV,  
                         A.G_L_CODE AS GL_CODE,  
                         A.BLOCKNO_NUMBER,  
                         A.PART_SIZE,  
                         A.SPEC_EN,  
                         A.SPEC,  
                         A.SPEC01, A.SPEC02, A.SPEC03, A.SPEC04, A.SPEC05, A.SPEC06, A.SPEC07, A.SPEC08, A.SPEC09, A.SPEC10, A.SPEC11, A.SPEC12, A.SPEC13, A.SPEC14, A.SPEC15, A.SPEC16,  
                         A.SPEC17, A.SPEC18, A.SPEC19, A.SPEC20  
                         --A.*  
                  FROM NORMALPART$VF A  
                  WHERE A.VF$OUID IN (SELECT * FROM OUID)  
                  --AND SUBSTR(A.BLOCKNO_NUMBER, 2,1) != '6'  
                  --AND SUBSTR(A.BLOCKNO_NUMBER, 2,1) != '5'                  --AND SUBSTR(A.BLOCKNO_NUMBER, 2,1) IN ('1','2','3');
```


| BLOCK 조회
```sql
SELECT  
    B.MD$NUMBER,  
    B.MD$DESC,  
    B.REMARKS,  
    B.MD$STATUS,  
    COD(B.BLOCK_OPT),  
    COD(B.GC_PRODUCT),  
    B.QTY_PID, B.CMT_PID,  
    B.SPEC01, B.SPEC02, B.SPEC03, B.SPEC04, B.SPEC05, B.SPEC06, B.SPEC07,  
    B.SPEC08, B.SPEC09, B.SPEC10, B.SPEC11, B.SPEC12, B.SPEC13, B.SPEC14,  
    B.SPEC15, B.SPEC16, B.SPEC17, B.SPEC18, B.SPEC19, B.SPEC20  
FROM BLOCKNO$SF B  
;
```