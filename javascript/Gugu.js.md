```TypeScript
import { useState } from "react";

interface StoreItem {
  state: any;
  dispatch: Set<any>;
}

interface Store {
  [key: string]: StoreItem;
}

interface KeyMap {
  [key: string]: boolean;
}

const isInitStore: KeyMap = {};
const store: Store = {};

function _setter<T>(key: string, value: T) {
  store[key].state = value;
  store[key].dispatch.forEach((cb: any) => {
    cb(value);
  });
}

function subscribeStore<T>(key: string): [T, (state: T) => void] {
  const [state, setState] = useState<T>(store[key].state);

  if (!store[key].dispatch.has(setState)) {
    store[key].dispatch.add(setState);
  }

  return [state, useDispatch<T>(key)];
}

export function useDispatch<T>(key: string) {
  return (value: T) => _setter(key, value);
}

export function createStore<T>(key: string, state: T) {
  // 如果没有被初始化，则初始化一次
  if (!isInitStore[key]) {
    store[key] = { state, dispatch: new Set() };
    isInitStore[key] = true;
  }
  return () => subscribeStore<T>(key);
}
```

「用法」

```TypeScript
export interface Counter {
  a: number;
  b: string;
}

export const useCounter = createStore<Counter>("counter", {
  a: 1,
  b: "2",
});

const [counter, setCounter] = useCounter();
```

