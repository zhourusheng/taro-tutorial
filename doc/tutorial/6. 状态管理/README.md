# 6. 状态管理

在 Taro 应用开发中，状态管理是构建复杂应用的关键环节。良好的状态管理可以让数据流更加清晰，组件间通信更加便捷，也使得应用更容易维护和扩展。本章将详细介绍 Taro 中常用的几种状态管理方案及其最佳实践。

## Taro 中的状态管理方案

Taro 作为一个基于 React 的框架，天然支持多种状态管理方案。从简单到复杂，我们可以根据应用规模和复杂度选择合适的方案：

1. React 内置状态管理（useState, useReducer）
2. React Context API
3. Redux
4. Mobx
5. Zustand

每种方案都有其适用场景和优缺点，下面我们将逐一介绍。

## React 内置状态管理

对于简单组件或小型应用，React 内置的状态管理机制已经足够使用。

### useState Hook

`useState` 是 React 提供的最基本的状态管理 Hook，适用于管理简单的状态。

```jsx
import React, { useState } from 'react'
import { View, Text, Button } from '@tarojs/components'

function Counter() {
  // 声明一个名为 count 的状态变量，初始值为 0
  const [count, setCount] = useState(0)
  
  return (
    <View className='counter'>
      <Text>当前计数: {count}</Text>
      <Button onClick={() => setCount(count + 1)}>增加</Button>
      <Button onClick={() => setCount(count - 1)}>减少</Button>
      <Button onClick={() => setCount(0)}>重置</Button>
    </View>
  )
}

export default Counter
```

当状态逻辑较为复杂时，可以使用函数式更新：

```jsx
function ComplexCounter() {
  const [count, setCount] = useState(0)
  
  // 使用函数式更新，确保使用最新的状态值
  const increment = () => {
    setCount(prevCount => prevCount + 1)
  }
  
  // 延迟增加
  const delayedIncrement = () => {
    setTimeout(() => {
      setCount(prevCount => prevCount + 1)
    }, 1000)
  }
  
  return (
    <View className='complex-counter'>
      <Text>当前计数: {count}</Text>
      <Button onClick={increment}>增加</Button>
      <Button onClick={delayedIncrement}>延迟增加</Button>
    </View>
  )
}
```

### useReducer Hook

当组件状态逻辑较为复杂，或者状态之间有依赖关系时，可以使用 `useReducer` 来管理状态：

```jsx
import React, { useReducer } from 'react'
import { View, Text, Button } from '@tarojs/components'

// 定义 reducer 函数
function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 }
    case 'decrement':
      return { count: state.count - 1 }
    case 'reset':
      return { count: 0 }
    case 'set':
      return { count: action.payload }
    default:
      throw new Error(`未知的 action 类型: ${action.type}`)
  }
}

function ReducerCounter() {
  // 使用 useReducer 管理状态
  const [state, dispatch] = useReducer(counterReducer, { count: 0 })
  
  return (
    <View className='reducer-counter'>
      <Text>当前计数: {state.count}</Text>
      <Button onClick={() => dispatch({ type: 'increment' })}>增加</Button>
      <Button onClick={() => dispatch({ type: 'decrement' })}>减少</Button>
      <Button onClick={() => dispatch({ type: 'reset' })}>重置</Button>
      <Button onClick={() => dispatch({ type: 'set', payload: 100 })}>设为 100</Button>
    </View>
  )
}

export default ReducerCounter
```

`useReducer` 的优势在于可以将复杂的状态逻辑集中在一个函数中处理，使组件代码更加清晰。

## React Context 使用

当需要在组件树中共享状态时，可以使用 React 的 Context API。Context API 适用于中小型应用或者局部的状态共享。

### 创建 Context

首先，我们需要创建一个 Context：

