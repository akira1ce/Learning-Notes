# 防抖函数实现原理

```TypeScript
function debounce( callback, delay = 300 ){
    let timer;
    return ( ...args ) => {
      clearTimeout( timer );
      timer = setTimeout( () => {
        callback( ...args );
      }, delay );
    }
}
```

问题：无法获取到函数的返回值。

原因：callback 在 settimeout 中执行，返回值自然是没办法返回给使用者的。

> 但实际上，一个函数在一段时间内不一定执行，也不应该拿到期待的返回值。

# 解决方案

将含有返回值的函数当做异步函数对待，通过 promise 承接返回值。

1. 回调函数中传递

    这种方式后续的操作都需要在回调中处理。

    ```TypeScript
    import { debounce } from "lodash-es";
    
    const test = (callback) => {
      console.log("test");
      
      let returnValue = 100
      callback && callback(returnValue);
      
      return returnValue
    };
    
    const debouncedTest = debounce(test, 100);
    
    console.log(
      "return",
      debouncedTest((value) => {
        console.log(value);
      })
    );
    ```

1. promise resolve

    ```TypeScript
    function debounce( callback, delay ) {
      let timer;
    
      return( ...args ) => {
        return new Promise( ( resolve, reject ) => {
          clearTimeout(timer);
          timer = setTimeout( () => {
              try {
                let output = callback(...args);
                resolve( output );
              } catch ( err ) {
                reject( err );
              }
          }, delay );
        })
    
      }
    }
    
    // demo
    const func =  val => {
      console.log(val)
      return val
    };
    
    const func1 = debounce(func, 1000);
    
    const a = () => {
      let t;
      t =  func1(1);
      t =  func1(2);
      t =  func1(3);
      console.log('akira', t.then(res => console.log('then', res)));
    };
    
    a();
    ```

