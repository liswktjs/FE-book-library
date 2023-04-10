## RecoilRoot

- Context API 처럼 상태를 공유할 컴포넌트를 `RecoilRoot`로 묶는다

```
<RecoilRoot>
  <App />
</RecoilRoot>
```

> 여러 RecoilRoot가 있을 경우 어떻게 될까?

- 각각의 RecoilRoot는 독립적인 atom 상태의 providers/store가 된다
  - 별개의 새로운 Recoil상태 트리이기 때문에 부모 자식 과 같은 관계로 중첩되어 있을 경우, 상태의 복제 또는 충돌되는 의도치 않은 결과가 발생할 수 있다
    => 동일한 계층 내에서 RecoilRoot를 중첩하여 사용할 것

### 속성

- initializeState ? : (MutateSnapshot) => void
- 원자 상태를 초기화하는 옵션 함수
- 초기 랜더링에 대한 상태를 설정한다

- override?: boolean
  - 기본값 : true
  - RecoilRoot가 다른 RecoilRoot와 중첩된 경우에
    - true
      : 루트는 새로운 Recoil 범위를 생성한다
    - false
      : RecoilRoot는 자식 렌더링 외에 다른 기능을 수행하지 않아, 루트의 자식들은 가장 가까운 조상 RecoilRoot에서 Recoil값을 엑세스 한다

## atoms

### 속성

- key
  - 내부적으로 atom을 식별하는 데 사용되는 고유한 문자열
- default
  - atom의 초깃값
  - Promise
  - 동일한 타입의 값을 나태내는 다른 atom이나 selector
- effect_UNSTABLE
  - atom effects 배열
- dangerouslyAllowMutability
  - true
    - useRecoilState의 setState 함수를 거치지 않고 직접적으로 상태를 수정할 수 있다.
    - 상태값이 변경되었는지 등록된 컴포넌트에 알리지 않기 때문에 재랜더링이 이루어지지 않는다
  - false
    - 정해진 setState함수를 통해서만 값을 업데이트 해야한다

## selector

### 특징

- get함수만 제공하면 Selector는 읽기만 가능한 RecoilValueReadOnly 객체를 반환

- set함수만 제공하면 쓰기 가능한 RecoilState 객체를 반환

```
function selector<T>({
  key: string,
  get: ({get: GetRecoilValue}) => T | Promise<T> | RecoilValue<T>,

  set?: ({
    get: GetRecoilValue,
    set: SetRecoilState,
    reset: ResetRecoilState
  },
  newValue: T | DefaultValue,) => void,

  dangerouslyAllowMutability?: boolean
})
```

### 속성

- key
  - 내부적으로 atom을 식별하는데 사용되는 고유한 문자열
- get
  - 파생된 상태의 값을 평가하는 함수
  - 값을 직접 반환하거나 비동기 Promise, 다른 atom이나 selector를 반환할 수 있다
  - get
    - 다른 atom이나 selector로부터 값을 찾는데 사용되는 함수
    - 해당 함수에 전달된 모든 atom과 selector는 암묵적으로 selector에 대한 의존성 목록에 추가된다
- set
  - selector를 쓰기 가능한 상태로 반환한다
  - callback get
    - 다른 atom이나 selector로 부터 ㄱ밧을 찾는데 사용되는 함수
    - 위의 get과 다르게 구독하지 않는다
  - callback set
    - Recoil 상태 값을 설정할 때 사용되는 함수

### selector 종류

#### 정적인 의존성

```
const mySelector = selector({
  key: 'MySelector',
  get: ({get}) => get(myAtom) * 100
})
```

#### 동적인 의존성

```
const toggleState = atom({key: 'Toggle', default: false});

const mySelector = selector({
  key: 'MySelector',
  get: ({get} => {
    const toggle = get(toggleState);
    if(toggle) {
      return get(selectorA);
    }else{
      return get(selectorB);
    }
  })
})
```

- get을 통해 구독한 toggleState가 변경될 때마다 mySelector는 다시 실행되어 값이 업데이트된다

