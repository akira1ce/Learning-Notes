useEffect 导致竟态问题？

![alt text](assets/image.png)

竟态来源实际不是 useEffect 导致的，是来自**异步请求导致的竞态问题，**这是客户端一直存在的问题**。**

**而不应该把这个问题跟 useEffect 强关联在一起，让 useEffect 来背锅！**

解决方案：发生后禁用、防抖、取消上次请求

