# 아이템 34. int 상수 대신 열거 타입을 사용하라.

## 결론.  필요한 원소를 컴파일 타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.

## 정수 열거 패턴과 열거 타입

```java
public static final int APPLE_PIE = 0;
public static final int APPLE_JAM = 1;

public static final int GRAPE_PIE = 0;
public static final int GRAPE_JAM = 1;
```

```java
public enum Apple { PIE, JAM }
public enum GRAPE { PIE, JAM }
```

- 정수 열거 패턴은 타입 안전을 보장할 수 없으며 다른 타입을 건네더라도 경고를 내보내지 않는다.
    
    ex) APPLE_PIE가 필요한데 GRAPE_PIE를 줘도 같은 int 0이므로 경고가 발생하지 않는다.
    
- 정수 열거 패턴은 상수 값이 바뀌면 모두 다시 컴파일해야 한다.
- 정수 열거 패턴은 문자열로 출력하기 어렵다.(숫자라 문자열로 출력해도 의미를 알기 어렵다.)

## 열거 타입

자바의 열거 타입은 완전한 형태의 클래스이다.

- 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final field로 공개한다.
- 열거 타입은 싱글턴을 일반화한 형태라 할 수 있다.
- 각자의 이름 공간이 존재하기 때문에 추가되거나 순서가 바뀌어도 클라이언트는 재 컴파일 할 필요가 없다.
- toString을 사용하기 적합하다.
- 열거타입을 삭제하면 해당 열거 타입을 참조하는 클라이언트는 컴파일 오류가 발생한다.

## 상수별 메소드 구현

열거타입에 따라 다른 메소드를 구현할 수 있다.

```java
public enum Operation {
    PLUS,MINUS,TIMES,DIVDE;

    public double apply(double x, double y) {
        switch (this) {
            case PLUS:
                return x + y;
            case MINUS:
                return x - y;
            case TIMES:
                return x * y;
            case DIVDE:
                return x / y;
        }
        throw new AssertionError("알 수 없는 연산:" + this);
    }
}
```

- 상수를 추가한다면 case문에 추가해야하며 안그런 경우 알 수 없는 연산이 되어버린다.

```java
public enum Operation {
    PLUS("+") {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVDE("/") {
        public double apply(double x, double y) {
            return x / y;
        }
    };

    private final String symbol;

    Operation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }

    public abstract double apply(double x, double y);
}
```

- 이는 기존 코드와 달리 각 각의 경우 apply연산을 구현해야 해서 case문을 까먹는 등의 행위를 하지 않을 수 있다.

## 열거타입의 참조

- 모든 열거타입에 대해 사용할 수 있는 코드

```java
private static final Map<String, Operation> stringToEnum =
            Stream.of(Operation.values())
                    .collect(Collectors.toMap(Operation::toString, operation -> operation));

//Optional로 반환하여 값이 존재하지않을 상황을 클라이언트에게 알린다.
public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));
}
```

- 빈 해시맵을 만든 후 순회할 수 있지만 컴파일 에러가 발생한다.
    - 열거타입 상수는 생성자에서 자신의 인스턴스를 추가할 수 없으며 이는 생성자의 실행 시점에서는 다른 필드들이 아직 초기화되기 전이라 참조를 막기위해 발생한다.

## 전략 열거타입 패턴

새로운 값을 열거타입에 추가하고 이를 잊지 않고 구현해야한다.

이를 쉽게 하기위해 전략 열거 타입 패턴을 사용할 수있다.

```java
public enum PayrollDay {
    MONDAY,
    TUESDAY,
    WEDNESDAY,
    THURSDAY,
    FRIDAY,
    SATURDAY,
    SUNDAY;

    private static final int MINS_PER_SHIFT = 8 * 60;

    int pay(int minutesWorked, int payRate) {
        //기본 급여
        int basePay = minutesWorked * payRate;
		//잔업수당
        int overtimePay;
        switch (this) {
        	//주말
            case SATURDAY:
            case SUNDAY:
                overtimePay = basePay / 2;
                break;
            //주중
            default:
                overtimePay = minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
        }

        return basePay + overtimePay;
    }
}
```

- 위 코드는 쉬는날(휴가)등이 추가된 경우 계속 case문을 추가해야 한다.

