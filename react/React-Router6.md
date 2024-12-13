> yarn add react-router-dom

# 路由配置

```TypeScript
import { createBrowserRouter } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/',
    /* Component/element: 路由组件 */
    Component: BaseLayout,
    /* ErrorBoundary/errorElement: 错误路由 */
    ErrorBoundary: ErrorPage,
    /* 子路由 */
    children: [
      /* index 类似默认子路由 */
      { index: true, Component: Counter1 },
      { path: 'counter2', Component: Counter2 },
    ],
  },
]);

export default router;
```

# RouterProvider

```TypeScript
import { RouterProvider } from "react-router-dom";
import router from "@/configs/router.config";

function App() {
  return (
    <>
      <div id="app">
        <RouterProvider router={router} />
      </div>
    </>
  );
}

export default App;
```

# 路由跳转

Link、Navigate、useNavigate

```TypeScript
export default function Comp () {
  const nav = useNavigate()
  return <>
    <Link to={}/>
  </>
}
```