#### 쓰기 가능한 selector

```
const counterAtom = atom({
  key: 'CounterAtom',
  default: 0,
});

const counterState = selector({
  key: 'CounterState',
  get: ({get}) => get(counterAtom),
  set: ({set}, newValue) => set(counterAtom, newValue);
});

function Counter() {
  const count = useRecoilValue(counterState);
  const setCount = useSetRecoilState(counterState);

  return (
    <div>
      <button onClick={() => {setCount(count + 1)}}>+</button>
    </div>
  )
}

```

## recoil Hook

### useRecoilState

`read` `write`

```
const [toggle, setToggle] = useRecoilState(toggleState);
```

- hook이 호출된 컴포넌트는 해당 상태를 구독하게 된다
- 상태를 setState함수를 통해서 업데이트 할 경우 컴포넌트가 리렌더링 되게 된다

### useRecoilValue

`read`

```
const toggle = useRecoilValue(toggleState);
```

- hook이 호출된 컴포넌트는 해당 상태를 구독하게 된다
- 공유된 atom이 update 되었을 때에 리렌더링 되게 된다

### useSetRecoilState

`write`

```
const toggleState = atom({
  key: 'toggleState',
  default: false
})

function Form(){
  const setToggleState = useSetRecoilSate(toggleState);

  return <FormContent setToggleState={setToggleState} / >
}

function FormContent({setToggleState}) {
  return (
    <>
      <button onClick={() => setToggleState( toggle => !toggle)}>toggle</button>
    </>
  )
}
```

- hook이 호출된 컴포넌트는 해당 상태를 구독하게 된다
- setter는 새로운 값이나 이전 값을 인수로 받아 updater 함수를 넘겨준다

### useResetRecoilState

`initialize`

```
const resetToggle = useResetRecoilState(toggleState);

return <button onClick={resetToggle}>Reset</button>;
```

## atomFamily

```
const todoListAtomFamily = atomFamily({
  key: 'todoList',
  default: [],
});

const todoList1 = useRecoilValue(todoListAtomFamily('list1'));
const todoList2 = useRecoilValue(todoListAtomFamily('list2'));

```

### 역할

- 작성 가능한 RecoilState atom을 반환하는 함수를 반환
- 여러 개의 유사한 atom을 생성하는 함수
  - 하나의 템플릿 atom을 기반으로 유사한 기능을 가진 여러 개의 atom을 생성할 수 있다
- 키 값을 인자로 넘긴 값을 기반으로 새롭게 생성한다
  - 위의 예제에서는 각각의 todoList에 `todoList:<name>`으로 key값이 생성된다

### 속성

- key
  - atom을 식별하는데 사용되는 고유한 문자열
- default
  - atom의 초기값
  - RecoilValue or Promise or 기본값 가져오는 함수
- dangerouslyAllowMutability

## selectorFamily

```
const myNumberState = atom({
  key: 'MyNumber',
  default: 2,
});

const myMultipleState = selectorFamily({
  key: 'MyMultipliedNumber',
  get: (multiplier) => ({get}) => {
    return get(myNumberState) * multiplier;
  },
  set : (multiplier) => ({set}, newValue) => {
    set(myNumberState, newValue  multiplier);
  }
})

function MyComponent() {
  const number = useRecoilValue(myNumberState);
  // default 2가 된다
  const multipliedNumber = useRecoilValue(myMultipliedState(100));
  // default 200이 된다

}
```

### 역할

- 읽기 전용 RecoilValueReadOnly 또는 수정 가능한 RecoilState selector를 반환하는 함수를 반환

### 특징

- selector와 유사하지만 get,set, selector와 같은 콜백을 매개변수로 전달할 수 있다.

### 속성

- key
  - 내부적으로 atom을 식별하는 데 사용되는 고유한 문자열
- get
  - selector 값을 반환하는 명명된 콜백들의 객체를 전달하는 함수
- set
  - 쓰기 가능한 selector를 생성하는 선택적 함수
