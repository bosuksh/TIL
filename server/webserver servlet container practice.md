# 웹서버와 서블릿 통신 실습

Created: Mar 9, 2021 10:46 PM
Last Edited Time: Mar 10, 2021 1:41 AM

> 환경: GCP Debian-10-buster

## 아파치 설치

```bash
sudo apt update && sudo apt -y install apache2
```

아파치를 설치하면 

확인방법은 http://{IP주소}:80 으로 들어가면 아파치 기본 페이지가 뜬다. 

![%E1%84%8B%E1%85%B0%E1%86%B8%E1%84%89%E1%85%A5%E1%84%87%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%A5%E1%84%87%E1%85%B3%E1%86%AF%E1%84%85%E1%85%B5%E1%86%BA%20%E1%84%90%E1%85%A9%E1%86%BC%E1%84%89%E1%85%B5%E1%86%AB%20%E1%84%89%E1%85%B5%E1%86%AF%E1%84%89%E1%85%B3%E1%86%B8%207dc8c521cbd44fbfa263e8f51e4cbd51/Untitled.png](%E1%84%8B%E1%85%B0%E1%86%B8%E1%84%89%E1%85%A5%E1%84%87%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%A5%E1%84%87%E1%85%B3%E1%86%AF%E1%84%85%E1%85%B5%E1%86%BA%20%E1%84%90%E1%85%A9%E1%86%BC%E1%84%89%E1%85%B5%E1%86%AB%20%E1%84%89%E1%85%B5%E1%86%AF%E1%84%89%E1%85%B3%E1%86%B8%207dc8c521cbd44fbfa263e8f51e4cbd51/Untitled.png)

## 아파치와 내장 톰캣 연동

아파치와 내장 톰캣을 연동하기 위해서는 mod_jk, mod_proxy 방식이 있는데 일단 mod_jk방식부터 설명하겠다. 

### mod_jk 설치

mod_jk는 아파치와 톰캣을 연동하기 위한 모듈로 AJP 통신을 가능하게 해준다. 

또한 Load Balancing과 Fail Over을 통해서 안정적인 운영을 할 수 있도록 해준다.

위 환경에서 설치 방법은 아래의 커맨드를 통해 할 수 있다. 

```bash
#mod_jk 설치
sudo apt-get install libapache2-mod-jk

#mod_jk 활성화
sudo a2enmod
```

### [workers.properties](http://workers.properties)  생성

이후 workers.properties를 **$APACHE_HOME/conf**에 생성한다. 

*현재 서버에서는 /etc/apache2/conf/ 에 생성하도록 한다.*

**workers.properties**의 내용은 아래와 같다.

여기서는 두개의 톰캣을 띄워서 연결한다고 가정한다. 두개를 로드밸런싱 해줄 수 있다.

```bash
worker.list=worker1

worker.list=worker1,worker2,load_balance //톰캣 두개 사용시 worker1,2를 넣고 각각 설정을 해주면 된다.

worker.worker1.port=18001 //AJP포트는 밑에서 내장 톰캣에서 세팅을 해줄 것이다.

worker.worker1.host=127.0.0.1

worker.worker1.type=ajp13

worker.worker1.lbfactor=1

worker.worker2.port=18002 //두개 이상의 tomcat을 사용하는 경우 두번째 worker 설정

worker.worker2.host=127.0.0.1

worker.worker2.type=ajp13

worker.worker2.lbfactor=1

worker.load_balance.type=lb

worker.load_balance.balanced_workers=worker1,worker2

```

### jk.conf 파일 수정

/etc/apache2/mods-available/jk.conf

방금 생성해준 workers.properties를 가리키도록 수정을 해준다.

```yaml
<IfModule jk_module>

    # We need a workers file exactly once

    # and in the global server

    #JkWorkersFile /etc/libapache2-mod-jk/workers.properties

    JkWorkersFile /etc/apache2/workers.properties

```

### 000-default.conf 파일 수정

/etc/apache2/sites-available/000-default.conf

만들어준 worker들과 url을 매핑해준다. 

우리는 로드밸런싱을 통해 두개의 인스턴스에 나눠서 띄울거기 때문에 load_balance를 설정한다.

```yaml
<VirtualHost *:80>

  ServerAdmin webmaster@localhost //도메인이 연동되어 있으면 도메인으로 수정한다
	#DocumentRoot /var/www/html
  ServerName  localhost
  ServerAlias localhost

  JkMount /* load_balance

</VirtualHost>

```

## SpringBoot 설정

스프링부트 프로젝트를 하나 생성한다. 여기서는 [김영한님의 스프링부트와 JPA 활용](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1)의 프로젝트를 이용했다.

### application.yml 설정

yaml파일이나 properties의 값을 사용할 것이다.

