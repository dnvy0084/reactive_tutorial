# Reactive Programming
리액티브 프로그래밍을 공부하면서 느꼈던 부분이나 도움이 된 글들을 정리하고, 리액티브 프로그래밍 라이브러리 중 하나인 rxjs를 통해 프로그래밍 방법에 대해 소개합니다. 주로 사용하는 객체지향 언어와의 차이점을 통해 OOP 베이스인 개발자들의 이해를 도울 수 있도록 하였습니다. 

### Reactive Programming?

리액티브 관련 검색을 하면 보통 다음처럼 나옵니다.

***"Reactive programming is programming with asynchronous data streams.***

***You can listen to that stream and react accordingly."***

"리액티브 프로그래밍은 비동기 데이터 스트림을 사용한 프로그래밍이며, 듣고 반응할 수 있다." 정도로 해석할 수 있는데 당연하게도 처음 리액티브 프로그래밍을 접하는 사람에게는 별 도움이 되지 못하는 구문입니다. "객체 지향 프로그래밍에서 특정 객체를 생성하기 위한 일종의 틀"로 설명되는 클래스를 처음 배웠을 때와 비슷한 느낌이네요.
클래스도 그렇지만 리액티브 프로그래밍도 어느정도 알고나야 이해가 될 구문이라 고개만 끄덕이고 넘어가면 될 것 같습니다. 

저 구문은 비동기 데이터 스트림을 우선 하고 있는데, 데이터 스트림은 도구이고 주 목적은 reactive에 있다고 생각합니다. "반응하는"이란 뜻인데 말 그대로 어떤 값의 변경이 그와 관련된 다른 값을 자동으로 갱신하거나 정해진 일을 수행하도록 프로그래밍하는 거라고 볼 수 있습니다.

```javascript 
let a = 3
  , c = a + 1;
  
 a = 4;
 console.log(c); // 4
```
+ 예제 코드 1

예제 코드 1을 실행하면 c의 값은 4인데요, 리액티브 프로그래밍의 관점에서 보면 ```a = 4```를 입력했으니 ```c = a + 1```이 갱신되어 log에 5가 출력되어야 합니다.
예제 코드가 마이크로소프트의 엑셀이고 c 셀은 a 셀에 1을 더한 값을 가지도록 함수를 설정한거라고 보면 될 것 같네요. 당연한거지만 c가 5가되려면 ```c = a + 1```을 한번 더 호출해줘야 되는데요, 리액티브 프로그래밍이라고 다르진 않습니다. 

a의 값이 변경되면 c 변수를 갱신해줘야 하는건 마찬가지이지만 절차적 언의의 방법과는 차이가 있습니다. 바로 스트림이라는 개념을 가져온건데요, 리액티브 프로그래밍에서는 "Everything is Stream"이란 말이 있을 정도로 스트림을 이용해 모든것을 처리하고 있습니다. 객체 지향 언어가 각 할일을 가진 여러개의 객체를 통해 주어진 일을 처리한다면, 리액티브 프로그래밍은 스트림을 통해 데이터의 흐름을 미리 선언함으로서 일을 처리하도록 만듭니다. 앞으로 예제 코드1을 직접 구현해보면서 스트림이 무엇이고 왜 사용하는지에 대해 알아보겠습니다.

### 반응형 변수

예제코드 1에서 나온 변수 c는 a의 값에 **반응**합니다. 그러기 위해서 c는 a를 **관찰**하고 있어야 됩니다. 많이들 사용하는 EventEmitter가 비슷한 일을 하고 있는데요, EventEmitter를 통해 c가 자동갱신되는 코드를 만들어봤습니다. 

```javascript 
function watch(target, prop) {
    const emitter = new EventEmitter();
    
    Object.defineProperty(target, prop, {
    	get() {
	    return this[`_${prop}`];
	},
        set(value) {
            this[`_${prop}`] = value;
            emitter.emit('change', value);
        }
    });
    
    return emitter
}

watch(window, 'a').on('change', value => window.c = value + 1);

a = 1;
console.log(a, c); // 1 2

a = 10;
console.log(a, c); // 10 11
```
+ 예제 코드 2 [fiddle](https://jsfiddle.net/dnvy0084/2vanhgpq/)

watch라는 함수를 통해 원하는 이름의 getter/setter를 설정하고 setter 함수에서 값이 변경될 경우 eventEmitter를 이용해 변경된 값을 알려주는 형태입니다. a가 변경되면 event listener에서 c를 갱신해주고 있는데요, 별 문제없이 작동하며 watch 함수를 제외하면 로직 자체도 굉장히 간단합니다.

```javascript
var a = 1
  , b = 2
  , c = a + b;
 
a = 10;
console.log(
```
