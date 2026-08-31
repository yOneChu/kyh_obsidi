---
작성일: 2026-08-31
---





1. 테스트호기 선택
2. 영업사양 수정(사람이) - 오늘날짜로 수정
3. 종속사양 산출 수행
4. bom 계산 수행
5. bom 저장
6. 영업사양/bom 비교해서 차이점 분석


## PLM API (with Python)
1. wip 생성 - PLM_AUTO_FUNCTION > make_wip

2. 호기의 ouid 조회 - PLM_AUTO_FUNCTION > projectNo

```
https://plmpro.hdel.co.kr/jsp/help/ouidList.jsp?md%24number=N28866L01
```


3. 종속사양 산출 - PLM_AUTO_FUNCTION > jongsoksung


4. BOM 계산 - PLM_AUTO_FUNCTION > bom_calculation



## 번위
1. 공사정보 변경
PLM_AUTO_FUNCTION > site_info_change

2. 로그인 토큰 관련
PlmInterface.get_authtoken

3. 공사정보 승인
PLM_AUTO_FUNCTION > approval




## 파라미터 변수
-cmd : 'makeWip'
-objectOuid : elv_info$vf@AC443BE1

api : migraion_result = req.post("http://plmpro.hdel.co.kr/SalesObject.do", data= inputstr) # wip 만들기