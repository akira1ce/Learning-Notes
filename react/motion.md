# overview

## 动画允许的值

所谓动画，其实质是在起始两个状态下，某些**css数值**的变化，**当然也包括 css 变量**。

在 motion 中支持动画的数值类型有：

1. number
2. string
3. color
4. Complex strings containing multiple numbers and/or colors (like box-shadow).
5. display

支持的css属性：

1. 位置大小
   1. x, y
   2. top, left, right, bottom
   3. width, height
   4. minWidth, maxWidth, minHeight, maxHeight
2. 变换
   1. rotate（旋转，单位为度数 deg 或弧度 rad）
   2. scale, scaleX, scaleY
   3. skew, skewX, skewY
   4. translateX, translateY, translateZ
3. 透明度和颜色
   1. opacity
   2. backgroundColor
   3. borderColor
   4. color
   5. boxShadow
   6. textShadow
4. 边框和圆角
   1. borderRadius,borderWidth
5. 滤镜
   1. filter（支持 blur, brightness, contrast, grayscale, 等）
6. 其他
   1. zIndex
   2. clipPath
   3. perspective
7. SVG
   1. pathLength
   2. pathOffset
8. CSS 变量


> 对于 x, y, width, height, top, left, right and bottom 这类数值，可在不同数据类型之间变化，eg：「100% -> 100px」。

```jsx
<motion.div
  initial={{ x: "100%" }}
  animate={{ x: "calc(100vw - 50%)" }}
/>
```

## 变体

多种形态之间的变换，每一种形态可通过对象中的属性配置。

```jsx
const list = {
    visible: { opacity: 1 },
    hidden: { opacity: 0 },
  };

const item = {
  /* 函数配置-参数「index」需要通过custom传入 */
  visible: (index) => ({ opacity: 1, y: 0, transition: { delay: index * 0.3 } }),
  hidden: { opacity: 0, y: -100 },
};

<motion.ul initial="hidden" whileInView="visible" variants={list}>
  {[1, 2, 3].map((i) => (
    <motion.li key={i} custom={i} variants={item} className={baseCls} />
  ))}
</motion.ul>
```

# transition

动画过渡配置。

可配置在任何 `animation prop` 中。

1. duration - 持续时间
2. visualDuration - 视觉可持续时间「会覆盖 duration」
3. ease - 贝塞尔曲线
4. times - 多个关键帧动画时间
5. type - 
6. damping - 阻尼「相当于阻力，设为0将无限弹跳」
7. mass - 物体质量
8. stiffness - 弹簧的刚度
9. velocity - 弹簧的初始速度

```js
<motion.div
  animate={{ x: 100 }}
  transition={{ duration: 0.3, delay: 1, ease: "linear" }}
  whileHover={{
    scale: 1.1,
    transition: { 
      duration: 0.2,
      ease: ["easeIn", "easeOut"],
      times: [0, 0.3, 1],
      type: "spring",
      bounce: 0.25,
      visualDuration: 0.5,
      damping: 300,
    }
  }}
/>
```

# gestures

1. whileHover - 悬浮
2. whileTap - 点击
3. onHoverStart
4. onHoverEnd
5. onPan - 按下并平移
6. drag - 拖动
7. whileDrag
8. dragConstraints - 拖动边界
9. whileFocus - 焦点

**注意：事件存在传播**

# layout

通常的动画是无法作用在布局中的，包括 flex、grid 等，但是通过 layout 属性，在 motion 中可以实现完美的动画效果。

多个dom，可通过 layoutId 界定。

# components

1. motion
2. MotionConfig - 全局配置
3. AnimatePresence - dom 移除前的动画
4. LayoutGroup

# hooks

motion 中有两种 hooks，一种是普通的 react-hook，另一种是 motion-hook。

motion-hook 内部实现了响应式数据，其数据变化不会导致组件 re-render。

## motion-hooks

**motion-value-api**

[概述](https://motion.dev/docs/react-motion-value#getvelocity())

1. get
2. getVelocity - 返回最新的运动速度
3. set
4. jump - 值跳转
5. isAnimating
6. stop
7. on - 「change、animationStart、animationCancel、animationComplete」其返回一个函数用于取消订阅。
8. destroy

**hooks**

1. [useMotionValueEvent](https://motion.dev/docs/react-use-motion-value-event)
2. [useMotionTemplate](https://motion.dev/docs/react-use-motion-template)
3. [useScroll](https://motion.dev/docs/react-use-motion-template)
4. [useSpring](https://motion.dev/docs/react-use-spring)
5. [useTime](https://motion.dev/docs/react-use-time)
6. [useTransform](https://motion.dev/docs/react-use-transform)
7. [useVelocity](https://motion.dev/docs/react-use-velocity)

## hooks

1. [useAnimate](https://motion.dev/docs/react-use-animate)
2. [useAnimationFrame](https://motion.dev/docs/react-use-animation-frame)
3. [useDragControls](https://motion.dev/docs/react-use-drag-controls)
4. [useInview](https://motion.dev/docs/react-use-in-view)
5. [useReducedMotion](https://motion.dev/docs/react-use-reduced-motion)