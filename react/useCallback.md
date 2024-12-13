缓存函数地址。

只有在不传参的回调，useCallback才会发挥作用。

自定义hooks中暴露出的函数可能需要useCallback包裹。



需要和memo配合使用

说白了还是要减少re-render。
re-render核心还是fiber的diff比对。



所以将组件拆分，并将组件数据保存在组件自身（不要留在上层），能有效减少fiber树的深度，进而减少父层对子集的影响。



不留在上层的原因是，diff比对的时候还是props是按照全等来比较的（===）

`{} === {}` 的结果都是false

React组件的re-render机制，需要同时保证state、props、context都不变，组件才不会re-render

