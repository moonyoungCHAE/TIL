### type '_Type' is not a subtype of type 'Widget'
날짜: 2020-01-13
해결 방법:
1. 에러가 정확히 발생하는 부분을 찾는다. (2의 A 코드)
2. 왜 에러가 발생하는지 이유를 찾는다.  (widget.value 이런 식으로 접근해서)
3. 이런 저런 시도를 하며 에러를 해결했다. (다른 변수를 만들어본다.)

상황:
1. Stateful Widget 원소 보내기 -> List 보내기로 수정
-- 1) 원소 하나의 정보만 사용할 때
-- 2) List 전체 정보로 사용할 때 *(추가해야 되는 것)*
두가지 상황에 따라 코드가 달라졌다. 그래서 일단 List로 받아서 1)의 경우에는 list[0] List의 첫번째 원소에만 접근해서 사용하려고 함
그런데 **List로 받아오는 것으로 바꾸었는데, 원소가 2개이상이면 에러가 발생**
``` Dart
class TimeRow extends StatefulWidget {  
  final List<Appointment> appointments; 
  
  const TimeRow({Key key, this.appointments}) : super(key: key);  
  
  @override  
  _TimeRowState createState() => _TimeRowState();  
} 
````

``` Dart
// List로 보내면 에러
TimeRow(  
  appointments: _eventList ?? []
)

// List의 개별 원소로 보내면 에러 X
_eventList  
  .map((e) => TimeRow(  
  appointments: [e]
)
```

2. Stateful Widget의 Child에서는 받아온 변수를 ```widget.appointments.length``` 이런 식으로 사용하는데, 이 과정에서 에러가 발생함.
이상한 점은 하나의 원소만 보냈을 때와 여러 개 원소를 보냈을 때 모두 DataType은 동일하게 List로 나왔다.

3. 
``` Dart
class _TimeRowState extends State<TimeRow> {  
  DateTime _startTime;  
  DateTime _endTime;  
  String _format = 'HH:mm';  
  
  @override  
  Widget build(BuildContext context) {  
    return Column(  
      children: <Widget>[  
	      // 에러가 나는 부분 @@@@@@@ A @@@@@@@
        if (widget.appointments.length == 1)
	      // Do Something (Widget 그리기)
        _buildTimeRow(context),  
      ],  
    );  
  }  
  
  // 여기도 화면 구성하는 부분
  Widget _buildTimeRow(BuildContext context) {  
  // @@@@@@@ B @@@@@@@
    return Padding(
  ```
  Stateful Widget의 Child에서는 widget.xxx 이렇게 사용할 수 밖에 없는데, A에서는 이런 접근 방식으로 에러가 나고, B에서는 에러가 안남 (B에도 ```widget.appointments.length``` 사용한 부분 있었음)

4. 에러가 발생하는 부분을 다른 변수로 만들어줌