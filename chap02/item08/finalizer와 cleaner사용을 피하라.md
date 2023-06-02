# 아이템8. finalizer와 cleaner 사용을 피하라
finalizer와 cleaner 모두 예측할 수 없고 일반적으로 불필요하다. 그냥 쓰지 말자.

C++의 소멸자와 다른개념이며 쓰지 말자.

### finalizer, cleaner의 문제점

1. 수행 여부, 수행 시점 보장X
2. finalizer동작 과정에서발생한 예외는 무시된다.
3. 낮은 성능
4. finalizer 공격에 취약

### finalizer 공격

: finalizer에 코드가 존재하며 해당 코드가 객체를 참조하게 되면 참조가 발생하므로 가비지컬렉팅 대상에서 제외된다. 하위 객체에서 예외가 발생하는 것을 막아야 한다.

- 해결방법: Java에서는 생성자에 상위객체의 생성된다. 이때 파라미터를 처리하는 과정에서 유효성 검사를 실행하여 finalizer대신 예외를 던진다.

## AutoCloseable로 대체

```java
public class Room implements AutoCloseable{
    private static final Cleaner cleaner = Cleaner.create();
    
    private static class  State implements Runnable{
        int numJunkPiles;

        State(int numJunkPiles){
            this.numJunkPiles = numJunkPiles;
        }

        @Override
        public void run() {
            numJunkPiles = 0;
        }
    }
    
    private final State state;
    
    private final Cleaner.Cleanable cleanable;
    
    public Room(int numJunkPiles){
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }
    
    @Override
    public void close() throws Exception {
        cleanable.clean();;
    }
}
```

- State 인스턴스는 절대 Room 인스턴스를 참조하면 안된다. (순환참조 발생)
    - 따라서 static inner class로 생성한다.

## finalizer와 cleaner의 사용 예시

### 1. 자원 회수에 대한 안전망

사용자가close메소드를 직접 호춣해야 하는 경우 안정망 역할을 한다. 즉시 회수될거란 보장은 없지만 언젠가는 회수될 것이라는 안정망의 역할을 하게 된다.

FileInputStream, FileOutputStream, ThreadPoolExecutor에는 finalizer를 통해 구현되어 있다.

### 2. 네이티브 피어와 연결된 객체

네이티브 객체는 자바에서 생성된 객체와 달리 네이티브 메소드(java 가 아닌 다른 언어로 쓰여진)에 의해 생성된 객체로 가비지컬렉터가 존재를 알지 못한다. 이 객체는 즉시 close를 통해 회ㅅ되어야 한다.

## 참고

[What is a native peer?](https://stackoverflow.com/questions/48260485/what-is-a-native-peer)

[자바에서 이런게 된다고? 네.. 가능합니다. 한번 보시죠!](https://www.youtube.com/watch?v=6kNzL1bl1kI)