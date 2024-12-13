```JavaScript
import { delay } from 'dva/saga';

...
* effect({ payload }, { put, call, select, race }) {
  const { res, timeout } = yield race({
      res: call(xxxx),
      timeout: delay(60000)
  })
}
```

