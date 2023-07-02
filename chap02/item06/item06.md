### 불필요한 객체 생성을 피하라

## 문자열

1. 절대 하면 안되는 것

```java
String s = new String ("bikini");
```

- 실행될 때마다 String 인스턴스를 새로 만드므로 호출이 빈번해지면 성능이 안좋아진다

1. 개선된 버전

```java
String s = "bikini";
```

- 하나의 String 인스턴스를 사용한다
- 같은 virtual machine안에서 같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 사용함을 보장한다.

## Static Factory Method

```java
Boolean bikini1 = Boolean.valueOf("bikini");
Boolean bikini2 = Boolean.valueOf("bikini");
```

- bikini1 == bikini2

## 재사용률이 높고 생성 비용이 비싼 객체

- 캐싱하여 재사용하자!

Ex) 문자열 확인할때

1. 정규표현식

```java
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

- String.matches는 반복해서 사용할 시 성능이 저하된다
- **Pattern**은 입력받은 정규표현식에 해당하는 **finate state machine (**[https://www.youtube.com/watch?v=ZfW7FwuBd90](https://www.youtube.com/watch?v=ZfW7FwuBd90))을 만들기 때문에 인스턴스 생성 비용이 높다.

1. 캐싱

```java
public class RomanNumerals {

    private static final Pattern ROMAN = Pattern.compile(
        "^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    public static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

- Pattern 인스턴스를(Roman) 초기화 하면서 직접 생성해 캐싱해두고 isRomanNumeral이 호출될때마다 인스턴스를 재사용한다

## 어댑터를 사용한 불필요 객체 생성 방지

Ex) Map 인터페이스의 KeySet 메서드는 호출될 때마다 같은 set 인스턴스를 반환한다

```java
for (String key : map.keySet()) {
	String value = map.get(key);
	System.out.println("[key]:" + key + ", [value]:" + value);
}
```

+ 이 Set 인스턴스의 상태에 변경이 일어나면, 뒷단 객체에 해당하는 Map 인스턴스도 맞춰서 상태 변경이 일어난다. 
+ keySet이 Set 인터페이스를 매번 새로 만들어도 상관은 없지만, 그럴 필요도 없고 이득도 없다? 

### from 백기선님

+ keySet 으로부터 받은 Set 인스턴스가 다른 곳에서도 사용중이고 심지어 상태값을 변화시킨다면 현재 내가 **사용중인 Set 인스턴스와 Map 인스턴스의 값에 확신을 할 수 없다.** 
+ 그렇기 때문에 매번 복사해서 새로운 객체를 반환하도록 하는(item 50) 방어적 복사 방식을 선호한다고 했다.

## 오토박싱

- 오토박싱은 프로그래머가 기본 타입과 박싱된 기본 타입(Wrapper Class)를 섞어서 사용할 때 자동으로 상호 변환해주는 기술이다.

```java
public static long sum(){
    Long sum = 0L;
    for (long i = 0; i<= Integer.MAX_VALUE; i++){
        sum += i;
    }
    return sum;
}
```

- 교훈 : 박싱된 기본타입 보다는, 기본 타입을 사용하고 의도치 않은 오토박싱이 숨어들지 않도록 주의하자

## 정리

- 이번 아이템은 요약하자면 "**기존 객체를 재사용해야 한다면 새로운 객체를 만들지 마라**" 인 반면,
- 아이템 50의 경우 "**새로운 객체를 만들어야 한다면 기존 객체를 재사용하지 마라**"이다.
- 방어적 복사가 필요한 상황에서 객체를 재사용했을 때의 피해가, 필요 없는 객체를 반복 생성했을 때의 피해보다 훨씬 크다는 사실을 기억하자.