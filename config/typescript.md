
| 配置项            | 说明                                                   |
| ----------------- | ------------------------------------------------------ |
| `compilerOptions` | 编译器选项，包含多个子选项来控制 TypeScript 编译行为。 |
| `include`         | 指定要包含在编译中的文件或目录。                       |
| `exclude`         | 指定要排除在编译之外的文件或目录。                     |
| `files`           | 明确指定要编译的文件列表。                             |
| `extends`         | 继承另一个配置文件的配置。                             |
| `references`      | 用于配置项目引用，以支持多项目构建。                   |
| `typeRoots`       | 指定 TypeScript 应该包含的类型声明文件的目录。         |
| `types`           | 指定需要包含的类型声明包。                             |
| `baseUrl`         | 用于解析非相对模块名称的基准目录。                     |
| `paths`           | 模块名到基准目录的映射，配合 `baseUrl` 使用。          |
| `rootDir`         | 用于控制输出目录结构的根目录。                         |
| `outDir`          | 指定编译输出的目录。                                   |
| `sourceMap`       | 生成对应的 `.map` 文件，便于调试。                     |
| `declaration`     | 生成对应的 `.d.ts` 类型声明文件。                      |
| `removeComments`  | 从输出的文件中移除注释。                               |
| `noEmit`          | 不生成输出文件，仅进行类型检查。                       |
| `strict`          | 启用所有严格类型检查选项。                             |
| `esModuleInterop` | 允许对 CommonJS 和 ES 模块之间进行互操作。             |
| `skipLibCheck`    | 跳过对库文件的类型检查。                               |
| `target`          | 指定 ECMAScript 目标版本。                             |
| `module`          | 指定模块系统。                                         |

# alias

`yarn add @types/node -D`

```TypeScript
{
  "compilerOptions": {
    /* path */
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```