```jsx
// contexts/ThemeContext.js
import React, { createContext, useState } from 'react'

// 创建 Context
export const ThemeContext = createContext({
  theme: 'light',
  toggleTheme: () => {}
})

// 创建 Provider 组件
export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light')
  
  const toggleTheme = () => {
    setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light')
  }
  
  // 提供给子组件的值
  const contextValue = {
    theme,
    toggleTheme
  }
  
  return (
    <ThemeContext.Provider value={contextValue}>
      {children}
    </ThemeContext.Provider>
  )
}
```

### 在应用中使用 Context

在应用的根组件中使用 Provider：

```jsx
// app.js
import React, { Component } from 'react'
import { ThemeProvider } from './contexts/ThemeContext'

class App extends Component {
  render() {
    return (
      <ThemeProvider>
        {this.props.children}
      </ThemeProvider>
    )
  }
}

export default App
```

在子组件中消费 Context：

```jsx
// pages/theme-demo/index.jsx
import React, { useContext } from 'react'
import { View, Text, Button } from '@tarojs/components'
import { ThemeContext } from '../../contexts/ThemeContext'

function ThemeDemo() {
  // 使用 useContext 获取 Context 值
  const { theme, toggleTheme } = useContext(ThemeContext)
  
  // 根据主题设置样式
  const styles = {
    container: {
      backgroundColor: theme === 'light' ? '#ffffff' : '#333333',
      padding: '20px'
    },
    text: {
      color: theme === 'light' ? '#333333' : '#ffffff'
    }
  }
  
  return (
    <View style={styles.container}>
      <Text style={styles.text}>当前主题: {theme}</Text>
      <Button onClick={toggleTheme}>切换主题</Button>
    </View>
  )
}

export default ThemeDemo
```

### 组合多个 Context

在复杂应用中，可能需要多个 Context 来管理不同的状态：

```jsx
// app.js
import React, { Component } from 'react'
import { ThemeProvider } from './contexts/ThemeContext'
import { UserProvider } from './contexts/UserContext'
import { CartProvider } from './contexts/CartContext'

class App extends Component {
  render() {
    return (
      <ThemeProvider>
        <UserProvider>
          <CartProvider>
            {this.props.children}
          </CartProvider>
        </UserProvider>
      </ThemeProvider>
    )
  }
}

export default App
```

为了避免 Context 嵌套过深，可以创建一个组合 Provider：

```jsx
// contexts/AppContext.js
import React from 'react'
import { ThemeProvider } from './ThemeContext'
import { UserProvider } from './UserContext'
import { CartProvider } from './CartContext'

export function AppProvider({ children }) {
  return (
    <ThemeProvider>
      <UserProvider>
        <CartProvider>
          {children}
        </CartProvider>
      </UserProvider>
    </ThemeProvider>
  )
}
```

然后在应用中使用：

```jsx
// app.js
import React, { Component } from 'react'
import { AppProvider } from './contexts/AppContext'

class App extends Component {
  render() {
    return (
      <AppProvider>
        {this.props.children}
      </AppProvider>
    )
  }
}

export default App
```

## Redux 集成与使用

对于大型应用或状态复杂的应用，Redux 是一个强大的状态管理解决方案。Taro 官方提供了 `@tarojs/redux` 和 `@tarojs/redux-h5` 来支持在 Taro 应用中使用 Redux。

### 安装 Redux 相关依赖

```bash
npm install redux @tarojs/redux @tarojs/redux-h5 redux-thunk redux-logger --save
```

### 创建 Redux Store

首先，我们需要创建 actions, reducers 和 store：

```jsx
// store/actionTypes.js
export const INCREMENT = 'INCREMENT'
export const DECREMENT = 'DECREMENT'
export const RESET = 'RESET'
```

```jsx
// store/actions/counter.js
import { INCREMENT, DECREMENT, RESET } from '../actionTypes'

export const increment = () => ({
  type: INCREMENT
})

export const decrement = () => ({
  type: DECREMENT
})

export const reset = () => ({
  type: RESET
})

// 异步 action
export const incrementAsync = () => {
  return dispatch => {
    setTimeout(() => {
      dispatch(increment())
    }, 1000)
  }
}
```

