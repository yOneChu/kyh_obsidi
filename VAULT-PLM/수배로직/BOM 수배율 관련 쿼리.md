---
tags:
  - PLM
---
---


### 테이블 생성
```
CREATE TABLE subaeProduct (  
    PRODUCTNO NVARCHAR(100) NOT NULL,  
    BATCHDATE NVARCHAR(100),  
    PRODUCTVER NVARCHAR(100),  
    APPDATE NVARCHAR(100),  
    PARTNO NVARCHAR(100),  
    PARTNAME NVARCHAR(200),  
    BLOCKNO NVARCHAR(100),  
    QTY NVARCHAR(100),  
    BLOCK_OPT NVARCHAR(100),  
    CMT NVARCHAR(4000),  
    UCHECK NVARCHAR(100),  
    GLCODE NVARCHAR(100),  
    m_ModCount NVARCHAR(100),  
    c_ModCount NVARCHAR(100),  
    one_ModCnt NVARCHAR(100),  
    two_ModCnt NVARCHAR(100),  
    three_ModCnt NVARCHAR(100),  
    allPartCnt NVARCHAR(100)  
                             );
```

### 삭제
```
drop table subaeProduct;
```