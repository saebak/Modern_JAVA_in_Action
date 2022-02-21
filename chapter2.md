# 동작 파라미터화 코드 전달하기

 - 변화하는 요구사항에 대응
 - 동작 파라미터화
 - 익명 클래스
 - 람다 표현식 미리보기


------------------------------------------------------------------

## 동작 파라미터화
    아직 어떻게 실행할 것인지 결정하지 않은 코드 블록
    
 - 자주 바뀌는 요구사항에 효과적으로 대응할수 있음
 - 나중에 실행될 메서드의 인수로 코드 블록을 전달할 수 있음
 - 동작 파라미터화를 추가하려면 쓸데없는 코드가 늘어나는데 자바8에서는 람다표현식으로 해결
 
 <br/>
 
### 변화하는 요구사항에 대응하기

#### 첫 번째 시도 : 녹색 사과 필터링

<pre>
<code>
public static List<Apple> filterGreenApples(List<Apple> inventory) {
   List<Apple> result = new ArrayList<>(); 사과 누적 리스트
   for (Apple apple: inventory) {
     if (GREEN.equals(apple.getColor()) { 녹색 사과만 선택
       result.add(apple);
     }
   }
 return result;
}
</code>
</pre>
 
 여기서 녹색 사과가 아닌 빨간 사과로 필터링하고 싶다면?
 
 > 거의 비슷한 코드가 반복 존재한다면 그 코드를 추상화한다.
 
 #### 두 번째 시도 : 색을 파라미터화
 
 색을 파라미터화할 수 있도록 메서드에 파라미터를 추가하면 변화하는 요구사항에 좀 더 유연하게 대응하는 코드를 만들 수 있다.
 
<pre>
<code>
public static List<Apple> filterApplesByColor(List<Apple> inventory, Color color)
{
   List<Apple> result = new ArrayList<>();
   for (Apple apple: inventory) {
   if ( apple.getColor().equals(color) ) {
     result.add(apple);
   }
 }
 return result;
}

List<Apple> greenApples = filterApplesByColor(inventory, GREEN); List<Apple>
redApples = filterApplesByColor(inventory, RED);
</code>
</pre>

색 이외에도 가벼운 사과와 무거운 사과로 구분하고 싶다면?

<pre>
<code>
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight)
{
   List<Apple> result = new ArrayList<>();
   for (Apple apple: inventory) {
     if ( apple.getWeight() > weight ) {
       result.add(apple);
     }
   }
 return result;
}
</code>
</pre>

위의 코드는 목록을 검색하고 각 사과에 필터링 조건을 적용하는 부분의 코드가 색 필터링 코드와 대부분 중복!!
이는 같은 것을 반복하지 말 것이라는 소프트웨어 공학의 원칙을 어기는것


#### 세 번째 시도 : 가능한 모든 속성으로 필터링

색과 무게를 filter라는 메서드로 합치고 어떤 기준으로 사과를 필터링할지 구분하는 플래그 추가

<pre>
<code>
public static List<Apple> filterApples(List<Apple> inventory, Color color,
                                            int weight, boolean flag) { 
   List<Apple> result = new ArrayList<>();
   for (Apple apple: inventory) {
   if ((flag && apple.getColor().equals(color)) || 
      (!flag && apple.getWeight() > weight)) {
      result.add(apple);
   }
  }
 return result;
}

List<Apple> greenApples = filterApples(inventory, GREEN, 0, true);
List<Apple> heavyApples = filterApples(inventory, null, 150, false);
</code>
</pre>

위 코드는 형편없는 코드이다.
요구사항이 바뀌었을 때 유연하게 대응할 수 없다.
결국 여러 중복된 필터 메서드를 만들거나 아니면 모든 것을 처리하는 거대한 하나의 필터 메서드를 구현해야 한다.
문제가 잘 정의되어있는 상황에서는 이 방법이 잘 동작하겠지만,
filterApples에 어떤 기준으로 사과를 필터링할 것인지 효과적으로 전달할 수 있다면 더 좋을것이다.


