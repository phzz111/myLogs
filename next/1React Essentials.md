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

## Patterns

### Moving Client Components to the Leaves

为了提高应用的性能, 我们建议尽可能将客户端组件移到组件树的叶子上.

例如, 您可能有一个 Layout，它包含静态元素（logo、链接等）和一个使用了状态的交互式搜索栏。

与其将整个布局设置为客户端组件，不如将交互式逻辑移动到客户端组件（例如“＜ SearchBar/＞”），并将布局保留为服务器组件。这意味着您不必将布局的所有组件 Javascript 发送到客户端。

```tsx filename="app/layout.tsx" switcher
// SearchBar is a Client Component
import SearchBar from "./searchbar";
// Logo is a Server Component
import Logo from "./logo";

// Layout is a Server Component by default
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <>
      <nav>
        <Logo />
        <SearchBar />
      </nav>
      <main>{children}</main>
    </>
  );
}
```

```jsx filename="app/layout.js" switcher
// SearchBar is a Client Component
import SearchBar from "./searchbar";
// Logo is a Server Component
import Logo from "./logo";

// Layout is a Server Component by default
export default function Layout({ children }) {
  return (
    <>
      <nav>
        <Logo />
        <SearchBar />
      </nav>
      <main>{children}</main>
    </>
  );
}
```

### Composing Client and Server Components

服务器组件和客户端组件可以组合在同一个组件树中.

在幕后，React 按如下方式处理渲染：

- 在服务器上，React 渲染**所有**服务器组件，然后将结果发送给客户端。
  - 这包括嵌套在客户端组件中的服务器组件。
  - 跳过在此阶段遇到的客户端组件。
- 在客户端, React 渲染客户端组件 和有 _slots in_ 标记的服务器组件合并.
  - 如果任何服务器组件嵌套在客户端组件内，则其呈现的内容将正确放置在客户端组件中。

> **Good to know**: 在 Next.js 中, 初始页面时, 上述步骤中的服务器组件和客户端组件的渲染结果都会[在服务器上预渲染为 HTML](/docs/app/building-your-application/rendering) 来加快初始加载速度

### Nesting Server Components inside Client Components

考虑到上面概述的呈现流程，将服务器组件导入客户端组件是有限制的，因为这种方法需要额外的服务器往返。

#### Unsupported Pattern: Importing Server Components into Client Components

下例是不支持的,你不能把服务器组件嵌套在客户端组件里(译者:不能直接导入,但你可以通过参数传递的形式实现客户端组件嵌套服务器组件):

```tsx filename="app/example-client-component.tsx" switcher highlight={5,18}
"use client";

// This pattern will **not** work!
// You cannot import a Server Component into a Client Component.
import ExampleServerComponent from "./example-server-component";

export default function ExampleClientComponent({
  children,
}: {
  children: React.ReactNode;
}) {
  const [count, setCount] = useState(0);

  return (
    <>
      <button onClick={() => setCount(count + 1)}>{count}</button>

      <ExampleServerComponent />
    </>
  );
}
```

#### Recommended Pattern: Passing Server Components to Client Components as Props

作为替代, 当你设计客户端组件时,你可以通过使用 React props 为服务器组件标记 _"slots"_

这个服务器组件会在服务器渲染完成, 并且当客户端组件在客户端渲染完成后, _"slot"_ 标记将被服务器组件的渲染结果填充

一个通用的做法是用 React `children` prop 来创建 _"slot"_. 我们可以重构 `<ExampleClientComponent>`组件来接受 `children` prop 并将“＜ ExampleClientComponent ＞”的导入和显式嵌套上移到父组件。

```tsx filename="app/example-client-component.tsx" switcher highlight={6,16}
"use client";

import { useState } from "react";

export default function ExampleClientComponent({
  children,
}: {
  children: React.ReactNode;
}) {
  const [count, setCount] = useState(0);

  return (
    <>
      <button onClick={() => setCount(count + 1)}>{count}</button>

      {children}
    </>
  );
}
```

现在，`<ExampleClientComponent>`不知道“children”是什么。它也不知道“children”最终会由什么服务器组件的渲染结果填充。

