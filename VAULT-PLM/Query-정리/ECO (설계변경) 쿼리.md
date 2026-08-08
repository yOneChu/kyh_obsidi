---
작성일: 2026-08-06
---

---

#### | 정리중 설계변경 쿼리
```SQL
SELECT
    A.ISTRANSFERED,  
       A.SF$OUID,  
       A.MD$NUMBER, -- ECONO  
       A.MD$STATUS, -- 상태  
       A.MD$DESC, -- ECO명  
       a.MD$USER AS 등록자,  
       (SELECT B.USER_NAME FROM V_USER_INFO B WHERE B.USER_ID = A.MD$USER) AS CRE_USER, -- 등록자  
       --A.EC_CLASS, -- 구분 (제품신규/제품수정/ 등)  
       CODN(A.EC_CLASS) AS EC_CLASS_KO, --구분 (제품신규/제품수정/ 등)  
       --A.EC_CLASS_DESC,       CODN(A.EC_CLASS_DESC) AS 구분내역_실시시기, -- 구분내역/실시시기  
       A.PROD_MODIFY_ETC AS 기타변경, --  
       A.EC_REMARK_DESC AS 내용및사유, -- 내용및사유  
       CODN(A.VERIFICATION) AS 검증유무, --검증유무  
       CODN(A.LOGIC_EXCEPTION) AS 정합성예외, --정합성 예외  
       CODN(A.PHANTOM_EXCEPTION) AS Phantom, --Phantom 하위 자재 검사 예외  
       SUBSTR(A.MD$CDATE, 0 , 8) AS CREDAY,  
       DATEFORMAT(A.MD$CDATE, 'YYYYMMDDHH24MISS', 'YYYY-MM-DD HH24:MI:SS') AS CRE_DATE, 
       DATEFORMAT(A.MD$MDATE, 'YYYYMMDDHH24MISS', 'YYYY-MM-DD HH24:MI:SS') AS MOD_DATE  
       --,A.*  
FROM CHANGEORDER$SF A  
WHERE  
--SUBSTR(A.MD$MDATE, 1, 6) = '202501'  
--AND A.MD$STATUS = 'RLS'  
A.MD$NUMBER = 'ECO2608-0877'  
--AND A.EC_CLASS IN ('2500087942', '2500087944', '2500087943') -- 제품신규-전체/의장/구조
```


| 설계변경 참고
```SQL
--설계변경 구분 종류
SELECT
       DISTINCT CODN(A.EC_CLASS) AS EC_CLASS_KO, A.EC_CLASS AS OID
FROM CHANGEORDER$SF A;
-- EL_CLASS
-- 2500087942 : 제품신규-전체
-- 2500087944 : 제품신규 - 의장
-- 2500087943 : 제품신규 - 구조
-- 2500087945 : 제품수정
-- 2500087955 : 제품 - JQPR
-- 2500087950 : 도면신규



-- 설계변경 테이블
SELECT
       A.SF$OUID,
       A.MD$NUMBER, -- ECONO
       A.MD$STATUS, -- 상태
       A.MD$DESC, -- ECO명
       a.MD$USER AS 등록자,
       (SELECT B.USER_NAME FROM V_USER_INFO B WHERE B.USER_ID = A.MD$USER) AS CRE_USER, -- 등록자
       A.EC_CLASS, -- 구분 (제품신규/제품수정/ 등)
       CODN(A.EC_CLASS) AS EC_CLASS_KO,
       A.EC_CLASS_DESC,
       CODN(A.EC_CLASS_DESC) AS EC_CLASS_DESC_KO,
       A.PROD_MODIFY_ETC AS 내용, -- 내용 및 사유
       A.EC_REMARK_DESC AS 기타변경, -- 기타변경
       DATEFORMAT(A.MD$CDATE, 'YYYYMMDDHH24MISS', 'YYYY-MM-DD HH24:MI:SS') AS CRE_DATE,
       DATEFORMAT(A.MD$MDATE, 'YYYYMMDDHH24MISS', 'YYYY-MM-DD HH24:MI:SS') AS MOD_DATE
       -- ,A.*
FROM CHANGEORDER$SF A
WHERE
SUBSTR(A.MD$MDATE, 1, 6) = '202501'
AND A.MD$STATUS = 'RLS'
AND A.EC_CLASS IN ('2500087942', '2500087944', '2500087943') -- 제품신규-전체/의장/구조
```