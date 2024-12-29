# npm 核心

npm 包核心的几个配置文件：

1. package.json
2. tsconfig.json
3. rollup.config.js 「也可是其他的打包工具」
4. LICENSE

# package.json

这里主要配置包的相关信息、以及各种出入口文件。

```js
{
  "name": "@akira1ce/r-hooks",
  "version": "1.0.6",
  // 工程构建模式 es or cjs
  "type": "module",
  // 入口文件
  "main": "dist/lib/index.js",
  // es 下入口文件
  "module": "dist/es/index.js",
  // 类型入口文件
  "types": "dist/es/hooks/index.d.ts",
  // 导出的文件目录
  "files": [
    "dist"
  ],
  "scripts": {
    "build": "rm -rf dist && rollup -c",
    "lint": "eslint src --ext .ts,.tsx",
    "format": "prettier --write src/**/*.ts"
  },
  // 关键词
  "keywords": [
    "react",
    "react-hooks"
  ],
  // license 类型
  "license": "MIT",
  "description": "react hooks",
  "browserslist": [
    ">1%",
    "last 1 version",
    "Firefox ESR",
    "not dead"
  ],
  // 仓库地址
  "repository": {
    "type": "git",
    "url": "https://github.com/akira1ce/r-hooks.git"
  },
  "author": {
    "name": "Akira Ice",
    "email": "akiraice@163.com"
  },
  "devDependencies": {
    "@rollup/plugin-commonjs": "^28.0.2",
    "@rollup/plugin-node-resolve": "^16.0.0",
    "@types/react": "^19.0.2",
    "eslint": "^9.17.0",
    "prettier": "^3.4.2",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "rollup": "^4.29.1",
    "rollup-plugin-typescript2": "^0.36.0",
    "tslib": "^2.8.1",
    "typescript": "^5.7.2"
  },
  "peerDependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  }
}
```

# tsconfig.json

```js
{
  "compilerOptions": {
    // 指定 ECMAScript 目标版本
    "target": "ES2016",
    // 指定模块代码生成
    "module": "ESNext",
    // 启用所有严格类型检查选项
    "strict": true,
    // 启用 CommonJS 和 ES 模块之间的互操作性
    "esModuleInterop": true,
    // 生成 .d.ts 声明文件
    "declaration": true,
    // 将输出结构重定向到目录
    "outDir": "dist",
    // 跳过声明文件的类型检查
    "skipLibCheck": true,
    // 指定模块解析策略
    "moduleResolution": "node",
    // 从 tslib 导入辅助函数
    "importHelpers": true,
    // 基础目录以解析非相对模块名称
    "baseUrl": "./",
    // 模块解析的路径映射
    "paths": {
      "@/*": ["src/*"]
    },
    // 指定要包含在编译中的库文件
    "lib": ["ES2016", "DOM"]
  },
}
```

# rollup.config.js

这里根据不同的项目或有差异，主要是用于打包的配置，将源文件打包成产物。

```js
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import typescript from 'rollup-plugin-typescript2';

export default {
  input: 'src/hooks/index.ts',
  output: [
    {
      dir: 'dist/lib',
      format: 'cjs',
    },
    {
      dir: 'dist/es',
      format: 'esm',
    },
  ],
  plugins: [resolve(), commonjs(), typescript()],
  external: ['react', 'react-dom'],
};
```