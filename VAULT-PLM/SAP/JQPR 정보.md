---
tags:
  - 공통DB
  - PLM
---
---
```
네, ZQMT007, ZQMT009 두 개 테이블 활용하시면 되실 겁니다. 김영진 매니저님이 관리하는 BI대시보드 플랫폼 이용하시는 거라면 김영진 매니저님께 현재 사용중인 ZPPT206테이블 데이터세트에 대해 한번 여쭤보고 확인해보세요
```


### | 📝 JQPR 정보
```
SELECT  
       B.JQPRNUM, -- 관리번호  
       B.JQPRNO, B.HOGI, B.GUBUN,  
       B.STATUS, -- 상태 > 9:종결완료, 2:접수완료  
       B.BOM_STAT, -- BOM상태 > C:BOM완료, A:품질BOM, B-1:기계설계BOM,  
       B.CREDT, --작성일  
       B.CRENM, --작성자  
       B.POST1, -- 프로젝트명  
       B.SPEC, -- 사양  
       B.MANDT,  
       B.ATYPE, -- 기종/전기  
       B.POST1,  
       B.MATCOST, --재료비  
       B.IWBTR, -- 노무비  
       B.IMPKTL, -- 내부부서1  
       B.IMPKTL3_P,  
       B.REJTXT, -- 문제점 제목  
       B.REJLT, -- 문제점 상세  
       B.REQLT, -- 요청사항  
       B.CAUSETXT, -- 고장원인  
       B.PHENOTXT, -- 고장현상  
       B.CORLT, -- 조치확인  
       B.CLODT, --종결처리일  
       B.CLOID  
       , B.WRKLFN,  
       B.CATCODE -- 품목번호  
       , B.ZPROFCHK  
--, B.*  
FROM ZQMT007 B  
WHERE  
  B.JQPRNUM IS NOT NULL  
  --AND B.JQPRNUM IN ('1100037734', '1100038091')  
  and SUBSTR(b.CREDT, 1, 6)  = '202501'  
AND B.ZPROFCHK IS NOT NULL  
;
```