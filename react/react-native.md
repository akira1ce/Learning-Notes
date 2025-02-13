# animated

> rotateX、rotateY 等变化时，需要加上 perspective 属性，否则在 Android 上将无法呈现。


```jsx
<Animated.View
  style={{
    transform: [
      {scale: this.state.scale},
      {rotateY: this.state.rotateY},
      {perspective: 1000}, // without this line this Animation will not render on Android while working fine on iOS
    ],
  }}
/>
```

## animated.event

```ts
type Mapping = {[key: string]: Mapping} | AnimatedValue;

interface EventConfig<T> {
  listener?: ((event: NativeSyntheticEvent<T>) => void) | undefined;
  /* 原生驱动 */
  useNativeDriver: boolean;
}

export function event<T>(
  /* 参数映射 */
  argMapping: Array<Mapping | null>,
  /* 配置 */
  config?: EventConfig<T>,
): (...args: any[]) => void;
```

参数映射实际上是将 event 事件中的参数，直接映射到 animated-value 上的，最终会调用 animated.setValue 方法，实现数据绑定。

## panResponder

[panresponder](https://reactnative.cn/docs/panresponder)

nativeEvent + gestureState 事件合成。

> 原生动画驱动「useNativeDriver」 在 panResponder 中暂时不可使用。
>
> 设为 true 会报错：onpanrespondermove is not a function.



# Components

1. [Pressable](https://reactnative.cn/docs/pressable) - 触摸事件
2. [Animated](https://reactnative.cn/docs/animated) - 动画

# libs

1. [react-native-reanimated](https://github.com/software-mansion/react-native-reanimated) - 动画
2. [nativewind](https://github.com/nativewind/nativewind) - tailwindcss
3. [react-native-gesture-handler](https://github.com/software-mansion/react-native-gesture-handler) - 手势