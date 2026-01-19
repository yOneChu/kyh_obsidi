
---



| PLM에 등록된 14자리 부품-제품

```
with ouid as  
         (select A.vf$ouid AS VFOID  
          from product$vf A,  
               product$id B  
          where A.vf$identity = B.id$ouid  
            and A.vf$ouid = B.id$wip  
            AND A.MD$NUMBER NOT LIKE 'TEST%'  
            AND A.MD$NUMBER NOT LIKE 'Q%'  
            AND SUBSTR(A.MD$MDATE, 0, 4) = '2026'  
          )  
 SELECT  
                      PE.SEQ  
                     , PE.PRODUCTOUID AS PRODUCT_ID  
                     , PE.PARTOUID AS PARTEND2_OID  
                      , LOWER(CONCAT('PRODUCT$VF@', DECTOHEX(PE.PRODUCTOUID))) END1HEX  
                      , LOWER(CONCAT('NORMALPART$VF@', DECTOHEX(PE.PARTOUID))) END2HEX  
                     , (SELECT MD$NUMBER FROM PRODUCT$VF WHERE VF$OUID = PE.PRODUCTOUID) AS PARENTNO  
                     , (SELECT F.VF$VERSION FROM PRODUCT$VF F WHERE F.VF$OUID = PE.PRODUCTOUID) AS PARENT_VER  
                     , (SELECT TO_CHAR(TO_DATE(PRODUCT.MD$CDATE, 'YYYYMMDDHH24MISS'), 'YYYY-MM-DD') FROM PRODUCT$VF PRODUCT WHERE PRODUCT.VF$OUID = PE.PRODUCTOUID) AS PROD_CREDATE  
                     , (SELECT TO_CHAR(TO_DATE(PRODUCT.MD$MDATE, 'YYYYMMDDHH24MISS'), 'YYYY-MM-DD') FROM PRODUCT$VF PRODUCT WHERE PRODUCT.VF$OUID = PE.PRODUCTOUID) AS PROD_MODDATE  
                     , (SELECT TO_CHAR(TO_DATE(PRODUCT.APP_DATE, 'YYYYMMDDHH24MISS'), 'YYYY-MM-DD') FROM PRODUCT$VF PRODUCT WHERE PRODUCT.VF$OUID = PE.PRODUCTOUID) AS PROD_APP_DATE  
                     , (SELECT PRODUCT.MD$STATUS FROM PRODUCT$VF PRODUCT WHERE PRODUCT.VF$OUID = PE.PRODUCTOUID) AS PROD_STATUS  
                     , (SELECT COD(E.EL_ATYP) FROM ELV_INFO$ID A, ELV_INFO$VF E  
                        WHERE A.ID$OUID = E.VF$IDENTITY AND E.vf$ouid = A.id$wip  
                        AND E.MD$NUMBER = (SELECT F.MD$NUMBER FROM PRODUCT$VF F WHERE F.VF$OUID = PE.PRODUCTOUID) ) AS GISONG  
                     , (SELECT COD(E.EL_ABRAND) FROM ELV_INFO$ID A, ELV_INFO$VF E  
                        WHERE A.ID$OUID = E.VF$IDENTITY AND E.vf$ouid = A.id$wip  
                        AND E.MD$NUMBER = (SELECT F.MD$NUMBER FROM PRODUCT$VF F WHERE F.VF$OUID = PE.PRODUCTOUID) ) AS BRAND  
                     , (SELECT COD(E.EL_ASPD) FROM ELV_INFO$ID A, ELV_INFO$VF E  
                        WHERE A.ID$OUID = E.VF$IDENTITY AND E.vf$ouid = A.id$wip  
                        AND E.MD$NUMBER = (SELECT F.MD$NUMBER FROM PRODUCT$VF F WHERE F.VF$OUID = PE.PRODUCTOUID) ) AS EL_ASPD -- 속도  
                     , (SELECT COD(E.EL_ASPSCD) FROM ELV_INFO$ID A, ELV_INFO$VF E  
                        WHERE A.ID$OUID = E.VF$IDENTITY AND E.vf$ouid = A.id$wip  
                        AND E.MD$NUMBER = (SELECT F.MD$NUMBER FROM PRODUCT$VF F WHERE F.VF$OUID = PE.PRODUCTOUID) ) AS ASPSCD  
                     , (SELECT COD(E.EL_ACAPA) FROM ELV_INFO$ID A, ELV_INFO$VF E  
                       WHERE A.ID$OUID = E.VF$IDENTITY AND E.vf$ouid = A.id$wip  
                       AND E.MD$NUMBER = (SELECT F.MD$NUMBER FROM PRODUCT$VF F WHERE F.VF$OUID = PE.PRODUCTOUID) ) AS EL_ACAPA -- 용량  
                     , (SELECT E.EL_ECWW FROM ELV_INFO$ID A, ELV_INFO$VF E  
                       WHERE A.ID$OUID = E.VF$IDENTITY AND E.vf$ouid = A.id$wip  
                       AND E.MD$NUMBER = (SELECT F.MD$NUMBER FROM PRODUCT$VF F WHERE F.VF$OUID = PE.PRODUCTOUID) ) AS EL_ECWW  
                     , (SELECT E.EL_ECWBG FROM ELV_INFO$ID A, ELV_INFO$VF E  
                       WHERE A.ID$OUID = E.VF$IDENTITY AND E.vf$ouid = A.id$wip  
                       AND E.MD$NUMBER = (SELECT F.MD$NUMBER FROM PRODUCT$VF F WHERE F.VF$OUID = PE.PRODUCTOUID) ) AS EL_ECWBG  
                    , (SELECT E.EL_ECBG FROM ELV_INFO$ID A, ELV_INFO$VF E  
                       WHERE A.ID$OUID = E.VF$IDENTITY AND E.vf$ouid = A.id$wip  
                       AND E.MD$NUMBER = (SELECT F.MD$NUMBER FROM PRODUCT$VF F WHERE F.VF$OUID = PE.PRODUCTOUID) ) AS EL_ECBG  
                     , (SELECT COD(E.EL_ETHRU) FROM ELV_INFO$ID A, ELV_INFO$VF E  
                       WHERE A.ID$OUID = E.VF$IDENTITY AND E.vf$ouid = A.id$wip  
                       AND E.MD$NUMBER = (SELECT F.MD$NUMBER FROM PRODUCT$VF F WHERE F.VF$OUID = PE.PRODUCTOUID) ) AS EL_ETHRU  
                     , (SELECT COD(E.EL_COB) FROM ELV_INFO$ID A, ELV_INFO$VF E  
                       WHERE A.ID$OUID = E.VF$IDENTITY AND E.vf$ouid = A.id$wip  
                       AND E.MD$NUMBER = (SELECT F.MD$NUMBER FROM PRODUCT$VF F WHERE F.VF$OUID = PE.PRODUCTOUID) ) AS EL_COB -- 전망종류  
                     , NP.MD$NUMBER AS PARTNO  
                     , cod(NP.NATION) AS NATION  
                     , NP.compen_part AS COMPEN_PART  
                     , NP.MD$DESC AS PARTNAME  
                     , NP.VF$VERSION AS PART_VERSION  
                     , PE.CMT AS CMT  
                     , NVL(NP.G_L_CODE, '') AS GLCODE  
                     , NVL(NP.SPEC, '') AS SPEC  
                     , NVL(NP.PART_SIZE, '') AS PART_SIZE  
                     , (SELECT MD$NUMBER FROM BLOCKNO$SF WHERE SF$OUID =  DECODE(NP.BLOCKNO, NULL, NULL, HEXTODEC(UPPER(SUBSTR(NP.BLOCKNO, 12))))) AS BLOCKNO  
                     , (SELECT COD(BLOCK_OPT) FROM BLOCKNO$SF WHERE SF$OUID =  DECODE(NP.BLOCKNO, NULL, NULL, HEXTODEC(UPPER(SUBSTR(NP.BLOCKNO, 12))))) AS BLOCK_OPT  
                     , (SELECT MD$NUMBER FROM BLOCKNO$SF WHERE SF$OUID =  DECODE(NP.UPPERBLOCKNO, NULL, NULL, HEXTODEC(UPPER(SUBSTR(NP.UPPERBLOCKNO, 12))))) AS UPPERBLOCKNO  
                     , NVL(COD(NP.UOM), '') AS UOM  
                     , PE.QTY AS PART_QTY  
                     , VP.WORK_QTY  
                     , VP.WORK_CMT  
                     , PE.COLOR  
                     , VP.WORK_COLOR  
                     , NVL(COD(NP.ORIGIN_DIV), '') DIV  
                     , NVL(PE.MBOM, '') MBOM  
                     , NVL(COD(NP.PART_MBOM), '') PART_MBOM  
                     , (SELECT MD$DESC FROM FUSER$SF WHERE MD$NUMBER = NP.MD$USER) USERNAME  
                     , NP.MD$USER USERID  
                     , COD(NP.PARTMPCHECK) AS PARTMPCHECK  
                     , (SELECT MD$DESC FROM FUSER$SF WHERE MD$NUMBER = PE.CUSER) AS CUSERNAME  
                     , PE.CUSER AS CUSERID  
                     , 1 LEV  
                     , 'F' ISLEAF  
                     , VP.UCHECK AS UCHECK  -- 수정여부  
                     , VP.MCHECK  
                     , NVL(COD(NP.PART_DIVISION), '') AS PART_DIVISION  
                     , PE.CDATE  
                     , VP.MDATE  
                    -- , DATEFORMAT(VP.MDATE, 'YYYYMMDDHH24MISS', 'YYYY-MM-DD HH24:MI:SS') AS 등록일  
                     , VP.user5  
                     , (SELECT COUNT(1) FROM PARTOFPART$AC WHERE AS$END1=NP.VF$OUID AND ROWNUM=1) AS HASCHILD -- 하위BOM 존재여부  
                    FROM  
                     PARTOFEBOM PE  
                    INNER JOIN NORMALPART$VF NP ON PE.PARTOUID = NP.VF$OUID  
                    LEFT OUTER JOIN VARIABLEPART_NEW VP ON VP.PRODUCTOUID = PE.PRODUCTOUID AND VP.ASSOOUID = PE.ASSOOUID  
                    WHERE  
                    PE.PRODUCTOUID IN (SELECT VFOID FROM ouid)  
                    AND LENGTH(NP.MD$NUMBER) = 14
```