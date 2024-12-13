「新的 API」

- use

- useActionState

- useFormStatus

- useOptimistic



「核心」

- 19 的开发者意图引导用户**弱化**对于 **useEffect** 的依赖。

- **并发模式**

- **React Compiler**

- **Form Action**



# use

> promise → state

利用 use 可以获取到 promise、context 中的值。

这里我们需要特别注意的是，**Promise** 是指的一个已经创建好的 Promise 对象，并且，在该 promise 对象中已经有了确定的 `resolve` 的结果，use 读取的是 resolve 的值。

```TypeScript
const _api2 = new Promise((resolve) => {
  resolve({ value: '_api2' })
})

// good
const result = use(_api2)
```

但 use 不能读取到 return Promise 中的值，且会报错。

```TypeScript
const _api3 = () => {
  return new Promise(resolve => {
    resolve({ value: '_api3' })
  })
}

// bad: get an error
const result = use(_api3())
```

![alt text](assets/image-9.png)

此时就需要 Suspense 的介入。

> 与其他 hooks 不一样的是，use 可以在循环和条件判断语句中使用。
可将其当做工具函数看待。

# Suspense

`Suspense` 能够捕获到子组件首次渲染的异常。因此我们常常将 `Suspense` 当成一种组件错误边界来处理。

```JavaScript
import {use, Suspense} from 'react'
import Message from './Message'

const _api3 = () => {
  return new Promise(resolve => {
    resolve({ value: 'React does not preserve any state for renders that got suspended before they were able to mount for the first time. When the component has loaded, React will retry rendering the suspended tree from scratch.' })
  })
}

export default function Demo01() {
  const promise = _api3()
  return (
    <Suspense fallback=''>
      <Content promise={promise} />
    </Suspense>
  )
}

function Content(props) {
  const {value} = use(props.promise)
  return (
    <Message message={value} />
  )
}
```

## 工作原理

Suspense 提供了一个加载数据的标准。在源码中，Suspense 的子组件被称为 `primary`。

当 react 在 beginWork 的过程中（diff 过程），遇到 `Suspense` 时，首先会尝试加载 `primary` 组件。如果 `primary` 组件只是一个普通组件，那么就顺利渲染完成。

如果 `primary` 组件是一个包含了 use 读取异步 promise 的组件，它会在首次渲染时，**抛出一个异常**。react 捕获到该异常之后，发现是一个我们在语法中约定好的 promise，那么就会将其 `then` 的回调函数保存下来，并将下一个 `next` beginWork 的组件重新指定为 `Suspense`。

此时 promise 在请求阶段，因此再次 beginWork Suspense 组件时，会跳过 `primary` 的执行而直接渲染 `fallback`

当 `primary` 中的 promise 执行完成时「resolve」，会执行刚才保存的 `then` 方法，此时会触发 `Suspense` 再次执行「调度一个更新任务」。由于此时 `primary` 中的 promise 已经 resolve，因此此时就可以拿到数据直接渲染 `primary` 组件。

```Shell
Suspense ->
primary -> 
Suspense -> 
fallback -> 
waiting -> resolve() -> 
Suspense -> 
primary -> 
```

在 Suspense 的加持下，异步请求不再需要在 useEffect 中执行，也不再需要 re-render 一遍。

## 对比

Suspense + use

```JavaScript
export default function Index() {
  const promise = getMessage()
  return (
    <Suspense fallback={<Skeleton />}>
      <Message promise={promise} />
    </Suspense>
  )
}
```

useEffect

```JavaScript
export default function Index() {
  const [content, update] = useState({value: ''})
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    getMessage().then(res => {
      update(res)
      setLoading(false)
    })
  }, []);

  if (loading) {
    return <Skeleton />
  }

  return (
    <Message message={content.value} />
  )
}
```

## 初始化不请求

默认 `promise` 为 `null`，通过事件更新 promise 状态。

```JavaScript
import {use, useState, Suspense} from 'react'
import Message from './Message'
import Skeleton from './Skeleton'
import Button from './Button'
import {getMessage} from './api'

export default function Demo01() {
  const [promise, update] = useState(null)

  function __handler() {
    update(getMessage())
  }

  return (
    <>
      <div className='text-right mb-4'>
        <Button onClick={__handler}>更新数据</Button>
      </div>
      <Suspense fallback={<Skeleton />}>
        <Content promise={promise} />
      </Suspense>
    </>
  )
}

function Content(props) {
  if (!props.promise) {
    return <Message message='' />
  }

  const {value} = use(props.promise)
  return (
    <Message message={value} />
  )
}
```