```jsx
// store/reducers/counter.js
import { INCREMENT, DECREMENT, RESET } from '../actionTypes'

const initialState = {
  count: 0
}

export default function counter(state = initialState, action) {
  switch (action.type) {
    case INCREMENT:
      return {
        ...state,
        count: state.count + 1
      }
    case DECREMENT:
      return {
        ...state,
        count: state.count - 1
      }
    case RESET:
      return {
        ...state,
        count: 0
      }
    default:
      return state
  }
}
```

```jsx
// store/reducers/index.js
import { combineReducers } from 'redux'
import counter from './counter'
import user from './user'

export default combineReducers({
  counter,
  user
})
```

```jsx
// store/index.js
import { createStore, applyMiddleware, compose } from 'redux'
import thunkMiddleware from 'redux-thunk'
import { createLogger } from 'redux-logger'
import rootReducer from './reducers'

const middlewares = [
  thunkMiddleware,
  createLogger()
]

// 开发环境下使用 Redux DevTools
const composeEnhancers =
  typeof window === 'object' &&
  window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ ?
    window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({}) : compose

const enhancer = composeEnhancers(
  applyMiddleware(...middlewares)
)

export default function configStore() {
  const store = createStore(rootReducer, enhancer)
  return store
}
```

### 在应用中使用 Redux

在应用入口文件中配置 Redux：

```jsx
// app.js
import React, { Component } from 'react'
import { Provider } from '@tarojs/redux'
import configStore from './store'

const store = configStore()

class App extends Component {
  render() {
    return (
      <Provider store={store}>
        {this.props.children}
      </Provider>
    )
  }
}

export default App
```

在页面组件中使用 Redux：

```jsx
// pages/redux-demo/index.jsx
import React, { Component } from 'react'
import { View, Text, Button } from '@tarojs/components'
import { connect } from '@tarojs/redux'
import { increment, decrement, reset, incrementAsync } from '../../store/actions/counter'

// 映射 Redux 状态到组件 props
const mapStateToProps = (state) => ({
  count: state.counter.count
})

// 映射 Redux actions 到组件 props
const mapDispatchToProps = (dispatch) => ({
  increment: () => dispatch(increment()),
  decrement: () => dispatch(decrement()),
  reset: () => dispatch(reset()),
  incrementAsync: () => dispatch(incrementAsync())
})

class ReduxDemo extends Component {
  render() {
    const { count, increment, decrement, reset, incrementAsync } = this.props
    
    return (
      <View className='redux-demo'>
        <Text>Redux 计数器: {count}</Text>
        <Button onClick={increment}>增加</Button>
        <Button onClick={decrement}>减少</Button>
        <Button onClick={reset}>重置</Button>
        <Button onClick={incrementAsync}>异步增加</Button>
      </View>
    )
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(ReduxDemo)
```

### 使用 Redux Hooks

在函数组件中，可以使用 Redux 提供的 Hooks API：

```jsx
// pages/redux-hooks-demo/index.jsx
import React from 'react'
import { View, Text, Button } from '@tarojs/components'
import { useSelector, useDispatch } from '@tarojs/redux'
import { increment, decrement, reset, incrementAsync } from '../../store/actions/counter'

function ReduxHooksDemo() {
  // 使用 useSelector 获取状态
  const count = useSelector(state => state.counter.count)
  
  // 使用 useDispatch 获取 dispatch 函数
  const dispatch = useDispatch()
  
  return (
    <View className='redux-hooks-demo'>
      <Text>Redux Hooks 计数器: {count}</Text>
      <Button onClick={() => dispatch(increment())}>增加</Button>
      <Button onClick={() => dispatch(decrement())}>减少</Button>
      <Button onClick={() => dispatch(reset())}>重置</Button>
      <Button onClick={() => dispatch(incrementAsync())}>异步增加</Button>
    </View>
  )
}

export default ReduxHooksDemo
```

