## 1. HealthKit 연동
- 기존에 서베이했던 내용 적용
## 2. drink || reset 메소드 시 헬스킷 적용
## 3. 달마다 가져오기
- datePicker 이용해서 달마다의 수분량을 가져와 리스트로 보여주기


![](F-lab/10주차/Pasted%20image%2020240801211145.png)

# HistoryView
- datePicker: 연월일  고르는 데이트피커
- List View리스트로 해당 달의 일별 수분 리스트 보여줌
# HistoryViewModel
- pickedYearMonth: 선택한 연월일
- histories: [History] 타입의 일별 수분 배열
# History
- date
- mililiter
# HistoryRepository
- HealthKitService를 통해 월별 수분량 가져와서 [History]으로 반환
# HealthKitService
- predicate를 이용하여 월별 수분량 가져오기
- + DrinkView에서 수분량 저장을 위해 setWater 메소드 추가(reset도)

# Class diagram
![](F-lab/10주차/Pasted%20image%2020240801212045.png)

## Sequence diagram
![](F-lab/10주차/Pasted%20image%2020240801212105.png)