## 初始化请求

只需要给 promise 一个默认的初始值即可。

```JavaScript
import {use, useState, Suspense} from 'react'
import Message from './Message'
import Skeleton from './Skeleton'
import Button from './Button'
import {getMessage} from './api'

export default function Demo02() {
  const [promise, update] = useState(getMessage())

  function __handler() {
    update(getMessage())
  }

  return (
    <>
      <div className='text-right mb-4'>
        <Button onClick={__handler}>更新数据</Button>
      </div>
      <Suspense fallback={<Skeleton />}>
        <Content promise={promise} />
      </Suspense>
    </>
  )
}

function Content(props) {
  const {value} = use(props.promise)
  return (
    <Message message={value} />
  )
}
```

## 嵌套

Suspense 嵌套会导致后续的请求都依赖前面的请求结果。

> 没有严格的先后顺序，没必要嵌套。

# Ref

在 react 19 中 ref 由独立的参数变为 props 中的一员。

forwardRef 被无情哈拉少了。

> 通常透传 ref 的作用是将自身的一些控制权交给外部（也可暴露一些状态），也被称作「控制反转」。

```TypeScript
const Comp = (props) => {
  const {ref, ...others} = props;
  return <>xxx</>
}
```

# Context

创建：`Context.Provider` → `Context`

获取：`useContext` → `use`

旧版

```TypeScript
import { createContext, useState, useContext } from 'react';

export const Context = createContext({});

export const Provider = (props: any) => {
  const [value, setValue] = useState({});
  return <Context.Provider value={{ value, setValue }}>{props.children}</Context.Provider>;
};

export const Comp = () => {
  const {value, setValue} = useContext(Context)
  return <>Comp</>;
};
```

新版（React19）

```TypeScript
import { createContext, useState, use } from 'react';

export const Context = createContext({});

export const Provider = (props: any) => {
  const [value, setValue] = useState({});
  return <Context value={{ value, setValue }}>{props.children}</Context>;
};

export const Comp = () => {
  const {value, setValue} = use(Context)
  return <>Comp</>;
};
```

# 并发API

降低 UI 任务更新的优先级（推迟UI渲染任务）

1. useDeferredValue

    其他任务会中断其状态更新。

1. useTransition

    不会中断回调中任务的执行，而是推迟UI更新。（适用于低优先级任务）

## useDeferredValue

```TypeScript
export function useDeferredValue<T>(value: T): T;
```

推迟 UI 的更新，将对应任务的优先级降低，使其可以被插队与中断。

这里将 useDeferredValue 传递给子组件（耗时子组件），可以发现此时第二个 counter 的更新已经落后了。

但是快速点击的时候，明显第一个 counter 已经出现卡顿现象了。

其实很简单，因为在我们的模拟案例中，**并没有把耗时定位在渲染上**。这就和实际的情况不太一样了。我们把耗时写在了 Expensive 函数里，而这个函数每次都会执行，它的执行阻塞了渲染。所以我们会觉得第一个 counter 的更新变得比较卡顿

```TypeScript
import {useState, useDeferredValue} from 'react'

export default function Index() {
  const [counter, setCounter] = useState(0)
  const deferred = useDeferredValue(counter, 0)

  function __clickHanler() {
    setCounter(counter + 1)
  }

  return (
    <div className='flex justify-between items-center'>
      <div>
        <div>counter: {counter}</div>
        <Expensive counter={deferred} />
      </div>
      <button onClick={__clickHanler}>counter++</button>
    </div>
  )
}

const Expensive = ({counter}) => {
  const start = performance.now()
  while (performance.now() - start < 200) {}
  return (
    <div className="mt-4">Deferred: {counter}</div>
  )
}
```

> 所以这里我们一定要区分开**渲染任务**和 **Expensive** 函数，他们是不同的，UI 渲染是一个异步任务，而 Expensive 函数是**同步执行**的。useDeferredValue 推迟的是 UI 渲染任务。因此，我们需要特别注意的是，不要在同步逻辑上执行过多的耗时任务。

