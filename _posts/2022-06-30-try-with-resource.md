---
title: "자바 자원 해제(close) 간단하게 하는 방법"
excerpt_separator: "<!--more-->"
categories:
- development

tags:
- Java
- try catch
- close
- try-with-resource
---

오늘은 자바에서 파일 등을 읽을때 자원 해제(close)를 해줘야 하는데, 매번 finally 절에 일일히 기입할 필요 없이, 간편하게 해주는 코드를 살펴 보도록 하겠습니다.

<!--more-->

오늘 설명할 구문은 Java 7 버전에서 업데이트된 내용으로 얼핏 이런 기능이 있다고만 알고 있었는데요.

실무에서 이번에 처음 사용하게 되면서 다른 분들도 필요하시지 않을까 싶어서 포스팅을 남기게 되었습니다.

### Java 7 이전 자원 해제 방법

보통 자원 관련해서 try catch 문으로 감싸서 사용하면서 finally 구문을 통해 사용한 자원을 해제 해주는 코드들을 직접 기입 해주었습니다.

```java
public static void main(String args[]) {
    FileInputStream is = null;
    BufferedInputStream bis = null;
    try {
        is = new FileInputStream("file.txt");
        bis = new BufferedInputStream(is);
        int data = -1;
        while((data = bis.read()) != -1){
            System.out.print((char) data);
        }
    } catch (IOException e) {
	    e.printStackTrace();
	} finally {
        // close resources
        if (is != null) is.close();
        if (bis != null) bis.close();
    }
}
```

### Java 7 이후 자원 해제 방법
얼핏 보면 차이점을 잘 모를 수 있지만, try (...) 구문 내에서 객체 선언 및 자원 할당을 하게 되면 try 문의 끝날때 자동으로 자원을 해제시켜 준다고 합니다.

개발자들이 직접 finally 문과 close() 메소드를 기입해줄 필요가 없습니다.

```java
public static void main(String args[]) {
    try (
        FileInputStream is = new FileInputStream("file.txt");
        BufferedInputStream bis = new BufferedInputStream(is)
    ) {
        int data = -1;
        while ((data = bis.read()) != -1) {
            System.out.print((char) data);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

이런 기능은 AutoCloseable을 구현한 객체에만 해당이 되며, 해당 객체가 try-with-resources에서 자동으로 close가 호출해주기 때문에 가능하다고 합니다.

자세한 내용은 참고자료의 블로그 글을 참고 하시기 바랍니다.

### 마무리
개발자에게 코드 한줄이라도 덜 쓸 수 있게 해주는 건 참 중요하죠 👍🏻

그럼 이만. 🥕👋🏼🖐🏼

### 참고자료
[https://codechacha.com/ko/java-try-with-resources/](https://codechacha.com/ko/java-try-with-resources/)