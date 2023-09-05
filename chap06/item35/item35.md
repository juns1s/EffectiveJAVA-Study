# ordinal 메서드 대신 인스턴스 필드를 사용하라


열거 타입 상수는 하나의 정수값에 대응되며, 모든 열거 타입은 해당 상수가 몇 번째 위치인지 반환하는 `ordinal()` 메서드를 제공한다.

- ordinal을 잘못 사용한 예

```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET,
    SEXTET, SEPTER, OCTET, DOUBLE_QUARTET,
    NONET, DECTET, TRIPLE_QUARTET;

    public int numberOfMusicians() { return ordinal() + 1; }
```

하지만 해당 코드는 상수 선언 순서를 바꾸는 순간 오작동하며, 이미 사용 중인 정수와 값이 같은 상수는 추가할 방법이 없다. 즉, 동작은 하지만 유지보수하기가 힘들다.

### 해결책

**열거 타입 상수에 연결된 값은, ordinal 메서드로 얻지 말고 인스턴스 필드에 저장하자.**

- 인스턴스 필드에 정수 데이터를 저장하는 열거 타입

```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

> 핵심 정리
> 
> 
> ordinal 메서드는, EnumSet과 EnumMap과 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었다. 따라서, 이런 용도가 아니라면 열거 타입 상수에 연결된 값은 ordinal 메서드를 절대 사용하지 말자.
>