## Mobx 集成与使用

Mobx 是另一个流行的状态管理库，相比 Redux，它更加简单和灵活。Taro 也支持在应用中使用 Mobx。

### 安装 Mobx 相关依赖

```bash
npm install mobx mobx-react --save
```

### 创建 Mobx Store

```jsx
// stores/counterStore.js
import { observable, action, makeObservable, computed } from 'mobx'

class CounterStore {
  constructor() {
    // 使用 makeObservable 来启用装饰器语法
    makeObservable(this)
  }
  
  @observable count = 0
  
  @action increment = () => {
    this.count++
  }
  
  @action decrement = () => {
    this.count--
  }
  
  @action reset = () => {
    this.count = 0
  }
  
  @action incrementAsync = () => {
    setTimeout(() => {
      this.increment()
    }, 1000)
  }
  
  @computed get doubleCount() {
    return this.count * 2
  }
}

export default new CounterStore()
```

```jsx
// stores/index.js
import counterStore from './counterStore'
import userStore from './userStore'

export default {
  counterStore,
  userStore
}
```

### 在应用中使用 Mobx

在应用入口文件中配置 Mobx：

```jsx
// app.js
import React, { Component } from 'react'
import { Provider } from 'mobx-react'
import stores from './stores'

class App extends Component {
  render() {
    return (
      <Provider {...stores}>
        {this.props.children}
      </Provider>
    )
  }
}

export default App
```

在页面组件中使用 Mobx：

```jsx
// pages/mobx-demo/index.jsx
import React, { Component } from 'react'
import { View, Text, Button } from '@tarojs/components'
import { inject, observer } from 'mobx-react'

// 使用 inject 注入 store，使用 observer 响应状态变化
@inject('counterStore')
@observer
class MobxDemo extends Component {
  render() {
    const { counterStore } = this.props
    
    return (
      <View className='mobx-demo'>
        <Text>Mobx 计数器: {counterStore.count}</Text>
        <Text>双倍计数: {counterStore.doubleCount}</Text>
        <Button onClick={counterStore.increment}>增加</Button>
        <Button onClick={counterStore.decrement}>减少</Button>
        <Button onClick={counterStore.reset}>重置</Button>
        <Button onClick={counterStore.incrementAsync}>异步增加</Button>
      </View>
    )
  }
}

export default MobxDemo
```

### 使用 Mobx Hooks

在函数组件中，可以使用 Mobx 提供的 Hooks API：

```jsx
// pages/mobx-hooks-demo/index.jsx
import React from 'react'
import { View, Text, Button } from '@tarojs/components'
import { useLocalObservable, Observer } from 'mobx-react'

function MobxHooksDemo() {
  // 使用 useLocalObservable 创建本地 observable 状态
  const counter = useLocalObservable(() => ({
    count: 0,
    get doubleCount() {
      return this.count * 2
    },
    increment() {
      this.count++
    },
    decrement() {
      this.count--
    },
    reset() {
      this.count = 0
    },
    incrementAsync() {
      setTimeout(() => {
        this.increment()
      }, 1000)
    }
  }))
  
  return (
    // 使用 Observer 组件包裹，响应状态变化
    <Observer>
      {() => (
        <View className='mobx-hooks-demo'>
          <Text>Mobx Hooks 计数器: {counter.count}</Text>
          <Text>双倍计数: {counter.doubleCount}</Text>
          <Button onClick={() => counter.increment()}>增加</Button>
          <Button onClick={() => counter.decrement()}>减少</Button>
          <Button onClick={() => counter.reset()}>重置</Button>
          <Button onClick={() => counter.incrementAsync()}>异步增加</Button>
        </View>
      )}
    </Observer>
  )
}

export default MobxHooksDemo
```