这里我们可以通过任务拆分的方式，把执行耗时时间分散到更多的子组件中去，这样 React 就可以利用任务中断的机制，在不阻塞渲染的情况下，中断低优先级的任务。

**下游组件的任务优先级相较于上游一定是更低的。**

下面例子中采用了 React Complier，组件均有相应缓存优化。

```TypeScript
import {useState, useDeferredValue} from 'react'

export default function Index() {
  const [counter, setCounter] = useState(0)
  const deferred = useDeferredValue(counter)

  function __clickHanler() {
    setCounter(counter + 1)
  }

  return (
    <div className='flex justify-between items-center'>
      <div>
        <div>counter: {counter}</div>
        <Expensive counter={deferred} />
      </div>
      <button onClick={__clickHanler}>counter++</button>
    </div>
  )
}

const Expensive = ({counter}) => {
  let items = []
  for (let i = 0; i < 200; i++) {
    items.push(<SlowItem key={i} counter={counter} />);
  }

  return (
    <div className='mt-4 text-green-500'>
      <div>Deferred: {counter}</div>
      <ul className='h-32 hidden'>
        {items}
      </ul>
    </div>
  );
}

function SlowItem({counter}) {
  let startTime = performance.now();
  while (performance.now() - startTime < 1) {
    // Do nothing for 1 ms per item to emulate extremely slow code
  }

  return (
    <li>{counter}</li>
  )
}
```

此时一个很明显的区别就是，counter 的 UI 变化变得更加流畅了。这是因为耗时被拆分到了多个子组件中，React 就有机会中断这些函数的执行，并执行优先级更高的任务，以确保高优先级任务的流畅。

如果没有使用 React Complier ，则需要对 Expensive 组件通过 memo 包裹，达到缓存效果。

**当更新发生时，useDefferdValue 会首先使用旧值传递给组件。**

因此，当 counter 发生变化时，deferred 依然是旧值，那么此时，如果我们使用 memo 包裹，Expensive 的 props 就没有发生变化，我们可以跳过此次针对 Expensive 的更新。

所以当我们快速点击时，Expensive 总是接受到旧值，它本身的渲染也会被中断，因此最终的表现结果是，当我们连续点击 7 次，counter 从 0 依次变成 7，而 deferred 会直接从 0 变成 7。

### **运行原理**

useDeferredValue 会尝试将 UI 任务更新两次。



第一次，会给子组件传递旧值。此时 `Expensive` 接收到的 props 会与上一次完全相同。如果结合了 React.memo，那么该组件就不会重新渲染。该组件可以重复使用之前的渲染结果

> Compiler 编译之后不需要 memo

此时，高优先级的任务渲染会发生，渲染完成之后，将会开始第二次渲染。此时，将会传入刚才更新之后的新值。对于 `Expensive` 而言，props 发生了变化，整个组件会重新渲染。



我们通常会将已经非常明确的耗时任务标记为 deferred，因此，这些任务都被视为低优先级。当重要的高优先级更新已经完成，低优先级任务在第二次渲染时尝试更新...

在它第二次更新的过程中，如果又有新的高优先级任务进来，那么 React 就会中断并放弃第二次更新，去执行高优先级的任务。



这样的运行机制有一个非常重要的好处。

那就是，如果你的电脑性能足够强悍，那么第二次的更新可能会快速完成，高优先级的任务来不及中断，那么我们的页面响应就是非常理想的。

但是如果我们的电脑性能比较差，第二次更新还没完成，新的高优先级任务又来了，那么就可以通过中断的方式，降级处理，保证重要 UI 的流畅，放弃低优先级任务。



## useTransition

与 useDeferredValue 一样，降低 UI 更新优先级，只是用法不一样。

```TypeScript
export function useTransition(): [boolean, TransitionStartFunction];

const [isPending, startTransition] = useTransition()
```

`startTransition` 可以标记一个或者多个状态的 `set` 方法，将他们标记为 `transition`，也就是调低他们更新的优先级。

> 但是这里需要注意的是，被调低的不是 set 方法本身的执行，而是其对应的 UI 更新。

`isPending` 表示是否还有未完成的 UI 更新任务。我们可以利用这个状态来判断请求是否正在发生。

