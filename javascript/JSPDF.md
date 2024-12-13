# p 标签导致 canvas 中元素错位问题

导出的PDF异常：

![alt text](assets/image-1.png)

页面上显示正常：

![alt text](assets/image-2.png)

原因：原生 p 标签的样式在转换为 canvas 的过程中样式有所丢失。（或行内元素都存在这类问题）

解决方案：不使用 p 标签，改为 div。

修改后的pdf：

![alt text](assets/image-3.png)