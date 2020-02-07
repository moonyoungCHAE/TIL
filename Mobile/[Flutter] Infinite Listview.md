이거 때문에 ㄹㅇ 며칠을 헤맴..!!!
내가 해야하는 것:  
1. 사용자가 스크롤 맨 아래까지 내리면 그것을 탐지하고  
2. 그에 맞는 서버 데이터를 받아와서  
3. 기존 데이터에 추가한다  

원래 다른 개발자분이 짜놓은 코드에는 future builder로 되어있었는데, future builder에서는 이것을 구현할 수 없다.  
내가 데이터를 갱신해도 그것이 반영이 안되었음..   
(setStatus로 widget를 rebuild하면 맨 처음 리스트 아이템부터 출력되는 문제 발생함)  
why? future builder 의 사용 목적 자체가 데이터를 build 전에 받고, 데이터를 받으면 build하는게 목적이기 때문이다.
이런 상황처럼 데이터를 지속적으로 받아서 추가하는 것에 적합하지 않음

---
The future must have been obtained earlier, e.g. during State.initState, State.didUpdateConfig, or State.didChangeDependencies. It must not be created during the State.build or StatelessWidget.build method call when constructing the FutureBuilder. If the future is created at the same time as the FutureBuilder, then every time the FutureBuilder's parent is rebuilt, the asynchronous task will be restarted.
---

해결 방법:
future builder를 사용하지 않고,
데이터를 처음에 initState에서 await 로 온전히 받은 후에 build 실행하도록 함.

배운 점 및 느낀 점:  
* 관련 reference가 없어서 어려웠다. future builder 같은 그 아이템(?) 자체에 대해 공부를 해야 해결되는 문제.. 영문 docs가 버겁게 느껴지지만 적응하고 있다.
* 비동기 프로그램 방식을 익힐 수 있었다.
* 데이터가 없을 때 예외 사항을 꼭 다 추가해줘야 완성도있는 프로그램을 만들 수 있당.