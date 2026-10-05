---
작성일: 2026-10-04
---


```text
COMPRD에서 원가 관련 테이블

DB에는 접속하지 않고 코드만 분석했습니다. 코드에서 COMPRD로 조회하는 테이블 중에 원가 전용 테이블은 없습니다. 금액 컬럼이 있는 테이블은 2개입니다.

테이블: ZQMT007 (JQPR 품질 문제)
금액 컬럼: MATCOST, IWBTR
의미: MATCOST = 재료비, IWBTR = 노무비 (주석: "뒤에 00 붙여야됨")
사용 위치: PartCommonUtil.getJQPRfromSAP() (src/dyna/plmetc/bom/PartCommonUtil.java:1371-1372)
────────────────────────────────────────
테이블: zmaster02 (수주/WBS 마스터)
금액 컬럼: NETWR
의미: 수주 금액 (판매가, 원가 아님)
사용 위치: commonDB.getzmaster02NETWR() (src/dyna/plmetc/commonDB/commonDB.java:91),
eaiProductDAO.java:65에서 "수주금액 없음" 체크용

원가에 가장 가까운 것은 ZQMT007의 MATCOST/IWBTR 입니다. 다만 이 값은 JQPR(품질 문제) 건별 처리 비용(재료비·노무비)이고, 제품 표준원가가 아닙니다.

코드에서 COMPRD에 접속하는 경로

1. MyBatis: comdbMybatisConfig.xml과 COMDBMyBatisConnectionFactory를 거쳐 commonDB.java의 어노테이션 매퍼를 씁니다.
2. Spring: db.properties의 mdm.* 설정으로 mdmJdbcTemplate이 만들어지고, CharVariantsService에서 씁니다.
3. 직접 JDBC: CommonDBConnection, PartCommonUtil, LayoutDrawingNotifier, BatchremodelingLayoutAttacher, MANUFACTUREDRAWING_EVENT_*_AFTER.java에서 하드코딩된 계정으로 접속합니다.

그 밖에 COMPRD에서 쓰는 테이블 (금액 없음)

- 영업/수주: ZSDT0005(호기 영업사양), VW_CRMQUOTATIONINFO(견적 사양. 코드에서는 EL_* 사양 컬럼만 읽음), ABENGBYSALES$SF
- 생산/출하: ZPPT027(블록별 출하일), ZPPT034(ERP 상태·수량), ZMMT025(출하일자)
- 구매: SAPHEE.EKKO/EKPO/ZMMT013. EKPO는 SAP 표준상 단가(NETPR)가 있는 테이블이지만, 코드에서는 EBELN, EBELP, MATNR, MENGE만 조회합니다.
- 생산오더: SAPHEE.AFKO/AFPO/PRPS
- 기준정보: zmdat1040, ZMDAT1230, ZMDAT3020/3020T, SALESMASTER_HISTORY
- 인사/협력사: VW_ZHRT001/002/005, VW_ZMASTER02, VW_ZMMT012, TB_PST0001/0011
- 기타: ZQM_PLM_PPAPDWG(INSERT)

참고

- COMPRD가 아닌 곳: 원가 PID 로직(COSTVARIANT_H/D, CostVariant, CostSubaeManager)은 PLM DB(PDMMyBatisConnectionFactory)를 씁니다. eaiProductDAO의 saphee.mara/marc/ZCOMOVEXTRANS는 ERP DB2(db2pdm) 접속입니다.
- 한계: 코드에서 실제로 참조하는 테이블만 확인한 결과입니다. COMPRD 스키마에 원가 테이블(예: SAP의 KEKO/KEPH, MBEW의 STPRS/VERPR 등)이 따로 있는지는 코드로는 알 수 없습니다. 직접 확인하시려면 COMPRD로 접속해서 이 쿼리를 실행해 보세요: SELECT table_name FROM all_tables WHERE owner='COMPRD' AND (table_name LIKE '%COST%' OR table_name LIKE 'KE%' OR table_name='MBEW')
- 보안: 여러 소스 파일(CommonDBConnection.java, PartCommonUtil.java 등)에 COMPRD 계정과 비밀번호가 평문으로 하드코딩되어 있습니다.
```