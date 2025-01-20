# 理解服务端渲染

服务端渲染「SSR」，server-side-rendering。

这是一种将页面拼接的工作放到服务端的渲染方案。ssr 主要是针对动态内容的页面提出的一种解决方案。

## SSR 流程

1. 客户端访问 url，向服务端发起请求

2. 服务端接收到请求之后，根据路由执行不同的页面组件

3. 页面组件的执行过程中，会向其他服务端或者数据库请求数据，请求成功之后再与模板渲染成字符串

4. 页面组件执行完毕，得到页面内容，向客户端返回该内容

5. 客户端得到内容之后，由浏览器渲染页面，这样可以做到页面直出

## SSR 面临的问题

带来优异 SEO 表现的同时，原先分发在用户客户端上的压力，直接转移到服务器，当用户访问规模上来之后，服务器会承受巨大压力。

# 同构

同构组件组件指的是一个 React 组件**既能在服务端执行渲染得到静态 html 内容，也能够在客户端执行得到动态交互逻辑**。

服务端渲染的产物是简单的html字符串，但是组件除了静态部分之外，还有动态的交互的逻辑。

最好的方式就是通过 js 获取到元素来完成这个需求，也就是早期的 JSP/PHP+JQUERY。

React 同构组件得益于 React 的虚拟 DOM，可以存在前后端两种展现形式。

在服务端，React 提供了 `ReactDOMServer.renderToString` 与 `ReactDOMServer.renderToStaticMarkup` 将虚拟 DOM 渲染成 HTML 字符串。

在客户端，虚拟 DOM 通过 `ReactDOM.render` 方法渲染成真实 DOM。

## 水合「注水」

水合实际上就是在客户端给静态组件增加动态交互能力的过程。

hydrate -> JSX -> fiber -> fiber.stateNode -> 复用 DOM -> cilent diff

hydrate 触发的 diff 过程并不完全是一次 DOM 节点全部替换的过程。而是在执行过程中，判断已经渲染到页面上的真实 DOM 是否可以被复用的过程。

React 底层源码中提供了如下几个方法用于判断对应的节点是否可以被注水，如果此时可以注水则表示可以复用。如下高亮显示的代码，就是核心的判断逻辑，如下：

```js
export function canHydrateInstance(
  instance: HydratableInstance,
  type: string,
  props: Props,
): null | Instance {
  if (
    instance.nodeType !== ELEMENT_NODE ||
    type.toLowerCase() !== instance.nodeName.toLowerCase()
  ) {
    return null;
  }
  // This has now been refined to an element node.
  return ((instance: any): Instance);
}

export function canHydrateTextInstance(
  instance: HydratableInstance,
  text: string,
): null | TextInstance {
  if (text === '' || instance.nodeType !== TEXT_NODE) {
    // Empty strings are not parsed by HTML so there won't be a correct match here.
    return null;
  }
  // This has now been refined to a text node.
  return ((instance: any): TextInstance);
}

export function canHydrateSuspenseInstance(
  instance: HydratableInstance,
): null | SuspenseInstance {
  if (instance.nodeType !== COMMENT_NODE) {
    // Empty strings are not parsed by HTML so there won't be a correct match here.
    return null;
  }
  // This has now been refined to a suspense node.
  return ((instance: any): SuspenseInstance);
}
```

但是这里需要特别注意的是，由于是第一次执行，因此此时内存中的 fiber 树需要全部根据 JSX 重新创建，而复用的仅仅只是真实 DOM 节点。因此，当我们判断出来真实 DOM 节点可以被复用之后，会直接将其添加到对应的 fiber.stateNode 上，在后续的 completeWork 流程中，就不需要创建新的真实 DOM，而是直接复用。

## 面临的问题

1. 运行时环境不同
2. 三方包依赖包的冗余

客户端是在浏览器 JS 环境，服务端则是 NodeJS 环境。

有些逻辑只有在服务端执行，那么相关逻辑的三方包，就没必要打到客户端。

因此随着同构的发展，将服务端组件和客户端组件分开，慢慢变成了刚需，React Server Component 就是在这个基础之上，演变出来的解决方案。

# RSC

React Server Component 简称 RSC，和同构组件最大的区别在于，他要求开发者在开发中就将客户端组件与服务端组件严格区分开。把服务端组件从同构组件中拆离出来，让服务端组件只在服务端运行，是 RSC 的核心理念之一。

在 nextjs 中可通过 `use cilent` 和 `use server` 在界定组件类型。

当然默认不写就是服务端组件。

优势：

1. 客户端的打包体积大幅减小
2. 直接访问服务器资源
3. 安全性提升

# React 优化

## React Fiber Diff

React三大元素对象：虚拟DOM、Fiber对象、真实DOM。

