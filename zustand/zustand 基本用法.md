一个小型、快速、可扩展的状态管理解决方案，基于简化的 flux 原则。它有一个友好的基于 hook 的 api，而不是那种样板式的或者固执己见的。

不要因为它的可爱而忽视了它。它有相当多的爪子，花了很多时间来处理常见的陷阱，比如可怕的[zombie child 问题](https://react-redux.js.org/api/hooks#stale-props-and-zombie-children)、[react concurrency](https://github.com/bvaughn/rfcs/blob/useMutableSource/text/0000-use-mutable-source.md)，以及混合渲染器之间的[上下文丢失](https://github.com/facebook/react/issues/13332)。它可能是 React 领域中唯一一个能解决所有这些问题的状态管理库。

你可以试一下这个[在线 demo](https://githubbox.com/pmndrs/zustand/tree/main/examples)。

```bash
npm install zustand # or yarn add zustand
```

## 首先创建一个 store

你的 store 是一个 hook！你可以在里面放任何东西：基本类型值、对象、函数。而`set`函数会*合并*状态。

```jsx
import create from "zustand";

const useStore = create((set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
  removeAllBears: () => set({ bears: 0 }),
}));
```

## 然后绑定你的组件，就是这么简单!

在任何地方使用这个 hook，不需要 provider。组件会在你选择的状态变化时重新渲染。

```jsx
function BearCounter() {
  const bears = useStore((state) => state.bears);
  return <h1>{bears} around here ...</h1>;
}

function Controls() {
  const increasePopulation = useStore((state) => state.increasePopulation);
  return <button onClick={increasePopulation}>one up</button>;
}
```

### 为什么是 zustand 而不是 redux？

- 简单而不固执己见
- 使得 hook 成为消费状态的主要手段
- 不需要把你的 app 包裹在 context provider 中
- 可以为组件提供瞬时状态(不引起渲染)

### 为什么是 zustand 而不是 context?

- 更少的样板代码
- 只在状态变化时渲染组件
- 集中的、基于 action 的状态管理

---

# 用法

## 获取所有状态

你可以这么做，但请记住，这将导致该组件在每一个状态变化时都要进行更新

```jsx
const state = useStore();
```

## 选择多个状态切片

默认情况下，它基于严格相等来检测变化(old === new)，这对原子状态的选择是有效的。

```jsx
const nuts = useStore((state) => state.nuts);
const honey = useStore((state) => state.honey);
```

如果你想构造一个内部有多个状态的对象，类似于 redux 的 mapStateToProps，你可以通过传递`shallow`比较函数来告诉 zustand 你想让这个对象被浅层 diff。

```jsx
import shallow from "zustand/shallow";

// 对象选取，当state.nuts或state.honey改变时，重新渲染组件。
const { nuts, honey } = useStore(
  (state) => ({ nuts: state.nuts, honey: state.honey }),
  shallow
);

// 数组选取，当state.nuts或state.honey改变时，重新渲染组件。
const [nuts, honey] = useStore((state) => [state.nuts, state.honey], shallow);

// 映射选取，当state.treats在顺序、数量或对象键上发生变化时，重新渲染组件
const treats = useStore((state) => Object.keys(state.treats), shallow);
```

为了对重新渲染进行更多控制，你可以提供自定义的比较函数。

```jsx
const treats = useStore(
  (state) => state.treats,
  (oldTreats, newTreats) => compare(oldTreats, newTreats)
);
```

## 记忆化选择器

通常建议用 useCallback 来记忆选择器。这将避免在每次渲染时进行不必要的计算。它也允许 React 在 concurrent 模式下优化性能。

```jsx
const fruit = useStore(useCallback((state) => state.fruits[id], [id]));
```

如果一个选择器不依赖于作用域，你可以在渲染函数(组件)之外定义它，以获得一个固定的引用，而无需使用 useCallback。

```jsx
const selector = state => state.berries

function Component() {
  const berries = useStore(selector)
```

## 覆盖状态

`set`函数有第二个参数，默认为`false`。它将替换状态而不是合并它们。注意不要覆盖了你依赖的部分，比如 action。

```jsx
import omit from "lodash-es/omit";

const useStore = create((set) => ({
  salmon: 1,
  tuna: 2,
  deleteEverything: () => set({}, true), // 清楚整个store，包括action
  deleteTuna: () => set((state) => omit(state, ["tuna"]), true),
}));
```

## 异步 action

当你准备好时，只需调用`set`，zustand 并不关心你的 action 是否是异步的。

```jsx
const useStore = create((set) => ({
  fishies: {},
  fetch: async (pond) => {
    const response = await fetch(pond);
    set({ fishies: await response.json() });
  },
}));
```

## 从 action 中读取状态

`set`允许函数式更新:`set(state => result)`，但你仍然可以通过`get`访问它之外的状态。

```jsx
const useStore = create((set, get) => ({
  sound: "grunt",
  action: () => {
    const sound = get().sound
    // ...
  }
})
```

## 读取/写入状态并对组件外的变化做出响应

有时你需要以非响应式的方式访问状态，或者对 store 进行操作。对于这些情况，返回的 hook 在其原型上附加了一些实用函数。

```jsx
const useStore = create(() => ({ paw: true, snout: true, fur: true }))

// 获得最新的且非响应式的状态
const paw = useStore.getState().paw
// 监听所有的变化，每次变化是将同步触发
const unsub1 = useStore.subscribe(console.log)
// 更新状态，将触发监听器
useStore.setState({ paw: false })
// 取消订阅
unsub1()
// 销毁store(删除所有订阅)。
useStore.destroy()

// 当然，你可以像往常一样使用hook
function Component() {
  const paw = useStore(state => state.paw)
```

### 使用带选择器的订阅者

如果你需要带选择器的订阅者，`subscribeWithSelector`中间件会有帮助。

通过这个中间件，`subscribe`接受一个额外参数。

```ts
subscribe(selector, callback, options?: { equalityFn, fireImmediately }): Unsubscribe
```

```js
import { subscribeWithSelector } from "zustand/middleware";
const useStore = create(
  subscribeWithSelector(() => ({ paw: true, snout: true, fur: true }))
);

// 监听选定的状态变化，在这个例子中是 "paw"
const unsub2 = useStore.subscribe((state) => state.paw, console.log);
// subscribe也会变化前的值
const unsub3 = useStore.subscribe(
  (state) => state.paw,
  (paw, previousPaw) => console.log(paw, previousPaw)
);
// subscribe还支持一个可选的比较函数
const unsub4 = useStore.subscribe(
  (state) => [state.paw, state.fur],
  console.log,
  { equalityFn: shallow }
);
// 订阅并立即触发
const unsub5 = useStore.subscribe((state) => state.paw, console.log, {
  fireImmediately: true,
});
```

> 如何在 TS 中让`subscribeWithSelector`带有类型

```ts
import create, { GetState, SetState } from "zustand";
import {
  StoreApiWithSubscribeWithSelector,
  subscribeWithSelector,
} from "zustand/middleware";

type BearState = {
  paw: boolean;
  snout: boolean;
  fur: boolean;
};
const useStore = create<
  BearState,
  SetState<BearState>,
  GetState<BearState>,
  StoreApiWithSubscribeWithSelector<BearState>
>(subscribeWithSelector(() => ({ paw: true, snout: true, fur: true })));
```

对于有多个中间件的更复杂的类型请参考[middlewareTypes.test.tsx](https://github.com/pmndrs/zustand/blob/main/tests/middlewareTypes.test.tsx)。

## 在没有 React 的情况下使用 zustand

Zustands 的核心可以在不依赖 React 的情况下被导入和使用。唯一的区别是，创建函数不返回 hook，而是返回一系列 api 函数。

```jsx
import create from 'zustand/vanilla'

const store = create(() => ({ ... }))
const { getState, setState, subscribe, destroy } = store
```

你甚至可以用 React 消费现有的 vanilla store。

```jsx
import create from "zustand";
import vanillaStore from "./vanillaStore";

const useStore = create(vanillaStore);
```

注意修改`set`或`get`的中间件不应用于`getState`和`setState`。

## 瞬时更新(对于经常发生的状态变化)。

subscribe 函数允许组件绑定到状态端，而不需要在变化时强制重新渲染。最好把它和 useEffect 结合起来，以便在卸载时自动取消订阅。当你允许它直接改变视图时，这可能会对性能产生[剧烈](https://codesandbox.io/s/peaceful-johnson-txtws)的影响。

```jsx
const useStore = create(set => ({ scratches: 0, ... }))

function Component() {
  // 获取初始状态
  const scratchRef = useRef(useStore.getState().scratches)
  // 挂载时连接到store，卸载时断开连接，在ref中捕捉状态变化
  useEffect(() => useStore.subscribe(
    state => (scratchRef.current = state.scratches)
  ), [])
```

## 厌倦了 reducers 和变更嵌套状态？使用 Immer!

Reducing 一个嵌套结构是很累人的。你试过[immer](https://github.com/mweststrate/immer)吗？

```jsx
import produce from "immer";

const useStore = create((set) => ({
  lush: { forest: { contains: { a: "bear" } } },
  clearForest: () =>
    set(
      produce((state) => {
        state.lush.forest.contains = null;
      })
    ),
}));

const clearForest = useStore((state) => state.clearForest);
clearForest();
```

## 中间件

你可以函数式地以任何你喜欢的方式组合你的 store。

```jsx
// 每次状态变化时打印输出
const log = (config) => (set, get, api) =>
  config(
    (args) => {
      console.log("  applying", args);
      set(args);
      console.log("  new state", get());
    },
    get,
    api
  );

// 对set方法使用immer代理
const immer = (config) => (set, get, api) =>
  config(
    (partial, replace) => {
      const nextState =
        typeof partial === "function" ? produce(partial) : partial;
      return set(nextState, replace);
    },
    get,
    api
  );

const useStore = create(
  log(
    immer((set) => ({
      bees: false,
      setBees: (input) => set((state) => (state.bees = input)),
    }))
  )
);
```

> 使用 pipe 连接中间件

```js
import create from "zustand";
import produce from "immer";
import pipe from "ramda/es/pipe";

/* 前面例子中的log和immer函数 */
/* 你可以随意连接多个中间件 */
const createStore = pipe(log, immer, create);

const useStore = createStore((set) => ({
  bears: 1,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
}));

export default useStore;
```

关于 TS 的例子，请看以下[讨论](https://github.com/pmndrs/zustand/discussions/224#discussioncomment-118208)

> 如何在 TS 中让 immer 中间件带有类型

在[middlewareTypes.test.tsx](https://github.com/pmndrs/zustand/blob/main/tests/middlewareTypes.test.tsx)中有一个实现和一些用例。

## 持久化中间件

你可以使用任何一种存储来持久化你 store 里的数据。

```jsx
import create from "zustand";
import { persist } from "zustand/middleware";

export const useStore = create(
  persist(
    (set, get) => ({
      fishes: 0,
      addAFish: () => set({ fishes: get().fishes + 1 }),
    }),
    {
      name: "food-storage", // 唯一键
      getStorage: () => sessionStorage, // (可选)默认使用'localStorage'
    }
  )
);
```

[该中间件的完整文档](https://github.com/pmndrs/zustand/wiki/Persisting-the-store's-data)

## 离不开类似 redux 的 reducers 和 action types？

```jsx
const types = { increase: "INCREASE", decrease: "DECREASE" };

const reducer = (state, { type, by = 1 }) => {
  switch (type) {
    case types.increase:
      return { grumpiness: state.grumpiness + by };
    case types.decrease:
      return { grumpiness: state.grumpiness - by };
  }
};

const useStore = create((set) => ({
  grumpiness: 0,
  dispatch: (args) => set((state) => reducer(state, args)),
}));

const dispatch = useStore((state) => state.dispatch);
dispatch({ type: types.increase, by: 2 });
```

或者使用我们的 redux 中间件。它连接你的主 reducer，设置初始状态，并为状态本身和 vanilla api 添加一个调度函数。试试[这个](https://codesandbox.io/s/amazing-kepler-swxol)例子。

```jsx
import { redux } from "zustand/middleware";

const useStore = create(redux(reducer, initialState));
```

## 在 React event handler 之外调用 action

因为如果在外部调用 event handler，React 会同步地`setState`。在 event handler 之外更新状态将迫使 React 同步更新组件，因此增加了遇到 zombie-child 效应的风险。
为了解决这个问题，这个 action 需要用`unstable_batchedUpdates`来包装。

```jsx
import { unstable_batchedUpdates } from "react-dom"; // or 'react-native'

const useStore = create((set) => ({
  fishes: 0,
  increaseFishes: () => set((prev) => ({ fishes: prev.fishes + 1 })),
}));

const nonReactCallback = () => {
  unstable_batchedUpdates(() => {
    useStore.getState().increaseFishes();
  });
};
```

更多细节见: <https://github.com/pmndrs/zustand/issues/302>

## Redux devtools

```jsx
import { devtools } from "zustand/middleware";

// 使用普通store，它将将action打印为"setState"
const useStore = create(devtools(store));
// 与redux store一起使用，它将打印完整的action types
const useStore = create(devtools(redux(reducer, initialState)));
```

devtools 把 store 函数作为它的第一个参数，你可以选择给 store 命名或用第二个参数配置[serialize](https://github.com/zalmoxisus/redux-devtools-extension/blob/master/docs/API/Arguments.md#serialize)选项。

命名 store: `devtools(store, {name: "MyStore"})`，这将在 devtools 中创建一个单独的名为 "MyStore" 的实例。

Serialize 选项: `devtools(store, { serialize: { options: true } })`。

#### 打印 action

devtools 只打印每个分离的存储空间的操作，与典型的*组合 reducers* redux store 不同。参见组合 store 的方法<https://github.com/pmndrs/zustand/issues/163>

你可以通过传递第三个参数来打印每个`set`函数的特定 action 类型。

```jsx
const createBearSlice = (set, get) => ({
  eatFish: () =>
    set(
      (prev) => ({ fishes: prev.fishes > 1 ? prev.fishes - 1 : 0 }),
      false,
      "bear/eatFish"
    ),
});
```

如果没有提供 action 类型，它将被默认为 "unknown"。你可以通过提供一个`anonymousActionType`参数来定制这个默认值。

```jsx
devtools(..., { anonymousActionType: 'unknown', ... })
```

## React context

用`create`创建的 store 不需要 context providers。在某些情况下，你可能想使用 context 来进行依赖注入，或者如果你想用组件的 props 来初始化你的 store。因为 store 是一个 hook，把它作为一个普通的 context 值传递可能会违反 hook 的规则。为了避免误用，我们提供了一个特殊的`createContext`。

```jsx
import create from 'zustand'
import createContext from 'zustand/context'

const { Provider, useStore } = createContext()

const createStore = () => create(...)

const App = () => (
  <Provider createStore={createStore}>
    ...
  </Provider>
)

const Component = () => {
  const state = useStore()
  const slice = useStore(selector)
  ...
}
```

> createContext 在实际组件中的应用

```jsx
import create from "zustand";
import createContext from "zustand/context";

// 最佳实践: 你可以把createContext()和createStore移到一个单独的文件(store.js)，然后在在你需要的地方导入Provider, useStore

const { Provider, useStore } = createContext();

const createStore = () =>
  create((set) => ({
    bears: 0,
    increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
    removeAllBears: () => set({ bears: 0 })
  }));

const Button = () => {
  return (
      {/** store() - 这将在每次使用Button组件的时候创建一个store，而不是所有组件使用一个store。 **/}
    <Provider createStore={createStore}>
      <ButtonChild />
    </Provider>
  );
};

const ButtonChild = () => {
  const state = useStore();
  return (
    <div>
      {state.bears}
      <button
        onClick={() => {
          state.increasePopulation();
        }}
      >
        +
      </button>
    </div>
  );
};

export default function App() {
  return (
    <div className="App">
      <Button />
      <Button />
    </div>
  );
}
```

> 使用 props 初始化 createContext(在 TS 中)

```tsx
import create from "zustand";
import createContext from "zustand/context";

type BearState = {
  bears: number;
  increase: () => void;
};

// 将类型传递给`createContext`，而不是传递给`create`。
const { Provider, useStore } = createContext<BearState>();

export default function App({ initialBears }: { initialBears: number }) {
  return (
    <Provider
      createStore={() =>
        create((set) => ({
          bears: initialBears,
          increase: () => set((state) => ({ bears: state.bears + 1 })),
        }))
      }
    >
      <Button />
    </Provider>
  );
}
```

## 为 store 提供类型和`combine`中间件

```tsx
// 你可以用 `type`
type BearState = {
  bears: number;
  increase: (by: number) => void;
};

// 或 `interface`
interface BearState {
  bears: number;
  increase: (by: number) => void;
}

// 它们都有效
const useStore = create<BearState>((set) => ({
  bears: 0,
  increase: (by) => set((state) => ({ bears: state.bears + by })),
}));
```

或者，使用`combine`，让 tsc 推断出类型。这将浅层合并两个状态。

```tsx
import { combine } from "zustand/middleware";

const useStore = create(
  combine({ bears: 0 }, (set) => ({
    increase: (by: number) => set((state) => ({ bears: state.bears + by })),
  }))
);
```

使用多个中间件的类型可能需要一些 TypeScript 知识。参考[middlewareTypes.test.tsx](https://github.com/pmndrs/zustand/blob/main/tests/middlewareTypes.test.tsx)中的一些实例。

## 最佳实践

- 你可能想知道如何组织你的代码以更好地维护: [将 store 分割成独立的片断](https://github.com/pmndrs/zustand/wiki/Splitting-the-store-into-separate-slices)。

- 对于这个库的推荐用法: [Flux 启发的实践](https://github.com/pmndrs/zustand/wiki/Flux-inspired-practice)。

## 测试

关于 Zustand 的测试信息，请访问专门的[Wiki 页面](https://github.com/pmndrs/zustand/wiki/Testing)。

## 第三方库

一些用户可能想要扩展 Zustand 的功能集，这可以通过社区制作的第三方库来完成。有关 Zustand 的第三方库的信息，请访问专门的[Wiki 页面](https://github.com/pmndrs/zustand/wiki/3rd-Party-Libraries)。

## 与其他库的比较

- [Zustand 和 valtio 的区别](https://github.com/pmndrs/zustand/wiki/Difference-between-zustand-and-valtio)
