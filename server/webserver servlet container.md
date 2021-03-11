# 웹서버와 서블릿 컨테이너의 통신

Created: Mar 7, 2021 11:28 PM
Last Edited Time: Mar 8, 2021 1:17 AM

우리가 잘 아는 웹서버는 아파치, Nginx 등이 있다. 

또한 서블릿 컨테이너(WAS)에는 톰캣, Jetty, WebLogic, Jeus 등이 있다. 

보통 웹서버는 정적 리소스를 처리할 때 사용한다고 알려져 있고, 서블릿 컨테이너는 동적 리소스를 처리한다고 알려져 있다. 웹서버는 또한 로드밸런싱을 통해 WAS인스턴스 여러 대에 분산 처리가 가능하다.

그렇다면 WebServer와 WAS는 어떻게 통신하는 걸까?

방법은 AJP 프로토콜과 reverse proxy를 사용할 수 있다. 

AJP 프로토콜은 아파치 재단에서 만든 프로토콜로, 아파치 웹서버와  JAVA EE 서버간의 연결을 위한 프로토콜이다. 

Reverse Proxy는 아파치와 자바에 국한되지 않고 모든 서버에서 사용가능하다. 

오늘은 AJP 프로토콜로 통신하는 법에 대해 설명하려고 한다.

추후에 Reverse Proxy도 첨부하겠다. 

## AJP 프로토콜

AJP(Apache Jserv Protocol)는 웹서버로 들어오는 요청을 웹서버 뒤에 있는 애플리케이션 서버로 위임할 수 있는 바이너리 프로토콜이다.

애플리케이션 서버로 핑을 할 수 있는 모니터링 기능을 지원하고 일반적으로 하나 이상의 프론트엔드 웹서버가 하나 이상의 애플리케이션 서버에 로드 밸런싱되는 배포에서 AJP를 사용한다. 

세션은 라우팅 메커니즘을 사용하여 올바른 애플리케이션 서버로 리디렉션 된다.

AJP는 HTTP 포워딩 용도로만 사용되기 때문에 안전하지 않다.

### 동작 방식

1. 아파치 웹서버의 httpd.conf에 톰캣 연동을 위한 설정을 추가하고 톰캣에서 처리할 요청을 지정한다.
2. 사용자 브라우저가 서버에 접속한다(80번포트)
3. 아파치 웹서버는 사용자의 요청이 톰캣에서 처리하도록 되어있는 요청인지 확인 후, 톰캣에서 처리해야하는 경우 톰캣의 AJP포트로 요청을 위임한다.(보통 8000번포트)
4. 톰캣는 웹서버로 부터 요청을 받아 처리한 후, 처리 결과를 아파치 웹서버에 돌려준다.
5. 아파치 웹서버는 톰캣으로부터 받은 처리 결과를 사용자에게 전달한다.

### mod_jk

mod_jk는 웹서버와 톰캣을 AJP프로토콜을 통해 연결하기 위한 아파치 모듈이다.

아파치를 설치한 후 httpd.conf 파일에 아래의 내용들을 넣어준다. 

```java
LoadModule jk_module modules/mod_jk.so 
# mod_jk 모듈 로딩

JkWordersFile conf/workers.properties 
# jk 모듈 실행을 위해 선언된 속성 값 (실행 대상 이름 등)

JkLogFile logs/mod_jk.log 
# mod_jk 모듈 실행중 발생하는 로그를 기록하는 파일 선언

JkLogLevel INFO 
# INFO 이상의 레벨에 대해 로깅

JkMount /* worker1  
# 도메인 / url로 접근한 경우 worker1 로 재전송
```

모듈 실행을 위한 설정값이 담겨있는 [worker.properties](http://worker.properties) 파일은 아래와 같다.

```java
worker.list=worker1 
# 실행할 노드 서버 이름

worker.worker1.port=8009 
# Tomcat 에서 ajp 요청받을 포트번호

worker.worker1.host=127.0.0.1 
# 톰캣 호스트 ip

worker.worker1.type=ajk13 
# 사용 중인 프로토콜
```

출처

[https://ehdvudee.tistory.com/20](https://ehdvudee.tistory.com/20)

[https://en.wikipedia.org/wiki/Apache_JServ_Protocol](https://en.wikipedia.org/wiki/Apache_JServ_Protocol)

[https://noobnim.tistory.com/26](https://noobnim.tistory.com/26)

[https://has3ong.github.io/ajpprotocol/](https://has3ong.github.io/ajpprotocol/)