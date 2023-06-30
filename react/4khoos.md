### hooks

**useState**
useState 不必把状态全部声明在组件的顶部

> Call useState at the top level of your component

这句话指的是作用域的 top level，所以 useState 不能放倒 if 条件判断，或者循环中。

如果给 useState 传递函数，那么这个函数应该是个没有参数的纯函数。因为 react 在调用 initializer function 的时候是不会给你传参的。

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

**useEffect**

用于处理副作用，但不强求，根据业务来。官方推荐，因为可以追踪。
两个参数   
- setup 因为useEffect的执行时机所以起名叫初始化（setup）
- dependencies 必须是一个数组  

1. 当我们通过useEffect注册了setup之后，会在每次挂载到页面中都会【异步】执行setup  
2. 当依赖项发生变更的时候，useEffect会重新执行setup.当然依赖也得是react构建的数据，比如useState。
3. 如果没有依赖项，和vue的onMounted很像。可以在里面获取dom，请求api
4. 返回值是一个cleanup函数，用于清理副作用。比如事件绑定，计时器的清理。它在组件卸载时执行。


### useCallback   
用来长期稳定维护一个函数引用的，他会将函数引用保存，当函数组件下一次重新渲染，他会直接返回之前保存的引用而不是重新创建
1. 第一个参数给函数声明
2. 第二个参数是依赖项，当依赖项改变时，函数引用会更新。
3. 它也是顶层声明，react会在初次渲染而非调用时返回callback?
4. 它会返回callback,我们自己决定什么时候调用返回值

组件的每一次重新渲染 都会执行一遍函数，函数内部的函数声明也会每次重新创建，导致了性能开销。useCallback缓存了第一次的函数，但那个函数中的所有引用都和当时的上下文形成了闭包。导致无法获取最新的值，所以需要我们手动的传入依赖值已更改回调函数的值引用。