![alt text](assets/image-2.png)

根因在某些复杂场景下，React 的代码被打包多份，在运行时产出了多个 React 实例。解法通过 Module Federation 的 `shared` 配置来避免多实例的出现。 如果有其他依赖出现多实例的问题，可以通过类似的方式解决。

```js
mfsu: {
  shared: {
    react: {
      singleton: true,
    },
  },
},
```

