### hooks

**useState**
useState不必把状态全部声明在组件的顶部

 > Call useState at the top level of your component  
  
这句话指的是作用域的top level，所以useState不能放倒if条件判断，或者循环中。
 
 如果给useState传递函数，那么这个函数应该是个没有参数的纯函数。因为react在调用initializer function的时候是不会给你传参的。

 ```js
 const [state,setState] = useState(1)
 ```

 set function 调用后不会立即更新，而是下一次渲染阶段更新。react底层有两个阶段【 commit render】

 