JSX -> vNode -> Fiber -> Dom

React Diff 三要素：state、props、context。

优化思路：

核心在于，如何确保一个稳定的环境或引用。

1. 拆分解耦
2. 全局状态管理

拆分能力 -> nextjs 中对于服务端、客户端组件的拆分

只渲染一次的拆分为服务端组件，需要多次渲染的拆分为客户端组件。

# next

`"use client"` 当做客户端与服务端之间的边界。

> 注意： 这里的边界指的是，**模块结构**的边界。
> 
> 模块边界是由模块间`import`组成的，它不是文件结构树。也不是 Fiber tree，不是父子组件之间的关系。

🌰 如下：

```tsx
// Layout.tsx
import './globals.css'
import {ThemeProvider} from '@/components/switch-themes'

export const metadata = {
  title: '这波能反杀的付费课',
  description: 'Generated by Next.js',
}

export default async function RootLayout({ children }: any) {
  return (
    <html lang="zh" suppressHydrationWarning className='bg-background h-full'>
      <body className='text-sm text-foreground h-full'>
        {/* ThemeProvider */}
        <ThemeProvider defaultTheme='light'>
          {/* Page */}
          {children}
        </ThemeProvider>
      </body>
    </html>
  )
}
```

```txt
// 注意，这里指的是模块结构树. 该模块结构树，由 import 引入组成
//  + 表示还有子节点，- 表示叶子节点
+ Layout.tsx
  - ThemeProvider.tsx // 此时 context 是树形结构中的叶子结点
  + page.tsx
    + component1.tsx
    + component2.tsx
    + ...
```

## project-structure

next 中采用的是文件系统路由，因此项目结构中，文件夹和文件的层级结构，决定了路由的结构。

[project-structure](https://nextjs.org/docs/app/getting-started/project-structure)


## app-router

next 也是标准的基于文件系统的路由系统。

实际上，我们常用的配置式路由，鬼使神差的我们也是根据文件来来界定我们的路由的。

文件式路由反而省去了我们配置的历程，在后续路由出现变更的时候也相对比较好来进行变更。

例如我们之前项目上存在，一次几乎一般的路由需要调整位置，虽然我们也是动态后端路由，可在页面直接配置，但是也导致后续修改过的路由和文件没有一一对应，这也给后续维护带来了一定成本。

但基于文件可能就不一样了，我们可直接通过 copy 的方式，拷贝新的路由的同时，保留原路由一段时间，此时我们可以在这些旧的路由做显式的跳转提示，并告知后续会不在维护，一方面给用户缓冲的时间，再者也保证了后续路由和文件的一一对应关系。

### 路由跳转

1. Link
2. useRouter
3. redirect
4. history api

同样路由可传递 state 或 query 参数，可通过 `useSearchParams` 获取。

### 路由加载原理

1. 混合
2. 预加载
3. 路由缓存
4. 部分渲染

*混合*

在 history router 下，纯客户端的路由跳转，不会往服务端发送请求，服务端也无法感知到路由切换。

而由浏览器刷新操作，发起的硬导航，会重新向服务器发送请求，如果服务端没有做特殊处理，就会出现 404 的报错。

在 NextJS 的 App Router 设计中，对这两种情况做了混合处理。

在服务端，代码会按照路由段进行拆分，从而分割成更小的包。当刷新浏览器访问到服务端时，可以直接仅下载该路由对应的代码。

在客户端，路由所对应的代码可以通过预读取的方法，确保路由切换时的流畅体验。

这种混合的方式有效解决了我们刚才所说的问题。

**预加载**

见字如面，预先加载路由。

默认情况下，`<Link>` 组件出现在视口可视区域，就会进行预加载路由所需要的内容。

当然我们也课可以通过 `prefetch` 属性控制。

> 预加载的内容主要为 Rreact Server Component Payload，这是 RSC 服务端与客户端通信的主要传输内容和数据格式。它与每次渲染生成的 Fiber 树严格对应，是 Fiber 的压缩形式。

**路由缓存**

客户端访问过的路由页面的信息与预加载的信息「RSC Payload」，会被缓存在 `React Cache` 中。

在后续路由切换的时候，尽可能的使用缓存数据，而不是总是向服务端请求。

这里缓存的是页面信息，而不是页面，页面缓存可通过 `staleTimes` 配置「next.config.js」。

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    staleTimes: {
      dynamic: 30, // 针对 ssr，非静态、Link 的 prefetch 不为 true
      static: 180, // 静态，或者 Link 的 prefetch 为 true
    },
  },
}

