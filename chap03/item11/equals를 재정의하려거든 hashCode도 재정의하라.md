# 아이템11. equals를 재정의하려거든 hashCode도 재정의하라

equals를 재정의 할때  hashCode를 재정의 하지 않으면 HashMap이나 HashSet의 원소로 사용할 때 문제를 일으킨다.

따라서, 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다. (해당 클래스가 Key가 될 것 같으면)

## 결론. equals를 재정의 하면 hashCode도 같이 재정의하자. hashTable의 성능을 높이기 위해 다른 값에는 다른 hashCode가 올 수 있도록 하자.

## Object의 hashCode 규약

1. equals 비교에 사용되는 정보가 변경되지 않았다면, 어플리케이션이 실행되는 동안 객체의 hashCode메소드는 몇 번 호출해도 일관적인 값을 반환해야 한다.
2. equals가 두 객체가 같다고 판단했으면 hashCode는 같은 값을 반환해야 한다.
3. equals가 두 객체가 다르다고 판단했을 때 꼭 서로 다른 hashCode를 반환할 필요는 없다. 하지만 다른 값을 반환해야 해시 테이블에서 성능이 좋아진다.

## HashCode의 구현

1. 모든 객체에 같은 HashCode를 반환
    
    ```java
    @Override
    public int hashCode(){
    	return 42;
    }
    ```
    
    위 코드는 Object의 hashCode 규약을 모두 만족시킨다. 하지만 모든 객체의 hashCode가 같은 값을 가지게 된다.
    
    따라서 해시 테이블에서 연결리스트 처럼 동작하게 되며, 평균 수행시간이 O(1)에서 O(n)으로 증가하게 된다.
    
2. HashCode의 구현
    
    ```java
    @Ovverride
    public int hashCode(){
    	int result = Short.hashCode(areaCode);
    	result = 31 * result + Short.hashCode(prefix);
    	result = 31 * result + Short.hashCOde(lineNum);
    	return result;
    }
    ```
    
    - result는 해당 객체의 첫번째 핵심 필드의 해시코드를 저장한다.
    - 이후 result = 31 * result + 각 필드의 해시코드 를 통해 해시코드를 계산한다.
        - 기본타입: Type.hashCode(필드)로 계산
        - 참조타입: 해당 하면 해당 필드의 hashCode를 재귀적으로 호출(계산이 복잡하면 표준형 사용)
        - 배열: 원소를 각각의 필드처럼 다룬다. 모두가 핵심 필드라면 Arrays.hashCode 사용
    - Object.hash(필드1, 필드 2, …)를 통해 해시 값을 계산할 수 도 있지만 성능이 별로다.
        
        ```java
        @Override
        public int hashCode() {
            return Objects.hash(getCity(), getDong(), getRoadName(), getBuildingNumber(), getZipcode());
        }
        ```
        
    
    <aside>
    💡 31을 곱하는 이유: 홀수 이며 소수이기 때문
    - 홀수: 짝수는 시프트 연산과 같은 결과를 내므로 오버플로 발생 시 정보를 잃게 된다.
    
    </aside>
    
3. Key로 사용될 클래스가 불변이고 해시코드를 계산하는 비용이 큰 경우 → 캐싱
    
    ```java
    private int hashCode;
    
    //인스턴스가 생성될 때 hashCode 저장
    ```
    
4. Key로 사용되지 않고  클래스가 불변이고 계산하는 비용이 큰 경우 → 지연 초기화
    
    ```java
    private int hashCode; //0으로 초기화 되어 있음
    
    @Override
    public int hashCode(){
    	int result = hashCode;
    	//처음 사용될 때 계산
    	if(result == 0){
    		//해시코드 계산
    		hashCode = result;
    	}
    	return result;
    }
    ```
    

## 주의사항

1. 핵심 필드를 생략하면 안된다.
2. 생성 규칙을 API사용자에게 알려주지 말자.

## 참고
[[자료구조] 해시테이블(HashTable)이란?](https://mangkyu.tistory.com/102)