可同时包裹多个 set （状态的set方法）

```TypeScript
function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('about');
  const [other, setOther] = useState(false);

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
      setOther(true)
    });
  }
  // ...
}
```

# Compiler

React Forget → React Compiler

缘由：状态变更 → 顶层向下的更新机制 → 冗余 re-render

> **对比的成本非常小，但是 re-render 的成本较高**

## 编译对比

源代码

```TypeScript
import {useState} from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)

  function __clickHanler() {
    setCount(count + 1)
  }

  return (
    <div>
      <div>A Base Case</div>
      <div className="flex items-center justify-between">
        <div>currnt count is: {count}</div>
        <button onClick={__clickHanler}>Increment</button>
      </div>
    </div>
  )
}
```

compiler 编译后：

```TypeScript
import {useState} from 'react'
import Button from './Button'
import {_c} from './useCache.js'

export default function Counter() {
  const $ = _c(10);

  if ($[0] !== "a13b836c47c4cd480504d73b45661476522265776f255f2150833079731132ac") {
    for (let $i = 0; $i < 10; $i += 1) {
      $[$i] = Symbol.for("react.memo_cache_sentinel");
    }
    $[0] = "a13b836c47c4cd480504d73b45661476522265776f255f2150833079731132ac";
  }

  const [count, setCount] = useState(0);
  let t0;

  if ($[0] !== count) {
    t0 = function __clickHanler() {
      setCount(count + 1);
    };

    $[0] = count;
    $[1] = t0;
  } else {
    t0 = $[1];
  }

  const __clickHanler = t0;
  let t1;

  if ($[2] === Symbol.for("react.memo_cache_sentinel")) {
    t1 = <div>A Base Case</div>;
    $[2] = t1;
  } else {
    t1 = $[2];
  }

  let t2;

  if ($[3] !== count) {
    t2 = <div>currnt count is: {count}</div>;
    $[3] = count;
    $[4] = t2;
  } else {
    t2 = $[4];
  }

  let t3;

  if ($[5] !== __clickHanler) {
    t3 = <Button onClick={__clickHanler}>Increment</Button>;
    $[5] = __clickHanler;
    $[6] = t3;
  } else {
    t3 = $[6];
  }

  let t4;

  if ($[7] !== t2 || $[8] !== t3) {
    t4 = (
      <div>
        {t1}
        <div className="flex items-center justify-between">
          {t2}
          {t3}
        </div>
      </div>
    );
    $[7] = t2;
    $[8] = t3;
    $[9] = t4;
  } else {
    t4 = $[9];
  }

  return t4;
}
```

这里出现大量 Symbol.for。

> Symbol 也是用来创建全局变量的，Symbol.for 也是一样，但其创建重复 key 的时候会走缓存里面去拿数据。

```TypeScript
var a = Symbol.for('for')
var b = Symbol.for('for')

a === b // true

// 创建一个 symbol 并放入 symbol 注册表中，键为 "foo"
Symbol.for("foo");

// 从 symbol 注册表中读取键为"foo"的 symbol
Symbol.for("foo"); 
```

实际上原理很简单，就是将所有的计算结果都缓存下来。

注意这里最后 t4 这一段，是将分段的缓存进行整合并返回，特别需要注意的是这里没有对 t1 做条件判断，原因在于 t1 比较稳定，可以理解为静态节点，自然 compiler 在转译的过程中进行了优化。

> 1、我希望首次渲染时，页面渲染更少的内容，因此此时，只能先渲染默认的 Panel。其他 Panel 需要在点击对应的按钮时，才渲染出来。

    2、在切换过程中，我希望能够缓存已经渲染好的 Panel，只需要在样式上做隐藏，而不需要在后续的交互中重复渲染内容
    
    3、当四个页面都渲染出来之后，再做切换时，此时只会有两个页面会发生变化，上一个选中的页面与下一个选中的页面。另外的页面不参与交互，则不应该 re-render。

