



버퍼는 새로운 버퍼가 나오는거 아니면 그냥 냅둬도 됨. 
카 자중 변경되는거에 따라 버퍼 적용 하중 범위 맞춰서 넣어놓는거다.


---

### | CALL
CAL_BUFON_EL - CAR BUFFER 신구형 적용
CAL_BUFWT - BUFFER 적용하중
- EL_ECSL: ◎ 적용하중 (CAR)
CAL_BUFQT : MRL BUFFER 수량 체크
- CAL_DURTB : 우레탄 버퍼 적용 조건 
AUTO_EL_ECSL - 적용하중 계산
- EL_ECW : CAR자중
- EL_ACAPA : 용량

EL_AUSE : 용도

### | 연관 PID
- CERTI_B183A01 - 완충기 부품인증
- EL_PB183B01 - BUFFER BLOCKING

### | CAL_BUFWT
- NN : 인승
- EL_ECSL : ◎ 적용하중 (CAR)

---

  
| SPEC    |            | CON        | 내용              |
| ------- | ---------- | ---------- | --------------- |
| EL_AUSE | 용도         | !(?N?,?O?) | 전만,장애,누드가 아닐 경우 |
| EL_ASPC | 시방서        | N          |                 |
| EL_ECSL | 적용하중 (CAR) |            |                 |
| EL_ZFDA | 기계구조 최초설계  |            |                 |

---


|           |       |     |
| --------- | ----- | --- |
| B183AA_BH | 전체 높이 | A 값 |


![[Pasted image 20251201130638.png]]

### 용어
- [[STROKE(스트로크)]] :  버퍼가 충격을 받았을 때 눌리면서 이동할 수 있는 거리. 즉 버퍼가 최대한으로 눌리는 변위
- BUFFER(버퍼) : 엘리베이터나 기계 장비에서 충격을 흡수하는 완충장치

### 📢참고사항
- MGD 사 북미용 버퍼오일 단종에 따라 정일산기 M3 버퍼오일로 변경됨(2025.11.27)
- 18300080G0600 → 18300064G0300 (150mpm)

