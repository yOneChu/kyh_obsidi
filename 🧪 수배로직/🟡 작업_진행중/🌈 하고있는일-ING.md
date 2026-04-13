
---

**| 모닝 루틴**
- [ ] 설치 메뉴얼 읽기 - GTLX
- [ ] 박코치 매일 3회 쉐도잉
- [ ] 마천문 10문장 외우기
- [ ] 다른사람 로직보기


**| 진행 중 학습**
- 나도코딩 머신러닝 : 15분~
- 인프런 머신러닝 : ~6강
	- 집: ~7강
- 데이터시각화 POWER BI : ~47p
- 리액트 : 
- TailwindCSS : 
- 

**| 개발 해야될 건**
- 최초설계완료BOM 이력 쌓아서 조회하는 기능
- 로직 비교기능(추가, 삭제 변경 나오게)
- 원가절감 실적 조회를 공통DB로 조회하면 2레벨도 조회할 수 있을거 같다.
	- [[원가절감실적조회_공통DB]]
- PID시뮬레이터
	- [[PID_시뮬레이터]]


**| 데이터 브릭스**
브릭스에 있는 테이블 목록 요청
수익률 분석 관련 데이터 > 데이터브릭스 에 있다.


**| 자동설계 작성**
- 사유 그럴싸하게 쓰기
- CWT SAFETY의 연관부품 수정사항 없다면 자동설계가 가능하다.


**| SAFETY GUARD 전산화 하기**



**| CWT SHEAVE**
후락을 횡락으로 돌려서 쓰기도 한다. 즉 수평횡락으로.

EL_PB124B
78라인
EL_ECWTP	!R	EL_EMCBD	SH


EL_PB181A03의 7라인 참고


```
SELECT V.MD$DESC,  
       V.MD$NUMBER AS PRODUCTNO,  
       COD(V.EL_ECWTP), -- CWT_위치  
       V.CO_DPEXQ1, --  교환기(1) 대수  
       V.CO_DPEXQ2, --  교환기(2) 대수  
       V.CO_DPEXQ3, --  교환기(3) 대수  
       V.CO_ELQTY, -- EL대수  
       COD(V.CO_LAND1), --국가코드  
       COD(V.EL_AARRT), -- CAR배열 형식  
       COD(V.EL_ACD2), --적용코드  
       V.EL_ADRV, --운행방식  
       V.EL_AEVAUX, --  
       V.EL_AEVFQ,  
       V.EL_AEXP,  
       V.EL_AFF,  
       V.EL_AFFQ,  
       V.EL_AFFT0, EL_AFFT1,EL_AFFT2, EL_AFFT3, EL_AFFT4, EL_AFFT5, EL_AFFT6, EL_AFFT7,  
       V.EL_AFQ, --층수  
       COD(V.EL_AOPEN) AS EL_AOPEN, -- 열림방식  
       CODN(v.EL_AUSE) AS EL_AUSE, --용도  
       COD(V.EL_DCRG) AS RGS적용,  
       COD(V.EL_BCL) AS EL_BCL,  
       COD(V.EL_BWCAD) AS EL_BWCAD,  
       COD(V.EL_ECWTP), -- CWT_위치  
       V.EL_ECWBUFBH, --CWT BUFFER BLOCKING 높이  
       V.EL_ECCH, --CAR 높이; CH  
       V.EL_ECBG, --CAR:BG  
       V.EL_ECEE, --CAR 무게중심;EE  
       V.EL_ECJJ, --도어폭;JJ  
       V.EL_ERPW,  
       COD(V.EL_ECWRL) AS EL_ECWRL, --CWT RAIL(K)  
       COD(V.EL_ETM) AS EL_ETM, --권상기  
       V.EL_ECWBG, --CWT; BG  
       V.EL_ECWW, --CWT;폭  
       COD(V.EL_ECSF), --CAR; SAFETY  
       COD(V.EL_ASPC), --시방서  
       COD(V.EL_ASPCD), -- 시방서 DEVIATION 여부  
       COD(V.EL_BCL) AS EL_BCL, -- 천장종류  
       V.EL_AMAN AS EL_AMAN, --인승  
       COD(V.EL_ASPSCD) AS EL_ASPSCD, --생산거점(설계)  
       CONCAT('elv_info$vf@', LOWER(DECTOHEX(V.vf$ouid))) OUID,   -- 영업사양 객체  
       CODN(V.EL_ABRAND) AS EL_ABRAND, -- 브랜드  
       CODN(V.EL_ATYP) AS EL_ATYP, -- 기종  
       CODN (V.EL_ASPD) AS EL_ASPD, -- 속도  
       CODN (V.EL_ACAPA) AS EL_ACAPA, --용량  
       V.EL_ZTEXT_B, --가내 특기사항  
       V.EL_ZTEXT_C, --승장 특기사항  
       V.EL_ZTEXT_D, --옵션 특기사항  
       V.EL_ZTEXT_E, --L/O 특기사항  
       V.EL_ZERR_M3_1, --기계 에러 메시지  
       V.EL_ZERR_E3_1, --전기 에러 메시지  
       V.EL_ZERR_M5_1, --기계 미품목,  
       V.EL_ZERR_E5_1, --전기 미품목  
       V.EL_ZERR_C_1, --공통 에러 메시지  
       V.EL_ZERR_A_1, --자동 입력 오류  
       V.MD$USER,  
       V.MD$CDATE,  
       CODN(V.EL_ETM)  
       --V.*  
FROM ELV_INFO$VF V, ELV_INFO$ID A  
WHERE  
    V.vf$identity = A.id$ouid and V.vf$ouid = A.id$wip  
    AND SUBSTR(V.MD$CDATE, 0, 4) = '2026'  
--AND CODN(V.EL_ATYP) LIKE '%WBLX%'  
--AND V.MD$NUMBER = '206504L01';  
  --AND V.MD$NUMBER NOT LIKE 'TEST%'AND COD(V.EL_ECWTP) = 'R'  
;
```