```TypeScript
import React, { useState } from 'react';

const Panel1 = () => {
  console.log('Rendering Panel 1');
  const [count, setCount] = useState(1);
  return <div onClick={() => setCount(count + 1)}>count1 {count}</div>;
};

const Panel2 = () => {
  console.log('Rendering Panel 2');
  const [count, setCount] = useState(1);
  return <div onClick={() => setCount(count + 1)}>count2 {count}</div>;
};

const Panel3 = () => {
  console.log('Rendering Panel 3');
  const [count, setCount] = useState(1);
  return <div onClick={() => setCount(count + 1)}>count3 {count}</div>;
};

const Panel4 = () => {
  console.log('Rendering Panel 4');
  const [count, setCount] = useState(1);
  return <div onClick={() => setCount(count + 1)}>count4 {count}</div>;
};

const App = () => {
  const items = [
    { id: 'Panel1', content: 'Panel 1', component: <Panel1 /> },
    { id: 'Panel2', content: 'Panel 2', component: <Panel2 /> },
    { id: 'Panel3', content: 'Panel 3', component: <Panel3 /> },
    { id: 'Panel4', content: 'Panel 4', component: <Panel4 /> },
  ];

  const [activePanel, setActivePanel] = useState('Panel1'); // 当前选中的 Panel
  const [renderedPanels, setRenderedPanels] = useState(['Panel1']); // 缓存已渲染的 Panel

  // 处理 Panel 切换
  const handlePanelClick = (panelId) => {
    setActivePanel(panelId);

    // 如果 Panel 没有渲染过，则将其添加到缓存中
    if (!renderedPanels.includes(panelId)) {
      setRenderedPanels([...renderedPanels, panelId]);
    }
  };

  return (
    <div>
      <div className="button-group">
        {items.map((item) => (
          <button className="mr-2" key={item.id} onClick={() => handlePanelClick(item.id)}>
            Show {item.content}
          </button>
        ))}
      </div>

      <div className="panel-container">
        {items.map((item) => (
          <div
            key={item.id}
            style={{
              display: activePanel === item.id ? 'block' : 'none',
            }}
          >
            {/* 只有已渲染过的 Panel 才会显示内容 */}
            {renderedPanels.includes(item.id) && item.component}
          </div>
        ))}
      </div>
    </div>
  );
};

export default App;
```

不同于 Vue 等框架，React 依然不做**依赖收集**。

Compiler编译之后，会将几乎所有应该缓存的元素和函数缓存起来，命中率很好。

最终还是通过自上而下的 diff 来找出需要更新的节点，过程中存在大量的**判断**。

> 实际上大量的判断并不耗时

## 最佳实践

1. 不再使用 useCallback、useMemo、Memo 等缓存函数

2. 丢掉闭包的心智负担，放心使用即可

3. 引入严格模式

4. 在你不熟悉的时候引入 eslint-plugin-react-compiler

5. 当你熟练之后，弃用它，因为有的时候我们就是不想让它编译我们的组件

6. 更多的使用 use 与 Action 来处理异步逻辑

7. 尽可能少的使用 useEffect

当你不希望你的组件被 Compiler 编译的时候，只需要通过 var 定义状态即可。

var [counter, setCounter] = useState(0)

因为这不符合它的语法规范。

### 优化耗时组件

```TypeScript
import { useState } from 'react'

function App() {
  var [counter, setCounter] = useState(0)

  function __clickHanler() {
    setCounter(counter + 1)
  }

  return (
    <div>
      <div>A Expensive Case</div>
      <div className="flex items-center justify-between mt-4">
        <div className="counter">current counter is: {counter}</div>
        <button onClick={__clickHanler}>counter++</button>
      </div>
      <Expensive/>
    </div>
  )
}

export default App


function Expensive() {
  var cur = performance.now()
  while (performance.now() - cur < 100) {
    // block 100ms
  }

  return (
    <div className='border border-red-500 rounded p-4 mt-4 text-red-500 text-sm leading-6'>
      这是模拟的一个耗时子组件，我的渲染至少要损耗 100ms 的时间，因此如果不做任何优化的情况下，当你快速点击按钮时，你会感受到 counter 的递增会有明显的掉帧。
    </div>
  )
}
```

Compiler 对于耗时组件编译后结果：

```TypeScript
let t5;

if ($[10] === Symbol.for("react.memo_cache_sentinel")) {
  t5 = <Expensive />;
  $[10] = t5;
} else {
  t5 = $[10];
}
```

由于没有依赖任何状态，当作静态节点缓存。

But，实际开发过程中，很少有这么独立的个体存在。



