**React 的函数式组件，其实就是纯函数**。

> UI = f(state)

```TypeScript
function Demo() {
  const [count, setCount] = useState(0)
  ...
}
```

**每一个函数的状态都被存在了另外一个模块里（Fiber tree）**。

也就是说，只要 React 允许，我们甚至可以在别的组件访问到任意一个组件里的状态。当然 React 对这种情况做了限制，只允许通过特定的语法来做到这个事情。

**函数组件中的所有的 hook 都是从外部传入的。**



例如我们有这样一个函数

```TypeScript
function Counter({x}) {
  const [count, setCount] = useState(0)
  return (
    <div>{x + count}</div>
  )
}
```

他可以等价于

```TypeScript
function Counter({x}, [count = 0, setCount]) {
  return (
    <div>{x + count}</div>
  )
}
```



这也是为什么 hook 不能在 if 环境下使用。

函数参数原本就有严格的顺序要求，若个别 hook 状态生效，会导致 state 参数错乱，那后续所有的操作都会有问题。

所以这并不是 react 的缺陷，设计就是如此。



为什么 state 一定要是不可变数据？

在纯函数的理念中，即相同输入必定会产生相同输出，而且不产生副作用（不修改函数外部的状态）。不可变数据是及其重要的环节。

如果所有的数据是可变的，那么数据就可能在函数执行的过程中发生变化，就可能导致输出产生变化。

