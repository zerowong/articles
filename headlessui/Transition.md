## 过渡

Transition 组件允许你向有条件渲染的元素添加进入/离开过渡，使用 CSS 类来控制不同过渡阶段的实际过渡样式。

### 基本示例

`Transition` 接受一个 `show` 属性，用于控制是否应该显示或隐藏子元素，以及一组生命周期属性（如 `enterFrom` 和 `leaveTo`），可让你在过渡的特定阶段添加 CSS 类。

```tsx
import { Transition } from '@headlessui/react'
import { useState } from 'react'

function MyComponent() {
  const [isShowing, setIsShowing] = useState(false)

  return (
    <>
      <button onClick={() => setIsShowing((isShowing) => !isShowing)}>
        Toggle
      </button>
      <Transition
        show={isShowing}
        enter="transition-opacity duration-75"
        enterFrom="opacity-0"
        enterTo="opacity-100"
        leave="transition-opacity duration-150"
        leaveFrom="opacity-100"
        leaveTo="opacity-0"
      >
        I will fade in and out
      </Transition>
    </>
  )
}
```

### 显示和隐藏内容

将应该有条件渲染的内容包裹在一个 `<Transition>`组件中，并使用 `show` 属性来控制内容是可见还是隐藏。

```tsx
import { Transition } from '@headlessui/react'
import { useState } from 'react'

function MyComponent() {
  const [isShowing, setIsShowing] = useState(false)

  return (
    <>
      <button onClick={() => setIsShowing((isShowing) => !isShowing)}>
        Toggle
      </button>
      <Transition show={isShowing}>I will appear and disappear.</Transition>
    </>
  )
}
```

默认情况下，`Transition` 组件将呈现一个 `div`，但如果需要，你可以使用 `as` 属性来呈现不同的元素。 任何其他 HTML 属性（如 `className`）都可以像添加到常规元素一样直接添加到 `Transition` 中。

```tsx
import { Transition } from '@headlessui/react'
import { useState } from 'react'

function MyComponent() {
  const [isShowing, setIsShowing] = useState(false)

  return (
    <>
      <button onClick={() => setIsShowing((isShowing) => !isShowing)}>
        Toggle
      </button>
      <Transition show={isShowing} as="a" href="/my-url" className="font-bold">
        I will appear and disappear.
      </Transition>
    </>
  )
}
```

### 过渡动画

默认情况下，一个 `Transition` 组件的进入和离开是立即的，如果你正在使用此组件，这可能不是你想要的。

要为你的进入/离开过渡设置动画，请在以下属性上添加为过渡的每个阶段提供样式的 CSS 类：

- `enter`：在元素进入的整个时间应用。 通常，你可以在此处定义持续时间以及要过渡的属性，例如 `transition-opacity duration-75`。
- `enterFrom`：进入开始阶段，例如，某些内容应该淡入，则设置 `opacity-0`。
- `enterTo`：进入的结束阶段，例如淡入后设置 `opacity-100`。
- `leave`：在元素离开的整个时间应用。 通常，你可以在此处定义持续时间以及要过渡的属性，例如 `transition-opacity duration-75`。
- `leaveFrom`：离开的开始阶段，例如，如果某些东西应该淡出，设置 `opacity-100`。
- `leaveTo`：离开的结束阶段，例如淡出后的 `opacity-0`。

这有个例子：

```tsx
import { Transition } from '@headlessui/react'
import { useState } from 'react'

function MyComponent() {
  const [isShowing, setIsShowing] = useState(false)

  return (
    <>
      <button onClick={() => setIsShowing((isShowing) => !isShowing)}>
        Toggle
      </button>
      <Transition
        show={isShowing}
        enter="transition-opacity duration-75"
        enterFrom="opacity-0"
        enterTo="opacity-100"
        leave="transition-opacity duration-150"
        leaveFrom="opacity-100"
        leaveTo="opacity-0"
      >
        I will fade in and out
      </Transition>
    </>
  )
}
```

在这个例子中，过渡元素将需要 75ms 才能进入（即 `duration-75` 类），并将在此期间过渡不透明度属性（即`transition-opacity`）。

它将在进入之前完全透明（即 `enterFrom` 阶段中的 `opacity-0`），并在完成时淡入到完全不透明（`opacity-100`）（即 `enterTo` 阶段）。

当元素被移除时（离开阶段），它会过渡 `opacity` 属性，并持续 150ms（`transition-opacity duration-150`）。

它将以完全不透明的形式开始（ `leaveFrom` 阶段中的 `opacity-100`），并以完全透明的形式结束（ `leaveTo` 阶段中的 `opacity-0` ）。

所有这些属性都是可选的，并且默认为空字符串。

### 协调多个过渡

有时你需要使用不同的动画过渡多个元素，但所有元素都基于相同的状态。 例如，假设用户单击一个按钮以打开在屏幕上滑动的侧边栏，同时你还需要淡入背景叠加层。

你可以通过使用父 `Transition` 组件包装相关元素来完成此操作，并使用 `Transition.Child` 组件包装每个需要自己的过渡样式的子组件，该组件将自动与父 `Transition` 通信并继承父级的 `show` 状态。

```tsx
import { Transition } from '@headlessui/react'

function Sidebar({ isShowing }) {
  return (
    /* 这个 `show` 属性控制所有嵌套的 `Transition.Child` 组件。 */
    <Transition show={isShowing}>
      {/* 背景叠加层 */}
      <Transition.Child
        enter="transition-opacity ease-linear duration-300"
        enterFrom="opacity-0"
        enterTo="opacity-100"
        leave="transition-opacity ease-linear duration-300"
        leaveFrom="opacity-100"
        leaveTo="opacity-0"
      >
        {/* ... */}
      </Transition.Child>

      {/* 滑动侧边栏 */}
      <Transition.Child
        enter="transition ease-in-out duration-300 transform"
        enterFrom="-translate-x-full"
        enterTo="translate-x-0"
        leave="transition ease-in-out duration-300 transform"
        leaveFrom="translate-x-0"
        leaveTo="-translate-x-full"
      >
        {/* ... */}
      </Transition.Child>
    </Transition>
  )
}
```

`Transition.Child` 组件具有与 `Transition` 组件完全相同的 API，但没有 `show` 属性，因为 `show` 的值由父级控制。

父级 `Transition` 组件在卸载之前将始终自动等待所有子组件完成过渡，因此你无需自己管理任何时间。

### 在初始挂载时过渡

如果你希望元素在第一次渲染时进行过渡，请将 `appear` 属性设置为 `true`。

如果你希望在初始页面加载时或在有条件地呈现其父级时过渡某些内容，这将非常有用。

```tsx
import { Transition } from '@headlessui/react'

function MyComponent({ isShowing }) {
  return (
    <Transition
      appear={true}
      show={isShowing}
      enter="transition-opacity duration-75"
      enterFrom="opacity-0"
      enterTo="opacity-100"
      leave="transition-opacity duration-150"
      leaveFrom="opacity-100"
      leaveTo="opacity-0"
    >
      {/* 内容 */}
    </Transition>
  )
}
```

### [组件API](https://headlessui.dev/react/transition#component-api)
