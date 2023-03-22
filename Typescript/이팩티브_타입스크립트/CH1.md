## 타입스크립트 알아보기

### 아이템 1. 타입스크립트와 자바스크립트의 관계 이해하기

- 타입스크립트는 문법적으로 자바스크립트의 상위 집합
  - 자바스크립트 프로그램에 문법 오류가 없다면, 유효한 타입스크립트 프로그램이라고 할 수 있다
  - 자바스크립트에서 이슈가 존재한다면, 문법 오류가 아니더라도 타입 체커에 지적당할 수 있다
    - 문법 유효성과 동작의 이슈는 독립적이므로 여전히 자바스크립트로 변환할 수 있다

=> 해당 특성은 일부분에만 타입스크립트를 적용할 수 있게 해주므로 타입스크립트로 마이그레이션하는데 도움이 된다

- 타입스크립트는 자바스크립트가 아닌 프로그램(문법)이 존재한다

- 타입스크립트의 타입 체커

  1. 타입 구문이 없어도 오류를 찾아낼수 있다
  2. 타입 구문을 추가했을 때, 코드의 동작과 의도가 다른 부분을 찾을 수 있다

  ```
  interface State {
      name : string;
      capital : string;
  }
  const states : State [] = [
    {name : 'Alaska', capitol: 'Montogomery'} // type checker에 의해서 capital에 대한 오타인 capitol을 지적해준다
  ]
  ```

- 타입스크립트의 타입 시스템
  - 자바스크립트의 런타임 동작을 `모델링`한다
  - 그렇기 때문에, 런타임 오류를 발생시키는 코드를 찾아내려고 한다 하지만 타입 체커에 벗어나더라도 오류가 발생하는 경우도 있다

### 아이템 2. 타입스크립트 설정 이해하기

#### noImplicitAny

- 변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어
- 해당 값을 설정함으로써 변수를 선언할 떄에 타입을 명시하는 것을 강제할 수 있다

```
function add(a,b) {
  return a + b;
}
```

=> 암시적으로 any를 추론하게 되면 noImplicitAny의 설정 상에서는 에러가 발생하게 된다

#### strictNullChecks

- null, undefined가 모든 타입에서 허용되는지 확인하는 설정

```
const x: number = null;  // strictNullChecks 해제되었을 때에 유효한 설정이다
const x: number = null;  // 더 이상 유효한 값이 되지 못한다 (undefined일 때에도 동일한 에러가 발생한다 )
```

- 만약, null을 사용하고 싶다면 null인지 여부를 확인하는 분기문과 타입 단언을 활용하여 null에 대한 처리를 해주어야 한다

```
const el = document.getElementById('status');
el.textContent = 'Ready'; // el이 존재하지 않을 수 있어 null일 수 있으므로 null로 추론되게 된다

if(el){ // null이 아닌 경우에 대해서 분기 처리를 진행한다
  el.textContent = 'Ready';
}
el!.textContent = 'Ready'; // null이 아님을 단언한다
```

### 아이템 3. 코드 생성과 타입이 관계없음을 이해하기

- 타입스크립트 컴파일러의 역할

1. 최신 타입스크립트 / 자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일한다
2. 코드의 타입 오류를 체크한다

=> 이 두 가지 역할은 독립적이여서 타입 오류가 발생해도 구버전의 자바스크립트로 트랜스파일이 가능하다

> 타입스크립트 컴파일러는 타입 오류가 있는 코드도 컴파일이 가능하다

- 타입 오류가 있더라도 컴파일이 되는 것의 이점은?
  : 웹 어플리케이션을 만들면서 타입 오류가 발생하여도 여전히 컴파일된 산출물을 생성하기 때문에 오류를 수정하지 않더라도 다른 부분 테스트가 가능하다

- 만약, 타입 오류가 났을 때 컴파일이 불가능하게 하려면?
  : noEmitOnError을 설정한다

> 런타임에는 타입 체크가 불가능하다

