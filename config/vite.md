# 配置 alias

> yarn add @types/node -D

```TypeScript
import { resolve } from 'path'

{
  resolve: {
    alias: {
      "@": resolve(__dirname, "src"),
      // ....
    },
  },
}
```

# Tsconfig

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

# Tailwindcss

```Shell
yarn add -D tailwindcss postcss autoprefixer && npx tailwindcss init -p && yarn add -D prettier prettier-plugin-tailwindcss
```

```JavaScript
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

```JavaScript
/** @type {import('tailwindcss').Config} */
export default {
  content: ["./src/**/*.{js,ts,jsx,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

```CSS
@tailwind base;
@tailwind components;
@tailwind utilities;
```

```TypeScript
import tailwindcss from "tailwindcss";
import autoprefixer from "autoprefixer";

{
  css: {
    postcss: {
      plugins: [tailwindcss, autoprefixer],
    },
  }
}
```

# Prettier

```Shell
yarn add -D prettier prettier-plugin-tailwindcss && yarn add -D @trivago/prettier-plugin-sort-imports
```

**不生效重启**



```JSON
{
  "singleQuote": true,
  "jsxSingleQuote": false,
  "trailingComma": "all",
  "printWidth": 100,
  "proseWrap": "never",
  "plugins": ["@trivago/prettier-plugin-sort-imports","prettier-plugin-tailwindcss"],
  "importOrder": ["^(react|react-dom)$", "^([a-z]|@[a-z])", "", ".*"],
  "overrides": [
    {
      "files": ".prettierrc",
      "options": {
        "parser": "json"
      }
    }
  ]
}
```

```JSON
// .prettierignore
node_modules

dist
public
```

# Require

> yarn add vite-plugin-require-transform -S -D

```TypeScript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react-swc'
import requireTransform from 'vite-plugin-require-transform'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    react(),
    requireTransform({
      fileRegex: /.js$|.jsx$/,
    }),
  ],
})
```

