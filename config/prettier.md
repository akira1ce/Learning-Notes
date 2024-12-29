# .prettierrc

**不生效重启**

```JSON
{
  "singleQuote": true,
  "jsxSingleQuote": false,
  "trailingComma": "all",
  "printWidth": 100,
  "proseWrap": "never",
  // 需要 yarn add xxx
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
# .prettierignore

```JSON
node_modules

dist
public
```