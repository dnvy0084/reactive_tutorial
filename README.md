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
+ 예제 코드 2 [fiddle](https://jsfiddle.net/dnvy0084/m9hvxmdc/)

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

```Rx.Observable.create```를 이용하여 원하는 어떤것도 Observable로 만들 수 있는데요, 여기서 이야기 하는 Observable 즉 **관찰가능한**이 리액티브 프로그래밍에서 말하는 스트림입니다. 외부로부터의 데이터만 스트림이 아니라 사용자 입력이나 값의 변경, 배열같은 Iterator나 Generator 심지어 1, 2, 3... 같은 단순한 Number까지 이 모든것을 **스트림화**하여 프로그래밍합니다. 여기서는 setter 함수에서 변경되는 값을 observer라는 객체의 next 메소드를 이용해 스트림으로 바꿔주고 있습니다. 이렇게 next로 넘겨진 값은 operator를 거쳐 최종적으로 subscribe의 콜백에 도착합니다. 여기서는 다른 operator없이 바로 subscribe로 전달되어 EventEmitter 예제처럼 최종적으로 window.c 변수에 +1되어 저장됩니다. 

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

rxjs의 operator가 처음으로 등장했네요. [combineLatest](http://reactivex.io/documentation/operators/combinelatest.html)는 두 개 이상의 스트림을 합치고 각 스트림의 값을 순서대로 배열로 만들어 다음 스트림으로 전달합니다. 그래서 subscribe에서는 ES6 비구조화 할당을 이용해 [a, b]로 받아 c에 대입하고 있습니다. 참고로 변수명을 a$라고 선언했는데요, reactivex의 암묵적인 룰 같은건데 $로 끝나는 변수는 스트림을 뜻합니다. combineLatest는 모든 스트림으로 부터 최소 한번은 데이터를 전달받아야 다음 스트림으로 데이터를 전달하기 때문에 undefined 체크가 필요없습니다. 

비슷한 일을 하는 operator로 [zip](http://reactivex.io/documentation/operators/zip.html)이 있는데요, zip은 각 스트림의 값이 모두 변경되어야 다음으로 전달되는 반면, combineLatest는 가지고 있는 스트림 중 하나만 변경이 되어도 바로 다음으로 전달됩니다. 이를 이용하면 ```n = a + b + c + d + e```같은 코드도 예제 3-1과 크게 다를것 없이 구현할 수 있을 것 같네요. EventEmitter는... 확실히 간단하지는 않을 모양새입니다. 

마지막으로 예제 코드 1이 아래 1-2처럼 변경이 되었다면 어떨까요? EventEmitter와 Observable로 각각 구현해 보겠습니다. 

```javascript
var a = 1
  , b = 2
  , c = a * a + b * b;
```
+ 예제 코드 1-2

먼저 EventEmitter입니다. 
```javascript
watch(window, 'a').on('change', value => add(a, b));
watch(window, 'b').on('change', value => add(a, b));

function add(a, b) {
    if(typeof a === 'undefined' || typeof b === 'undefined') return;
    
    window.c = a * a + b * b;
}

a = 1;
b = 2
console.log(c); // 3

a = 3;
console.log(c); // 13
b = 5;
console.log(c); // 34
```
+ 예제 코드 2-2 [fiddle](https://jsfiddle.net/dnvy0084/fu40m7bw/)

바뀐 곳은 add 함수 내 window.c 값을 갱신하는 부분입니다. 다음은 Observable 코드와 비교해 보겟습니다.

```javascript
const square = n => n * n;
const a$ = watch(window, 'a').map(square);
const b$ = watch(window, 'b').map(square);

a$.combineLatest(b$).subscribe(([a, b]) => window.c = a + b);

a = 1;
b = 2;
console.log(c); // 5

a = 3;
console.log(c); // 13

b = 5;
console.log(c); // 34
```
+ 예제 코드 3-2 [fiddle](https://jsfiddle.net/dnvy0084/k2hoaL0g/)

rxjs operator [map](http://reactivex.io/documentation/operators/map.html)을 사용했는데요, js array의 map과 동일한 함수로 콜백을 호출한 결과값을 새로운 데이터로 배출합니다. 여기서는 ```n => n * n```이라는 콜백을 적용하여 subscribe전에 제곱한 값을 전달합니다. EventEmitter와 Observable 코드의 가장 큰 차이점은 로직의 분리라고 할 수 있습니다. ```window.c = a + b``` 코드를 dom 업데이트나 instance 변수에 값을 할당하는 등의 side effect가 발생할 여지가 있는 코드라고 보면, EventEmitter는 그런 코드와 데이터를 제어하는 코드가 혼재해 있는 반면, Observable은 자연스럽게 분리되 있습니다. 

```javascript
a$.combineLatest(b$)
  .map(([a, b]) => a + b)
  .subscribe(n => window.c = n);
```
+ 예제 코드 3-3

사실 Observable도 데이터 제어와 값 할당을 완벽히 분리하려면 위처럼 바꾸는게 맞을 것 같네요. 이런 로직의 분리는 함수형 언어의 지향점과 같다고 볼 수 있습니다. 함수형 언어는 "부가적인 입력이나 출력이 없는 순수 함수와, io를 처리하거나 UI를 처리하기 위해 부득이하게 글로벌 변수등을 참조해야 하는 코드를 철저히 분리하자" 라는 컨셉을 가지고 있는데요, 그런면에서 EventEmitter는 add 함수를 따로 리팩토링할 필요가 있지만, Observable은 좀 더 쉽게(?) 분리된 코드를 짤 수 있습니다. 예제 코드 3-3에서 map의 콜백 ```([a, b]) => a + b``` 같은 함수는 단위 테스트하기 쉽다는 장점도 있겠네요.

여기까지 EventEmitter와 Observable을 비교해서 보여드렸는데, 어떻게 다르고 어떤 장점을 가지고 있는지 잘 전달이 되었나 모르겠네요. 다음은 Observable을 이용해 실질적인 도움이 될 만한 코드를 구현해보겠습니다. 

### Vue.js

FE 업무 중 큰 부분이 dom을 제어하는 일이라 생각하는데요, 위에 만든 watch 함수를 이용해 object의 속성을 바꾸면 해당되는 엘리먼트가 업데이트 되면 좋을 것 같습니다. [Vue.js](https://kr.vuejs.org/v2/guide/index.html)가 이런 형식으로 dom을 제어하고 있으니 인터페이스를 동일하게 구현해보겠습니다.

```html
<div id="target">
    <div class="name">
        <span>name:</span>
        <span>{{name}}</span>    
    </div>
    <div class="age">
        <span>age:</span>
        <span>{{age}}</span>    
    </div>
</div>
```
+ 예제 코드 4-1

코드 4-1처럼 html을 만들었습니다. span 태그의 textContent중 동적 제어가 필요한 곳에 mustache 템플릿 구문으로 변수 이름을 쓰고 해당 변수에 값을 바꾸면 textContent가 업데이트 되도록 하겠습니다. 그러려면 우선 mustache를 포함하고 있는 Text element 찾아야 됩니다. 

```javascript
/**
 * nodelist용 forEach
 **/
function forEach(nodelist, f) {
    for(let i = 0; i < nodelist.length; i++) 
    	f(nodelist[i], i, nodelist);
}

/**
 * el안의 Text element를 배열로 반환한다. 
 **/
function getTextNodes(el, nodes = []) {
    if(el instanceof Text && /{{[^}]+}}/.test(el.textContent)) 
    	nodes.push(el);
    
    if(!el.childNodes) return;
    
    forEach(el.childNodes, child => getTextNodes(child, nodes));
    
    return nodes;
}
```
+ 예제 코드 4-2

코드 4-2의 forEach는 element.childNodes 같은 NodeList 객체가 인터페이스는 배열이면서 .forEach를 지원하지 않아 forEach 대용으로 만들었습니다. 그리고 getTextNodes를 통해 Text element이면서 ```{{}}``` 이런 mustache 구문을 가진 element만 찾아서 반환합니다. 

```javascript
/**
 * target 객체에 mustache 구문 내 변수 이름으로 속성을 추가하고 해당 엘리먼트에 바인딩한다. 
 **/
function bindAsText(target, node) {
    const textContent = node.textContent;
    const regex = /{{\w+}}/g;
    const key = textContent.match(regex)[0].match(/\w+/g)[0];
    
    return watch(target, key)
    	.subscribe(value => node.textContent = textContent.replace(regex, value));
}
```
+ 예제 코드 4-3

이제 target object에 찾은 Text node를 바인딩 할 차례인데요, watch 후 suscribe 콜백에서 적절한 정규식으로 replace만 해주면 간단히 해결됩니다. subscribe 콜백은 bindAsText를 함수 스코프(부모 함수)로 갖는 클로져이기 때문에 bindAsText가 실행되는 순간 watch로 생성되는 Observable이 제어해야 될 node를 특정지을 수 있습니다. 그래서 따로 개별 Observable과 제어해야 될 node를 저장하지 않아도 됩니다. 글로벌로 접근할 수 있는 map같은 객체에 map.set('name', node) 형식으로 저장한다면, target이 여러개인 경우는 따로 생성을 해야 될지 등 신경써야 될 부분이 늘어나게 될 것 같네요. [관심사의 분리](https://en.wikipedia.org/wiki/Separation_of_concerns)라는 입장에서 보면 좋은 방법은 아닙니다. 

```javascript
const vue = {}
const el = document.querySelector('#target');

const subscriptions = getTextNodes(el).map(child => bindAsText(vue, child));

vue.name = 'moonee';
vue.age = 6;
```
+ 예제 코드 4-4

이제 마지막으로 적용할 element를 찾아 각 Text node마다 bindAsText를 호출해 주면 됩니다. vue라는 object와 연결을 해놓았으니 vue.name, vue.age를 바꾸면 html의 textContent가 바로 업데이트 될 것입니다. 전체 코드는 [fiddle](https://jsfiddle.net/dnvy0084/90uj35sx/)에서 확인하실 수 있습니다.




리액티브 프로그래밍(RP)은 보통 함수형 리액티브 프로그래밍(FRP)과 같이 나오곤 합니다. 그만큼 함수형 언어가 가지고 있는 특징들을 많이 구현하고 있습니다. 그래서 처음에는 ramda같은 함수형 라이브러리 + rxjs를 이용해 FRP의 형태로 코드를 짜려고 노력했었는데요, 리액티브도 생소한데 몸에 맞지도 않는 함수형 언어 형태로 개발을 하려니 진행은 안되고 시간만 잡아먹었던 것 같습니다. 우선 배우는 단계에서는 ugly한 코드라도 구현을 우선하고 

### Method Observable

다음은 조금 더 복잡한 예제입니다. 임의의 함수 호출 결과를 textContent로 대입할 수 있도록 할건데요, 리액티브 프로그래밍답게 함수 내부에서 참조한 변수 값의 변경이 있으면 함수를 호출해서 해당 엘리먼트가 업데이트 될 수 있도록 하겠습니다. [i18n](https://ko.wikipedia.org/wiki/%EA%B5%AD%EC%A0%9C%ED%99%94%EC%99%80_%EC%A7%80%EC%97%AD%ED%99%94) 같은 다국어 처리가 예제로 적격일 것 같네요. FE에서 다국어 처리 시 언어코드에 따라 해당 언어에 맞는 텍스트를 보여주기 위해 ```i18n(locale, key)``` 같은 형태의 함수 호출로 텍스트를 가져오는데요, 리액티브 프로그래밍스럽게 가져온 텍스트가 보여져야 할 엘리먼트까지 길(stream)을 잡아주고, 관찰중인 변수(locale)가 업데이트 되면 해당 언어로 바뀌도록 하겠습니다.

이전 예제에 기능을 더하는 형식으로 코드를 구현하고, 리팩토링도 같이 진행하겠습니다. 

```javascript
<div id="target">
    <div class="name">
        <span>{{i18n(NAME)}}:</span>
        <span>{{name}}</span>    
    </div>
    <div class="age">
        <span>{{i18n(AGE)}}:</span>
        <span>{{age}}</span>    
    </div>
</div>
```
+ 예제 코드 5

렌더링할 html인데, name과 age앞에 다국어 처리되어야 할 text가 추가되었습니다. locale이 바뀌면 NAME, AGE가 해당 언어로 업데이트 될 것입니다. 다음으로 vue라는 target 객체의 data model이 수정되었습니다. 

```javascript
const vue = {
  data: {
    name: 'moonee',
    age: 6,
    locale: 1
  },
  methods: {
    i18n(key) {
    	return i18n(this.locale, key);
    }
  }
}
```
+ 예제 코드 5-1

단순히 데이터만 가지고 있는 data object와 함수를 가지고 있는 methods object 두 부분으로 나눴습니다.(Vue.js와 동일한 인터페이스입니다.) data는 직접 '관찰'하거나 함수를 통해 '관찰' 가능한 반응형 변수들을 모아두고, methods는 그런 변수가 변경되면 해당 함수를 호출하여 필요한 결과값을 전달하기 위한 함수를 모아놓았습니다. 그래서 methods의 i18n 함수 내부에서 참조하는 this.locale은 data의 locale을 ```watch```하도록 할건데요, 그러기 위해 bindAsText에서 필요한 key를 찾아 stream을 구독하는 로직이 아래처럼 변경되었습니다.

```javascript
/**
 * target 객체에 mustache 구문 내 변수 이름으로 속성을 추가하고 해당 엘리먼트에 바인딩한다. 
 **/
function bindAsText(target, node) {
  const textContent = node.textContent;
  const regex = /{{[^}]+}}/g;
  const key = textContent.match(regex)[0].match(/[^{}]+/g)[0];
    
  return getObservable(target, key)
    .subscribe(value => node.textContent = textContent.replace(regex, value));
}
```
+ 예제 코드 5-2

{{}} 내부에 변수명 뿐만 아니라 함수 호출을 위해 괄호나 쉼표가 올 수 있기 때문에 정규식이 ```/{{\w}}/g```에서 ```/{{[^}]+}}/g```로 변경되었고, watch를 바로 호출 할 수 없어 getObservable을 통해 data observable인지 method observable인지를 구분했습니다. 

```javascript
function getObservable(target, key) {
  const propertyName = key.match(/^\w+/)[0];
  const params = (key.match(/\([^\)]+\)/g) || [''])[0].match(/[^,()]+/g);

  if(target.data.hasOwnProperty(propertyName))
    return getDataObservable(target, propertyName);
      
  if(target.methods.hasOwnProperty(propertyName))
    return getMethodsObservable(target, propertyName, params);
    
  console.warn('cannot find key', key);
}
```
+ 예제 코드 5-3

getObservable에서는 propertyName과 함수 매개변수 용 params를 분리하고, hasOwnProperty를 통해 data와 methods로 분기처리 하였습니다. getDataObservable에서는 이전처럼 watch stream을 반환해주면 되고 getMethodsObservable은 참조 변수가 변경되면 해당 함수를 실행하여 결과값을 전달하는 stream을 반환하면 됩니다. 

```javascript
function getDataObservable(target, property) {
  return watch(target, property);
}

function getMethodsObservable(target, property, params) {
  const func = target.methods[property];
  const funcStr = func.toString();
  const data$ = funcStr                 // 함수 코드
    .match(/this\.\w+/g)                // "this.variable" 텍스트 매칭
    .map(k => k.replace(/^this\./, '')) // 매칭된 텍스트에서 "this." 제거
    .map(k => getObservable(target, k)) // 해당 변수의 Observable 배열로 변환
    .reduce((a$, b$) => a$.combineLatest(b$)); // combineLatest를 이용해 하나의 Stream으로 변환

  return data$
    .map(_ => func.apply(target, params));
}
```
+ 예제 코드 5-3

[Function.prototype.toString](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/toString)은 함수 코드를 string으로 반환해 주는데요, 거기서 ```this.variable```을 정규식을 이용해 바인딩 해야 될 변수명을 알 수 있습니다. 바인딩 변수가 하나 이상일 수 있으니 찾은 변수를 getObservable을 이용해 모두 Observable로 변환 후 combineLatest를 통해 하나의 Observable로 합칩니다. 이렇게 하나 이상의 Observable을 합칠 수 있는 operator는 combineLatest이외에도 zip과 merge, concat, join등이 있는데요, zip은 합친 Observable 모두가 변경되어야 다음 stream으로 전달되고, merge는 하나만 변경되도 다음 stream으로 전달되는 등 동작 방식이 조금씩 다릅니다. 그래서 적절한 operator를 사용하기 위해 
