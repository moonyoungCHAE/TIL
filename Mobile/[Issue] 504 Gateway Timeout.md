> The **504 Gateway Timeout error** is an HTTP status code that means that one server didn't receive a timely response from another server that it was accessing while attempting to load the web page or fill another request by the browser.



nginx와 upstream이 통신할 때, nginx에서 설정한 시간 제한보다 오래 걸리면 504 timeout 문제가 발생한다.



![img](https://blog.lael.be/wp-content/uploads/2019/09/proxy.png)

NGINX : Reverse Proxy Server

* 외부 프로그램과 통신하여 결과를 출력하는 프로그램

* 이 때 외부 프로그램: Upstreams (node.js, tomcat etc)

  * Proxy Server: 

    * Web Caches: origin server의 overhead를 줄여주는 역할

    * Client (Browser)로부터  Proxy Server가 Request를 받는다.

    * 캐시 처리된 요청 결과는 바로 Client로 보낸다.

    * 캐시 처리되지 않은 요청 결과는 Origin Server에 Request한 후,

      그 결과를 Client로 보내준다.

      * Client 입장에서는 Server역할

      * Server 입장에서는 Client 역할