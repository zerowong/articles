### 基本示例

Dialog由`Dialog`、`Dialog.Overlay`、`Dialog.Title`和`Dialog.Description`组件构成。

当Dialog的`open`属性为`true`时，Dialog的内容将被渲染。当用户循环浏览可关注的元素时，焦点将被移到Dialog内并停留在那里。滚动被锁定时，你的应用UI的其余部分将对屏幕阅读器隐藏，点击Dialog外或按Escape键将调用`onClose`函数并关闭Dialog。

```tsx
import { useState } from 'react'
import { Dialog } from '@headlessui/react'

function MyDialog() {
  let [isOpen, setIsOpen] = useState(true)

  return (
    <Dialog open={isOpen} onClose={() => setIsOpen(false)}>
      <Dialog.Overlay />

      <Dialog.Title>Deactivate account</Dialog.Title>
      <Dialog.Description>
        This will permanently deactivate your account
      </Dialog.Description>

      <p>
        Are you sure you want to deactivate your account? All of your data will
        be permanently removed. This action cannot be undone.
      </p>

      <button onClick={() => setIsOpen(false)}>Deactivate</button>
      <button onClick={() => setIsOpen(false)}>Cancel</button>
    </Dialog>
  )
}
```

### 显示或隐藏Dialog

Dialog没有对打开/关闭状态的自动管理。要显示和隐藏Dailog，请将React状态传递到`open`属性。当`open`为`true`时，Dialog将显示，当它为`false`时，Dialog将卸载。

`onClose`回调在打开的Dialog被关闭时触发，比如当用户在Dialog内容之外单击或按下Escape。可以使用此回调将`open`设置回`false`并关闭Dialog。

```tsx
import { useState } from 'react'
import { Dialog } from '@headlessui/react'

function MyDialog() {
  // 打开/关闭状态存在于Dialog之外并自己手动管理
  let [isOpen, setIsOpen] = useState(true)

  function handleDeactivate() {
    // ...
  }

  return (
    /*
      将`isOpen`传递给`open`属性，并当用户在Dialog外单击或按下Escape键时使用`onClose`将状态设置回`false`。
    */
    <Dialog open={isOpen} onClose={() => setIsOpen(false)}>
      <Dialog.Overlay />

      <Dialog.Title>Deactivate account</Dialog.Title>
      <Dialog.Description>
        This will permanently deactivate your account
      </Dialog.Description>

      <p>
        Are you sure you want to deactivate your account? All of your data will
        be permanently removed. This action cannot be undone.
      </p>

      {/*
        你可以通过将`isOpen`设置为`false`来渲染其它按钮以关闭Dialog。
      */}
      <button onClick={() => setIsOpen(false)}>Cancel</button>
      <button onClick={handleDeactivate}>Deactivate</button>
    </Dialog>
  )
}
```

### 在Dialog中管理焦点

出于可访问性的原因，你的Dialog应至少包含一个可聚焦元素。默认情况下，`Dialog`组件在渲染后将聚焦第一个可聚焦元素（按DOM顺序），按Tab键将循环浏览内容中的所有其它可聚焦元素。

只要Dialog被渲染，焦点就会被困在Dialog中，所以跳到结尾将再次开始循环回到开头。Dialog之外的所有其它应用元素将被标记为惰性，因此不可聚焦。

如果你希望在初始呈现Dialog时让第一个可聚焦元素以外的其它元素接收初始焦点，则可以使用`initialFocus`ref：

```tsx
import { useState, useRef } from 'react'
import { Dialog } from '@headlessui/react'

function MyDialog() {
  let [isOpen, setIsOpen] = useState(true)
  let completeButtonRef = useRef(null)

  function completeOrder() {
    // ...
  }

  return (
    /* 使用`initialFocus`将初始焦点强制到特定的ref */
    <Dialog
      initialFocus={completeButtonRef}
      open={isOpen}
      onClose={() => setIsOpen(false)}
    >
      <Dialog.Overlay />

      <Dialog.Title>Complete your order</Dialog.Title>

      <p>Your order is all ready!</p>

      <button onClick={() => setIsOpen(false)}>Cancel</button>
      <button ref={completeButtonRef} onClick={completeOrder}>
        Complete order
      </button>
    </Dialog>
  )
}
```

### 为overlay设定样式

通常，Dialog将呈现在透明的深色背景之上。你可以设置`Dialog.Overlay`组件的样式以实现此外观。

