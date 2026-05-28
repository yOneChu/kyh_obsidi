---
작성일: 2026-05-28
---
---



### | ABENGBYSALES$SF
- ABENGBYSALES$SF 수주 테이블이다.
- 수주가 영업에서 안넘어오면 해당 테이블에 해당 현장의 데이터가 없다.



| 테이블
```SQL
SELECT MD$NUMBER, EL_ZFDA, EL_ZFDB, EL_ZFDC, EL_ZFDD from ABENGBYSALES$SF where MD$NUMBER = 'N28112L04';
```