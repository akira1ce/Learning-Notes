Tree 数据超大时，需要通过 height 开启虚拟滚动，优化性能。

但这会导致横向滚动失效，某中场景会出现内容显示不全的问题。

官方也明确表示开启虚拟滚动后会导致无法横向滚动，但我们可以通过 css 强制开启横向滚动。

```less
.ant-tree-list-holder {
  /* 滚动槽 */
  &::-webkit-scrollbar {
    width: 8px !important;
    height: 8px !important;
  }
  &::-webkit-scrollbar-track {
    background: transparent;
    border-radius: 8px;
    -webkit-box-shadow: none;
  }
  /* 滚动条滑块 */
  &::-webkit-scrollbar-thumb {
    background: rgba(0, 0, 0, 0.5);
    border-radius: 4px;
    -webkit-box-shadow: none;
  }

  // 开启虚拟滚动时，适配横向滚动
  & > div {
    overflow: visible !important;
  }
}
```

> Reference
> 
> [issues](https://github.com/ant-design/ant-design/issues/31226)