## Todo App

create-react-app v3.3.0

### 라이브러리

- node-sass
- classnames
- react-icons (https://react-icons.netlify.com/)

### 성능 최적화 과정

**문제 상황**
 
- 2,500개의 데이터를 일정 목록에 렌더링 

- App 컴포넌트의 state가 변경되면서 --> App 컴포넌트가 리렌더링 --> TodoList 컴포넌트 리렌더링 --> 무수한 하위 컴포넌트들 리렌더링

#### 성능 모니터링

크롬 개발자 도구 Performance 탭에서 녹화 버튼 누른 후, 어떤 동작을 하고 Stop 버튼 누릅니다.
그러면 어떤 동작을 하는 것에 대한 성능 분석 결과가 나타납니다.

![performance_before](./img/performance_before.png)

2,500개의 할 일 목록에서 하나를 체크하는 데, 1.38초가 걸립니다.

지금처럼 약 2,000개가 넘어가면 성능이 저하되므로, 리렌더링을 방지해 성능을 최적화해 주는 작업을 해보자!

### 성능 최적화 방법

**1) React.memo 사용**

컴포넌트를 감싸주면 됩니다.

`export default React.memo(TodoListItem);`

<br>

**2) useState의 함수형 업데이트**

setTodos를 사용할 때 새로운 상태를 파라미터로 넣는 대신, 상태 업데이트를 어떻게 할지 정의해 주는 업데이트 함수를 넣는다.
useCallback을 사용할 때 두 번째 파라미터로 넣지 않아도 된다.

[Before 코드]   
``` JavaScript
const onRemove = useCallback( id => {
    setTodos(todos.filter(todo => todo.id !== id));
}, [todos]);
```  

[After 코드]   
``` JavaScript
const onRemove = useCallback( id => {
    setTodos(todos => todos.filter(todo => todo.id !== id));
}, []);
```  

<br>

**성능 향상 결과**

![performance_after](./img/performance_after.png)

1.38초에서 0.112초로 훨씬 향상되었습니다!!

<br>

**3) useReducer 사용**

useState의 함수형 업데이트를 사용하는 대신, useReducer를 사용해도 문제를 해결할 수 있습니다.
기존 코드를 많이 고쳐야 한다는 단점이 있지만, 상태를 업데이트하는 로직을 모아서 컴포넌트 바깥에 둘 수 있다는 장점이 있습니다.

성능상으로는 두 가지 방법이 비슷하기 떄문에 어떤 방법을 선택하는지는 취향에 따라 결정하면 됩니다.

``` JavaScript
...

function todoReducer(todos, action) {
    switch(action.type) {
        ...
        case 'REMOVE':
            return todos.filter(todo => todo.id !== action.id);
        ...
        default:
            return todos;
    }
}

const App = () => {
    const [todos, dispatch] = useReducer(todoReducer, undefined, createBulkTodos);

    ...
    const onRemove = useCallback(id => {
        dispatch({ type: 'REMOVE', id });
    }, []);
    ...
}
```  

useReducer의 두 번째 파라미터에는 초기 상태를 넣어 줍니다. 지금은 2,500개의 데이터를 만드는 createBulkTodos 함수를 맨 처음 렌더링될 떄만 호출하기 위해 세 번째 파라미터에 넣어줍니다.

<br>

**4) react-virtualized를 사용한 렌더링 최적화**

2,500개 컴포넌트 중 2,491개 컴포넌트는 스크롤하기 전에 보이지 않음에도 불구하고 렌더링이 이루어집니다.
react-viertualized를 사용하면 스크롤되기 전에 보이지 않는 컴포넌트는 렌더링하지 않고 크기만 차지하게끔 할 수 있습니다.
이 라이브러리를 사용하면 낭비되는 자원을 쉽게 아낄 수 있습니다.

- `$ yarn add react-virtualized`

- 리스트 하나의 항목 가로, 세로 크기 알아내기 (가로 512px, 세로 57px)

- 자세한 코드는 src/components/TodoList.js 참고

![performance_virtualized](./img/performance_virtualized.png)

React.memo를 통해 0.112초까지 줄였는데, 0.008초로 훨씬 더 줄었습니다!!!