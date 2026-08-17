---
작성일: 2026-08-17
---



## 관련 쿼리 정리

```SQL
  
  
CREATE TABLE PLM_LLM_METADATA (  
    CATEGORY      VARCHAR(50) PRIMARY KEY,   -- 예: 'PID_QUERY', 'SALES_SPEC', 'DUTY_REVIEW'  
    TITLE         NVARCHAR(100),             -- 메타데이터 명칭 (다국어 지원)  
    CONTENT       NVARCHAR(MAX),             -- 마크다운/프롬프트 지침 전문  
    VERSION       INT DEFAULT 1,             -- 문서 버전  
    UPDATED_BY    NVARCHAR(50),              -- 최종 수정자 이름  
    UPDATED_AT    DATETIME DEFAULT GETDATE() -- 최종 수정일시  
);  
  
  
INSERT INTO PLM_LLM_METADATA  
VALUES('LOGIC_VERIFY'  
      , '로직 검증'  
      ,'# PLM 영업사양(공사) 자연어 기반 SQL 생성 학습 문서'  
      , '1'  
      ,'KYH'  
      ,GETDATE())  
;  
  
SELECT * FROM PLM_LLM_METADATA;
```