**一个完整的函数执行是不可以中断的**

```TypeScript
btn.onclick = () => {
  let i = 0
  for (; i < 100000; i++) {
    let span = document.createElement('span')
    span.innerText = 1
    container.appendChild(span)
  }
}
```

通过将任务拆分，放到任务队列中，实现可中断。

```TypeScript
function performWorkUnit() {
  // 任务执行完毕后结束递归
  if (taskQueue.length === 0) {
    btn.innerText = '执行'
    return
  }

  requestIdleCallback(deadline => {
    let task;
    while ((task = taskQueue.pop()) && !deadline.didTimeout && deadline.timeRemaining() > 0) {
      task()
    }
    performWorkUnit()
  })
}
```

甚至这时候，我们可通过操作任务队列的方式，实现操作任务执行的顺序、插队任务等。



