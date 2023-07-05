### hooks

## useState

1. useState 不必把状态全部声明在组件的顶部

> Call useState at the top level of your component

这句话指的是作用域的 top level，所以 useState 不能放倒 if 条件判断，或者循环中。

2. 如果给 useState 传递函数，那么这个函数应该是个没有参数的纯函数。因为 react 在调用 initializer function 的时候是不会给你传参。

```js
const [state, setState] = useState(1);
```

set function 调用后不会立即更新，而是下一次渲染阶段更新。react 底层有两个阶段【 commit render】

**如果更新引用类型的值，必须传递一个新的引用**
immutable state 不可变状态，意味着每一次传递的值是不可变的也是一次性的。
所以变更数组、对象通常都是在新的数组、对象中解构再赋值

```
setObj((prev)=>({
   ...prev,
   a:1
}))
```

## useEffect

用于处理副作用，但不强求，根据业务来。官方推荐，因为可以追踪。
两个参数

- setup 因为 useEffect 的执行时机所以起名叫初始化（setup）
- dependencies 必须是一个数组

1. 当我们通过 useEffect 注册了 setup 之后，会在每次挂载到页面中都会【异步】执行 setup
2. 当依赖项发生变更的时候，useEffect 会重新执行 setup.当然依赖也得是 react 构建的数据，比如 useState。
3. 如果没有依赖项，和 vue 的 onMounted 很像。可以在里面获取 dom，请求 api
4. 返回值是一个 cleanup 函数，用于清理副作用。比如事件绑定，计时器的清理。它在组件卸载时执行。

### useCallback

用来长期稳定维护一个函数引用的，他会将函数引用保存，当函数组件下一次重新渲染，他会直接返回之前保存的引用而不是重新创建

1. 第一个参数给函数声明
2. 第二个参数是依赖项，当依赖项改变时，函数引用会更新。
3. 它也是顶层声明，react 会在初次渲染而非调用时返回 callback?
4. 它会返回 callback,我们自己决定什么时候调用返回值

组件的每一次重新渲染 都会执行一遍函数，函数内部的函数声明也会每次重新创建，导致了性能开销。useCallback 缓存了第一次的函数，但那个函数中的所有引用都和当时的上下文形成了闭包。导致无法获取最新的值，所以需要我们手动的传入依赖值已更改回调函数的值引用。

## useMemo

**示例：渲染从服务器拿到的 studentList,和一个 studentNameList**
和 vue 的计算属性比较像。缓存数据，只有当依赖项改变时才会再执行第一个参数。

1. 第一个参数是一个函数 这个函数会被 react 直接执行，将其返回值进行缓存
2. 第二个参数是依赖项，当依赖项变化时，react 会重新执行第一个参数，拿到最新的返回值，并缓存

## useRef

**示例 1：一个按钮点击造成 input 聚焦**  
**示例 2：一个数字，两个按钮（start,stop 倒计时）**

1. 构建一个状态，这个状态是脱离 react 控制的，不会造成重新渲染，状态也不会因为组件的重新渲染而初始化
2. 返回一个对象，current 里是初始值。这个 current 是可读可写，随意改动的。
3. 如果用 useRef 获取 dom 可以用下面例子的语法糖

```jsx
const inputRef = useRef(null);

return <input ref={inputRef} />;
```

4. 为什么不用 getElementById？ 因为用全局变量会造成闭包，用 useEffect 需要构建状态，造成无意义的重新渲染

## forwardRef

1. forwardRef 用于解决 useRef 无法绑定函数组件，却又想拿子组件的状态、元素的场景
2. forwardRef 是一个高阶组件，接受组件做参数，返回一个新的组件，这个新的组件可以用 ref
3. forwardRef 给函数组件扩展了第二个参数，叫做 ref，此时的 ref 可以用来传递任何你想传递的数据。
4. forwardRef 一般和 useRef 一起使用

## useImperativeHandle

1. 第一个参数是一个 ref ,第二个参数是个函数，函数返回值会被赋值到 ref.current, 第三个参数是依赖项，依赖项不变 Current 不变
2. 给 ref 加响应式依赖 -我的理解

## createContext && useContext

1. 一般会有单独目录 context 统一管理 context
2. 用于组件内共享上下文

```jsx
// context/theme.js
import { createContext } from "react";

const ThemeContext = createContext("light"); //light 是个默认值

export default ThemeContext;

//App.jsx

return (
  <div>
    <ThemeContext.Provider value={theme}>
      内部组件实现共享上下文
      <Children />
    </ThemeContext.Provider>
  </div>
);

//Children.jsx

import { useContext } from "react";
import ThemeContext from "../context/Theme";

//light
const contextValue = useContext(ThemeContext);
```

## useLayoutEffect

1. 功能上和 useEffect 一致，只有运行规则上有区别
   - useEffect：首次渲染完成并 mounted 后将回调函数放到异步队列中等待执行
   - useLayoutEffect 。。。，推入到同步队列中等待执行，所以会阻塞后续的更新工作
2. 如果在用 useEffect 上遇到问题可以考虑这个 Hook
