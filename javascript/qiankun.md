# qiankun

qiankun 微前端框架， 主应用和子应用的通信， 主应用和子应用的样式隔离， 主应用和子应用的 js 隔离， 主应用和子应用的 html 隔离。

实质上是通过监控 url 的变化， 来决定子应用的挂载和卸载。

而挂载的过程则是，通过 webpack 的打包配置， 将子应用打包成一个 js 文件， 然后通过 webpack 的 devServer 配置， 将子应用的 js 文件代理到主应用中。

demo 地址：

1. https://github.com/fengxianqi/qiankun-example
2. https://github.com/akira1ce/demo-qiankun

# 主应用

主应用中需要配置 qiankun 的配置， 在 qiankun 中注册子应用。

当微应用信息注册完之后，一旦浏览器的 url 发生变化，便会自动触发 qiankun 的匹配逻辑，所有 activeRule 规则匹配上的微应用就会被插入到指定的 container 中，同时依次调用微应用暴露出的生命周期钩子。



`yarn add qiankun`

```js
// 主入口，例如：main.js、app.js
import { registerMicroApps, start } from 'qiankun';

registerMicroApps([
  {
    name: 'react app', // app name registered
    entry: '//localhost:7100',
    /* 容器id 需要在 index.html 中存在 */
    container: '#yourContainer',
   /**
    * 匹配规则- 匹配到 /yourActiveRule 的 url 时， 会插入到 container 中
    * 子应用中有几点注意：
    * 1. 路由的 baseUrl
    * 2. 子应用的项目基础路径 - 例如 vite 中的 base
    * 3. 若存在跨域问题，需要在子应用中配置跨域
    */
  },
  {
    name: 'vue app',
    entry: { scripts: ['//localhost:7100/main.js'] },
    container: '#yourContainer2',
    activeRule: '/yourActiveRule2',
  },
]);

start();
```

# 子应用

入口文件中暴露 qiankun 生命周期。

qiankun 默认不支持 vite， 需要使用 vite-plugin-qiankun 插件。

其他 webpack 构建的项目，等同的方式去调整打包配置即可。

```js
import ReactDOM from 'react-dom/client';
import App from './App.tsx';
import './tailwind.css';
import React from 'react';

import { renderWithQiankun, qiankunWindow } from 'vite-plugin-qiankun/dist/helper';

function render(props: any = {}) {
  // 获取挂载点，确保在 qiankun 环境中，挂载点为 props.container，否则为 document.getElementById('root')
  const root = props.container
    ? props.container.querySelector('#root')
    : document.getElementById('root');
  ReactDOM.createRoot(root!).render(
    <React.StrictMode>
      <App />
    </React.StrictMode>
  );
}

//@ts-ignore
renderWithQiankun({
  // 挂载
  mount(props) {
    console.log('mount');
    render(props);
  },
  // 启动
  bootstrap() {
    console.log('bootstrap');
  },
  // 卸载
  unmount(props: any) {
    console.log('unmount');
    const { container } = props;
    const mountRoot = container?.querySelector('#root');
    ReactDOM.unmountComponentAtNode(mountRoot || document.querySelector('#root'));
  },
});

// 非 qiankun 环境，直接渲染
if (!qiankunWindow.__POWERED_BY_QIANKUN__) {
  render({});
}

```