overlay组件接受像`style`和`className`这样的普通 React 属性，所以你可以使用任何你喜欢的技术来设置它的样式。确保将它放在DOM中Dialog的其余内容之前，以免遮挡内容的交互元素。

```tsx
import { useState } from 'react'
import { Dialog } from '@headlessui/react'

function Example() {
  let [isOpen, setIsOpen] = useState(true)

  return (
    <Dialog
      open={isOpen}
      onClose={() => setIsOpen(false)}
      className="fixed z-10 inset-0 overflow-y-auto"
    >
      {/* 使用overlay为你的Dialog设置一个暗淡的背景 */}
      <Dialog.Overlay className="fixed inset-0 bg-black opacity-30" />

      {/* ... */}
    </Dialog>
  )
}
```

### 为dialog设定样式

Dialog的内容没有什么特别之处——你可以使用任何你喜欢的HTML和CSS。典型的Dialog将有一个最大宽度并在屏幕中居中，如下例所示，但在较小屏幕上的全屏处理也很常见。

```tsx
import { useState } from 'react'
import { Dialog } from '@headlessui/react'

function Example() {
  let [isOpen, setIsOpen] = useState(true)

  return (
    <Dialog
      open={isOpen}
      onClose={() => setIsOpen(false)}
      className="fixed z-10 inset-0 overflow-y-auto"
    >
      <div className="flex items-center justify-center min-h-screen">
        <Dialog.Overlay className="fixed inset-0 bg-black opacity-30" />

        <div className="bg-white rounded max-w-sm mx-auto">
          <Dialog.Title>Complete your order</Dialog.Title>

          {/* ... */}
        </div>
      </div>
    </Dialog>
  )
}
```

### 使用Title和Description组件

出于可访问性的原因，在呈现标记和描述Dialog内容的内容时，你应该使用`Dialog.Title`和`Dialog.Description`组件。 它们将通过`aria-labelledby`和`aria-descriptionby`属性自动链接到根`Dialog`组件，并且它们的内容将对屏幕阅读器用户友好。

```tsx
import { useState } from 'react'
import { Dialog } from '@headlessui/react'

function MyDialog() {
  let [isOpen, setIsOpen] = useState(true)

  return (
    <Dialog open={isOpen} onClose={() => setIsOpen(false)}>
      <Dialog.Overlay />

      {/*
        在适当的时候使用 Title 和 Description 组件来提高自定义Dialog的可访问性。
      */}
      <Dialog.Title>Deactivate account</Dialog.Title>
      <Dialog.Description>
        This will permanently deactivate your account
      </Dialog.Description>

      {/* ... */}
    </Dialog>
  )
}
```

### 渲染到Portal

