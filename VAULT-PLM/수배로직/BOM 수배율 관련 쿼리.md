---
tags:
  - PLM
---
---

BI의 설계일은 PLM의 "EAI_BOM_BLOCKOPT" 테이블에서 모든블럭 컬럼이 "X"되어 있는 "CODAT' 컬럼의 MIN값을 참고한다.


### 테이블 생성
```
CREATE TABLE subaeProduct (  
    PRODUCTNO    nvarchar(100) not null,  
	BATCHDATE    nvarchar(100),  
	PRODUCTVER   nvarchar(100),  
	M_MANAGER    nvarchar(100),  --기계담당자
	E_MANAGER    nvarchar(100),  --전기담당자
	APPDATE      nvarchar(100),  
	PARTNO       nvarchar(100),  
	PARTNAME     nvarchar(200),  
	BLOCKNO      nvarchar(100),  
	QTY          nvarchar(100),  
	BLOCK_OPT    nvarchar(100),  
	CMT          nvarchar(4000),  
	UCHECK       nvarchar(100),  
	GLCODE       nvarchar(100),  
	m_ModCount   nvarchar(100),  
	c_ModCount   nvarchar(100),  
	one_ModCnt   nvarchar(100),  
	two_ModCnt   nvarchar(100),  
	three_ModCnt nvarchar(100),  
	allPartCnt   nvarchar(100),  
	CREDATE      nvarchar(100),  
	MODDATE      nvarchar(100),  
	GISONG       nvarchar(100)
                             );
```

### 삭제
```
drop table subaeProduct;
```