# UI 라이브러리 만들기 시리즈 - 생각정리

## 시작하며
나는 프론트엔드 개발을 하며 새로운 프로젝트를 할 때마다 똑같은 컴포넌트를 항상 새로만드는 것과 똑같은 컴포넌트라도 만드는 사람에 따라 제각각인 일관성 없는 인터페이스로 인한 비효율이 싫었다. 그리고 웹 접근성은 당연하게 무시되어지는 경향이 있었는데 항상 마음에 걸렸다.

그래서 위의 문제들을 해결해주는 Radix UI나 React Aria 와 같이 명확한 디자인 원칙을 기반으로 만들어진 UI 라이브러리들에 자연스럽게 관심이 갔던 거 같다.  언젠가부터 이런 UI 라이브러리를 직접 만들고 싶다는 생각을 했고, UX 엔지니어라는 포지션에도 관심이 생겼다.

사실, [react-dive-ui ](https://github.com/dev-hobin/react-dive-ui)라는 라이브러리를 여러 라이브러리를 참고해서 만들어보기도 했는데, 공부는 많이 됐지만 실제로 사용하기에 부족함이 많았다.

이번에는 적어도 내 사이드 프로젝트에 쓰일 수 있는 UI 라이브러리를 만들어내고 싶다.
## 주요 특징
- Uncontrolled 컴포넌트와 Controlled 컴포넌트 둘 다 지원하기  
  기본적으로는 Uncontrolled 컴포넌트로 작동하되, 필요에 따라 Controlled 컴포넌트로 전환 가능하도록 한다.
- prop 보다는 event 기반 동작 방식  
  ```jsx
    const [isOpen, setIsOpen] = useState(false);
    
    <Component isOpen={isOpen} onOpenChange={setIsOpen} />
    ```
    위와 같이 사용자에게 prop을 직접 관리하게 하는 방식보다는 
    ```jsx
    const store = useComponentStore({ ...initialValues });
    const { open, close, toggle } = store.events();
    
    <Component store={store} />
    ```
    컴포넌트가 적절히 추상화한 이벤트 핸들러를 제공하는 방식을 선호한다.
- 컴포넌트 자체적으로 애니메이션 관리
  라이브러리에서 제공하는 컴포넌트는 자체적으로 애니메이션이 끝날 때까지 언마운트 시점을 늦추는 기능을 가진다. 또한, framer motion 과 같은 외부 애니메이션 라이브러리에 애니메이션 관리를 위임 할 수 있다.
- 다형적인 컴포넌트 지원  
  asChild prop을 제공하여 사용자의 필요에 맞게 엘리먼트 타입을 변경할 수 있는 수단을 제공한다.
- 웹 접근성 준수
  컴포넌트들은 [ARIA design pattern](https://www.w3.org/WAI/ARIA/apg/patterns)을 기준으로하여 웹 접근성을 준수한다.
## 구현 결정사항
- Controlled와 Uncontrolled 컴포넌트를 모두 대응하려고 하면 필연적으로 내부 상태와 외부 상태의 싱크를 맞추는 데 useEffect를 남발하게 된다. 또한 상태가 변경될 때 필요한 사이드 이팩트를 적절하게 발생시키기 위해 여러가지 hacky한 방법을 사용하게 되어 구현의 복잡성이 매우 커진다.
  이 문제를 해결하기 위해서는 리액트의 렌더링 시스템에서 벗어날 필요가 있었고 컴포넌트의 상태를 리액트 외부의 스토어에서 관리하도록 하고 useSyncExternalStore 를 이용하여 리액트와 연결 시키는 방법을 선택했다. (구독 개념을 이용)
-  리액트 외부에서 컴포넌트의 상태를 다룰 때 사용할 기술로 xstate 를 이용하여 상태의 변경이 발생했을 때 사이드 이팩트를 발생시키는 방법을 챙기며, 컴포넌트의 동작을 추상화한다.
## 제공할 인터페이스의 형태

### Store
컴포넌트 내부, 외부에서 사용할 store : 컴포넌트의 모든 상태, 동작을 담고 있다.
```jsx
// - ref에 감싸져 있어 렌더링에 영향을 주어서는 안된다.
const store = useComponentStore({ ...initialValues });
// store는 watch 인터페이스를 제공하여 store의 상태를 부분 구독할 수 있게한다.
// - 내부적으로 useSyncExternalStore 를 이용한 구독 개념을 사용한다.
const watchState = store.watch(state => state.a.b);
// store의 현재 state를 스냅샷으로 찍는다. 
// - useEffect 안에서나 이벤트 핸들러에서 사용할 수 있다.
const snapshot = store.snapshot();
// store에서 컴포넌트의 동작을 가져온다.
// - events.toggle(), events.open(), ... 등등
const events = store.events();
```
### Component Specific Props
store의 상태를 기반으로 컴포넌트마다 가져아하는 prop 제공한다.
```jsx
const { rootProps, getItemProps, ... } = useComponentProps(store.watch(state => state));

<Root {...rootProps}>
  <Item {...getItemProps(...동적으로 넘겨줘야하는 prop)} />
</Root>
```
## 글을 마치며
이번 글은 내가 어떤 라이브러리를 만들고 싶은지 정리해보았는데, 다 쓰고보니 애매한 부분도 많이 있고 검증이 필요한 부분도 많이 있다. 앞으로 라이브러리를 직접 만들어나가며 더 구체적이고 주제별로 도움이 되는 글을 작성해나가려 한다.
