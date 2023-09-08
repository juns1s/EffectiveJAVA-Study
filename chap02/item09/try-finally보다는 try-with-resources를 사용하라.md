# 아이템9. try-finally보다는 try-with-resources를 사용하라

InputStream, OutputStream, java.sql.Connection등 close를 해야하는 자원은 try-with-resources를 사용하자.

## 기존 try-finally

```java
static void copy(String src, String dst) throws IOException {
  BufferedReader in = new BufferedReader(new FileReader(src));
  try{
      OutputStream out = new FileOutputStream(dst);
      try{
          byte [] buf = new byte[BUFFER_SIZE];
          int n;
          while((n=in.read(buf))>=0)
              out.write(buf, 0, n);
      }finally {
          out.close();
      }
  }finally {
      in.close();
  }
}
```

- 자원을 여러개 사용 시 중첩 try-finally문이 발생하여 복잡하다.

## try-with-resources

```java
static void copy(String src, String dst) throws IOException {
  try(InputStream in = new FileInputStream(src);
      OutputStream out = new FileOutputStream(dst)){
      byte [] buf = new byte[BUFFER_SIZE];
      int n;
      while ((n=in.read(buf))>=0)
          out.write(buf,0,n);
  }catch(Exception e){
		//예외 처리	
	}
}
```

- try(자원 객체)문이 끝나면 해당 자원은 반환된다.
- try-finally에 비해 깔끔하다.
- catch를 통해 특정 예외에 대해 처리가 가능하다.
    - 발생한 예외가 가려저도 suppressed꼬리표에 달고 출력된다.

## 참고
[Try-with-resources를 이용한 자원해제 처리 | Integerous DevLog](https://ryan-han.com/post/java/try_with_resources/)