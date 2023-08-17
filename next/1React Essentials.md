# React Essentials

要使用 Next.js 构建应用程序，熟悉 React 的新功能（例如服务器组件）会有所帮助。本页将介绍服务器和客户端组件之间的差异、何时使用它们以及推荐的模式。

### Server Components

服务器和客户端组件允许开发人员构建跨服务器和客户端的应用程序，将客户端应用程序的丰富交互性与传统服务端渲染的性能相结合。

### Thinking in Server Components

与 React 如何改变我们构建 UI 的方式类似，React 服务器组件引入了一种新的思维模型，用于构建利用服务器和客户端的混合应用程序

React 现在使您可以根据组件的用途灵活地选择在何处渲染组件，而不是在客户端渲染整个应用程序（例如在单页应用程序的情况下）。

例如:如果我们将页面拆分为更小的组件，您会注意到大多数组件都是非交互式的，并且可以作为服务器组件在服务器上呈现。对于较小的交互式 UI，我们可以添加客户端组件。这与 Next.js 服务器优先的思想一致。

### Why Server Components?

所以，您可能会想，为什么用服务器组件？与客户端组件相比，使用它们有什么优势？

服务器组件使开发人员能够更好地利用服务器的基础架构。例如，您可以将数据获取操作放到服务器,因为离数据库更近，并在服务器上保留客户端的 Js 依赖项，从而提高性能。服务器组件使编写 React 应用程序感觉类似于 PHP 或 RubyonRails，但具有 React 模板化 UI 的强大功能和灵活性.

使用服务器组件，初始页面加载速度更快，并且客户端 JavaScript 包大小减小。基本客户端运行时的大小是可缓存且可预测的，并且不会随着应用程序的增长而增加。仅当通过在应用程序中使用客户端组件交互时，才会添加其他 JavaScript。

当使用 Next.js 加载路由时，初始 HTML 将在服务器上呈现。然后，通过异步加载 Next.js 和 React 客户端运行时,来让浏览器接管应用程序并添加交互性。

为了更容易地过渡到服务器组件，App Router 中的所有组件(包括特殊文件和共存组件)默认为服务器组件。这使您可以在无需额外工作的情况下自动采用它们，并实现开箱即用的出色性能。您也可以使用“use Client”指令选择性的加入客户端组件。

---

### Client Components

客户端组件使您能够向应用程序添加客户端交互性。在 Next.js 中，它们在服务器上预渲染并在客户端上进行 hydrated(激活交互性)。

#### The "use client" directive

“use client”指令用来声明客户端组件,与服务器组件划分界限.

```tsx
"use client";

import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

"use client"位于仅服务器代码和客户端代码之间,它应该被写在模块的顶部,在 import 之上,来定义一个服务器部分和客户端部分代码的分界点.一旦在一个文件中定义了“use client”，导入其中的所有其他模块，包括子组件，都将被视为客户端的一部分。

由于服务器组件是默认的，所以所有组件都是服务器组件模块图的一部分，除非在以“use client”指令开头的模块中定义或导入。

> **Good to know**:
>
> - 服务器组件模块图中的组件保证仅在服务器上呈现。
> - 客户端组件模块图中的组件主要在客户端上渲染，但使用 Next.js，它们也可以在服务器上预渲染并在客户端上水合。
> - 不需要在每个文件中定义“使用客户端”。客户端模块边界只需要在“入口点”定义一次，就可以将导入其中的所有模块视为客户端组件。

## When to use Server and Client Components?

为了简化服务器组件和客户端组件之间的决策，我们建议使用服务器组件（默认在“app”目录中），直到您有了客户端组件的使用场景。

此表总结了服务器组件和客户端组件的不同使用场景：

| What do you need to do?                                                            | Server Component | Client Component |
| ---------------------------------------------------------------------------------- | ---------------- | ---------------- |
| 获取数据                                                                           | ✅               | ❌               |
| 直接获取后端资源                                                                   | ✅               | ❌               |
| 在服务器上保留敏感信息(access tokens, API keys, etc)                               | ✅               | ❌               |
| 保持对服务器的大量依赖/减少客户端 JavaScript                                       | ✅               | ❌               |
| 添加交互性和事件侦听器 (`onClick()`, `onChange()`, etc)                            | ❌               | ✅               |
| Use State and Lifecycle Effects (`useState()`, `useReducer()`, `useEffect()`, etc) | ❌               | ✅               |
| Use browser-only APIs                                                              | ❌               | ✅               |
| Use custom hooks that depend on state, effects, or browser-only APIs               | ❌               | ✅               |
| Use [React Class components](https://react.dev/reference/react/Component)          | ❌               | ✅               |
