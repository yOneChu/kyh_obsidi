
---



```
Variant.calcResultValues : 이 부분이 로직에서 CALL하는 PID 찾는 부분이다.
```



| 아래 컨트롤러가 수행하는 PID에 가장 가까운 날짜들 CALL해서 불러오는 거임
- VaultReportController
```
@RequestMapping(value = "/pidSelectExecuteV2", method = RequestMethod.GET)
```