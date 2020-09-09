2020-07-29



# Item9. try-finally보다는 try-with-resources를 사용하라.

자바 라이브러리에는 close 메소드를 호출해 직접 닫아줘야 하는 자원이 많다. 예를 들어, InputStream, OutputStream, java.sql.connection 등이 좋은 예다. 

전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 `try-finally` 가 쓰였다. 

```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    }finally {
        br.close();
    }
}
```

### 문제점 1. try-finally가 중첩되면 지저분해질 수 있다.

나쁘지 않지만 자원을 하나 더 사용하게 되면 너무 지저분해질 수 있다. 

```java
static void copy(String src, String dst) throws IOException{
    InputStream in = new FileInputStream(src);
    try {
    OutputStream out  = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while((n = in.read()) >= 0)
                out.write(buf,0,n);
        }finally {
            out.close();
        }
    }finally {
        in.close();
    }
}
```

### 문제점2. finally에서 발생할 경우 에러가 삼켜질 수 있다.

try-finally 문을 제대로 사용한 앞의 두 코드 예제에 조차 미묘한 결점이 있다. 예외는 try블록과 finally블록에서 모두 발생할 수 있는데, 예컨대 기기에 물리적인 문제가 생긴다면 firstLineOfFile 메소드 안의 readLine 메소드가 예외를 던지고 , 같은 이유로 close 메소드도 실패할 것이다. 이런 상황에서 close에서 발생한 두번째 에러가 첫번째에서 발생한 에러를 집어 삼키게 된다. 그래서 스택 추적 내역에 첫번째 에러에 대한 정보가 남지 않아 디버깅이 매우 어렵게 된다.

*그러나, 자바7에서 Throwable에 추가된 getSuppressed 메소드를 이용하면 프로그램 코드를 가져올 수 있다.* 

### 해결방법. `try-with-resource`를 사용한다.

**전제는 이 구조를 사용하기 위해서는 AutoCloseable 인터페이스를 구현해야한다.**

위의 코드들을 `try-with-resources` 로 재 작성한 예다. 

```java
static String firstLineOfFile(String path) throws IOException {
    try(BufferedReader br = new BufferedReader(new FileReader(path));) {
        return br.readLine();
    }
}
```

```java
static void copy(String src, String dst) throws IOException{
    try(InputStream in = new FileInputStream(src);OutputStream out  = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while((n = in.read()) >= 0)
            out.write(buf,0,n);
    }
}
```

`try-with-resources` 버전이 짧고 읽기 수월할 뿐 아니라 문제를 진단하기도 훨씬 좋다. 

보통의 `try-finally`에서처럼 `try-with-resources`에서도 **catch**절을 쓸 수 있다.