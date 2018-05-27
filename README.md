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

예제코드 1에서 나온 변수 c는 a의 값에 **반응**합니다. 그러기 위해서 c는 a를 **관찰**하고 있어야 됩니다. 많이들 사용하는 EventEmitter가 비슷한 일을 하고 있는데요, EventEmitter를 통해 c가 자동갱신되는 코드를 만들어 보겠습니다.

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

watch라는 함수를 통해 원하는 이름의 getter/setter를 설정하고 setter 함수에서 값이 변경될 경우 eventEmitter를 이용해 변경된 값을 알려주는 형태입니다. a가 변경되면 event listener에서 c를 갱신해주고 있는데요, 별 문제없이 작동하며 watch 함수를 제외하면 로직 자체도 굉장히 간단합니다. 예제 코드 2를 리액티브 프로그래밍 라이브러리인 [rxjs](https://github.com/ReactiveX/rxjs)를 이용해서 동일하게 구현해 볼건데요, 그에 앞서 rxjs가 무엇인지 부터 잠깐 소개하고 넘어가겠습니다. 

### Rxjs란

Rxjs는 [ReactiveX](http://reactivex.io/)라는 라이브러리의 javascript 버전입니다. ReactiveX는 마이크로소프트에서 시작한 리액티브 프로그래밍 라이브러리인데요, [Rx.NET](https://github.com/dotnet/reactive), [RxScala](https://github.com/ReactiveX/RxScala), [RxJava](https://github.com/ReactiveX/RxJava), [RxLua](https://github.com/bjornbytes/RxLua), [RxPY](https://github.com/ReactiveX/RxPY), [RxGo](https://github.com/ReactiveX/RxGo), [RxCpp](https://github.com/ReactiveX/RxCpp), [RxPHP](https://github.com/ReactiveX/RxPHP)등 들어봤을법한 언어는 모두 지원하고 있습니다. 그래서 어떤 언어로든 배우고 나면 다른 언어에서도 거의 동일한 인터페이스로 프로그래밍을 할 수 있습니다. 최근에 와서 Angular2에서 rxjs를 채용하거나 Netflix에서 RxJava를 만드는 등 reactivex가 주목을 받고 있는 것 같습니다. 

RxJS는 Observable, Observer, Subscriber 3가지 키워드와 Observer를 제어할 수 있는 operators로 이루어져 있습니다. Observable은 ES6의 Promise와 비슷하거나 Promise를 강화한 버전이라고 생각하시면 됩니다. Promise는 처리할 일을 wrapping해서 해당 과정의 비동기 여부와 상관없이 then() 체인으로 결과를 전달하는데요, Observable도 비슷하게 동작합니다. 다만 Observable은 map, filter, scan등 값을 원하는 형태로 변경하거나 zip, combineLatest, merge, forkJoin등 여러개의 Observable을 합치는 등 Promise에는 없는 굉장히 많은 기능들이 있습니다. 이런 (쓸데없을 정도로) 많은 기능이 rxjs를 배우는데 높은 진입장벽으로 작용하는 것도 사실입니다.;

각설하고 실제 RxJs를 가지고 예제코드 2를 똑같이 구현해보겠습니다. (최신 버전이 6.x인데 ES6를 위한 개별 import 지원으로 cdn link 상태에서도 pipe 구문을 사용해야 되서 혼란을 피하기 위해 5.5버전을 사용했습니다.)

```javascript
function watch(target, prop) {
    return Rx.Observable.create(observer => {
    	Object.defineProperty(target, prop, {
            get() {
                return this[`_${prop}`];
            },
            set(value) {
                this[`_${prop}`] = value;
                observer.next(value);
            }
        });
    });
}

watch(window, 'a').subscribe(value => window.c = value + 1);

a = 1;
console.log(a, c); // 1 2

a = 10;
console.log(a, c); // 10 11
```
+ 예제 코드 3 - [fiddle](https://jsfiddle.net/dnvy0084/s8sm2jsj/)

```Rx.Observable.create```를 이용하여 원하는 어떤것도 Observable로 만들 수 있는데요, 여기서 이야기 하는 Observable 즉 **관찰가능한**이 리액티브 프로그래밍에서 말하는 스트림입니다. 외부로부터의 데이터만 스트림이 아니라 사용자 입력이나 값의 변경, 배열같은 Iterator나 Generator 심지어 1, 2, 3... 같은 단순한 Number까지 이 모든것을 **스트림화**하여 프로그래밍합니다. 여기서는 setter 함수에서 변경되는 값을 observer라는 객체의 next 메소드를 이용해 스트림으로 바꿔주고 있습니다. 이렇게 next로 넘겨진 값은 operator거쳐 최종적으로 subscribe의 콜백에 도착합니다. 여기서는 다른 operator없이 바로 subscribe로 전달되어 EventEmitter 예제처럼 최종적으로 window.c 변수에 +1되어 저장됩니다. 

Promise가 생성자의 실행함수에서 resolve나 reject를 이용해 then으로 연결된 다음 콜백에 결과값이나 에러를 던져주는 것과 동일한 동작 방식입니다. 차이점은 Promise는 resolve나 reject를 호출하면 한번으로 끝인 반면 Observable은 observer.next()를 몇번이고 호출할 수 있습니다. 그래서 setter의 값을 여러번 바꿔도 매번 subsribe의 콜백을 거쳐 c변수를 갱신합니다. 참고로 종료를 위해서는 observer.error(e)나 observer.complete()을 호출하여 완료를 알리고 스트림을 종료 할 수 있습니다.

여기까지 봐서는 EventEmitter나 Observable이나 별 차이가 없어 보이는데요, 만약 아래처럼 c 변수가 a, b 두 개의 변수를 참조하고 있다면 어떻게 해야 될까요? EventEmitter로 먼저 구현해 보겠습니다. (watch 함수는 위와 같은 형태이니 생략하도록 하겠습니다.)

```javascript
var a = 1
  , b = 2
  , c = a + b;
  
console.log(c) // 3
 
a = 10;
console.log(c) // 12

b = 20;
console.log(c) // 30
```
+ 예제 코드 1-1

```javascript
watch(window, 'a').on('change', value => add(a, b));
watch(window, 'b').on('change', value => add(a, b));

function add(a, b) {
    if(typeof a === 'undefined' || typeof b === 'undefined') return;
    
    window.c = a + b;
}

a = 1;
b = 2
console.log(c); // 3

a = 10;
console.log(c); // 12
b = 20;
console.log(c); // 30
```
+ 예제 코드 2-1 [fiddle](https://jsfiddle.net/dnvy0084/gh84xovv/)

a, b 두 변수 모두 watch를 이용해 값이 변경되는 시점에 add 함수를 호출하고 있습니다. 다만 a나 b 둘 중 하나만 값이 있는 경우가 있어 if를 이용해 undefined인지 체크를 해줘야 합니다. 혹은 watch를 하기 전에 _a, _b에 기본값을 설정해 놓는 방법도 있을 것 같은데요, 어찌됐건 신경써야 될 부분이 늘어난건 마찬가지입니다. c에서 참조하는 변수가 많으면 많을수록 걱정거리도 늘어나겠네요.

이 코드를 Observable로 구현하면 이런류의 걱정거리를 줄일 수 있습니다. 다음은 Observable 코드입니다. 

```javascript
const a$ = watch(window, 'a');
const b$ = watch(window, 'b');

a$.combineLatest(b$).subscribe(([a, b]) => window.c = a + b);

a = 1;
b = 2;
console.log(c); // 3

a = 10;
console.log(c); // 12

b = 20;
console.log(c); // 30
```
+ 예제 코드 3-1 [fiddle](https://jsfiddle.net/dnvy0084/sz1s370d/)