## Zustand 集成与使用

Zustand 是一个轻量级的状态管理库，它使用 hooks 的方式来管理状态，API 简单直观，是 Redux 和 Mobx 的良好替代品。

### 安装 Zustand

```bash
npm install zustand --save
```

### 创建 Zustand Store

```jsx
// stores/useCounterStore.js
import create from 'zustand'

// 创建 store
const useCounterStore = create(set => ({
  count: 0,
  
  // 定义 actions
  increment: () => set(state => ({ count: state.count + 1 })),
  decrement: () => set(state => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
  incrementAsync: () => {
    setTimeout(() => {
      set(state => ({ count: state.count + 1 }))
    }, 1000)
  }
}))

export default useCounterStore
```

### 在组件中使用 Zustand

```jsx
// pages/zustand-demo/index.jsx
import React from 'react'
import { View, Text, Button } from '@tarojs/components'
import useCounterStore from '../../stores/useCounterStore'

function ZustandDemo() {
  // 从 store 中获取状态和 actions
  const count = useCounterStore(state => state.count)
  const { increment, decrement, reset, incrementAsync } = useCounterStore()
  
  return (
    <View className='zustand-demo'>
      <Text>Zustand 计数器: {count}</Text>
      <Button onClick={increment}>增加</Button>
      <Button onClick={decrement}>减少</Button>
      <Button onClick={reset}>重置</Button>
      <Button onClick={incrementAsync}>异步增加</Button>
    </View>
  )
}

export default ZustandDemo
```

### Zustand 的高级用法

Zustand 支持中间件，可以用来实现持久化、日志等功能：

```jsx
// stores/usePersistStore.js
import create from 'zustand'
import { persist } from 'zustand/middleware'

// 使用 persist 中间件实现状态持久化
const usePersistStore = create(
  persist(
    (set) => ({
      user: null,
      
      login: (userData) => set({ user: userData }),
      logout: () => set({ user: null }),
      updateProfile: (profile) => set(state => ({
        user: { ...state.user, ...profile }
      }))
    }),
    {
      name: 'user-storage', // 存储的名称
      getStorage: () => localStorage, // 存储方式
    }
  )
)

export default usePersistStore
```

## 状态管理最佳实践

在 Taro 应用中使用状态管理时，有一些最佳实践可以帮助你构建更加可维护和高效的应用。

### 选择合适的状态管理方案

根据应用的规模和复杂度选择合适的状态管理方案：

1. **小型应用或简单组件**：使用 React 内置的 useState 和 useReducer
2. **中型应用或局部状态共享**：使用 React Context API
3. **大型应用或复杂状态逻辑**：使用 Redux、Mobx 或 Zustand

### 状态分层管理

将应用状态分为不同的层次：

1. **本地组件状态**：只在单个组件内使用的状态，使用 useState 管理
2. **共享状态**：在多个组件间共享的状态，使用 Context API 或全局状态管理库
3. **应用全局状态**：整个应用都需要的状态，如用户信息、主题设置等，使用全局状态管理库

### 避免状态冗余

避免在多个地方存储相同的状态，这可能导致状态不一致：

```jsx
// ❌ 错误示例：状态冗余
const UserProfile = () => {
  const [user, setUser] = useState(null)
  const globalUser = useSelector(state => state.user)
  
  // 这里可能导致 user 和 globalUser 不一致
}

// ✅ 正确示例：使用单一数据源
const UserProfile = () => {
  const user = useSelector(state => state.user)
  
  // 所有组件都使用同一个用户状态
}
```

### 合理拆分状态

将复杂状态拆分为更小的、可管理的部分：

