# <mark style="background: #FF5582A6;">디자인 패턴 정의</mark>

프로그램을 설계할 때 발생했던 문제점들을 객체 간의 상호 관계 등을 이용하여 해결할 수 있도록 하나의 규약 형태로 만들어 놓은 것

# <mark style="background: #FF5582A6;">디자인 패턴의 종류</mark>

## <mark style="background: #FFB86CA6;">싱글톤 패턴</mark>

하나의 클래스에 오직 하나의 인스턴스만 가지는 패턴

하나의 클래스를 기반으로 여러 개의 개별적인 인스턴스를 만들어 이를 기반으로 로직을 만든다. (의존성은 상승하나 인스턴스 생성의 비용이 줄어든다.)

### <mark style="background: #FFF3A3A6;">JS의 싱글톤 패턴</mark>

JS에서는 `{}` 또는 new Object로 객체를 생성하는 것 만으로도 싱글톤 패턴을 구현할 수 있다. ( 다른 어떤 객체와도 같아질 수 없다. )

#### object 

```js
const obj = {
  a: 1,
};

const obj2 = {
  a: 2,
};

console.log(obj === obj2); // false
// obj와 obj2는 다른 인스턴스를 가진다.
```

#### class

new Object라는 클래스에서 나온 인스턴스들이니 싱글톤 패턴이라 할 수 있으나 실제 class를 이용해 구현해보자.

```js
class Singleton {
  constructor() {
    if (!Singleton.instance) {
      Singleton.instance = this;
    }
    return Singleton.instance;
  }
  getInstance() {
    return this;
  }
}
const a = new Singleton();
const b = new Singleton();
console.log(a === b); // true
```

#### db 관련 모듈

```js
const URL = 'mongodb://localhost:27017/test';
const createConnection = url=>{{"url":url}}
class DB{
    constructor(url){
      if (!DB.instance){
        DB.instance = createConnection(url);
      }
      return DB.instance;
    }
    connect(){
      return this.instance;
    }
}
const a = new DB(URL);
const b = new DB(URL);
console.log(a === b); // true
// 하나의 인스턴스를 기반으로 생성하기 때문에 생성 비용을 아낄 수 있다.
```

mongoose나 MYSQL를 연결할 때 싱글톤 패턴이 많이 쓰인다. 메인 모듈에서 데이터 베이스 연결에 관한 인스턴스를 정의하고 다른 모듈인 A 또는 B에서 해당 인스턴스를 기반으로 쿼리를 보내는 형식으로 쓰인다.

### 싱글톤 패턴의 단점

1. TDD를 할 때 걸림돌이 된다. 단위 테스트를 할 때 테스트가 서로 독립적이어야 하는데 독립적인 인스턴스를 만들기 어렵기 때문이다.
2. 모듈 간의 결합을 강하게 만들 수 있다.

### 의존성 주입

의존성 : A가 B에 의존성이 있다는 것은 B의 변경 사항에 대해 A 또한 변해야 된다는 것을 의미한다.

p23

메인 모듈과 하위 모듈 사이에 dependency injector를 넣어서 간접적으로 의존성을 주입하는 방식

#### 의존성 주입의 장점

1. 테스팅하기 쉽고 마이그레이션하기 쉽게 모듈을 교체할 수 있는 구조가 된다.
2. 추상화 레이어를 넣고 이를 기반으로 구현체를 넣어 주기 때문에 애플리케이션 의존성 방향이 일관되고, 애플리케이션을 쉽게 추론가능하다.

#### 의존성 주입의 단점

1. 모듈들이 더욱더 분리되므로 클래스 수가 늘어나 복잡성이 증가될 수 있으며 약간의 런타임 페널티가 생긴다.

#### 의존성 주입 원칙

상위 모듈은 하위 모듈에서 어떠한 것도 가져오지 않아야 하며 둘다 추상화에 의존해야 하고 추상화는 세부 사항에 의존하지 말아야 한다.

## 팩토리 패턴

객체를 사용하는 코드에서 객체 생성 부분을 떼어내 추상화한 패턴, 상속 관계에 있는 두 클래스에서 상위 클래스가 중요한 뼈대를 결정하고 하위 클래스에서 생성에 관한 구체적인 내용을 결정하는 패턴

객체 생성 로직이 떼어져 있기 때문에 더 많은 유연성을 갖게 된다.

### new Object

```tsx
const num = new Object(123);
const str = new Object('123');
num.constructor.name; // Number
str.constructor.name; // String
```

### Class를 활용한 예시

```ts
// 상위 클래스, 뼈대를 구성하는 클래스
class CoffeeFactory {
  static createCoffee(type) {
    // 정적 메서드를 이용하여 메모리 할당을 한 번만 할 수 있게 되고 클래스를 기반으로 호출이 가능하게 된다.
    const factory = factoryList[type];
    // 다른 factory의 인스턴스를 CoffeeFactory에 주입했다고 볼 수 있음 (의존성 주입)
    return factory.createCoffee();
  }
}

class Latte {
  constructor() {
    this.name = "latte";
  }
}

class Espresso {
  constructor() {
    this.name = "espresso";
  }
}

class LatteFactory extends CoffeeFactory {
  createCoffee() {
    return new Latte();
  }
}

class EspressoFactory extends CoffeeFactory {
  createCoffee() {
    return new Espresso();
  }
}

const factoryList = { LatteFactory, EspressoFactory };

const main = () => {
  const latte = CoffeeFactory.createCoffee("LatteFactory");
  console.log(latte.name);
};
```

