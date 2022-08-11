# TypeScript 

## 초과 프로퍼티 검사
```javascript
function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ colour: "red", width: 100 });
}
```
`createSquare` 의 매개변수가 `color` 가 아니라 `colour` 로 잘못 전달된 것에 집중해보자.
이 경우 JavaSript에선 오류가 발생하지 않는다.

`width` 프로퍼티는 적합하고, `color` 프로퍼티는 없고, 추가 `colour` 프로퍼티는 중요하지 않기 때문이다.

하지만, TypeScript에서는 버그가 있을 수 있다고 생각한다.

객체 리터럴은 다른 변수에 할당할 때나 인수로 전달할 때, 특별한 처리를 받고 초과 프로퍼티 검사를 받는다.

만약 객체 리터럴이 대상 타입이 갖고있지 않은 프로퍼티를 갖고 있으면 에러가 발생하게 된다.

```javascript
interface SquareConfig {
  color?: string;
  width?: number;
  [propName: string]: any;
}
```
  초과 프로퍼티 검사를 피하기 위해 사용하는 방법중에 하나다. 
  
  추가 프로퍼티가 있음을 확신한다면 `[propName: string]: any;` 와 같이 추가 프로퍼티에 대한 index signatuer를 추가해 준다.
  
  하지만 위 방법 처럼 검사를 피하는 방법은 시도하지 않는것이 좋다.
  
  초과 프로퍼티 에러의 대부분은 실제 버그이기 때문이다.
  
  만약 옵션 백 같은 곳에서 초과 프로퍼티 검사 문제가 발생하면, 타입 정의를 조정해야 할 필요가 있다.

## 함수

### 선택적 매개변수와 기본 매개변수

```javascript
function buildName(firstName: string, lastName: string) {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // 오류, 너무 적은 매개변수
let result2 = buildName("Bob", "Adams", "Sr.");  // 오류, 너무 많은 매개변수
let result3 = buildName("Bob", "Adams");         // 정확함
```
TypeScript 에서는 모든 매개변수가 함수에 필요하다고 가정한다.
이것은 `null` 이나 `undefined`를 줄 수 없다는 걸 의미하는 것이 아니라 컴파일러는 각 매개변수에 대해 사용자가 값을 제공했는지를 검사 한다는 것이다.
그렇기에 함수에 주어진 인자의 수는 함수가 기대하는 매개변수의 수와 일치해야 한다.

JavaScript 에서는 위의 설명과 다르게 매개변수에 값을 주지 않아도 된다.
그럴 경우 그 값은 `undefined` 가 된다.

TypeScript에서도 위와 같이 선택적 매개변수를 원한다면 매개변수 이름 끝에 `?`를 붙임으로써 해결할 수 있다. 그 예시를 아래에서 보자.

```javascript
function buildName(firstName: string, lastName?: string) {
  if (lastName)
    return firstName + " " + lastName;
  else
    return firstName;
}

let result1 = buildName("Bob"); // 바르게 동작한다. (lastName == undefined)
let result2 = buildName("Bob", "Adams", "Sr."); // 오류
let result3 = buildName("Bob", "Adams"); // 정확함
```

위처럼 제공하지 않은 매개변수 값에 대해서 `undefined`로 적용할 수도 있지만, 값을 제공하지 않아 `undefined`로 했을 때 할당 될 매개 변수의 값을 미리 지정해 놓을 수도 있다.

그 예시를 아래에서 보자.

``` javascript
function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // 올바르게 동작, "Bob Smith" 반환
let result2 = buildName("Bob", undefined);       // 여전히 동작, 역시 "Bob Smith" 반환
let result3 = buildName("Bob", "Adams", "Sr.");  // 오류, 너무 많은 매개변수
let result4 = buildName("Bob", "Adams");         // 정확함
```

### 나머지 매개변수

필수, 선택적, 기본 매개변수는 한번에 하나의 매개변수만을 가지고 이야기 한다.
때로는 다수의 매개변수를 그룹 지어 작업하기를 원하거나, 함수가 최종적으로 얼마나 많은 매개변수를 취할지 모를 때도 있을 것이다.

이 때, JavaScript에서는 모든 함수 내부에 위치한 `arguments` 라는 변수를 사용해 직접 인자를 가지고 작업한다.

TypeScript에서는 이 인자들을 하나의 변수로 모을 수 있다.

```javascript
function buildName(firstName: string, ...restOfName: string[]) {
    return firstName + " " + restOfName.join(" ");
}

// employeeName 은 "Joseph Samuel Lucas MacKinzie" 가 될것입니다.
let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

나머지 매개변수는 선택적 매개변수들의 수를 무한으로 취급한다. 나머지 매개변수로 인자들을 넘겨줄 때는 원하는 만큼 넘겨줄 수 있다. 또는 아무것도 넘기지 않을 수도 있다.

컴파일러는 생략 부호 (...) 뒤의 이름으로 전달된 인자 배열을 빌드하여 함수에서 사용할 수 있도록 한다.

생략 부호는 나머지 매개변수가 있는 함수의 타입에도 사용된다.

```javascript
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfname.join(" ");
}

