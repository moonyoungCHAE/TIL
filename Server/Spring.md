## WebClient and RestTemplate
* RestTemplate : Synchronous + blcoking : response가 올 때까지 기다린다.
* WebClient: response가 올 때까지 기다리지 않고, response가 오면 알려준다.
> WebClient is an interface representing the main entry point for performing web requests.

## Blocking / Non-Blocking
Spring: Blocking + Synchronous

Asynchronous(비동기)
* 어떤 기능이 완료될 때까지 기다리지 않는 방식

Non-Blocking + Synchronous:
Non-Blocking I/O + Asynchronous
* 입출력은 시작
* 완료되지 않은 상태에서 다른 처맂 ㅏㄱ업 진행 가능
<-> Blocking: 완료될 때까지 기다린다.

### exchange and retrieve
* return 타입이 다르다.
> They differ in return types; the exchange method provides a ClientResponse along with its status, headers while the retrieve method is the shortest path to fetching a body directly:

``` JAVA
String response2 = request1.exchange()
  .block()
  .bodyToMono(String.class)
  .block();
String response3 = request2
  .retrieve()
  .bodyToMono(String.class)
  .block();
  ```
  
  ### bodyToMono
  * 에러 throw한다.
>   Pay attention to the bodyToMono method, which will throw a WebClientException if the status code is 4xx (client error) or 5xx (Server error). We used the block method on Monos to subscribe and retrieve an actual data which was sent with the response.

* Mono Type의 객체를 추출한다.
The method bodyToMono() extracts the body to a Mono instance. The method Mono.block() subscribes to this Mono instance and blocks until the response is received.