## 전략 패턴

객체의 행위를 바꾸고 싶은 경우 직접 수정하지 않고 전략이라고 부르는 캡슐화한 알고리즘을 컨텍스트 안에서 바꿔주면서 상호 교체가 가능하게 만드는 패턴

### passport의 전략 패턴

node.js에서 인증 모듈을 구현할 때 쓰는 미들웨어 라이브러리

```js
var passport = require("passport");
var LocalStrategy = require("passport-local").Strategy;

passport.use(
  new LocalStrategy(function (username, password, done) {
    if (username === "test" && password === "test") {
      return done(null, { username: "test" });
    } else {
      return done(null, false);
    }
  })
);
```

위처럼 전략을 매개변수로 넣어서 로직을 수행한다.

## 옵저버 패턴

주체가 어떤 객체의 상태 변화를 관찰하다가 상태 변화가 있을 때마다 메서드 등을 통해 옵저버 목록에 있는 옵저버들에게 변화를 알려주는 디자인 패턴

### JS에서 옵저버 패턴

프록시 객체를 통해 구현한다.

#### 프록시 객체

어떠한 대상의 기본적인 동작의 작업을 가로챌 수 있는 객체를 뜻한다. JS에선 두 개의 매개변수를 가진다.
1. target
2. handler

```js
const handler = {
  // name으로 접근하면 target의 a와 b를 읽어서 합쳐라.
  get: function (target, name) {
    return name === "name" ? `${target.a} ${target.b}` : target[name];
  },
};

const p = new Proxy({ a: "Hello", b: "World" }, handler);
console.log(p.name); // Hello World
```

```js
function createReactiveObject(target, callback) {
  const proxy = new Proxy(target, {
    set(obj, prop, value) { // 객체, prop, 할당 받는 값(여기선 커플 혹은 솔로)
      if (value !== obj[prop]) { // 기본값은 형규: 솔로 인데 만약 솔로가 아니라 다른 값을 할당한다면 돌아감
        obj[prop] = value;
        callback(`${prop} changed to ${value}`);
      }
      return true;
    },
  });
  return proxy;
}
const a = { 형규: "솔로" };
const b = createReactiveObject(a, console.log);
b["형규"] = "솔로";
b["형규"] = "커플";
// 형규 changed to 커플
```

## 프록시 패턴

프록시 객체는 프록시 패턴이 녹아들어 있는 객체이다.

### Proxy Pattern

대상 객체에 접근하기 전 그 접근에 대한 흐름을 가로채 해당 접근을 필터링하거나 수정하는 등의 역할을 하는 계층이 있는 디자인 패턴

이를 통해 객체의 속성, 변환 등을 보완하며 보안, 데이터 검증, 캐싱, 로깅에 사용합니다. (서버에서도 사용된다.)

### Proxy Server

서버와 클라이언트 사이에서 클라이언트가 자신을 통해 다른 네트워크 서비스에 간접적으로 접속할 수 있게 해주는 컴퓨터 시스템이나 응용 프로그램을 가리킨다.

#### Proxy Server로 쓰는 nginx

비동기 이벤트 기반의 구조와 다수의 연결을 효과적으로 처리 가능한 웹 서버, 주로 Node.js 서버 앞단의 프록시 서버로 활용

Node.js 서버를 운영할 때 앞단에 nginx를 두어서 익명 사용자가 직접적으로 서버에 접근하는 것을 차단하고, 간접적으로 한 단계를 더 거치게 만들어서 보안을 강화할 수 있다.

버퍼 오버플로우 (메모리 공간을 벗어나게 해서 데이터를 덮어씌워 주소, 값이 바뀌는 것), gzip 압축 (CPU 오버헤드를 생각해서 사용 유무를 신중히 해야한다.)

#### proxy로 쓰는 CloudFlare

어떠한 시스템의 콘텐츠 전달을 빠르게 할 수 있는 CDN(Cross-Origin Resource Sharing) 서비스인 CloudFlare는 웹 서버 앞단에 프록시 서버로 두어 DDOS 공격 방어나 HTTPS 구축에 쓰인다.

서비스 배포시 트래픽이 이상한지를 CAPTCHA등으로 일정 부분 막아주는 역할도 한다.

DDOS는 네트워크 요청을 마구 보내 마비시키는 공격 유형인데 CloudFlare는 이를 잘 막아준다.
CloudFlare를 사용하면 인증서 설치 없이 좀 더 손쉽게 HTTPS를 구축할 수 있다.

