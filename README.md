# Reactive Programming
리액티브 프로그래밍을 공부하면서 느꼈던 부분이나 도움이 된 글들을 정리하고, 리액티브 프로그래밍 라이브러리 중 하나인 rxjs를 통해 프로그래밍 방법에 대해 소개합니다. 주로 사용하는 객체지향 언어와의 차이점을 통해 OOP 베이스인 개발자들의 이해를 도울 수 있도록 하였습니다. 

### Reactive Programming?

리액티브 관련 검색을 하면 보통 다음처럼 나옵니다.

***"Reactive programming is programming with asynchronous data streams.***

***You can listen to that stream and react accordingly."***

"리액티브 프로그래밍은 비동기 데이터 스트림을 사용한 프로그래밍이며, 듣고 반응할 수 있다." 정도로 해석할 수 있는데 당연하게도 처음 리액티브 프로그래밍을 접하는 사람에게는 별 도움이 되지 못하는 구문입니다. "객체 지향 프로그래밍에서 특정 객체를 생성하기 위한 일종의 틀"로 설명되는 클래스를 처음 배웠을 때와 비슷한 느낌이네요.
클래스도 그렇지만 리액티브 프로그래밍도 어느정도 알고나야 이해가 될 구문이라 고개만 끄덕이고 넘어가면 될 것 같습니다. 

그리고 저기서는 비동기 데이터 스트림을 우선 하고 있는데, 데이터 스트림은 도구이고 주 목적은 reactive에 있다고 생각합니다. "반응하는"이란 뜻인데 말 그대로 어떤 값의 변경이 그와 관련된 다른 값을 자동으로 갱신하거나 정해진 일을 수행하도록 프로그래밍하는 거라고 볼 수 있습니다. 마이크로소프트의 엑셀을 생각해보면 되는데요, 각 셀은 그와 관련된 다른 셀의 값이 변경되면 별도의 동작없이 바로 갱신되

```javascript
let a = 3
  , c = a + 1;
  
console.log(c); // 4
```