let buildNameFun: (fname: string, ...rest: string[]) => string = buildName;
```

## this

### this 와 화살표 함수

JavaScript 에서, `this`는 함수가 호출될 때 정해지는 변수이다. 매우 강력하고 유연한 기능이지만 이것은 항상 함수가 실행되는 콘텍스트에 대해 알아야 한다는 수고가 생긴다.

특히 함수를 반환하거나 인자로 넘길 때의 혼란스러움은 악명 높다.

예시를 보자.

```javascript
let deck = {
  suits: ["hearts", "spades", "clubs", "diamonds"],
  card: Array(52),
  createCardPicker: function() {
    return function() {
      let pickedCard = Math.floor(Math.random() * 52);
      let pickedSuit = Math.floor(pickedCard / 13);
      
      return {suit: this.suits[pickedSuit], card: pickedCard % 13};
    }
  }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

`createCardPicker`가 자기 자신을 반환하는 함수임을 주목하라. 이 예제를 실행 시키면 오류가 발생할 것이다.

`createCardPicker` 에 의해 생성된 함수에서 사용 중인 `this`가 `deck` 객체가 아닌 `window`에 설정 되었기 때문이다.

최상위 레벨에서의 비-메서드 문법의 호출은 `this`를 `window`로 한다.

이 문제는 나중에 사용할 함수를 반환하기 전에 바인딩을 알맞게 하는 것으로 해결할 수 있다.

이 방법대로라면 나중에 사용하는 방법에 상관없이 원본 `deck` 객체를 계속해서 볼 수 있다. 이를 위해 함수의 표현식을 ES6의 화살표 함수로 바꿀 것이다.

화살표 함수는 함수가 호출 된 곳이 아닌 함수가 생성된 쪽의 this를 캡처한다.

```javascript
let deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        // NOTE: 아랫줄은 화살표 함수로써, 'this'를 이곳에서 캡처할 수 있도록 한다
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

### this 매개변수

화살표 함수를 활용해 this에 발생하는 에러를 없앴지만 `this.suits[pickedSuit]` 의 타입은 여전히 `any` 이다.

`this`가 객체 리터럴 내부의 함수에서 왔기 때문인데, 이것을 고치기 위해 명시적으로 `this` 매개변수를 줄 수 있다. `this` 매개변수는 함수의 매개변수 목록에서 가장 먼저 나오는 가짜 매개변수이다.

명확하고 재사용하기 쉽게 `Card`와 `Deck` 두 가지 인터페이스 타입들을 예시에 추가해 보자.

```javascript
interface Card {
  suit: string;
  card: number;
}

interface Deck {
  suits: string[];
  cards: number[];
  createCardPicker(this: Deck): () => Card;
}

let deck: Deck = {
  suits: ["hearts", "spades", "clubs", "diamonds"],
  cards: Array(52),
  
  createCardPicker: function(this: Deck) {
    return () => {
      let pickedCard = Math.floor(Math.random() * 52);
      let pickedSuit = Math.floor(pickedCard / 13);
      
      return {suit: this.suits[pickedSuit], card: pickedCard % 13};
    }
  }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
  
```

이제 TypeScript는 `createCardPicker`가 `Deck` 객체에서 호출된다는 것을 알게 됐다. 
이것은 `this`가 `any` 타입이 아니라 `Deck` 타입이며 따라서 `--noImplicitThis` 플래그가 어떤 오류도 일으키지 않는다는 것을 의미한다.

## 리터럴 타입

### 리터럴 타입 좁히기

`var` 또는 `let`으로 변수를 선언할 경우 이 변수의 값이 변경될 가능성이 있음을 컴파일러에게 알린다. 반면, `const`로 변수를 선언하게 되면 TypeScript에게 이 객체는 절대 변경되지 않음을 알린다.

무한한 수의 잠재적 케이스를 유한한 수의 잠재적 케이스로 줄여나가는 것을 타입 좁히기라 한다.

### 문자열 리터럴 타입

실제로 문자열 리터럴 타입은 유니언 타입, 타입 가드 그리고 타입 별칭과 잘 결합된다. 이런 기능을 함께 사용하여 문자열로 enum과 비슷한 형태를 갖출 수 있다.
```javascript
type Easing = "ease-in" | "ease-out" | "ease-in-out";

class UIElement {
  animate(dx: number, dy: number, easing: Easing) {
    if (easing === "ease-in"){
     // ... 
    } else if (easing === "ease-out") {
    } else if (easing === "ease-in-out") {
    } else {
      // type을 무시하면 여기로.
    }
  }
}

let button = new UIElement();
button.animate(0, 0, "ease-in");
button.animate(0, 0, "uneasy"); // Easing에 포함되지 않음 -- error
```
허용된 세 개의 문자열이 아닌 다른 문자열을 사용하게 되면 오류가 발생한다.

문자열 리터럴 타입은 오버로드를 구별하는 것과 동일한 방법으로 사용될 수 있다.

```javascript
function createElement(tagName: "img"): HTMLImageElement;
function createElement(tagName: "input"): HTMLInputElement;
// ... 추가적인 중복 정의들 ... 
function createElement(tagName: string): Element {
  // ... 여기에 로직 추가 ...
}
```

### 숫자형 리터럴 타입

TypeScript에는 위의 문자열 리터럴과 같은 역할을 하는 숫자형 리터럴 타입도 있다.

```javascript
function rollDice(): 1 | 2 | 3 | 4 | 5 | 6 {
  return (Math.floor(Math.random() * 6) + 1) as 1 | 2 | 3 | 4 | 5 | 6;
}

const result = rollDice();
```

이는 주로 설정값을 설명할 때 사용된다.

```javascript
// loc/lat 좌표에 지도를 생성한다.
declare function setupMap(config: MapConfig): void;
// ---생략---
Interface MapConfig {
  lng: number;
  lat: number;
  tileSize: 8 | 16 | 32;
}

setupMap({ lng: -73.935242, lat: 40.73061, tileSize: 16 });
```

## 유니언과 교차 타입

### 유니언 타입

어떠한 function을 사용할 때 매개변수의 type을 `any`로 줘야하는 상황이 발생할 수 있고, 매개변수 type이 `any`일 경우에는 여러 런타임 오류의 원인이 되기도 한다.

```typescript
declare function padLeft(value: string, padding: any): string;
// ---생략---
// 컴파일 타임에는 통과하지만, 런타임에는 오류가 발생합니다.
let indentedString = padLeft("Hello world", true);
```

이렇게 하나의 매개변수에 여러 type이 필요한 경우 유니언 타입을 사용하게 된다.
아래 예시를 보자.

```typescript
function padLeft(value: string, padding: string | number) {
	if (typeof padding === "number") {
    return Array(padding + 1).join(" ") + value;
  }
  if (typeof padding === "string") {
    return padding + value;
  }
  throw new Error(`Expected string or number, got '${padding}'.`);
}

padLeft("Hello world", 4);
```

### 공통 필드를 갖는 유니언

유니언 타입인 값이 있으면, 유니언에 있는 모든 타입에 공통인 멤버들에만 접근할 수 있다.

```typescript
interface Bird {
  fly(): void;
  layEggs(): void;
}

interface Fish {
  swim(): void;
  layEggs(): void;
}

declare function getSmallPet(): Fish | Bird;

let pet = getSmallPet();
pet.layEggs();

// 두 개의 잠재적인 타입 중 하나에서만 사용할 수 있다.
pet.swim();
```

여기서 `pet`은 `Fish`와 `Bird` 타입 모두 가질 수 있다. 이 때, `pet.swim()`은 `Fish` 타입에만 존재하기 때문에 `pet`이 `Bird` 타입을 가진다면 오류가 발생하게 된다.

### 유니언 구별하기

공통된 하나의 필드를 가지는 유니언 타입을 만들고, `switch` 를 이용해 이를 활용 할 수 있다.

```typescript
type NetworkLoadingState = {
  state: "loading";
};

type NetworkFailedState = {
  state: "failed";
  code : number;
};

type NetworkSuccessState = {
  state: "success";
  response: {
    title: string;
    duration: number;
    summary: string;
  };
};

type NetworkState =
  | NetworkLoadingState
  | NetworkFailedState
  | NetworkSuccessState;

function networkStatus(state: NetworkState): string {
  // state 의 type에 따라 switch문을 활용해 다른 작업을 해준다.
  switch (state.state) {
    case "loading":
      return "Downloading...";
    case "failed":
      return `Error ${state.code} downloading`;
    case "success":
      return `Downloaded ${state.response.title} - ${state.response.summary}`;
  }
}
```

### 교차 타입

유니언 타입이 | (or) 라면 교차 타입은 & (and) 이다. 기존 타입을 합쳐 필요한 기능을 모두 가진 단일 타입을 얻는데에 사용한다.

예를 들어, 일관된 에러를 다루는 여러 네트워크 요청이 있으면, 에러 핸들링을 분리하여 하나의 응답 타입에 대응하는 결합된 자체 타입으로 만들 수 있다.

```typescript
interface ErrorHandling {
  success: boolean;
  error?: { message: string };
}

interface ArtworksData {
  artworks: { title: string }[];
}

interface ArtistData {
  artists: { name: string }[];
}

type ArtworksResponse = ArtworksData & ErrorHandling;
type ArtistsResponse = ArtistsData & ErrorHandling;

const handleArtistsResponse = (response: ArtistsResponse) => {
  if (response.error) {
    console.error(response.error.message);
    return;
  }
  
  console.log(response.artists);
};
```

### 교차를 통한 믹스인

교차는 믹스인 패턴을 실행하기 위해 사용된다.

```typescript
//봐도 모르겠다 이건
class Person {
  constructor(public name: string) {}
}

interface Loggable {
  log(name: string): void;
}

class ConsoleLogger implements Loggable {
  log(name: string) {
    console.log(`Hello, I'm ${name}.`);
  }
}

// 두 객체를 받아 하나로 합칩니다.
function extend<First extends {}, Second extends {}>(
  first: First,
  second: Second
): First & Second {
  const result: Partial<First & Second> = {};
  for (const prop in first) {
    if (first.hasOwnProperty(prop)) {
      (result as First)[prop] = first[prop];
    }
  }
  for (const prop in second) {
    if (second.hasOwnProperty(prop)) {
      (result as Second)[prop] = second[prop];
    }
  }
  return result as First & Second;
}

const jim = extend(new Person("Jim"), ConsoleLogger.prototype);
jim.log(jim.name);
```

## 클래스

### 간단한 클래스 기반 예제

```typescript
class Greeter {
  greeting: string;
  constructor(message: string){
    this.greeting = message;
  }
  greet() {
    return "Hello, " + this.greeting;
  }
}

let greeter = new Greeter("world");
```

`Greeter` 클래스는 3개의 멤버를 가지고 있다. `greeting` 프로퍼티, 생성자, 그리고 `greet` 메서드 이다.

클래스 안에서 클래스의 멤버를 참조할 때 `this.` 를 앞에 덧붙인다. 이것은 멤버에 접근하는 것을 의미한다.

`new`를 사용하여 `Greeter`클래스의 인스턴스를 생성한다. 이 코드는 이전에 정의한 생성자를 호출하여 `Greeter` 형태의 새로운 객체를 만들고, 생성자를 실행해 초기화한다.

### 상속

TypeScript에서는, 일반적인 객체-지향 패턴을 사용할 수 있다. 클래스-기반 프로그래밍의 가장 기본적인 패턴 중 하나는 상속을 이용하여 이미 존재하는 클래스를 확장해 새로운 클래스를 만들 수 있다는 것이다.

예제를 살펴보자.

```typescript
class Animal {
  move(distanceInMeters: number = 0) {
    console.log(`Animal moved ${distanceInMeters}m.`;
  }
}

class Dog extends Animal {
  bark() {
    console.log('왈왈!');
  }
}

const dog = new Dog();
dog.bark();
dog.move(10);
dog.bark();
```

상속 기능을 보여주는 가장 기본적인 예제이다. 자식 클래스는 기초 클래스로부터 프로퍼티와 메서드를 상속 받는다.

여기서, `Dog`는 `extends` 키워드를 사용해 `Animal`이라는 기초 클래스로부터 파생된 클래스 이다.

`Dog`는 `Animal`의 기능을 확장하기 때문에, `bark()`와 `move()` 를 모두 가진 `Dog` 인스턴스를 생성할 수 있다.

또 다른 예제를 보자.

```typescript
class Animal {
  name: string;
  constructor(theName: string) { this.name = theName; }
  move(distanceInMeters: number = 0) {
    console.log(`Animal moved ${distanceInMeters}m.`);
  }
}

class Snake extends Animal {
  constructor(name: string) { super(name); }
  move(distanceInMeters = 5) {
    console.log("Slithering...");
    super.move(distanceInMeters);
  }
}

class Horse extends Animal {
  constructor(name: string) { super(name); }
  move(distanceInMeters = 45) {
    console.log("Galloping...");
    super.move(distanceInMeters);
  }
}

let sam = new Snake("Sammy the Python");
let tom: Animal = new Horse("Tommy the palomino")

sam.move();
tom.move(34);
```

이 예제에서는 앞에서 언급하지 않은 몇가지 기능을 다룬다.

이전 예제와 한 가지 다른 부분은 파생된 클래스의 생성자 함수는 기초 클래스의 생성자를 실행할 `super()`를 호출해야 한다는 점이다.

더욱이 생성자 내에서 `this`에 있는 프로퍼티에 접근하기 전에 `super()`를 먼저 호출 해야한다. 이 부분은 TypeScript에서 중요한 규칙이다.

또한 이 예제는 기초 클래스의 메서드를 하위클래스에 특화된 메서드로 오버라이드 하는 방법을 보여준다.

여기서 `Snake`와 `Horse`는 `Animal`의 `move`를 오버라이드 해서 각각 클래스의 특성에 맞게 기능을 가진 `move`를 생성한다.

`tom`은 `Animal`로 선언되었지만, `Horse`의 값을 가지므로 `tom.move(34)` 는 `Horse`의 오버라이딩 메서드를 호출한다.

## Public, Private, Protected 지정자

### 기본적으로 Public

TypeScript에서는 기본적으로 각 멤버가 `public` 이다.

### TypeScript의 private 이해하기

TypeScript에는 멤버를 포함하는 클래스 외부에서 이 멤버에 접근하지 못하도록 멤버를 `private`로 표시하는 방법이 있다.

```typescript
class Animal {
  private name: string;
  constructor(theName: string) { this.name = theName; }
}

new Animal("Cat").name; // Error: 'name'은 private으로 선언 되어 있다.
```

TypeScript는 구조적인 타입 시스템이다. 두개의 다른 타입을 비교할 때 어디서 왔는지 상관없이 모든 멤버의 타입이 호환 된다면, 그 타입들 자체가 호환 가능하다고 말한다.

그러나 `private` 및 `protected` 멤버가 있는 타입들을 비교할 때는 타입을 다르게 처리한다.

호환된다고 판단되는 두 개의 타입 중 한 쪽에서 `private` 멤버를 가지고 있다면, 다른 한 쪽도 무조건 동일한 선언에 `private` 멤버를 가지고 있어야 한다.

그리고 이것은 `protected` 멤버에도 동일하게 적용 된다.

실제로 어떻게 작동하는지 보기위해 다음 예제를 살펴보자.

```typescript
class Animal {
  private name: string;
  constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
  constructor() { super("Rhino"); }
}

class Employee {
  private name: string;
  constructor(theName: string) { this.name = theName; }
}

let animal = new Animal("Goat");
let rhino = new Rhino();
let employee = new Employee("Bob");

animal = rhino;
animal = employee;
// Error: 'Animal'과 'Employee'는 호환될 수 없다.
```

`Animal`과 `Rhino`는 `Animal`의 `private name:string` 이라는 동일한 선언으로 부터 `private` 부분을 공유하기 때문에 호환이 가능하다. 하지만 `Employee`의 경우는 그렇지 않다.

### TypeScript의 protected 이해하기

`protected`는 `protected`로 선언된 멤버를 파생된 클래스 내에서 접근할 수 있다는 점만 제외하면 `private` 지정자와 매우 유사하게 동작한다.

예를 들면,

```javascript
class Person {
  protected name: string;
  constructor(name: string) {this.name = name;}
}

class Employee extends Person {
  private department: string;

  constructor(name: string, department: string){
      super(name);
      this.department = department;
  }

  public getElevatorPitch() {
    return `Hello, my name is ${this.name} and I work in ${this.department}.`;
  }
}

let chan = new Employee("Chan", "WINS");
console.log(chan.getElevatorPitch());
console.log(chan.name); // Error: protected -> 자식클래스 에서만 사용가능하다.
```

`Person` 외부에서 `name`을 사용할 수 없지만, `Employee`는 `Person`에서 파생되었기 때문에 `Employee`의 인스턴스 메서드 내에서는 여전히 사용할 수 있다.

이는 생산자에 대해서도 동일하게 적용이 가능하다.

### 읽기전용 지정자

`readonly` 키워드를 사용하여 프로퍼티를 읽기전용으로 만들 수 있다. 읽기 전용 프로퍼티들은 선언 또는 생성자에서 초기화 해야한다.

```typescript
class Octopus {
  readonly name: string;
  readonly numberOfLegs: number = 8;
  constructor (theName: string) {
    this.name = theName;
  }
}

let dad = new Octopus("Man with the 8 strong legs");
dad.name = "Man with the 3-piece suit"; // Error! name is a read_only property
```

### 매개변수 프로퍼티

위의 예제에 `Octopus` 클래스 내에서 `name` 이라는 읽기전용 멤버와 `theName` 이라는 생성자 매개변수를 선언했다. 이는 `Octopus`의 생성자가 수행된 후에 `theName`의 값에 접근하기 위해서다.

매개변수 프로퍼티를 사용하면 한 곳에서 멤버를 만들고 초기화할 수 있다. 다음은 매개변수 프로퍼티를 사용한 더 개정된 `Octopus` 클래스 이다.

```typescript
class Octopus {
  readonly numberOfLegs: number = 8;
  constructor(readonly name: string) {
  }
}
```

생성자에서 `readonly name: string` 파라미터를 사용하여 선언과 할당을 한 곳으로 통합했다.

### 접근자

TypeScript는 객체의 멤버에 대한 접근을 가로채는 방식으로 `getters`와 `setters`를 지원한다.

이를 통해 각 객체의 멤버에 접근하는 방법을 세밀하게 제어할 수 있다.

아래 예시와 같이 `getters`와 `setters`가 없는 간단한 클래스를 `get`과 `set`을 사용하도록 변환해보자.

```typescript
class Employee {
  fullName: string;
}

let employee = new Employee();
employee.fullName = "Kim Yeonchan";
if (employee.fullName) {
  console.log(employee.fullName);
}
```

임의로 `fullName`을 직접 설정할 수 있도록 허용하는 것은 매우 편리하지만, `fullName`이 설정될 때 몇 가지 제약 조건이 적용되는 것을 원할 수 있다.

여기서는 데이터베이스 필드의 최대 길이와 호환되는지 확인하기 위해 `newName`의 길이를 확인하는 `setter`를 추가한다.

만약 최대 길이를 초과한다면, 클라이언트 코드에 문제가 있다는 것을 알리기 위해 오류를 발생시킨다.

기존의 기능을 유지하기 위해, `fullName`을 수정하지 않는 간단한 getter도 추가한다.

```typescript
const fullNameMaxLength = 10;

class Employee {
  private _fullName: string;

  get fullName(): string {
    return this._fullName;
  }

  set fullName(newName: string) {
    if (newName && newName.length > fullNameMaxLength) {
      throw new Error("fullName has a max length of " + fullNameMaxLength);
    }

    this._fullName = newName;
  }
}

let employee = new Employee();
employee.fullName = "Yeonchan"
if (employee.fullName) {
  console.log(employee.fullName);
}
```

접근자가 값의 길이를 확인하고 있는지 검증하기 위해서, 10자가 넘는 이름을 할당하고 오류가 발생함을 확인할 수 있다.

### 전역 프로퍼티 (Static Properties)

우리는 인스턴스가 아닌 클래스 자체에서 보이는 전역 멤버를 생성할 수 있다.

이 예제에서는 모든 grid의 일반적인 값이기 때문에 origin에 `static`을 사용한다.

각 인스턴스는 클래스 이름을 앞에 붙여 이 값에 접근할 수 있다. 인스턴스 접근 앞에 `this.`를 붙이는 것과 비슷하게 여기선 전역 접근 앞에 `Grid.`을 붙인다.

```typescript
class Grid {
  static origin = {x:0, y:0};
  calculateDistanceFromOrigin(point: {x: number; y: number;}) {
    let xDist = (point.x - Grid.origin.x);
    let yDist = (point.y - Grid.origin.y);
    return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
  }
  constructor (public scale: number) { }
}

let grid1 = new Grid(1.0); // 1x scale
let grid2 = new Grid(5.0); // 5x scale

console.log(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
console.log(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```

### 추상 클래스

추상 클래스는 다른 클래스들이 파생될 수 있는 기초 클래스이다.

직접 인스턴스화할 수 없으며, 추상클래스는 인터페이스와 달리 멤버에 대한 구현 세부 정보를 포함할 수 있다.

`abstract` 키워드는 추상 클래스 뿐만 아니라 추상 클래스 내에서 추상 메서드를 정의하는데 사용된다.

```typescript
abstract class Animal {
  abstract makeSound(): void;
  move(): void {
    console.log("roaming the earth...");
  }
}
```

추상 클래스 내에서 추상으로 표시된 메서드는 구현을 포함하지 않으며 반드시 파생된 클래스에서 구현되어야 한다. 

추상 메서드는 대체적으로 인터페이스 메서드와 비슷한 문법을 공유하는데, 차이점은 추상 메서드는 반드시 `abstract` 키워드를 포함해야 한다는 것이다.

그리고 선택적으로 접근 지정자를 포함할 수 있다.

```typescript
abstract class Department {

  constructor(public name: string) {

  }

  printName(): void {
    console.log("Department name: " + this.name);
  }

  abstract printMeeting(): void; // 반드시 파생된 클래스에서 구현되어야 한다.
}

class accountingDepartment extends Department {

  constructor() {
    super("Accounting and Auditing"); // 파생된 클래스의 생성자는 반드시 super를 호출해야 한다.
  }

  printMeeting(): void {
    console.log("The Accounting Department meets each Mondat at 10am.");
  }

  generateReports(): void {
    console.log("Generating accounting reports...");
  }
}

let department: Department; // 추상 타입의 레퍼런스를 생성
department = new Department(); // Error: abstract 클래스는 인스턴스화할 수 없다.
department = new accountingDepartment(); // 추상이 아닌 하위 클래스를 생성하고 할당한다.
department.printName();
department.printMeeting();
department.generateReports(); // Error: 선언된 추상 타입에 메서드가 존재하지 않는다.
```

## 고급 기법

### 생성자 함수 (Constructor functions)

TypeScript에서는 클래스를 선언하면 실제로 여러 개의 선언이 동시에 생성된다.

첫번째로 클래스의 인스턴스 타입이다.

```typescript
class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }
  greet() {
    return "Hello, " + this.greeting;
  }
}

let greeter: Greeter;
greeter = new Greeter("world");
console.log(greeter.greet());
```

`let greeter: Greeter`라고 할 때, `Greeter` 클래스의 인스턴스 타입으로 `Greeter`를 사용한다. 객체 지향 언어에서 자연스러운 성질이다.

또한 생성자 함수라고 불리는 또 다른 값을 생성하고 있다.

생성자 함수는 클래스의 인스턴스를 `new` 할 때 호출되는 함수이다.

그 예시를 보여주는 예제를 살펴보자.

```typescript
let Greeter = (function () {
  function Greeter(message) {
    this.greeting = message;
  }
  Greeter.prototype.greet = function () {
    return "Hello, " + this.greeting;
  };
  return Greeter;
})();

let greeter;
greeter = new Greeter("world");
console.log(greeter.greet());
```

여기서, `let Greeter`는 생성자 함수를 할당 받는다. `new`를 호출하고 이 함수를 실행할 때, 클래스의 인스턴스를 얻는다.

또한, 생성자 함수는 클래스의 모든 전역 변수들을 포함하고 있다. 각 클래스를 생각하는 또 다른 방법은 인스턴스 측면과 정적 측면이 있다.

인스턴스 측면과 정적 측면의 차이를 보여주는 예제를 보자.

```typescript
class Greeter {
  static standardGreeting = "Hello, there";
  greeting: string;
  greet() {
    if (this.greeting) {
      return "Hello, " + this.greeting;
    }
    else {
      return Greeter.standardGreeting;
    }
  }
}

let greeter1: Greeter;
greeter1 = new Greeter();
console.log(greeter1.greet()); // "Hello, there"

let greeterMaker: typeof Greeter = Greeter;
greeterMaker.standardGreeting = "Hey there!";

let greeter2: Greeter = new greeterMaker();
console.log(greeter2.greet()); // "Hey there!";
```

이 예제에서 `greeter1`은 이전과 비슷하게 동작한다. `Greeter` 클래스를 인스턴스화 하고 그 객체를 사용하는 것이다.

`greeterMaker`는 클래스를 직접 사용한다. `greeterMaker`라는 변수를 생성했는데, 이 변수는 클래스 자체를 유지하거나 생성자 함수를 다르게 설명한다.

여기서 `typeof Greeter`를 사용하여 인스턴스 타입이 아닌 `Greeter` 클래스 자체의 타입을 제공한다. 더 정확하게 말하면 생성자 함수의 타입인 `Greeter`라는 심볼의 타입을 제공한다.

이 타입은 `Greeter` 클래스의 인스턴스를 만드는 생성자와 함께 Greeter의 모든 정적 멤버를 포함할 것이다.

`greeterMaker`에 `new`를 사용함으로써 `Greeter`의 새로운 인스턴스를 생성하고 이전과 같이 호출한다.

### 인터페이스로써 클래스 사용하기

앞서 언급 했듯이, 클래스 선언은 클래스의 인스턴스를 나타내는 타입과 생성자 함수라는 두가지를 생성한다.

클래스는 타입을 생성하기 때문에 인터페이스를 사용할 수 있는 동일한 위치에서 사용할 수 있다.

```typescript
class Point {
  x: number;
  y: number;
}

interface Point3d extends Point {
  z: number;
}

let point3d: Point3d = {x: 1, y: 2, z: 3};
```

## 열거형

열거형을 사용하면 의도를 문서화 하거나 구분되는 사례 집합을 좀 더 쉽게 만들 수 있다.

TypeScript는 숫자와 문자열 기반 열거형을 제공한다.

### 숫자 열거형 (Numeric enums)

열거형은 `enum` 키워드를 사용해 정의할 수 있다.

```typescript
enum Direction {
  Up = 1,
  Down,
  Left,
  Right,
}
```

위 코드에서 `Up`이 `1`로 초기화된 숫자 열거형을 선언했다. 그 지점부터 뒤따르는 멤버들은 `auto-increment`된 값을 갖는다. 

즉 `Up`은 `1`, `Down` = `2`, `Left` = `3`, `Right` = `4`의 값을 가진다.

원한다면 초기화 하지 않아도 된다.

```typescript
enum Direction {
  Up,
  Down,
  Left,
  Right,
}
```

위의 경우 `Up` = `0`으로 시작해 `auto-increment`된 값을 가지게 된다.

초기화 하지 않은 열거형의 경우 값이 같은 열거형의 다른 값과 구별해야 하는 경우에 유용하게 쓸 수 있다.

열거형을 사용하는 방법은 간단하게 열거형 자체에서 프로퍼티로 모든 멤버에게 접근하며, 열거형의 이름을 사용해 타입을 선언한다.

```typescript
enum Response {
  No = 0,
  Yes = 1,
}

function respond(recipient: string, message: Response): void {

}

respond("Princess Caroline", Response.Yes)
```

### 문자열 열거형 (String enums)

문자열 열거형은 유사한 개념이지만 아래 설명된 것과 같이 런타임에서 열거형의 동작이 약간 다르다.

문자열 열거형에서 각 멤버들은 문자열 리터럴 또는 다른 문자열 열거형 멤버로 상수 초기화 해야한다.

```typescript
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}
```

문자열 열거형은 `auto-increment`기능이 없지만, 직렬화를 잘한다는 이점이 있다.

만약 숫자 열거형을 이용해서 디버깅하고 있고, 그 값을 읽어야 한다면 종종 그 값이 불확실한 경우가 있다. (숫자만으로는 어떤 의미인지 유의미한 정보를 제공해주지 않기 때문)

반면 문자열 열거형을 이용하면 코드를 실행할 때, 열거형 멤버에 지정된 이름과 무관하게 유의미하고 읽기 좋은 값을 이용하여 실행할 수 있다.

### 유니언 열거형과 열거형 멤버 타입

열거형의 모든 멤버가 리터럴 열거형 값을 가지면 특별한 의미로 쓰이게 된다.

첫째로, 열거형 멤버를 타입처럼 사용할 수 있다. 예를 들어, 특정 멤버는 오직 열거형 멤버의 값만 가지게 할 수 있다.

```typescript
enum ShapeKind {
  Circle,
  Square,
}

interface Circle {
  kind: ShapeKind.Circle;
  radius: number;
}

interface Square {
  kind: ShapeKind.Square;
  sideLength: number;
}

let c: Circle = {
  kind: ShapeKind.Square, // Error: ShapeKind.Circle 타입에 Square를 할당할 수 없다.
  radius: 100,
}
```

두번째로, 열거형 타입 자체가 효율적으로 각각의 열거형 멤버의 유니언이 된다는 점이다.

유니언 타입 열거형을 사용하면 타입 시스템이 열거형 자체에 존재하는 정확한 값의 집합을 알고 있다는 사실을 활용할 수 있다는 점만 알아두면 된다.

이 때문에 TypeScript는 값을 잘못 비교하는 버그를 잡을 수 있다.

```typescript
enum E {
  Foo,
  Bar,
}

function f(x: E) {
  if (x !== E.Foo || x !== E.Bar) {
  	// Error: 위 조건은 항상 True를 만족한다.
  }
}
```

### 런타임에서 열거형

열거형은 런타임에 존재하는 실제 객체이다. 예를 들어 아래와 같은 열거형은

```typescript
enum E {
  X, Y, Z
}
// 실제로 아래와 같이 함수로 전달될 수 있다.
function f(obj: { Z: number }) {
  return obj.Z;
}
// E가 Z라는 프로퍼티를 가지고 있기 때문에 동작하는 코드다.
console.log(f(E)); // 2를 출력.
```

### 컴파일 시점에서 열거형

열거형이 런타임에 존재하는 실제 객체라도, `keyof` 키워드는 일반적인 객체에서 기대하는 동작과 다르게 동작한다.

대신 `keyof typeof`를 사용하면 모든 열거형의 키를 문자열로 나타내는 타입을 가져온다.

```typescript
enum LogLevel {
    ERROR, WARN, INFO, DEBUG
}

// 같은 의미의 코드 : type LogLevelStrings = 'ERROR' | 'WARN' | 'INFO' | 'DEBUG';
type LogLevelStrings = keyof typeof LogLevel;

function printImportant(key: LogLevelStrings, message: string) {
    const num = LogLevel[key];
    if (num <= LogLevel.WARN) {
       console.log('Log level key is: ', key);
       console.log('Log level value is: ', num);
       console.log('Log level message is: ', message);
    }
}
printImportant('WARN', 'This is a message');
```

### 역 매핑

숫자 열거형 멤버는 객체를 생성하는 것 외에도 열거형 값에서 열거형 이름으로 역 매핑을 받기도 한다.

```typescript
enum Enum {
  A
}

let a = Enum.A;
let nameOfA = Enum[a]; // "A"
```

이는 숫자 열거형에서만 가능하며 문자열 열거형은 역 매핑을 생성하지 않는다.

### const 열거형

종종 열거형의 요구사항이 좀 더 엄격해지는데, 열거형 값에 접근할 때 추가로 생성된 코드 및 추가적인 간접 참조에 대한 비용을 피하기 위해 `const` 열거형을 사용하곤 한다.

```typescript
const enum Enum {
  A = 1,
  B = A * 2
}
```

`const` 열거형은 상수 열거형 표현식만 사용될 수 있다.

일반 열거형과 달리 컴파일 과정에서 완전히 제거된다.

`const` 열거형이 계산된 멤버를 가지고 있지 않기 때문에 `const` 열거형은 사용하는 공간에 인라인된다.

```typescript
const enum Directions {
  Up,
  Down,
  Left,
  Right
}

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right]
/** 위 코드는 아래와 같이 컴파일 된다
var directions = [0, 1, 2, 3]; **/
```

## 제네릭

"어떠한 클래스 혹은 함수에서 사용할 타입을 그 함수나 클래스를 사용할 때 결정하는 프로그래밍 기법"

제네릭을 사용하면 단일 타입이 아닌 다양한 타입에서 작동하는 컴포넌트를 작성할 수 있다.

사용자는 제네릭을 통해 여러 타입의 컴포넌트나 자신만의 타입을 사용할 수 있다.

### 제네릭의 HELLO WORLD (identity)

`identity` 함수는 인수로 무엇이 오던 그대로 반환하는 함수이다.

`echo` 명령과 비슷하게 생각할 수 잇다.

제네릭이 없다면, `identity` 함수에 특정 타입을 주어야 한다.

```typescript
function identity(arg: number): number {
  return arg;
}
// 또는 any 타입을 사용하여 identity 함수를 기술할 수 있다.
```

`any`를 쓰는 것은 함수의 `arg`가 어떤 타입이든 받을 수 있다는 점에서 제네릭 이지만, 실제로 함수가 반환할 때 어떤 타입인지에 대한 정보를 잃게 된다.

만약 number타입을 넘긴다 해도 any 타입이 반환된다는 정보만 얻을 뿐이다.

일단 제네릭 identity 함수를 작성하고 나면, 두 가지 방법 중 하나로 호출할 수 있다.

첫 번째 방법은 함수에 타입 인수를 포함한 모든 인수를 전달하는 방법이다.

```typescript
let output = identity<string>("myString"); // 출력타입은 string이다.
```

두 번째 방법은 타입 인수 추론을 사용하는 것이다. 

즉, 우리가 전당하는 인수에 따라서 컴파일러가 자도응로 정하게 하는것이다.

```typescript
let output = identity("myString"); // 출력타입은 string이다.
```

이 방법은 코드의 가독성을 올려주지만 더 복잡한 예제에서 컴파일러가 타입을 유추할 수 없는 경우엔 명시적인 타입 인수 전달이 필요할 수 있다.

### 제네릭 타입 변수 작업

제네릭을 사용하기 시작하면, `identity`와 같은 제네릭 함수를 만들 때, 컴파일러가 함수 본문에 제네릭 타입화된 매개변수를 쓰도록 강요한다.

즉, 이 매개변수들은 실제로 모든 타입이 될 수 있는 것처럼 취급할 수 있게 된다.

```typescript
function identity<T>(arg: T): T {
  return arg;
}
```

위의 함수에서 호출 시마다 `arg`의 길이를 로그에 찍을 때,

```typescript
function loggingIdentity<T>(arg: T): T {
  console.log(arg.length);
  return arg;
}
```

제네릭에 대한 개념이 없다면 위와 같이 작성할 것입니다.

여기서 `arg`는 모든 타입이 될 수 있기 때문에, `number` 타입이 된다면 `.lengh` 멤버가 없으므로 Error가 나오게 된다.

이런 경우 아래처럼 함수가 `T`가 아닌 `T`의 배열에서 동작하도록 하는 방법이 있다.

```typescript
function loggingIdentity<T>(arg: T[]): T[] {
  console.log(arg.length);
  return arg;
}
```

만약에 number 배열을 넘기면 `T`가 `number`에 바인딩 되므로 함수는 number 배열을 얻게 된다.

이렇게 전제 타입변수를 쓰는 것 보다 하나의 타입으로써 제네릭 타입변수 `T`를 사용하는 것은 굉장한 유연함을 제공한다.
