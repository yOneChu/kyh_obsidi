
---


| 에러로그 조회
```SQL
SELECT PID, VERSION AS 버전,  
       NO AS LINE번호,  
       MD$DESC AS 등록자,  
       MSG AS 오류메시지,  
       LAST_OCCUR_DATE AS 최신발생시각,  
       LAST_OCCUR_HOGI AS 최신발생현장,  
       HIT AS오류횟수  
       FROM (  
              SELECT A.PID,  DECODE(A.VERSION,-1,'TEST',A.VERSION) VERSION, C.NO, E.MD$DESC, D.MSG, D.LAST_OCCUR_DATE, LAST_OCCUR_HOGI, D.HIT   
FROM VARIANT_H A   
LEFT OUTER JOIN VARIANT_ID B   
              ON A.PID = B.PID   
AND A.HOUID = B.LAST_HOUID   
JOIN VARIANT_D C   
              ON A.HOUID = C.HOUID   
JOIN VARIANT_ERRORLOG D   
              ON C.DOUID = D.DOUID   
JOIN FUSER$SF E   
              ON A.USERID = E.MD$NUMBER   
WHERE (A.VERSION=-1 OR B.PID IS NOT NULL)  
              UNION ALL  
              SELECT NULL, NULL, NULL, NULL, MSG, LAST_OCCUR_DATE, NULL, HIT  
              FROM VARIANT_ERRORLOG  
              WHERE DOUID=0  
              ORDER BY LAST_OCCUR_DATE desc, PID, NO  
              FETCH FIRST 200 ROWS ONLY  
              )
```


| 기간으로 에러로그 조회
```SQL
SELECT PID, VERSION AS 버전,  
       NO AS LINE번호,  
       MD$DESC AS 등록자,  
       MSG AS 오류메시지,  
       LAST_OCCUR_DATE AS 최신발생시각,  
       LAST_OCCUR_HOGI AS 최신발생현장,  
       HIT AS오류횟수  
from (SELECT A.PID,  
             DECODE(A.VERSION, -1, 'TEST', A.VERSION) VERSION,  
             C.NO,  
             E.MD$DESC,  
             D.MSG,  
             D.LAST_OCCUR_DATE,  
             D.LAST_OCCUR_HOGI,  
             D.HIT  
      FROM VARIANT_H A  
               LEFT OUTER JOIN VARIANT_ID B  
                               ON A.PID = B.PID  
                                   AND A.HOUID = B.LAST_HOUID  
               JOIN VARIANT_D C  
                    ON A.HOUID = C.HOUID  
               JOIN VARIANT_ERRORLOG D  
                    ON C.DOUID = D.DOUID  
               JOIN FUSER$SF E  
                    ON A.USERID = E.MD$NUMBER  
      WHERE (A.VERSION = -1 OR B.PID IS NOT NULL)  
        AND D.LAST_OCCUR_DATE >= TO_DATE('2026-03-19', 'YYYY-MM-DD')  
        AND D.LAST_OCCUR_DATE < TO_DATE('2026-03-20', 'YYYY-MM-DD')  
      UNION ALL  
      SELECT NULL,  
             NULL,  
             NULL,  
             NULL,  
             MSG,  
             LAST_OCCUR_DATE,  
             NULL,  
             HIT  
      FROM VARIANT_ERRORLOG  
      WHERE DOUID = 0  
        AND LAST_OCCUR_DATE >= TO_DATE('2026-03-19', 'YYYY-MM-DD')  
        AND LAST_OCCUR_DATE < TO_DATE('2026-03-20', 'YYYY-MM-DD')  
  
      ORDER BY LAST_OCCUR_DATE DESC, PID, NO)

```

