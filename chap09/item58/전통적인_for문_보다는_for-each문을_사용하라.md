# 전통적인 for문보다는 for-each 문을 사용하라

# 문제점

**컬렉션 순회하기 - 더 나은 방법 존재**

```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ...
}
```

**배열 순회하기 - 더 나은 방법 존재**

```java
for (int i = 0; i < a.length; i++) {
    ...
}
```

**반복자와 인덱스 변수를 코드를 지저분하게하고 오류가 생길 가능성이 높아진다.**

1회 반복에서 반복자는 세 번 등장하고 인덱스는 네 번 등장하고 있다. 혹시라도 잘못된 변수를 사용했을 때 컴파일러가 잡아주지 못할 수도 있다. 

### 또한 컬렉션이나 배열이냐에 따라 코드 형태도 달라지고 있다.

# 향상된 for-Each

하나의 관용구로 컬렉션과 배열 모두 처리할 수 있어서 어떤 컨테이너를 다루는지 신경쓰지 않아도 된다.

```java
for (Element e:  elements) {
    ...
}
```

반복 대상이 컬렉션이든 배열이든, for-each 문을 사용해도 속도는 그대로다.

컬렉션을 중첩해 순회해야 한다면 for-each문의 이점은 더욱 커진다.

**버그를 찾아보자**

```java
    // 버그를 찾아보자.
    enum Suit { CLUB, DIAMOND, HEART, SPADE }
    enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
        NINE, TEN, JACK, QUEEN, KING }

    static Collection<Suit> suits = Arrays.asList(Suit.values());
    static Collection<Rank> ranks = Arrays.asList(Rank.values());

    Card(Suit suit, Rank rank ) {
        this.suit = suit;
        this.rank = rank;
    }

    public static void main(String[] args) {
        List<Card> deck = new ArrayList<>();

        for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
            for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
                deck.add(new Card(i.next(), j.next()));
```

마지막 줄의 i.next()가 숫자 하나당 한번씩만 불려야하는데, 안쪽 반복문에서 호출되는 바람에 카드 하나당 한 번씩 불리고 있다. 그래서 숫자가 바닥나면 NoSuchElementException을 던진다.

이 문제를 해결하려면 바깥 반복문에 바깥 원소를 저장하는 변수를 추가해야한다.

```java
for (Suit suit: suits)
    for (Rank rank: ranks)
        deck.add(new Card(suit, rank));
```

이렇게 for-each문을 사용하면 간단하게 해결된다.

# for-each 문 사용 불가 상황

- **파괴적인 필터링**
    
    컬렉션을 순회하면서 선택된 원소를 제거해야 한다면 **remove** 메서드를 호출해야 한다. 자바 8부터는 Collection의 **removeIf** 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다.
    
- **변형**
    
    리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 리스트의 반복자나 배열의 인덱스를 사용해야 한다.
    
- **병렬 반복**
    
    여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 명시적으로 제어해야 한다.