### 동작 파라미터화

선택 조건을 결정하는 인터페이스를 정의 </br>

![image](https://user-images.githubusercontent.com/45276842/154943211-c2992ba0-c9f3-442f-81ed-f7da8dabd346.png)

위 조건에 따라 filter 메서드가 다르게 동작할 것이라고 예상할 수 있음
이를 '전략 디자인 패턴' 이라고 함
> 전략 디자인 패턴은 각 알고리즘 ( 전략이라 불리는 ) 을 캡슐화하는 알고리즘 패밀리를 정의해둔 다음에 런타임에 알고리즘을 선택하는 기법

filterApples에서 ApplePredicate 객체를 받아 애플의 조건을 검사하도록 메서드를 고쳐서 </br>
메서드가 다양한 동작을 받아서 내부적으로 다양한 동작을 수행할 수 있다(동작 파라미터화)


#### 네 번째 시도 : 추상적 조건으로 필터링

코드/동작 전달하기 </br>
![image](https://user-images.githubusercontent.com/45276842/154944931-0e74c2fe-5085-4210-900c-13426036f051.png)
 
한 개의 피라미터, 다양한 동작 </br>
![image](https://user-images.githubusercontent.com/45276842/154945801-746001c7-1ec6-49a0-8630-0d23a2d264b1.png)


### 복잡한 과정 간소화

위의 과정들을 거치면서 로직과 관련 없는 코드가 많이 추가되었다. </br>
자바는 클래스의 선언과 인스턴스화를 동시에 수행할 수 있도록 <b>익명 클래스</b>라는 기법을 제공한다.


#### 익명 클래스
> 익명 클래스는 자바의 지역 클래스local class (블록 내부에 선언된 클래스)와 비슷한 개념으로 말 그대로 이름이 없는 클래스다. 익명 클래스를 이용하면 클래스 선언과 인스턴스화를 동시에 할 수 있다. 즉, 즉석에서 필요한 구현을 만들어서 사용할 수 있다

#### 다섯 번재 시도 : 익명 클래스 사용

<pre>
<code>
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {
    public boolean test(Apple apple){
      return RED.equals(apple.getColor());
    }
});
</code>
</pre>

하지만 익명 클래스는 많은 공간을 차지하고(장황한 코드) <br/>
많은 프로그래머가 익명 클래스의 사용에 익숙하지 않는다는 단점이 있다.

#### 여섯 번째 시도 : 람다 표현식 사용

<pre>
<code>
List<Apple> result =
 filterApples(inventory, (Apple apple) -> RED.equals(apple.getColor()));
</code>
</pre>

![image](https://user-images.githubusercontent.com/45276842/154949400-c71474c6-f107-4aa6-9b0b-6bc9fe606d3f.png)


#### 일곱 번째 시도 : 리스트 형식으로 추상화

형식 파라미터 T 사용
<pre>
<code>
public interface Predicate<T> {
    boolean test(T t);
}
public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    List<T> result = new ArrayList<>();
    for(T e: list) {
      if(p.test(e)) {
       result.add(e);
      }
    }
 return result;
}
</code>
</pre>

람다 표현식을 사용한 예제
<pre>
<code>
List<Apple> redApples =
   filter(inventory, (Apple apple) -> RED.equals(apple.getColor()));
List<Integer> evenNumbers =
 filter(numbers, (Integer i) -> i % 2 == 0);
</code>
</pre>



### 실전 예제

자바 API의 많은 메서드를 다양한 동작으로 파라미터화 할 수 있다. </br>
이들 메서드를 익명 클래스와 자주 사용하기도 한다.

1. Comparator로 정렬하기
2. Runnable로 코드 블록 실행하기
3. GUI 이벤트 처리하기 (ExecutorService, EventHandler 인터페이스)


<br/><br/><br/>
----------------------------------------------------------------------------------------------------------

## 참고자료
[Refactoring - 리팩터링 원칙 7장 캡슐화](https://slog2.tistory.com/17).

