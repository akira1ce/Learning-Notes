# 配置 alias

> yarn add @types/node -D

```TypeScript
import tailwindcss from "tailwindcss";
import autoprefixer from "autoprefixer";
import { resolve } from 'path'

{
  resolve: {
    /* 需要配置 tsconfig.ts-compilerOptions-「baseUrl、paths」 */
    alias: {
      "@": resolve(__dirname, "src"),
      // ....
    },
  },
  css: {
    postcss: {
      plugins: [tailwindcss, autoprefixer],
    },
  }
}
```