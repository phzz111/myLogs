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

Props passed from the Server to Client Components need to be [serializable](https://developer.mozilla.org/en-US/docs/Glossary/Serialization). This means that values such as functions, Dates, etc, cannot be passed directly to Client Components.

> **Where is the Network Boundary?**
>
> In the App Router, the network boundary is between Server Components and Client Components. This is different from the Pages where the boundary is between `getStaticProps`/`getServerSideProps` and Page Components. Data fetched inside Server Components do not need to be serialized as it doesn't cross the network boundary unless it is passed to a Client Component. Learn more about [data fetching](/docs/app/building-your-application/data-fetching/patterns#fetching-data-on-the-server) with Server Components.

### Keeping Server-Only Code out of Client Components (Poisoning)

Since JavaScript modules can be shared between both Server and Client Components, it's possible for code that was only ever intended to be run on the server to sneak its way into the client.

For example, take the following data-fetching function:

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

At first glance, it appears that `getData` works on both the server and the client. But because the environment variable `API_KEY` is not prefixed with `NEXT_PUBLIC`, it's a private variable that can only be accessed on the server. Next.js replaces private environment variables with the empty string in client code to prevent leaking secure information.

As a result, even though `getData()` can be imported and executed on the client, it won't work as expected. And while making the variable public would make the function work on the client, it would leak sensitive information.

So, this function was written with the intention that it would only ever be executed on the server.

### The "server only" package

To prevent this sort of unintended client usage of server code, we can use the `server-only` package to give other developers a build-time error if they ever accidentally import one of these modules into a Client Component.

To use `server-only`, first install the package:

```bash filename="Terminal"
npm install server-only
```

Then import the package into any module that contains server-only code:

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

Now, any Client Component that imports `getData()` will receive a build-time error explaining that this module can only be used on the server.

The corresponding package `client-only` can be used to mark modules that contain client-only code – for example, code that accesses the `window` object.

### Data Fetching

Although it's possible to fetch data in Client Components, we recommend fetching data in Server Components unless you have a specific reason for fetching data on the client. Moving data fetching to the server leads to better performance and user experience.

[Learn more about data fetching](/docs/app/building-your-application/data-fetching).

### Third-party packages

Since Server Components are new, third-party packages in the ecosystem are just beginning to add the `"use client"` directive to components that use client-only features like `useState`, `useEffect`, and `createContext`.

Today, many components from `npm` packages that use client-only features do not yet have the directive. These third-party components will work as expected within your own [Client Components](#the-use-client-directive) since they have the `"use client"` directive, but they won't work within Server Components.

For example, let's say you've installed the hypothetical `acme-carousel` package which has a `<Carousel />` component. This component uses `useState`, but it doesn't yet have the `"use client"` directive.

If you use `<Carousel />` within a Client Component, it will work as expected:

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

However, if you try to use it directly within a Server Component, you'll see an error:

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

This is because Next.js doesn't know `<Carousel />` is using client-only features.

To fix this, you can wrap third-party components that rely on client-only features in your own Client Components:

```tsx filename="app/carousel.tsx" switcher
"use client";

import { Carousel } from "acme-carousel";

export default Carousel;
```

Now, you can use `<Carousel />` directly within a Server Component:

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

We don't expect you to need to wrap most third-party components since it's likely you'll be using them within Client Components. However, one exception is provider components, since they rely on React state and context, and are typically needed at the root of an application. [Learn more about third-party context providers below](#rendering-third-party-context-providers-in-server-components).

#### Library Authors

- In a similar fashion, library authors creating packages to be consumed by other developers can use the `"use client"` directive to mark client entry points of their package. This allows users of the package to import package components directly into their Server Components without having to create a wrapping boundary.
- You can optimize your package by using ['use client' deeper in the tree](#moving-client-components-to-the-leaves), allowing the imported modules to be part of the Server Component module graph.
- It's worth noting some bundlers might strip out `"use client"` directives. You can find an example of how to configure esbuild to include the `"use client"` directive in the [React Wrap Balancer](https://github.com/shuding/react-wrap-balancer/blob/main/tsup.config.ts#L10-L13) and [Vercel Analytics](https://github.com/vercel/analytics/blob/main/packages/web/tsup.config.js#L26-L30) repositories.

## Context

Most React applications rely on [context](https://react.dev/reference/react/useContext) to share data between components, either directly via [`createContext`](https://react.dev/reference/react/useContext), or indirectly via provider components imported from third-party libraries.

In Next.js 13, context is fully supported within Client Components, but it **cannot** be created or consumed directly within Server Components. This is because Server Components have no React state (since they're not interactive), and context is primarily used for rerendering interactive components deep in the tree after some React state has been updated.

We'll discuss alternatives for sharing data between Server Components, but first, let's take a look at how to use context within Client Components.

### Using context in Client Components

All of the context APIs are fully supported within Client Components:

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

However, context providers are typically rendered near the root of an application to share global concerns, like the current theme. Because context is not supported in Server Components, trying to create a context at the root of your application will cause an error:

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

To fix this, create your context and render its provider inside of a Client Component:

```tsx filename="app/theme-provider.tsx" switcher
"use client";

import { createContext } from "react";

export const ThemeContext = createContext({});

export default function ThemeProvider({ children }) {
  return <ThemeContext.Provider value="dark">{children}</ThemeContext.Provider>;
}
```

Your Server Component will now be able to directly render your provider since it's been marked as a Client Component:

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

With the provider rendered at the root, all other Client Components throughout your app will be able to consume this context.

> **Good to know**: You should render providers as deep as possible in the tree – notice how `ThemeProvider` only wraps `{children}` instead of the entire `<html>` document. This makes it easier for Next.js to optimize the static parts of your Server Components.

### Rendering third-party context providers in Server Components

Third-party npm packages often include Providers that need to be rendered near the root of your application. If these providers include the `"use client"` directive, they can be rendered directly inside of your Server Components. However, since Server Components are so new, many third-party providers won't have added the directive yet.

If you try to render a third-party provider that doesn't have `"use client"`, it will cause an error:

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

To fix this, wrap third-party providers in your own Client Component:

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

Now, you can import and render `<Providers />` directly within your root layout.

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

With the providers rendered at the root, all the components and hooks from these libraries will work as expected within your own Client Components.

Once a third-party library has added `"use client"` to its client code, you'll be able to remove the wrapper Client Component.

### Sharing data between Server Components

Since Server Components are not interactive and therefore do not read from React state, you don't need React context to share data. Instead, you can use native JavaScript patterns for common data that multiple Server Components need to access. For example, a module can be used to share a database connection across multiple components:

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

In the above example, both the layout and page need to make database queries. Each of these components shares access to the database by importing the `@utils/database` module. This JavaScript pattern is called global singletons.

### Sharing fetch requests between Server Components

When fetching data, you may want to share the result of a `fetch` between a `page` or `layout` and some of its children components. This is an unnecessary coupling between the components and can lead to passing `props` back and forth between components.

Instead, we recommend colocating data fetching alongside the component that consumes the data. [`fetch` requests are automatically memoized](/docs/app/building-your-application/caching#request-memoization) in Server Components, so each route segment can request exactly the data it needs without worrying about duplicate requests. Next.js will read the same value from the `fetch` cache.