如果你之前曾经实现过 Dialog，那么你可能已经在 React 中遇到过 [Portal](https://react.docschina.org/docs/portals.html)。Portal允许你从 DOM 中的一个位置调用组件（例如在你的应用 UI 深处），但实际上完全渲染到 DOM 中的另一个位置。

由于 Dialogs 及其 overlay 占据整个页面，你通常希望将它们呈现为 React 应用根节点的兄弟节点。这样你就可以依靠自然的 DOM 排序来确保它们的内容呈现在你现有的应用 UI 之上。这还可以轻松地将滚动锁定应用于应用的其余部分，并确保你的 Dialog 的内容和overlay不受阻碍以接收焦点和单击事件。

由于这些可访问性问题，Headless UI 的`Dialog`组件实际上在背后使用了 Portal。通过这种方式，我们可以提供诸如无障碍事件处理和使应用的其余部分变得惰性等功能。因此，在使用我们的 Dialog 时，你无需自己使用 Portal！我们已经处理好了。

### 过渡

要在打开/关闭Dialog的时候应用动画，请使用 [Transition](https://headlessui.dev/react/transition) 组件。 你需要做的就是将`Dialog`放在`<Transition>`中，Dialog将根据`<Transition>`上的`show`属性的状态自动转换。

在Dialog中使用`<Transition>`时，你可以删除`open`属性，因为Dialog会自动从`<Transition>`读取`show`状态。

```tsx
import { useState } from 'react'
import { Dialog, Transition } from '@headlessui/react'

function MyDialog() {
  let [isOpen, setIsOpen] = useState(true)

  return (
    <Transition
      show={isOpen}
      enter="transition duration-100 ease-out"
      enterFrom="transform scale-95 opacity-0"
      enterTo="transform scale-100 opacity-100"
      leave="transition duration-75 ease-out"
      leaveFrom="transform scale-100 opacity-100"
      leaveTo="transform scale-95 opacity-0"
    >
      <Dialog onClose={() => setIsOpen(false)}>
        <Dialog.Overlay />
        <Dialog.Title>Deactivate account</Dialog.Title>
        {/* ... */}
      </Dialog>
    </Transition>
  )
}
```

要分别为Dialog的overlay和内容设置动画，请使用`Transition`和`Transition.Child`：

```tsx
import { useState, Fragment } from 'react'
import { Dialog, Transition } from '@headlessui/react'

function MyDialog() {
  let [isOpen, setIsOpen] = useState(true)

  return (
    // 在根元素上使用 `Transition` 组件
    <Transition show={isOpen} as={Fragment}>
      <Dialog onClose={() => setIsOpen(false)}>
        {/*
          使用一个 Transition.Child 将一个过渡应用到overlay...
        */}
        <Transition.Child
          as={Fragment}
          enter="ease-out duration-300"
          enterFrom="opacity-0"
          enterTo="opacity-100"
          leave="ease-in duration-200"
          leaveFrom="opacity-100"
          leaveTo="opacity-0"
        >
          <Dialog.Overlay />
        </Transition.Child>

        {/*
          ...用另一个 Transition.Child 对内容应用单独的过渡。
        */}
        <Transition.Child
          as={Fragment}
          enter="ease-out duration-300"
          enterFrom="opacity-0 scale-95"
          enterTo="opacity-100 scale-100"
          leave="ease-in duration-200"
          leaveFrom="opacity-100 scale-100"
          leaveTo="opacity-0 scale-95"
        >
          <Dialog.Title>Deactivate account</Dialog.Title>

          {/* ... */}
        </Transition.Child>
      </Dialog>
    </Transition>
  )
}
```

如果你想使用其它动画库（如 [Framer Motion](https://www.framer.com/motion/) 或 [React Spring](https://www.react-spring.io/）)为Dialog设置动画并需要更多控制，你可以使用`static`属性告诉 Headless UI 不要管理渲染本身，并使用其它工具手动控制它：

```tsx
import { useState } from 'react'
import { Dialog } from '@headlessui/react'
import { AnimatePresence, motion } from 'framer-motion'

function MyDialog() {
  let [isOpen, setIsOpen] = useState(true)

  return (
    // 使用 `Transition` 组件 + open属性添加过渡。
    <AnimatePresence>
      {open && (
        <Dialog
          static
          as={motion.div}
          open={isOpen}
          onClose={() => setIsOpen(false)}
        >
          <Dialog.Overlay />
          <Dialog.Title>Deactivate account</Dialog.Title>

          {/* ... */}
        </Dialog>
      )}
    </AnimatePresence>
  )
}
```

`open`属性仍然用于管理滚动锁定和焦点捕获，但只要存在`static`，无论`open`值如何，实际元素将始终呈现，这允许你在外部自己控制它。

### 访问性说明

#### 焦点管理

当 Dialog 的`open`属性为`true`时，Dialog 的内容将被渲染，焦点将在 Dialog 内移动并被困在那里。 根据 DOM 顺序的第一个可聚焦元素将获得焦点，尽管你可以使用`initialFocus`引用来控制哪个元素获得初始焦点。 在打开的 Dialog 上按 Tab 会循环显示所有可聚焦元素。

#### 鼠标交互

呈现Dialog时，单击`Dialog.Overlay`将关闭Dialog。

没有开箱即用的鼠标交互来打开Dialog，但通常你会将`<button />`元素与`onClick`事件处理连接起来，该事件处理将 Dialog 的`open`属性切换为`true`。

#### 键盘交互

| 指令            |                         描述 |
| :-------------- | ---------------------------: |
| `Esc`           |         关闭所有打开的Dialog |
| `Tab`           |   循环浏览打开的Dialog的内容 |
| `Shift` + `Tab` | 在打开的Dialog内容中向后循环 |

#### 其它

当Dialog打开时，滚动被锁定，并且应用程序 UI 的其余部分对屏幕阅读器隐藏。

自动管理所有相关的 ARIA 属性。

### [组件API](https://headlessui.dev/react/dialog#component-api)
