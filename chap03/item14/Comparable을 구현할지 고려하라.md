# 아이템14. Comparable을 구현할지 고려하라
## 결론. 비교 연산자 대신 compare을 통해 비교하고 비교자를 사용하자.

- compareTo는 equals와 비슷하다.
    - Comparable에 정의됨
    - 단순 동치성이 아닌 순서까지 비교 가능
    - 타입이 다른 객체를 신경쓰지 않음
- Java의 대부분의 값 클래스와 열거형은 이를 구현했다.

## compareTo의 규약

- 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0, 크면 양의 정수를 반환한다.
    - 아래의 값들은 음수, 0, 양수인지에 대해 동치여부를 비교한다.
1. x.compareTo(y) == -y.compareTo(X)를 만족해야 한다. (한 쪽이 예외 발생시 다른 쪽도 예외 발생)
2. x.compareTo(y) > 0 && y.compareTo(z)>0인 경우 x.compareTo(z)>0이다.
3. x.compareTo(y)==0이면 x.compareTo(z) == y.compareTo(z)이다.
4. (x.compareTo(y) == 0) == (x.equals(y))이면 좋다. 
    - compareTo는 동치성까지 신경 쓴다.
        
        ⇒ equals랑 비슷하게  기존 클래스를 확장하는 경우 새 클래스를 만들고 원래 클래스의 인스턴스를 가르키는 필드를 만든다.
        
        [아이템10.  equals는 일반 규약을 지켜 재정의하라](https://www.notion.so/10-equals-359fa5b9e600481a9efdff6113d8c85e)
        
    - 이를 지키지 않은 예시(Math.BigDecimal)
    
    ```java
    public static void main(String[] args){
        TreeSet<BigDecimal> treeSet = new TreeSet<>();
        HashSet<BigDecimal> hashSet = new HashSet<>();
    
        treeSet.add(new BigDecimal("1.0"));
        treeSet.add(new BigDecimal("1.00"));
    
        hashSet.add(new BigDecimal("1.0"));
        hashSet.add(new BigDecimal("1.00"));
    
        System.out.println("treeSet.size() = " + treeSet.size());
        System.out.println("hashSet.size() = " + hashSet.size());
    }
    
    =>	결과
    		treeSet.size() = 1
    		hashSet.size() = 2
    ```
    

<aside>
💡 정렬된 컬렉션은 동치성을 비교할 때 equals가 아닌 compareTo를 사용한다.

</aside>

## compareTo의 구현

- Comparable의 구현을 통해 어떤 클래스랑 비교할지 지정할 수 있다.
1. 원시타입은 비교 연산자(<, >)가 아닌 박싱된 타입의 compare를 사용한다.
    
    ```java
    public static void main(String[] args){
        TestClass abc = new TestClass("abc");
        TestClass def = new TestClass("def");
        System.out.println("abc.compareTo(def) = " + abc.compareTo(def));
    }
    
    public static class TestClass implements Comparable<TestClass>{
        private String name;
    
        public TestClass(String name) {
            this.name = name;
        }
    
        @Override
        public int compareTo(TestClass o) {
            return String.CASE_INSENSITIVE_ORDER.compare(name, o.name);
    		}
    }
    ```
    
2. 비교자(comparator)를 사용할 수 있다.
    
    ```java
    public static void main(String[] args){
        TestClass abc = new TestClass("abc", 20);
        TestClass def = new TestClass("abc", 21);
        System.out.println("abc.compareTo(def) = " + abc.compareTo(def));
    }
    public static class TestClass implements Comparable<TestClass>{
        private String name;
        private int age;
        private static final Comparator<TestClass> COMPARATOR =
                Comparator.comparing((TestClass tc) -> tc.name)
                        .thenComparingInt(tc -> tc.age);
    
        public TestClass(String name, int age) {
            this.name = name;
            this.age = age;
        }
    
        @Override
        public int compareTo(TestClass o) {
            return COMPARATOR.compare(this, o);
        }
    }
    ```
    
    - 여러 필드에 대해 순서 지정 가능
3. 값의 차를 사용한 구현의 경우
    
    ```java
    @Override
    public int compareTo(TestClass o){
    	return this.age - o.age;
    }
    ```
    
    - 이 형식은 오버플로우 또는 부동소수점 연산에서 오류를 일으킬 수 있다.
    
    ```java
    @Override
    public int compareTo(TestClass o) {
        return Integer.compare(this.age, o.age);
    }
    
    또는 비교자를 통해 비교.
    ```