```jsx
// ❌ 错误示例：过大的状态对象
const initialState = {
  user: { /* 用户信息 */ },
  products: [ /* 产品列表 */ ],
  cart: [ /* 购物车 */ ],
  orders: [ /* 订单 */ ],
  ui: { /* UI 状态 */ }
}

// ✅ 正确示例：拆分状态
const userReducer = (state = initialUserState, action) => { /* ... */ }
const productsReducer = (state = initialProductsState, action) => { /* ... */ }
const cartReducer = (state = initialCartState, action) => { /* ... */ }
const ordersReducer = (state = initialOrdersState, action) => { /* ... */ }
const uiReducer = (state = initialUiState, action) => { /* ... */ }

// 组合 reducers
const rootReducer = combineReducers({
  user: userReducer,
  products: productsReducer,
  cart: cartReducer,
  orders: ordersReducer,
  ui: uiReducer
})
```

### 使用不可变数据

在更新状态时，始终创建新的对象或数组，而不是修改现有的：

```jsx
// ❌ 错误示例：直接修改状态
const addToCart = (product) => {
  const cart = useSelector(state => state.cart)
  cart.push(product) // 直接修改了原始数组
  dispatch(updateCart(cart))
}

// ✅ 正确示例：使用不可变更新
const addToCart = (product) => {
  const cart = useSelector(state => state.cart)
  const newCart = [...cart, product] // 创建新数组
  dispatch(updateCart(newCart))
}
```

### 处理异步操作

在状态管理中处理异步操作时，使用适当的中间件或工具：

1. **Redux**：使用 redux-thunk, redux-saga 或 redux-observable
2. **Mobx**：使用 async/await 或 flow
3. **Zustand**：直接在 store 中使用 async 函数

```jsx
// Redux Thunk 示例
const fetchUserAsync = (userId) => {
  return async (dispatch) => {
    dispatch(fetchUserStart())
    
    try {
      const response = await api.getUser(userId)
      dispatch(fetchUserSuccess(response.data))
    } catch (error) {
      dispatch(fetchUserFailure(error.message))
    }
  }
}

// Zustand 异步示例
const useUserStore = create((set) => ({
  user: null,
  loading: false,
  error: null,
  
  fetchUser: async (userId) => {
    set({ loading: true })
    
    try {
      const response = await api.getUser(userId)
      set({ user: response.data, loading: false, error: null })
    } catch (error) {
      set({ loading: false, error: error.message })
    }
  }
}))
```

### 性能优化

在使用状态管理时，注意性能优化：

1. **选择性订阅**：只订阅组件需要的状态部分
2. **避免不必要的渲染**：使用 memo, useMemo, useCallback 等
3. **批量更新**：将多个状态更新合并为一个

```jsx
// ❌ 错误示例：订阅整个状态树
const UserProfile = () => {
  const state = useSelector(state => state) // 订阅了整个状态树
  
  return <View>{state.user.name}</View>
}

// ✅ 正确示例：选择性订阅
const UserProfile = () => {
  const userName = useSelector(state => state.user.name) // 只订阅用户名
  
  return <View>{userName}</View>
}
```

### 调试工具

使用适当的调试工具来帮助开发和排错：

1. **Redux DevTools**：用于 Redux 调试
2. **MobX DevTools**：用于 Mobx 调试
3. **React DevTools**：用于 React 组件和 Context 调试

## 总结

本章详细介绍了 Taro 中常用的几种状态管理方案，包括 React 内置状态管理、React Context API、Redux、Mobx 和 Zustand。每种方案都有其适用场景和优缺点，开发者可以根据应用的规模和复杂度选择合适的方案。

关键要点：

1. **小型应用或简单组件**使用 React 内置的状态管理机制
2. **中型应用或局部状态共享**可以考虑使用 React Context API
3. **大型应用或复杂状态逻辑**推荐使用 Redux、Mobx 或 Zustand
4. 遵循状态管理的最佳实践，如状态分层、避免冗余、合理拆分等
5. 注意性能优化和调试工具的使用

通过合理使用状态管理，可以让 Taro 应用的数据流更加清晰，组件间通信更加便捷，也使得应用更容易维护和扩展。 