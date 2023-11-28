쓸모 있는 API에는 잘 작성된 문서도 필요하다.

자바에서는 자바독(Javadoc)이라는 유틸리티를 지원하고 있는데, 소스코드 파일에서 문서화 주석이라는 특수한 형태로 기술된 설명을 추려 API 문서로 변환해준다.

### 문서화 주석을 다는 방법

1) 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다. 단, 기본 생성자에는 주석을 달 방법이 없으니 공개 클래스는 절대 기본 생성자를 사용하면 안된다.

2) 메소드용 문서화 주석에서는 해당 메서드와 클라이언트 사이의 규약을 명료하게 기술해야 한다.

- 메서드의 동작 과정이 아닌 아닌 무엇을 하는지 기술
- 클라이언트가 해당 메서드를 호출하기 위한 전제조건 나열
- 메서드가 수행된 후에 만족해야 하는 사후조건 나열
- 사후조건으로 명확하게 나타나지 않지만 시스템 상태에 어떠한 변화를 가져오는 부작용 기술(ex) 백그라운드 슬드)

###  메서드 주석

메서드의 계약을 완벽히 기술하려면, 다음의 규칙을 따라 작성하면 된다.

> 모든 매개변수에 @param 태그반환 타입이 void가 아니라면 @return 태그발생할 가능성이 있는 모든 예외(검사/비검사)에 @throws 태그
> 

```java
   /**
     * Returns the element at the specified position in this list.  // 요약 설명
     *
     * <p>This method is <i>not</i> guaranteed to run in constant
     * time. In some implementations it may run in time proportional
     * to the element position.
     *
     * @param  index index of element to return; must be
     *         non-negative and less than the size of this list
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         ({@code index < 0 || index >= this.size()})
     */
    E get(int index) {
        return null;
    }
```

`@code` 는 태그로 감싼 내용을 코드용 폰트로 렌더링하며, 감싼 내용에 포함된 HTML 요소나 다른 자바독 태그를 무시하는 역할을 한다. 여러줄로 된 코드 예시를 넣으려면 `<pre>{@code ...}</pre>` 형태로 쓰면 된다.

###  상속용 클래스 문서화 주석

클래스를 상속용으로 설계할 때는, `@implSpec` 을 사용해 **자기사용 패턴**을 문서로 남겨 메서드를 올바르게 재정의하는 방법을 알려줘야 한다.(*아이템 15*)

>  @impleSpec
> 
> 
> 해당 메서드와 하위 클래스 사이의 계약을 설명
> 
> 하위 클래스들이 그 메서드를 상속하거나 super 키워드를 이용해 호출할 때 그 메서드가 어떻게 동작하는지 명확히 인지하고 사용할 수 있도록 도와주는 역할
> 

```java
  /**
     * Returns true if this collection is empty. // 요약 설명
     *
     * @implSpec This implementation returns {@code this.size() == 0}.
     *
     * @return true if this collection is empty
     */
    public boolean isEmpty() {
        return false;
    }
```

참고로, 자바 11 이전까지는 자바독 명령줄에서 다음의 스위치를 켜주지 않으면 해당 태그를 무시해버린다.

```java
-tag "implSpec:a:Implementation Requirements:"`
```

### 요약 설명

요약 설명(summary description)이란, 각 문서화 주석의 첫 번째 문장을 말한다.

```java
Collection.size() : 이 컬렉션 안의 원소 개수를 반환한다. // 동사
Instant : 타임라인상의 특정 순간(지점)   // 명사절

```

중요한 점은 반드시 대상의 기능을 고유하게 기술해야 한다는 것이다. 즉, 한 클래스 안에서 요약 설명이 똑같은 멤버 혹은 생성자가 둘 이상이면 안되므로 특히 다중정의된 메서드에서 조심하자.

또한 요약 설명은 **마침표**를 기준으로 구분되기 때문에, `{@literal}` 로 감싸주어 의도치 않은 마침표로 인한 혼동을 해결할 수 있다.

>  @literal
> 
> 
> 문서화 주석에 HTML이나 자바독 메타문자를 포함시키고 싶을 때 사용 가능
> 
> ex) `<`, `>`, `&`
> 

▶️ 문서화 주석 첫 '문장'에 마침표가 있을 때 요약 설명 처리

```java
 /**
 * A suspect, such as Colonel Mustard or {@literal Mrs. Peacock}.
 */
  public enum Suspect {...}
```

▶️ @Summary 이용: 자바 10 이후

```java
  /**
  *{@summary A suspect, such as Colonel Mustard or Mrs. Peacock.}
  */
  public enum Suspect {...}
```

### 🔗 제네릭 타입과 문서화 주석

제네릭 타입이나 제네릭 메서드를 문서화 할때는, 모든 **타입 매개변수**에 주석을 달아야 한다.

```java
/**
* An object that maps keys to values. A map cannot contain duplicate keys; each key can map to at most one value.
*
* (Remainder omitted)
*
* @param <K> the type of keys maintained by this map
* @param <V> the type of mapped values
*/
public interface Map<K, V> {...}
```

###  열거 타입과 문서화 주석

열거 타입을 문서화할 때는 **상수**들에도 주석을 달아야 한다.

```java
/**
* An instrument section of a symphony orchestra.
*/
public enum OrchestraSection {
    /** Woodwinds, such as flute, clarinet, and oboe. */
    WOODWIND,

    /** Brass instruments, such as french horn and trumpet. */
    BRASS,

    /** Percussion instruments, such as timpani and cymbals. */
    PERCUSSION,

   /** Stringed instruments, such as violin and cello. */
    STRING;
}
```

###  애너테이션과 문서화 주석

애너테이션 타입을 문서화할 때는 멤버들에도 모두 주석을 달아야 한다.

애너테이션 타입의 요약 설명에서는, 이 애너테이션을 단다는 것이 어떤 의미인지를 설명하며녀 된다.

```java
/**
* Indicates that the annotated method is a test method that
* must throw the designated exception to pass.
*/
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    /**
    * The exception that the annotated test method must throw
    * in order to pass. (The test is permitted to throw any
    * subtype of the type described by this class object.)
    */
    Class<? extends Throwable> value();
}
```

###  문서화 주석의 검색 기능

자바 9 이후부터는, 자바독이 생성한 HTML 문서에 검색(색인) 기능이 추가되었다. 클래스, 메서드, 필드 같은 API 요소의 색인은 자동으로 만들어지며 원한다면 `{@Index}` 태그를 사용해 API에서 중요한 용어를 추가로 색인화할 수 있다.

```java
/**
*This method complies with the {@index IEEE 754} standard.
**/
public void fragment2() {...}
```

>  핵심 정리
> 
> 
> 문서화 주석은 API를 문서화하는 가장 훌륭하고 효과적인 방법이다. 공개 API라면 모두 설명을 달아야 하며, 표준 규약을 일관되게 지키자. 문서화 주석에 임의의 HTML 태그를 사용할 수 있음을 기억하라. 단, HTML 메타문자는 특별하게 취급해야 한다.
>