- 타입스크립트의 타입은 자바스크립트로 컴파일 되는 과정에서 모두 제거된다

- 만약, 런타임시에 타입을 확인하고 싶다면?
  : instanceOf, in 과 같은 자바스크립트 문법을 활용하여 객체에 특정 프로퍼티가 있는지 확인하는 것이 가능하다

* class를 활용해 런타임시 타입 확인을 진행하자

- class로 타입을 선언하게 되면 타입과 값으로 모두 사용이 가능하다
- 이 점을 활용하여 타입 체크와 런타임시 타입 확인 모두 가능하게 된다

```
class Square {
  constructor(public width :number) {}
}
class Rectangle extends Square {
  constructor(public width: number, public height : number){
    super(width);
  }
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceOf Rectangle){ // instanceOf문법을 사용하여 shape가 Rectangle의 인스턴스인지(프로퍼티를 모두 가지고 있는지) 확인한다
    return shape.width * shape.height;
  }
  if (shape instanceOf Square){
    return shape.width * shape.width;
  }
}

```

> 타입 연산은 런타임에 영향을 주지 않는다

- 런타임시 값의 안정성을 보장하고 싶다면, 런타임의 타입 체크를 자바스크립트 문법으로 진행해주어야 한다

> 런타임 타입은 선언된 타입과 다를 수 있다

- 타입스크립트를 통해 지정한 타입들의 경우, 런타임시 모두 제거되게 된다
- 이 때문에 서버에서 오는 값들의 경우 타입스크립트로 선언된 타입과 맞지 않을 수 있다

> 타입스크립트 타입으로는 함수를 오버로드 할 수 없다

- 타입스크립트에서는 타입과 런타임 동작이 무관하기 때문에 함수 오버로딩이 불가능하다

```
function add(a: number, b : number) {return a + b}
function add(a: number, b : string) { return a + b} // 중복된 함수라는 에러 메세지가 보여지게 된다
```

> 타입스크립트 타입은 런타임 성능에 영향을 주지 않는다

- 타입과 타입 연산자는 자바스크립트로 컴파일 되는 과정에서 모두 제거되기 때문에 런타임 시점에 성능에 아무런 영향을 주지 못한다

### 아이템 4. 구조적 타이핑에 익숙해지기

- 자바스크립트는 덕 타이핑 기반이다

> 덕 타이핑이란?
> 만약 어떤 함수의 매개변수 값이 모두 제대로 주어진다면, 그 값이 어떻게 만들어졌는지 신경 쓰지 않고 사용한다

- 타입스크립트는 자바스크립트의 덕 타이핑을 모델링하여 `구조적 타이핑`을 지원한다

```
interface Vector2D {
  x: number;
  y: number;
}


function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}


interface NamedVector {
  name :string;
  x: number;
  y : number;
}

```

- NamedVector는 number 타입의 x와 y 속성이 있기 때문에 calculateLength로 호출이 가능하다

```
const v : NameVector = {x : 3, y : 4, name : 'Zee' };
calculateLength(v);  //  v가 x, y 속성을 지니고 있기 때문에 정상적으로 작동하게 된다
```

- 타입스크립트의 구조적 타이핑이 문제를 발생시킬 때

```
interface Vector3D {
  x: number;
  y: number;
  z: number;
}

function normalize (v: Vector3D){
  const length = calculateLength(v); // x , y가 존재하기 때문에 z에 대한 처리가 calculateLength에 존재하지 않아도
  // 에러를 발생시키지 않고 문제없이 작동하게 된다
  return {
    x: v.x / length;
    y: v.y / length;
    z: v.z / length;
  }
}

normalize({x: 3, y: 4, z: 5});
```

- 타입스크립트에서는 타입이 열려 있기 때문에 때로는 예상치 못한 문제가 발생할 수도 있다

* 열리 있다는 것
  : 타입 확장에 열려있다는 것, 즉 타입에 선언된 속성 외에 임의의 속성을 추가하더라도 오류가 발생하지 않는다