module.exports = nextConfig
```

**部分渲染**

只有正在切换的路由会重新渲染，而共享的部分则不会渲染「Layout、Template」。

## Link

主要参数：

1. href - 跳转的路由
2. prefetch - 预加载路由
3. shallow - 是否浅跳转
4. scroll - 是否滚动到顶部
5. replace - 是否替换当前路由
6. as - 路由分类「可以当作路由的别名-中间件中会使用到」

### 中间件

在中间件中，我们通常会进行身份验证，然后将页面根据验证结果跳转到不同的路由中。此时，href 的结果是动态的，但是我们需要提高预设一个需要预加载的页面内容。可以通过 as 属性来做到这个事情。

中间件：

```ts
// middleware.ts
import { NextResponse } from 'next/server'

export function middleware(request: Request) {
  const nextUrl = request.nextUrl
  if (nextUrl.pathname === '/dashboard') {
    if (request.cookies.authToken) {
      return NextResponse.rewrite(new URL('/auth/dashboard', request.url))
    } else {
      return NextResponse.rewrite(new URL('/public/dashboard', request.url))
    }
  }
}
```

page 中的逻辑：

```tsx
// page.tsx
'use client'

import Link from 'next/link'
import useIsAuthed from './hooks/useIsAuthed' // Your auth hook

export default function Page() {
  const isAuthed = useIsAuthed()
  const path = isAuthed ? '/auth/dashboard' : '/public/dashboard'
  return (
    <Link as="/dashboard" href={path}>
      Dashboard
    </Link>
  )
}
```

## 路由分组

next 中可通过 `(groupName)` 实现路由分组。

> 注意路由分组后，不会影响路由的路径。
> 
> 同样，分组后的路由中都可以自定义 layout、loading、error 等。

![alt text](assets/image-11.png)

## 动态路由

next 中可通过 `[xxx]` 实现动态路由。

1. [slug] - 单个动态路由段
2. [...slug] - 多个动态路由段
3. [[...slug]] - 所有动态路由段

我们可以在 layout page route generateMetadata 中获得动态路由段的参数。

```tsx
export async function generateStaticParams() {
  return [{ slug: 'article' }, { slug: 'demo' }];
}

export default function Page({ params }: { params: { slug: string } }) {
  return <div>Hello, {params.slug}!</div>;
}
```

但这里静态生成，如果数量很大，也会导致构建时间过长、内存占用过高。

因此静态生成，需要根据实际情况来决定。

## server-render

服务端组件：只在服务端执行的组件。

在 next 中，渲染工作按照路由段进行拆分，以实现流式渲染与部分渲染。

流式传输能够更快的让页面具备交互能力。由于首屏直出之后，页面虽然快速的呈现了有效内容，但是还无法执行交互逻辑，此时需要有一个完整的水合过程。流式传输可以将渲染工作拆分成多个块，提前完成一部分页面内容的水合，而不需要等待整个页面都水合完成之后才能交互

**服务端组件的好处**

1. 更近的数据获取
2. 更安全的数据操作
3. 缓存
4. 更小的打包体积
5. 首屏直出
6. SEO

在 next 中，我们在访问页面的时候，会发送一个请求，请求中会包含一个 `_rsc` 参数，该参数的值为 `rxx9e`，该参数的值是随机生成的，用于标识该请求是服务端渲染的。

返回的响应即 RSC Payload，该 Payload 中包含了页面中所有的组件，以及组件的 props。

RSC Payload 是渲染 React Server Components 树的压缩格式，它完整的记录了代码在项目中的模块结构、渲染结果、props 传递内容。

```
//article?_rsc=rxx9e