```java
public enum PayrollDay {
    MONDAY(PayType.WEEKDAY),
    TUESDAY(PayType.WEEKDAY),
    WEDNESDAY(PayType.WEEKDAY),
    THURSDAY(PayType.WEEKDAY),
    FRIDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND),
    SUNDAY(PayType.WEEKEND);
    
    private final PayType payType;

    PayrollDay(PayType payType) {
        this.payType = payType;
    }

    int pay(int minutesWorked, int payRate) {
        return payType.pay(minutesWorked,payRate);
    }

    private enum PayType {
        WEEKDAY {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked <= MINS_PER_SHIFT ?
                        0 : (minutesWorked - MINS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            int overtimePay(int minutesWorked, int payRate) {
                return minutesWorked * payRate / 2;
            }
        };

        abstract int overtimePay(int minutesWorked, int payRate);
        private static final int MINS_PER_SHIFT = 8 * 60;

        int pay(int minutesWorked, int payRate) {
            int basePay = minutesWorked * payRate;
            return basePay + overtimePay(minutesWorked,payRate);
        }
    }
}
```

- 이는 각각의 계산은 private 중첩 열거타입에 위임한 후 열거타입에서는 중첩 열거 타입에서 적절한 것을 선택하기만 하면 된다.
- 이는 추후 새로운 상수가 추가되어도 크게 수정할 필요 없다.
- case문은 대부분 적합하지 않지만 추가하려는 메서드가 의미상 열거타입에 속하지 않는다면 사용할 법 하다.
    
    ```java
    public static Operation inverse(Operation operation) {
            switch (operation) {
                case PLUS:
                    return Operation.MINUS;
                case MINUS:
                    return Operation.PLUS;
                case TIMES:
                    return Operation.DIVDE;
                case DIVDE:
                    return Operation.TIMES;
            }
            throw new AssertionError("알 수 없는 연산 : " +operation);
    }
    ```
    

## 열거타입의 사용 예시

프로젝트에서 에러타입을 정의할 때 열거타입을 사용했었다.

- BaseException
    
    ```java
    public abstract class BaseException extends RuntimeException {
        public abstract BaseExceptionType getExceptionType();
    }
    ```
    
- BaseExceptionType
    
    ```java
    	public interface BaseExceptionType {
        int getErrorCode();
    
        HttpStatus getHttpStatus();
    
        String getErrorMessage();
    }
    ```
    
- ExceptionAdvice
    
    : RestControllerAdvice를 사용하여 해당 예외가 발생하면 전역적으로 처리한다.
    
    ```java
    @RestControllerAdvice
    @Slf4j
    public class ExceptionAdvice {
    
        @ExceptionHandler(BaseException.class)
        public ResponseEntity handleBaseEx(BaseException exception) {
            log.error("CustomException errorMessage(): {}", exception.getExceptionType().getErrorMessage());
            log.error("CustomException errorCode(): {}", exception.getExceptionType().getErrorCode());
    
            return new ResponseEntity(new ExceptionDto(exception.getExceptionType().getErrorCode()), exception.getExceptionType().getHttpStatus());
        }
    
        /**
         * 커스텀 예외 발생시 예외 코드 반환 DTO
         */
        @Data
        @AllArgsConstructor
        static class ExceptionDto {
            private Integer errorCode;
        }
    }
    ```
    
- MemberException
    
    ```java
    public class MemberException extends  BaseException{
    
        private BaseExceptionType exceptionType;
    
        public MemberException(BaseExceptionType exceptionType){
            this.exceptionType = exceptionType;
        }
    
        @Override
        public BaseExceptionType getExceptionType(){
            return exceptionType;
        }
    }
    ```
    
- MemberExceptionType
    ```java
    public enum MemberExceptionType implements BaseExceptionType {
        //회원가입 시
        ALREADY_EXIST_USER_EMAIL(419, HttpStatus.CONFLICT, "이미 존재하는 이메일입니다."),
        ALREADY_EXIST_USER_NICKNAME(429, HttpStatus.CONFLICT, "이미 존재하는 닉네임입니다."),

        private int errorCode;
        private HttpStatus httpStatus;
        private String errorMessage;

        MemberExceptionType(int errorCode, HttpStatus httpStatus, String errorMessage) {
            this.errorCode = errorCode;
            this.httpStatus = httpStatus;
            this.errorMessage = errorMessage;
        }

        @Override
        public int getErrorCode() {
            return this.errorCode;
        }

        @Override
        public HttpStatus getHttpStatus() {
            return this.httpStatus;
        }

        @Override
        public String getErrorMessage() {
            return this.errorMessage;
        }
    }
    ```