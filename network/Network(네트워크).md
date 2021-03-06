# 2. Network(네트워크)


## 1. GET/ POST

### GET


HTTP 통신 방식으로 데이터가 HTTP Request Message 헤더부분에 URL에 담겨서 보내지게 된다. 

URL에 데이터가 같이 노출되기 때문에 보안에 민감한 데이터는 Get방식을 이용해서 통신하면 안된다.

또한 URL길이의 제한만큼 데이터를 보낼 수 있다. 

인터넷 익스플로러의 최대 URL 길이는 2,083자다. 다른 브라우저들 크롬, 파이어폭스, 사파리, 엣지, 오페라 모두 65,536자 이상의 URL을 사용할 수 있다.

**그러나 HTTP 자체에서는 URI의 길이에 대한 규약을 두지 않는다.** 

HTTP 규약은 URI의 길이에 대한 어떠한 사전 제한도 두지 않는다. 서버는 반드시 자신이 제 공하는 어떠한 자원의 URI도 처리할 수 있어야 하며 이러한 URI를 생성할 수 있는 GET에 기초한 폼을 (GET-based forms) 제공한다면 무제한 길이의 URI를 처리할 수 있어야만 한다. 서버는 URI의 길이가 자신의 처리할 수 있는 (10.4.15 절 참조) 것보다 긴 경우 414 (Request-URI Too Long)를 응답으로서 돌려주어야 한다.

GET방식은 브라우저에 caching 할 수 있다. 그래서 기존에 Caching 되어 있는 데이터가 나올 수 있다. 

### POST


HTTP 통신 방식으로 데이터가 HTTP Request Message Body부분에 담겨서 보내지게 된다. 

보안 면에서 살짝 나아보이지만 암호화를 하지 않는 이상 GET/POST 고만고만하다.

사용의 차이는 GET은 데이터를 가져올 때 사용되고 POST 데이터를 추가하거나, 변경할 때 변경/추가할 데이터를 Body에 넣어서 보내게 된다.

 

## 2. TCP/ UDP의 비교

### TCP(Transmission Control Protocol)


**신뢰성과 순차성**

데이터를 바이트의 나열로 전달하는데 효율성을 여러 바이트를 세그먼트로 묶어서 전송한다. 

전이중이며(full-duplex), 점대점(point to point) 방식이다.

전이중이란 양방향 전송이 동시에 가능하다는 의미이고, 점대점이란 정확히 2개의 종단점을 가지고 있음을 의미한다. 

그래서 멀티 캐스팅, 브로드 캐스팅이 되지 않는다. 

3-way handshake를 통해서 연결 설정이 이루어진다. 

### UDP (User Datagram Protocol)


**비신뢰성, 비연결성**

UDP는 흐름제어, 오류제어 손상된 세그먼트에 대해서 재전송을 하지 않는다. 

오버헤드가 작은 아주 단순한 프로토콜에 이용된다. 

연결설정과 해제가 따로 필요하지 않다. 

UDP 사용예: DNS서버로 host name을 포함한 udp 패킷을 주고 받는다.



## 3. TCP  3-way handshake

TCP는 신뢰성을 가진 프로토콜이기 때문에 3-way handshake를 통해서 연결 설정을 한다. 

A에서 B로 연결 설정을 한다고 할 때

1. 처음에 A에서 헤더 내에 syn flag를 set해서 B로 보낸다. 

2. B는 잘 받았으면 syn과 ack flag를 set해서 다시 A로 보낸다.

3. 그 후 A에서는 ack를 set해서 B로 보내면 연결 설정이 된다. 

    

    ![3-way handshake](https://t1.daumcdn.net/cfile/tistory/225A964D52F1BB6917)

    

## 4. TCP 4-way handshake

TCP는 연결 설정을 했으면 연결 해제도 해야 한다. 연결 해제는 4-way handshake를 통해서 이루어진다. 

A에서 B로 연결 해제를 한다고 할 때

1. A에서 fin을 set해서 B로 보낸다. 
2. B에서 A로 ack를 set해서 보낸다. 그리고 자신의 통신이 끝날 때까지 기다린다.
3. B로 들어오는 통신이 끝난 후 B에서 A로 fin을 set해서 보낸다.
4. A에서 B로 ack를 set해서 보낸다.

이렇게 되면 연결이 해제된다. 

만약 B에서 FIN 전에 보낸 패킷이 FIN패킷보다 늦게 도착한 경우는 어떻게 될까?

이럴 때를 대비해서 A는 FIN을 수신하더라도 일정시간(default 240초)동안 세션을 남겨둔다. 이과정을 time-wait이라 한다.

![4-way handshake](https://t1.daumcdn.net/cfile/tistory/2152353F52F1C02835)



reference

https://mindnet.tistory.com/entry/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-22%ED%8E%B8-TCP-3-WayHandshake-4-WayHandshake