1:"$Sreact.fragment"
2:I["[project]/node_modules/next/dist/client/components/layout-router.js [app-client] (ecmascript)",["static/chunks/_a91c21._.js","static/chunks/src_app_favicon_ico_mjs_ddfdf0._.js"],"default"]
3:I["[project]/node_modules/next/dist/client/components/render-from-template-context.js [app-client] (ecmascript)",["static/chunks/_a91c21._.js","static/chunks/src_app_favicon_ico_mjs_ddfdf0._.js"],"default"]
6:I["[project]/node_modules/next/dist/client/app-dir/link.js [app-client] (ecmascript)",["static/chunks/_e1f41f._.js","static/chunks/src_app_layout_tsx_61af54._.js","static/chunks/node_modules_3c5980._.js","static/chunks/src_2bfdc8._.js","static/chunks/node_modules_lucide-react_dist_esm_icons_86be41._.js","static/chunks/src_app_article_page_tsx_e73084._.js"],"default"]
7:I["[project]/src/app/article/tooltip.tsx [app-client] (ecmascript)",["static/chunks/_e1f41f._.js","static/chunks/src_app_layout_tsx_61af54._.js","static/chunks/node_modules_3c5980._.js","static/chunks/src_2bfdc8._.js","static/chunks/node_modules_lucide-react_dist_esm_icons_86be41._.js","static/chunks/src_app_article_page_tsx_e73084._.js"],"default"]
8:I["[project]/node_modules/next/dist/lib/metadata/metadata-boundary.js [app-client] (ecmascript)",["static/chunks/_a91c21._.js","static/chunks/src_app_favicon_ico_mjs_ddfdf0._.js"],"OutletBoundary"]
c:I["[project]/node_modules/next/dist/client/components/client-page.js [app-client] (ecmascript)",["static/chunks/_a91c21._.js","static/chunks/src_app_favicon_ico_mjs_ddfdf0._.js"],"ClientPageRoot"]
d:I["[project]/node_modules/next/dist/client/components/client-segment.js [app-client] (ecmascript)",["static/chunks/_a91c21._.js","static/chunks/src_app_favicon_ico_mjs_ddfdf0._.js"],"ClientSegmentRoot"]
e:I["[project]/node_modules/next/dist/client/components/error-boundary.js [app-client] (ecmascript)",["static/chunks/_a91c21._.js","static/chunks/src_app_favicon_ico_mjs_ddfdf0._.js"],"default"]
f:I["[project]/node_modules/next/dist/client/components/http-access-fallback/error-boundary.js [app-client] (ecmascript)",["static/chunks/_a91c21._.js","static/chunks/src_app_favicon_ico_mjs_ddfdf0._.js"],"HTTPAccessFallbackBoundary"]
10:I["[project]/node_modules/next/dist/lib/metadata/metadata-boundary.js [app-client] (ecmascript)",["static/chunks/_a91c21._.js","static/chunks/src_app_favicon_ico_mjs_ddfdf0._.js"],"MetadataBoundary"]
11:I["[project]/node_modules/next/dist/lib/metadata/metadata-boundary.js [app-client] (ecmascript)",["static/chunks/_a91c21._.js","static/chunks/src_app_favicon_ico_mjs_ddfdf0._.js"],"ViewportBoundary"]
12:"$SkResourceStore"
5:{"name":"Page","env":"Server","key":null,"owner":null,"props":{"params":"$@","searchParams":"$@"}}
4:D"$5"
4:[["$","$L6",null,{"href":"/article/111","children":"/article/111"},"$5"],["$","p",null,{"children":"These new APIs improve on renderToString by waiting for data to load for static HTML generation. They are designed to work with streaming environments like Node.js Streams and Web Streams. For example, in a Web Stream environment, you can prerender a React tree to static HTML with prerender"},"$5"],["$","p",null,{"children":"Prerender APIs will wait for all data to load before returning the static HTML stream. Streams can be converted to strings, or sent with a streaming response."},"$5"],["$","p",null,{"children":"... ..."},"$5"],["$","$L7",null,{},"$5"]]
a:{"name":"__next_outlet_boundary__","env":"Server","key":null,"owner":null,"props":{"ready":"$E(async function getMetadataAndViewportReady() {\n        await viewport();\n        await metadata();\n        return undefined;\n    })"}}
9:D"$a"
...
```

其中部分语义：

1. $：表示某个服务器组件生成的 DOM 定义。
2. I：表示一个模块，他们调用特定的脚本，这也是客户端组件如何被加载的方式（直接执行或者懒加载）。
3. HL：表示提示，并连接到特定的资源，例如 css 或者字体等

RSC Payload 包含的内容具体有

1. 服务器组件的渲染结果 HTML 片段
2. 客户端组件应该呈现的位置占位符，及其对代码模块的引用地址
3. 从服务器组件传递到客户端组件的任何 props

**渲染策略**

1. 静态渲染「SSG」
2. 动态渲染「SSR」

next 中会自动选择最优的渲染策略。

例如使用到一些获取动态数据的 API，则会选择 SSR 渲染：

1. cookies
2. headers
3. connection
4. draftMode
5. searchParams
6. unstable_noStore

**我们无需去记忆这些 API 具体有哪些，只有一个标准：那就是你试图获取最新数据、状态、参数时，你自然会使用到他们。**

## 最佳实践

1. Dark Mode

### Dark Mode

主要实现思路：html 标签中添加 className='dark' 属性，然后通过 css 来实现主题切换。

需要考虑的点是用户选择主题后，缓存的问题。

缓存的方案：

1. 使用 localStorage 缓存
2. 使用 cookie 缓存

localStorage 只能在客户端使用，导致页面会存在闪烁的问题，但是可以利用客户端组件会在服务端运行一次，提前设置`html-className`。

但此时，一定会导致 html 中的 className 属性的替换，导致水合失败。「暂时没找到好的解决方案」