> CDN : 각 사용자가 인터넷에 접속하는 곳과 가까운 곳에서 콘텐츠를 캐싱 또는 배포하는 서버 네트워크를 말한다. 이를 통해 사용자가 웹 서버로부터 콘텐츠를 다운로드하는 시간을 줄일 수 있다.

#### CORS와 frontend의 Proxy Server

CORS는 서버가 웹 브라우저에서 리소스를 로드할 때 다른 오리진을 통해 로드하지 못하게 하는 http 헤더 기반 메커니즘이다.

frontend server와 backend server가 통신할 때 이러한 문제를 마주할 수 있다. (frontend server와 backend server의 포트 번호가 다를 경우 문제 발생)

이때 frontend server 앞단에 Proxy Server를 둬서 오리진을 맞춤으로써 CORS 문제를 해결할 수 있다.

## 이터레이터 패턴

iterator를 사용하여 collection의 요소들에 접근하는 디자인 패턴

순회할 수 있는 여러 가지 자료형의 구조와는 상관없이 이터레이터라는 하나의 인터페이스로 순회가 가능

```js
// Map과 Set은 자료구조가 다르지만 for라는 이터레이터를 사용해서 둘 다 순회할 수 있다.
const mp = new Map();
mp.set("a", 1);
mp.set("b", 2);
mp.set("c", 3);
const st = new Set();
st.add("a");
st.add("b");
st.add("c");
for (let a of mp) {
  console.log(a);
}
for (let a of st) {
  console.log(a);
}
/*
[ 'a', 1 ]
[ 'b', 2 ]
[ 'c', 3 ]
1
2
3
*/
```

> iterator protocol: 이터러블한 객체들을 순회할 때 쓰이는 규칙
> iteratorable object: 반복 가능한 객체로 배열을 일반화한 객체

## 노출모듈 패턴 (revealing module pattern)

즉시 실행 함수를 통해 private, public 같은 접근 제어자를 만드는 패턴

JS에서는 private이나 public이 없기 때문에 접근 제어자가 존재하지 않고 전역 범위에서 스크립트가 실행된다. (클래스의 `#`을 활용한 private 제외)

```js
const pukuba = (() => {
  const a = 1;
  const b = () => 2;
  const public = {
    c: 2,
    d: () => 3,
  };
  return public;
})();
console.log(pukuba.a); // undefined
console.log(pukuba.b); // undefined
console.log(pukuba); // { c: 2, d: [Function: d] }
console.log(pukuba.c); // 2
```

a와 b는 함수 내에 있고 return이 없기 때문에 함수 내에서만 사용되는 프로퍼티이다.

## MVC 패턴

Modal, View, Controller로 이루어진 디자인 패턴

뷰 > (유저 이벤트) > 컨트롤러, 컨트롤러 > (갱신) > 모델,
모델 > (알림) > 컨트롤러, 컨트롤러 > (갱신) > 뷰

위의 과정으로 재사용성과 확장성이 용이해진다.

### Modal

데이터베이스, 상수, 변수 등을 뜻한다. 데이터 그 자체

### View

사용자 인터페이스 요소, 모델을 기반으로 사용자가 볼 수 있는 화면

### Controller

하나 이상의 모델과 하나 이상의 뷰를 잇는 다리 역할, 이벤트 등 메인 로직 담당

## MVP 패턴

MVC에서 Controller가 presenter로 교체된 패턴

### Presenter

뷰와 프레젠터는 일대일 관계이다.

## MVVM 패턴

MVC의 C가 VM (View Model)로 바뀐 패턴

MVC 패턴과는 다르게 커맨드와 데이터 바인딩을 가지는 것이 특징

뷰와 뷰모델 사이 양방향 데이터 바인딩 지원, UI를 별도의 코드 수정 없이 재사용, 단위 테스트 쉬움

# 프로그래밍 패러다임

## 선언형과 명령형

선언형 > 함수형

명령형 > 객체지향형, 절차지향형

JS에선 함수가 일급 객체이기 때문에 함수형 프로그래밍 방식이 선호된다.

### 함수형 프로그래밍

프로그램을 하나의 큰 함수로 본다.

#### 순수 함수

출력이 입력에만 의존하는 것 (a,b)=>a+b;

#### 고차 함수

함수가 함수를 값처럼 매개변수로 받아 로직을 생성할 수 있는 것

##### 일급 객체

고차 함수이기 위해선 함수의 정의가 변수, 메서드, 함수안에 함수, 함수가 함수 반환 등의 특징을 가진 일급 객체여야 한다.

### 객체지향 프로그래밍

프로그램의 상호 작용을 객체들의 집합으로 표현한다.

#### 특징

추상화, 캡슐화, 상속성, 다형성(하나의 메서드나 클래스가 다양한 방법으로 동작 (오버라이딩, 오버로딩))

#### SOLID

S
단일 책임 원칙

O
개방-폐쇄 원칙

L
리스코프 치환 원칙

I
인터페이스 분리 원칙

D
의존 역전 원칙

### 절차지향 프로그래밍

모듈화가 어렵지만 가독성이 좋으며 실행 속도가 빠름