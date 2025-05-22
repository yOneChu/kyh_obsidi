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
       B.JQPRNO,  
       B.HOGI,  
       B.GUBUN,  
       B.STATUS, -- 상태 > 9:종결완료, 2:접수완료, 1:반려, Z:반려  
       B.BOM_STAT, -- BOM상태 > C:BOM완료, A:품질BOM, B-1:기계설계BOM, D:BOM불필요  
       B.CREDT, --작성일  
       B.CRENM, --작성자명  
       B.POST1, -- 프로젝트명  
       B.SPEC AS 사양, -- 사양  
       B.MANDT,  
       B.ATYPE AS 기종, -- 기종/전기  
       B.POST1,  
       B.MATCOST AS 자재비, --재료비  
       B.IWBTR AS 노무비, -- 노무비  
       B.TEMNO,  
       B.IMPKTL, -- 내부부서1  
       B.IMPKTL3_P, -- 내부비용 퍼센테이지  
       B.IMPLFN,  
       B.IMPLFN_P,  
       B.REJTXT AS 문제점_제목, -- 문제점 제목  
       B.REJLT AS 문제점_상세, -- 문제점 상세  
       B.REQLT AS 요청사항, -- 요청사항  
       B.CAUSEGRP,  
       B.CAUSECOD,  
       B.CAUSETXT AS 고장원인, -- 고장원인  
       B.PHENOTXT AS 고장현상, -- 고장현상  
       B.CORLT AS 조치확인, -- 조치확인  
       B.CLODT AS 종결처리일, --종결처리일  
       B.CLOID,  
       B.WRKLFN AS 사업자등록번호, --사업자등록번호  
       B.CATCODE -- 품목번호  
       , B.ZPROFCHK AS 귀책증빙, -- 귀책증빙  
       B.PROG_STAT AS 진행현황  
, B.*  
FROM ZQMT007 B  
WHERE  
  B.JQPRNUM IS NOT NULL  
  AND B.JQPRNUM IN ('1100041877') -- 관리번호  
  and SUBSTR(b.CREDT, 1, 6)  = '202505'  
AND B.ZPROFCHK IS NOT NULL
;
```


### |  📝ZQMT009 : JQPR 상세화면의 원인/대책 부분 테이블  

```
SELECT  
       A.RQSTID, --대책요청자  
       A.RQSTDT, --대책요청일  
       A.FIXNM AS 대책작성자명,  
       A.APPNM AS대책작성부서장,  
       A.FIXDT, --대책완료일  
       A.SERNO, --결재행 순서  
       A.MCAUCHK, --체크되있는 행이 진짜임  
       A.CAUTXT, --  
       A.CAUDEP,  
       A.LTCAS, --현상 및 발생원인,  
       A.LTRST, --조치내용  
       A.LTFIX --재발방지대책  
FROM ZQMT009 A  
WHERE A.JQPRNUM = '1100042006';
```

---

### | 📝성준M JQPR 추출 방법
- ZQMAL4020D
- 작성일 기준으로 조회
- 엑셀로 추출
![[Pasted image 20250522091321.png]]