它唯一的职责是指定`ExampleClientComponent` 被放在哪里

在它的父组件, 你可以引入 `<ExampleClientComponent>` 和 `<ExampleServerComponent>` ,并把 `<ExampleServerComponent>` 作为子组件嵌入

```tsx filename="app/page.tsx"  highlight={11} switcher
// This pattern works:
// You can pass a Server Component as a child or prop of a
// Client Component.
import ExampleClientComponent from "./example-client-component";
import ExampleServerComponent from "./example-server-component";

// Pages in Next.js are Server Components by default
export default function Page() {
  return (
    <ExampleClientComponent>
      <ExampleServerComponent />
    </ExampleClientComponent>
  );
}
```

使用这种方法，“＜ ExampleClientComponent ＞”和“＜ ExaampleServerComponent ＞”的渲染是解耦的，并且可以独立渲染

> **Good to know**
>
> - 这个模式基于 `children` prop **已经应用** 在 [layouts and pages](/docs/app/building-your-application/routing/pages-and-layouts) 了,所以你不需要额外创建包装器组件
> - 将 React 组件（JSX）传递给其他组件并不是一个新概念，它一直是 React 组合模型的一部分。
> - 这种组合策略适用于服务器和客户端组件，因为接收 props 的组件可以选择放置 props 的位置，但不知道 props 是什么。
>   - 被传递的服务器组件可以独立的在服务器上渲染,时机上早于客户端组件
>   - 同样的内容提升策略也被用于当父组件状态改变导致的导入的子组件的重新渲染
> - 你不必将 prop 取名叫 children

### Passing props from Server to Client Components (Serialization)