```yaml
tomcat:
  ajp:
    protocol: AJP/1.3
    port: 8009
    enabled: true
```

### Tomcat 설정을 빈으로 등록

yaml파일에서 설정한 값을 사용해서 내장 톰캣을 설정한다.

```java
@Configuration
public class TomcatConfig {
  @Value("${tomcat.ajp.protocol}")
  String ajpProtocol;

  @Value("${tomcat.ajp.port}")
  int ajpPort;

  @Value("${tomcat.ajp.enabled}")
  boolean tomcatAjpEnabled;

  @Bean
  public ServletWebServerFactory servletContainer() {
    TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
    tomcat.addAdditionalTomcatConnectors(createAjpConnector());
    return tomcat;
  }

  private Connector createAjpConnector() {
    Connector ajpConnector = new Connector(ajpProtocol);
    ajpConnector.setPort(ajpPort);
    ajpConnector.setSecure(false);
    ajpConnector.setAllowTrace(false);
    ajpConnector.setScheme("http");
    ((AbstractAjpProtocol) ajpConnector.getProtocolHandler()).setSecretRequired(false);
    return ajpConnector;
  }
}
```

이렇게 설정이 완료 되었으면 빌들를 통해서 jar파일을 생성한다. 

이 프로젝트는 gradle을 사용했으며 아래의 명령어를 통해 자바 애플리케이션을 실행했다.

```java
./gradlew build -x test && java -jar build/lib/jpa-shop-0.0.1-SNAPSHOT.jar
```

이제 8080 포트를 통해 접속하면 자바 애플리케이션을 볼 수 있다. 

![%E1%84%8B%E1%85%B0%E1%86%B8%E1%84%89%E1%85%A5%E1%84%87%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%A5%E1%84%87%E1%85%B3%E1%86%AF%E1%84%85%E1%85%B5%E1%86%BA%20%E1%84%90%E1%85%A9%E1%86%BC%E1%84%89%E1%85%B5%E1%86%AB%20%E1%84%89%E1%85%B5%E1%86%AF%E1%84%89%E1%85%B3%E1%86%B8%207dc8c521cbd44fbfa263e8f51e4cbd51/Untitled%201.png](%E1%84%8B%E1%85%B0%E1%86%B8%E1%84%89%E1%85%A5%E1%84%87%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%A5%E1%84%87%E1%85%B3%E1%86%AF%E1%84%85%E1%85%B5%E1%86%BA%20%E1%84%90%E1%85%A9%E1%86%BC%E1%84%89%E1%85%B5%E1%86%AB%20%E1%84%89%E1%85%B5%E1%86%AF%E1%84%89%E1%85%B3%E1%86%B8%207dc8c521cbd44fbfa263e8f51e4cbd51/Untitled%201.png)

### 아파치 재시작

아파치를 재시작하면 80번 포트에서 자바 애플리케이션으로 접속 되는걸 볼 수 있다.

```bash
sudo service apache2 start
```

![%E1%84%8B%E1%85%B0%E1%86%B8%E1%84%89%E1%85%A5%E1%84%87%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%A5%E1%84%87%E1%85%B3%E1%86%AF%E1%84%85%E1%85%B5%E1%86%BA%20%E1%84%90%E1%85%A9%E1%86%BC%E1%84%89%E1%85%B5%E1%86%AB%20%E1%84%89%E1%85%B5%E1%86%AF%E1%84%89%E1%85%B3%E1%86%B8%207dc8c521cbd44fbfa263e8f51e4cbd51/Untitled%202.png](%E1%84%8B%E1%85%B0%E1%86%B8%E1%84%89%E1%85%A5%E1%84%87%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%A5%E1%84%87%E1%85%B3%E1%86%AF%E1%84%85%E1%85%B5%E1%86%BA%20%E1%84%90%E1%85%A9%E1%86%BC%E1%84%89%E1%85%B5%E1%86%AB%20%E1%84%89%E1%85%B5%E1%86%AF%E1%84%89%E1%85%B3%E1%86%B8%207dc8c521cbd44fbfa263e8f51e4cbd51/Untitled%202.png)

---

추가적으로  자바 애플리케이션을 두개를 띄우기 위해서

application.yml을 아래와 같이 수정한다.

```yaml
	server:
	  port: 8081
	
	tomcat:
	  ajp:
	    protocol: AJP/1.3
	    port: 8010
		    enabled: true
```

서버가 뜨는 포트는 8081번 포트이고 AJP 통신을 위한 포트를 8010을 사용한다.

이 후 빌드와 실행을 하면 자바 애플리케이션이 하나 더 뜨게 된다. 

그리고 80번포트에 접속하면 마찬가지로 자바 애플리케이션이 보이는데 이때 만약 첫번째 띄운 인스턴스를 kill 하더라도 자바 애플리케이션은 계속 유지되는걸 볼 수 있다.