### Refresh Indicator

* pull to refresh할 때 사용

* ```onRefresh ``` 함수 필수로 사용해야 한다.

  * ```onRefresh``` 함수는 ```future```를 retrun해야 한다.

    

  * 다음과 같은 2가지 방식으로 사용 가능

  * ```
    onRefresh: () {
    	return Future.valure();
    }
    
    onRefresh: () async {
    	// refresh를 위한 매서드
    }
    
    // Future Builder에서 사용할 경우 Future를 다시 설정해주는 방법이 있다.
    ```

* 위로 pull인지 아래로 pull인지 확인

  * ```
    void _onRefresh(bool up) {
    	if (up) {
    		// pull down
    	} else {
    		// pull up
    	}
    }
    ```



---

Issue

* 사용자가 pull to refresh하면 데이터를 server로부터 받아와 업데이트하려고 했다.
* 이 때 ```onRefresh```함수에서  async - await를 활용해서 데이터를 다시 받아왔다.
* 하지만, 데이터를 받아오는 사이 잠시동안 빨갛게 에러 화면이 나왔다.
  * 데이터를 받아올 때까지는 await해주지만, 그동안 화면에 보여질 위젯이 없기 떄문.
* ```Future Builder```에서 Future 데이터 자체를 업데이트하는 방식으로 해결 !