从服务器传递到客户端组件的 Props 需要是[可序列化的](https://developer.mozilla.org/en-US/docs/Glossary/Serialization)。这意味着函数、日期等值不能直接传递给客户端组件。

> **Where is the Network Boundary?**
>
> 在 App Router 中, 网络边界位于服务器组件和客户端组件之间. 这和 Pages Router 不同 (`getStaticProps`/`getServerSideProps`). 在服务器组件内部获取的数据不需要序列化，因为除非将其传递给客户端组件，否则它不会跨越网络边界.更多服务器组件的 [数据获取](/docs/app/building-your-application/data-fetching/patterns#fetching-data-on-the-server) .

### Keeping Server-Only Code out of Client Components (Poisoning)

由于 JavaScript 模块可以在服务器组件和客户端组件之间共享，因此原本只打算在服务器上运行的代码可能会偷偷进入客户端。

以获取数据为例

```ts filename="lib/data.ts" switcher
export async function getData() {
  const res = await fetch("https://external-service.com/data", {
    headers: {
      authorization: process.env.API_KEY,
    },
  });

  return res.json();
}
```

乍一看，“getData”似乎同时适用于服务器和客户端。但由于环境变量“API_KEY”没有前缀“NEXT_PUBLIC”，因此它是一个只能在服务器上访问的私有变量。Next.js 在客户端代码中将私有环境变量替换为空字符串，以防止泄露安全信息。

因此，即使“getData（）”可以导入并在客户端上执行，它也无法按预期工作。虽然公开变量会使函数在客户端上工作，但它会泄露敏感信息。

因此，编写此函数的目的是只在服务器上执行。

### The "server only" package

为了防止客户端意外使用服务器代码，我们可以使用“server-only”制造构建时错误。

使用 `server-only`, 第一步安装包:

```bash filename="Terminal"
npm install server-only
```

然后把 server-only 引入到只在服务器上运行的代码中

```js filename="lib/data.js"
import "server-only";

export async function getData() {
  const res = await fetch("https://external-service.com/data", {
    headers: {
      authorization: process.env.API_KEY,
    },
  });

  return res.json();
}
```

现在,客户端组件引用 getData 方法都会导致一个构建错误

相应的包“clinet-only”可用于标记包含仅客户端代码的模块，例如访问“window”对象的代码。

### Data Fetching

尽管在客户端组件获取数据是可行的,但我们建议获取数据应该在服务器组件上,除非你有特别的理由.在服务器端获取数据有助于提升性能和用户体验.

[了解更多数据获取](/docs/app/building-your-application/data-fetching).

### Third-party packages

因为服务器组件是一个新的概念,'use client'指令刚刚开始被加入到使用了`useState`, `useEffect`, `createContext`的第三方包中,

如今，“npm”包中许多使用仅客户端功能的组件还没有该指令。这些第三方组件将在您自己的客户端组件中按预期工作，因为你自己的客户端组件具有“use Client”指令，但在服务器组件中不起作用。

假如,你安装了 `acme-carousel`包,包里有个 `<Carousel />` 组件. 该组件使用了 `useState` hook, 但不包含'use client'指令

`<Carousel />` 如果在你的客户端组件中工作,那么没有问题

```tsx filename="app/gallery.tsx" switcher
"use client";

import { useState } from "react";
import { Carousel } from "acme-carousel";

export default function Gallery() {
  let [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>View pictures</button>

      {/* Works, since Carousel is used within a Client Component */}
      {isOpen && <Carousel />}
    </div>
  );
}
```

然而,如果你直接把它放到你的服务器组件,就会报错.

```tsx filename="app/page.tsx" switcher
import { Carousel } from "acme-carousel";

export default function Page() {
  return (
    <div>
      <p>View pictures</p>

      {/* Error: `useState` can not be used within Server Components */}
      <Carousel />
    </div>
  );
}
```

这是因为 Next.js 不知道`<Carousel />`正在使用仅在客户端可用的功能。

要解决这个问题，你可以将依赖于客户端特性的第三方组件包装在你自己的客户端组件中：

```tsx filename="app/carousel.tsx" switcher
"use client";

import { Carousel } from "acme-carousel";

export default Carousel;
```

现在 `<Carousel />` 可以直接放到服务器组件中了:

```tsx filename="app/page.tsx" switcher
import Carousel from "./carousel";

export default function Page() {
  return (
    <div>
      <p>View pictures</p>

      {/*  Works, since Carousel is a Client Component */}
      <Carousel />
    </div>
  );
}
```

我们不希望你需要包装大多数第三方组件，因为你很可能会在客户端组件中使用它们。然而，有一个例外是提供者组件(provider component)，因为它们依赖于 React 的状态和上下文，并且通常需要在应用程序的根部使用. [Learn more about third-party context providers below](#rendering-third-party-context-providers-in-server-components).

#### Library Authors

- 包作者可以使用'use-client'指令标记包的客户端入口,这可以让用户直接导入,而不需要创建 Wrapper boundary 包一层.
- 通过在树的更深层次上使用'use client'，可以优化您的包，从而使导入的模块成为服务器组件模块图的一部分。
- 值得注意的是,一些打包工具可能会删掉你的'use client'指令, [React Wrap Balancer](https://github.com/shuding/react-wrap-balancer/blob/main/tsup.config.ts#L10-L13) 和 [Vercel Analytics](https://github.com/vercel/analytics/blob/main/packages/web/tsup.config.js#L26-L30)通过配置 esbuild 来包含'use client'.

## Context

大量 react 程序依赖[context](https://react.dev/reference/react/useContext) 实现组件内的状态共享, 包括直接使用'useContext'或引入一些第三方包.

在 Next.js 13 中，客户端组件完全支持上下文，但**不能**在服务器组件中直接创建或使用上下文。这是因为服务器组件没有 React 状态（因为它们不是可交互式的），并且上下文主要用于在更新了某些 React 状态后重新渲染树深处的交互式组件。

我们将讨论在服务器组件之间共享数据的替代方案，但首先，让我们看看如何在客户端组件中使用上下文。

### Using context in Client Components

在客户端组件中,context APIs 是完全支持的

```tsx filename="app/sidebar.tsx" switcher
"use client";

import { createContext, useContext, useState } from "react";

const SidebarContext = createContext();

export function Sidebar() {
  const [isOpen, setIsOpen] = useState();

  return (
    <SidebarContext.Provider value={{ isOpen }}>
      <SidebarNav />
    </SidebarContext.Provider>
  );
}

function SidebarNav() {
  let { isOpen } = useContext(SidebarContext);

  return (
    <div>
      <p>Home</p>

      {isOpen && <Subnav />}
    </div>
  );
}
```

然而，上下文组件通常在根组件附近使用，以共享全局状态，如当前的主题。由于服务器组件不支持上下文，因此尝试在应用程序的根目录创建上下文将导致错误：

```tsx filename="app/layout.tsx" switcher
import { createContext } from "react";

//  createContext is not supported in Server Components
export const ThemeContext = createContext({});

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <ThemeContext.Provider value="dark">{children}</ThemeContext.Provider>
      </body>
    </html>
  );
}
```

为了修复,你只能在客户端组件中创建上下文和 provider

```tsx filename="app/theme-provider.tsx" switcher
"use client";

import { createContext } from "react";

export const ThemeContext = createContext({});

export default function ThemeProvider({ children }) {
  return <ThemeContext.Provider value="dark">{children}</ThemeContext.Provider>;
}
```

你的服务器组件现在可以渲染 provider,因为 provider 被标记为客户端组件

```tsx filename="app/layout.tsx" switcher
import ThemeProvider from "./theme-provider";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html>
      <body>
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  );
}
```

通过在根目录中呈现提供程序，整个应用程序中的所有其他客户端组件都将能够使用此上下文。

> **Good to know**: 您应该在树中尽可能深入地使用 Provider——注意“ThemeProvider”如何只包装“｛children｝”而不是整个“＜ html ＞”文档。这使得 Next.js 更容易优化服务器组件的静态部分。

### Rendering third-party context providers in Server Components

第三方 npm 包经常含有 provider,并需要你放到你应用的根节点.如果这些 providers 包含'use-client'指令,它们就可以直接在你的服务器组件中使用,然而,因为服务器组件是一个新的概念,大多数第三方的 provider 还没有添加这个指令.

如果你试图渲染一个没有 `"use client"` 指令的 provier,会报错:

```tsx filename="app/layout.tsx" switcher
import { ThemeProvider } from "acme-theme";

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {/*  Error: `createContext` can't be used in Server Components */}
        <ThemeProvider>{children}</ThemeProvider>
      </body>
    </html>
  );
}
```

所以你还是需要包一层

```jsx filename="app/providers.js"
"use client";

import { ThemeProvider } from "acme-theme";
import { AuthProvider } from "acme-auth";

export function Providers({ children }) {
  return (
    <ThemeProvider>
      <AuthProvider>{children}</AuthProvider>
    </ThemeProvider>
  );
}
```

现在你可以直接用了

```jsx filename="app/layout.js"
import { Providers } from "./providers";

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

通过在根节点渲染 provider,库中的所有组件和 Hooks 都会在客户端组件中如期工作

只需要在第三方库中添加'use-client'一次,你就可以移除组件的 client wrapper 了.

### Sharing data between Server Components

因为服务器组件不是可交互的,因此不需要使用 React state,你也不需要 React context 来共享数据.相反,你可以使用原生的 Js 写法来为多个服务器组件创建公共数据.比如,把数据库连接分享到多个服务器组件中.

```ts filename="utils/database.ts" switcher
export const db = new DatabaseConnection();
```

```tsx filename="app/users/layout.tsx" switcher
import { db } from "@utils/database";

export async function UsersLayout() {
  let users = await db.query();
  // ...
}
```

```tsx filename="app/users/[id]/page.tsx" switcher
import { db } from "@utils/database";

export async function DashboardPage() {
  let user = await db.query();
  // ...
}
```

在上述示例中,layout 和 page 都可以进行数据库的查询,每个组件都可以通过导入'@utils/database'模块来实现共享数据库的连接.这种模式称作全局单例模式

### Sharing fetch requests between Server Components

fetch data 时，您可能希望在“page”或“layout”及其某些子组件之间共享“fetch”的结果。这是组件之间不必要的耦合，可能导致组件之间来回传递“道具”。

我们建议将数据获取逻辑放在组件中。在服务器组件中，fetch 请求会自动进行记忆化，因此每个路由片段可以准确地请求所需的数据，而无需担心重复请求。[ext.js 将从 fetch 缓存中读取相同的值。](/docs/app/building-your-application/caching#request-memoization)
