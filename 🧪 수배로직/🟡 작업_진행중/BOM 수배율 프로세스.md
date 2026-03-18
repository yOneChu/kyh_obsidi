
---



### BOM 수배율 프로세스 문서 작성

**1. 정보기술팀에서 개발한 해당 날짜의 API 호출**
1) 금일(타겟일자) 최초설계된 호기 검색
- https://plmpro.hdel.co.kr/jsp/help/gethogilistByBlockopt.jsp?searchdate=타겟일자
- gethogilistByBlockopt.jsp
- 최초설계된 호기 검색의 기준조건
	- `eai_bom_blockopt` DB테이블에 ECO가 승인될때마다 모든 품목이 있는지 체크하여, 체크되어 있다면 최초 설계 완료된 것으로 간주.

1) 위 링크로 검색된 해당날자의 각 호기들에 대한 최초설계 BOM검색
- https://plmpro.hdel.co.kr/jsp/help/getBomByBlockoptList.jsp?proNo=타겟호기



**2. 데이터팀에서 해당 API호출하여 데이터브릭스에 데이터 정제하여 적재**

1) API -> Azure DW (매일 4시)
- 정보기술팀에서 개발한 위의 1), 2) API를  -1일 기준날짜로 순차적으로 수행
- API에서 가져온 데이터가 매일 새벽 4시에 Azure DW(Data Warehouse) 로 적재됨

2) Azure DW -> PB Mart 매일 5시
- Azure DW에 들어간 데이터가 매일 새벽 5시에 PB Mart 로 한 번 더 가공/전송됨

```
PB: POWER BI용 데이터 마트
MART: 분석/리포트용으로 사용하기 위한 데이터 마트(테이블)
```


- 테이블명: T_PLM_BOM
- 매일 새벽 4시에 -1일 기준으로 정보기술팀에서 개발한 API 호출하여 데이터 적재


**4.데이터 마트(최종 정제된 데이터)에 있는 데이터를 POWER BI로 불러와서 화면UI 구성(김영진M)**
- CODAT = 최초설계완료 ECO 승인일 기준으로 집계



