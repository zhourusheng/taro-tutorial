# 4. 全局生命周期

在 Taro 应用中，生命周期函数是理解和控制应用运行过程的关键。Taro 提供了三个层级的生命周期：应用级别、页面级别和组件级别。本章将详细介绍这些生命周期函数及其最佳实践。

## 应用级别生命周期函数

应用级别的生命周期函数定义在 `app.js` 文件中，用于控制整个应用的运行过程。

### 应用生命周期函数列表

| 生命周期方法 | 说明 | 触发时机 |
|------------|------|---------|
| onLaunch | 应用初始化 | 应用第一次启动 |
| onShow | 应用显示 | 应用启动或从后台切换到前台 |
| onHide | 应用隐藏 | 应用从前台切换到后台 |
| onError | 错误监听 | 应用发生 JS 错误 |
| onPageNotFound | 页面不存在监听 | 打开的页面不存在 |
| onUnhandledRejection | 未处理的 Promise 拒绝 | 当 Promise 被 reject 且没有被捕获 |
| onThemeChange | 主题变化 | 系统主题变化时 |

### 应用生命周期示例

```jsx
import React, { Component } from 'react'
import './app.scss'

class App extends Component {
  // 应用初始化时触发，全局只触发一次
  onLaunch(options) {
    console.log('App onLaunch', options)
    
    // 可以获取启动参数
    const { path, query, scene } = options
    
    // 执行初始化逻辑，如检查更新、初始化存储等
    this.checkUpdate()
    this.initStorage()
  }
  
  // 应用显示/切前台时触发
  onShow(options) {
    console.log('App onShow', options)
    
    // 可以在这里恢复应用状态、重新请求数据等
    this.resumeAudio()
    this.refreshData()
  }
  
  // 应用隐藏/切后台时触发
  onHide() {
    console.log('App onHide')
    
    // 可以在这里保存应用状态、暂停操作等
    this.saveAppState()
    this.pauseAudio()
  }
  
  // 应用发生错误时触发
  onError(error) {
    console.error('App onError', error)
    
    // 可以在这里进行错误上报
    this.reportError(error)
  }
  
  // 打开的页面不存在时触发
  onPageNotFound(res) {
    console.error('App onPageNotFound', res)
    
    // 可以在这里进行重定向
    Taro.redirectTo({
      url: '/pages/404/index'
    })
  }
  
  // 自定义方法示例
  checkUpdate() {
    // 检查小程序更新
    if (Taro.canIUse('getUpdateManager')) {
      const updateManager = Taro.getUpdateManager()
      updateManager.onCheckForUpdate(res => {
        if (res.hasUpdate) {
          console.log('有新版本')
        }
      })
      
      updateManager.onUpdateReady(() => {
        Taro.showModal({
          title: '更新提示',
          content: '新版本已准备好，是否重启应用？',
          success: res => {
            if (res.confirm) {
              updateManager.applyUpdate()
            }
          }
        })
      })
    }
  }
  
  // 渲染方法，必须实现
  render() {
    // App 组件不需要实际的 UI 渲染，只需返回 this.props.children
    return this.props.children
  }
}

export default App
```

### 应用配置文件

除了生命周期函数，应用级别的配置在 `app.config.js` 中定义：

```js
// app.config.js
export default {
  pages: [
    'pages/index/index',
    'pages/user/index',
    'pages/detail/index'
  ],
  window: {
    backgroundTextStyle: 'light',
    navigationBarBackgroundColor: '#fff',
    navigationBarTitleText: 'Taro 应用',
    navigationBarTextStyle: 'black'
  },
  tabBar: {
    color: '#999',
    selectedColor: '#1296db',
    backgroundColor: '#fff',
    list: [
      {
        pagePath: 'pages/index/index',
        text: '首页',
        iconPath: './assets/home.png',
        selectedIconPath: './assets/home_selected.png'
      },
      {
        pagePath: 'pages/user/index',
        text: '我的',
        iconPath: './assets/user.png',
        selectedIconPath: './assets/user_selected.png'
      }
    ]
  }
}
```

## 页面级别生命周期函数

页面级别的生命周期函数定义在页面组件中，用于控制页面的加载、显示、隐藏等过程。

### 页面生命周期函数列表

| 生命周期方法 | 说明 | 触发时机 |
|------------|------|---------|
| onLoad | 页面加载 | 页面首次加载 |
| onShow | 页面显示 | 页面显示/切入前台 |
| onReady | 页面初次渲染完成 | 页面首次渲染完成 |
| onHide | 页面隐藏 | 页面隐藏/切入后台 |
| onUnload | 页面卸载 | 页面卸载/销毁 |
| onPullDownRefresh | 下拉刷新 | 用户下拉刷新页面 |
| onReachBottom | 上拉触底 | 用户上拉触底 |
| onShareAppMessage | 用户点击分享 | 用户点击页面分享按钮 |
| onPageScroll | 页面滚动 | 页面滚动时触发 |
| onTabItemTap | 点击 Tab 项 | 用户点击 Tab 时触发 |
| onResize | 页面尺寸变化 | 页面尺寸发生变化时 |
| onAddToFavorites | 收藏 | 用户点击收藏时 |

### 页面生命周期示例

```jsx
import React, { Component } from 'react'
import { View, Text, Button } from '@tarojs/components'
import Taro from '@tarojs/taro'
import './index.scss'

export default class IndexPage extends Component {
  constructor(props) {
    super(props)
    this.state = {
      data: [],
      loading: true
    }
  }
  
  // 页面首次加载时触发，只会触发一次
  onLoad(options) {
    console.log('Page onLoad', options)
    
    // 获取路由参数
    const { id } = options
    
    // 根据参数加载数据
    this.loadData(id)
  }
  
  // 页面显示/切入前台时触发
  onShow() {
    console.log('Page onShow')
    
    // 可以在这里刷新数据
    this.refreshData()
    
    // 记录页面访问
    this.recordPageView()
  }
  
  // 页面初次渲染完成时触发，只会触发一次
  onReady() {
    console.log('Page onReady')
    
    // 可以在这里操作 DOM，如获取元素高度等
    this.initPageLayout()
  }
  
  // 页面隐藏/切入后台时触发
  onHide() {
    console.log('Page onHide')
    
    // 可以在这里暂停一些操作
    this.pauseVideo()
  }
  
  // 页面卸载/销毁时触发
  onUnload() {
    console.log('Page onUnload')
    
    // 可以在这里清理资源
    this.clearTimer()
    this.savePageState()
  }
  
  // 用户下拉刷新时触发
  onPullDownRefresh() {
    console.log('Page onPullDownRefresh')
    
    // 刷新数据
    this.setState({ loading: true })
    this.loadData().then(() => {
      // 停止下拉刷新动画
      Taro.stopPullDownRefresh()
    })
  }
  
  // 用户上拉触底时触发
  onReachBottom() {
    console.log('Page onReachBottom')
    
    // 加载更多数据
    if (!this.state.loading) {
      this.loadMoreData()
    }
  }
  
  // 用户点击分享按钮时触发
  onShareAppMessage(res) {
    console.log('Page onShareAppMessage', res)
    
    // 自定义分享内容
    return {
      title: '分享标题',
      path: '/pages/index/index?id=123',
      imageUrl: 'https://example.com/share-image.png'
    }
  }
  
  // 页面滚动时触发
  onPageScroll(e) {
    // 获取滚动距离
    const { scrollTop } = e
    
    // 根据滚动距离控制元素显示/隐藏
    if (scrollTop > 100 && !this.state.showBackTop) {
      this.setState({ showBackTop: true })
    } else if (scrollTop <= 100 && this.state.showBackTop) {
      this.setState({ showBackTop: false })
    }
  }
  
  // 点击 Tab 项时触发
  onTabItemTap(item) {
    console.log('Page onTabItemTap', item)
    
    // item 包含 index, pagePath, text 属性
    const { index, pagePath, text } = item
    
    // 可以根据 Tab 信息执行特定操作
    if (index === 0) {
      this.refreshHomeData()
    }
  }
  
  // 自定义方法：加载数据
  loadData = async (id) => {
    try {
      const response = await Taro.request({
        url: 'https://api.example.com/data',
        data: { id }
      })
      
      this.setState({
        data: response.data,
        loading: false
      })
    } catch (error) {
      console.error('加载数据失败', error)
      this.setState({ loading: false })
      
      Taro.showToast({
        title: '加载失败',
        icon: 'none'
      })
    }
  }
  
  // 渲染页面
  render() {
    const { data, loading, showBackTop } = this.state
    
    return (
      <View className='index-page'>
        <Text className='title'>页面生命周期示例</Text>
        
        {loading ? (
          <View className='loading'>加载中...</View>
        ) : (
          <View className='data-list'>
            {data.map(item => (
              <View key={item.id} className='data-item'>
                {item.name}
              </View>
            ))}
          </View>
        )}
        
        {showBackTop && (
          <Button 
            className='back-top' 
            onClick={() => Taro.pageScrollTo({ scrollTop: 0 })}
          >
            返回顶部
          </Button>
        )}
      </View>
    )
  }
}
```

### 页面配置文件

每个页面可以有自己的配置文件，例如 `index.config.js`：

```js
// index.config.js
export default {
  navigationBarTitleText: '首页',
  enablePullDownRefresh: true,
  backgroundColor: '#f5f5f5',
  onReachBottomDistance: 50
}
```

## 组件生命周期与 React 对比

Taro 组件的生命周期基于 React 组件生命周期，但由于多端兼容性考虑，有一些差异和限制。

### React 组件生命周期

React 组件生命周期可以分为三个阶段：

1. **挂载阶段**：组件被创建并插入到 DOM 中
   - constructor()
   - static getDerivedStateFromProps()
   - render()
   - componentDidMount()

2. **更新阶段**：组件重新渲染，可由 props 或 state 变化触发
   - static getDerivedStateFromProps()
   - shouldComponentUpdate()
   - render()
   - getSnapshotBeforeUpdate()
   - componentDidUpdate()

3. **卸载阶段**：组件从 DOM 中移除
   - componentWillUnmount()

### Taro 组件生命周期

Taro 支持 React 的标准生命周期，但在小程序环境中可能存在一些差异：

```jsx
import React, { Component } from 'react'
import { View, Text } from '@tarojs/components'
import './style.scss'

export default class MyComponent extends Component {
  constructor(props) {
    super(props)
    console.log('constructor')
    
    this.state = {
      count: 0
    }
  }
  
  // 挂载阶段
  static getDerivedStateFromProps(props, state) {
    console.log('getDerivedStateFromProps', props, state)
    // 根据 props 更新 state
    if (props.initialCount !== undefined && state.count === 0) {
      return { count: props.initialCount }
    }
    return null
  }
  
  componentDidMount() {
    console.log('componentDidMount')
    // 组件挂载后执行，如发起网络请求、添加事件监听等
    this.timer = setInterval(() => {
      console.log('timer tick')
    }, 1000)
  }
  
  // 更新阶段
  shouldComponentUpdate(nextProps, nextState) {
    console.log('shouldComponentUpdate', nextProps, nextState)
    // 控制组件是否重新渲染
    return nextState.count !== this.state.count
  }
  
  getSnapshotBeforeUpdate(prevProps, prevState) {
    console.log('getSnapshotBeforeUpdate', prevProps, prevState)
    // 在 DOM 更新前获取一些信息
    return { scrollPosition: 100 }
  }
  
  componentDidUpdate(prevProps, prevState, snapshot) {
    console.log('componentDidUpdate', prevProps, prevState, snapshot)
    // 组件更新后执行，可以操作 DOM
    if (this.state.count !== prevState.count) {
      console.log('count changed to', this.state.count)
    }
  }
  
  // 卸载阶段
  componentWillUnmount() {
    console.log('componentWillUnmount')
    // 组件卸载前执行，清理资源
    clearInterval(this.timer)
  }
  
  // 错误处理
  static getDerivedStateFromError(error) {
    console.log('getDerivedStateFromError', error)
    // 渲染备用 UI
    return { hasError: true }
  }
  
  componentDidCatch(error, info) {
    console.log('componentDidCatch', error, info)
    // 记录错误信息
  }
  
  // 自定义方法
  handleIncrement = () => {
    this.setState(prevState => ({
      count: prevState.count + 1
    }))
  }
  
  // 渲染方法
  render() {
    console.log('render')
    
    if (this.state.hasError) {
      return <View>出错了</View>
    }
    
    return (
      <View className='my-component'>
        <Text>计数: {this.state.count}</Text>
        <View onClick={this.handleIncrement}>增加</View>
      </View>
    )
  }
}
```

### React Hooks 生命周期

在函数组件中，可以使用 Hooks 来模拟生命周期行为：

```jsx
import React, { useState, useEffect, useRef } from 'react'
import { View, Text } from '@tarojs/components'

function HooksComponent(props) {
  // 状态管理
  const [count, setCount] = useState(props.initialCount || 0)
  const [data, setData] = useState(null)
  
  // 相当于 componentDidMount
  useEffect(() => {
    console.log('Component mounted')
    
    // 加载数据
    fetchData()
    
    // 设置定时器
    const timer = setInterval(() => {
      console.log('timer tick')
    }, 1000)
    
    // 相当于 componentWillUnmount
    return () => {
      console.log('Component will unmount')
      clearInterval(timer)
    }
  }, []) // 空依赖数组表示只在挂载和卸载时执行
  
  // 相当于 componentDidUpdate (针对 count 的变化)
  useEffect(() => {
    console.log('count changed to', count)
    
    // 可以执行一些副作用
    document.title = `Count: ${count}`
  }, [count]) // 依赖 count，当 count 变化时执行
  
  // 自定义方法
  const fetchData = async () => {
    try {
      const response = await fetch('https://api.example.com/data')
      const result = await response.json()
      setData(result)
    } catch (error) {
      console.error('Failed to fetch data', error)
    }
  }
  
  const handleIncrement = () => {
    setCount(prevCount => prevCount + 1)
  }
  
  // 渲染
  return (
    <View className='hooks-component'>
      <Text>计数: {count}</Text>
      <View onClick={handleIncrement}>增加</View>
      
      {data ? (
        <View className='data'>
          {data.name}
        </View>
      ) : (
        <View>加载中...</View>
      )}
    </View>
  )
}

export default HooksComponent
```

## 生命周期最佳实践

### 应用级别最佳实践

1. **在 onLaunch 中进行全局初始化**
   - 检查更新
   - 初始化全局状态
   - 配置网络请求拦截器

2. **在 onShow 中处理应用恢复**
   - 刷新过期数据
   - 恢复被暂停的操作
   - 检查登录状态

3. **在 onHide 中保存状态**
   - 保存未提交的数据
   - 暂停后台任务
   - 释放不必要的资源

### 页面级别最佳实践

1. **合理使用生命周期函数**
   - onLoad: 获取路由参数、初始化页面数据
   - onShow: 刷新可能过期的数据
   - onReady: 执行依赖 DOM 的操作
   - onHide/onUnload: 保存页面状态、清理资源

2. **避免生命周期函数中的重复操作**
   - 使用标志变量控制只执行一次的操作
   - 区分 onShow 和 onLoad 的职责

3. **合理处理页面间数据传递**
   - 路由参数：适用于简单数据
   - 全局状态：适用于复杂共享数据
   - 缓存：适用于大量数据

### 组件级别最佳实践

1. **保持组件生命周期函数简洁**
   - 将复杂逻辑抽离到独立方法
   - 避免在生命周期中执行过多操作

2. **正确清理资源**
   - 在 componentWillUnmount 中清理定时器、事件监听等
   - 取消未完成的网络请求

3. **使用 shouldComponentUpdate 优化性能**
   - 避免不必要的重渲染
   - 结合 immutable 数据进行高效比较

4. **函数组件中的 Hooks 使用建议**
   - 使用 useEffect 的依赖数组控制执行时机
   - 使用 useCallback 和 useMemo 避免不必要的重复计算
   - 使用自定义 Hooks 抽离和复用逻辑

## 常见生命周期问题处理

### 1. 数据请求时机问题

**问题**：在哪个生命周期中请求数据最合适？

**解决方案**：
- 类组件：通常在 componentDidMount 中请求数据
- 页面组件：在 onLoad 或 onShow 中请求数据
- 函数组件：在 useEffect 中请求数据

```jsx
// 类组件
componentDidMount() {
  this.fetchData()
}

// 页面组件
onLoad() {
  this.fetchData()
}

// 函数组件
useEffect(() => {
  const fetchData = async () => {
    // 请求数据
  }
  fetchData()
}, [])
```

### 2. 重复请求数据问题

**问题**：页面显示/隐藏多次时可能重复请求数据

**解决方案**：
- 使用标志变量控制
- 缓存数据并设置过期时间

```jsx
onShow() {
  const now = Date.now()
  const lastFetchTime = this.lastFetchTime || 0
  
  // 如果距离上次请求超过5分钟，则重新请求
  if (now - lastFetchTime > 5 * 60 * 1000) {
    this.fetchData()
    this.lastFetchTime = now
  }
}
```

### 3. 生命周期执行顺序问题

**问题**：多层组件嵌套时生命周期执行顺序混乱

**解决方案**：
- 理解生命周期的执行顺序
- 避免在父组件生命周期中依赖子组件状态

**执行顺序**：
1. 挂载时：先父组件 render，然后子组件完整生命周期，最后父组件 componentDidMount
2. 更新时：先父组件 render，然后子组件更新，最后父组件 componentDidUpdate
3. 卸载时：先子组件 componentWillUnmount，然后父组件 componentWillUnmount

### 4. 内存泄漏问题

**问题**：组件卸载后仍有异步操作试图修改状态

**解决方案**：
- 使用标志变量控制
- 在 componentWillUnmount 中清理所有资源

```jsx
componentDidMount() {
  this.mounted = true
  
  this.fetchData().then(data => {
    // 检查组件是否仍然挂载
    if (this.mounted) {
      this.setState({ data })
    }
  })
}

componentWillUnmount() {
  this.mounted = false
  // 清理其他资源
}
```

### 5. 页面返回数据刷新问题

**问题**：从详情页返回列表页，需要刷新列表数据

**解决方案**：
- 使用页面栈和页面间通信
- 在 onShow 中检查是否需要刷新

```jsx
// 列表页
onShow() {
  // 从全局变量或缓存中获取标志
  const needRefresh = Taro.getStorageSync('needRefreshList')
  
  if (needRefresh) {
    this.fetchListData()
    Taro.removeStorageSync('needRefreshList')
  }
}

// 详情页
handleDelete() {
  // 删除操作成功后
  Taro.setStorageSync('needRefreshList', true)
  Taro.navigateBack()
}
```

### 6. 生命周期中的异步操作

**问题**：生命周期中的异步操作可能导致状态更新延迟

**解决方案**：
- 使用 Promise 或 async/await 处理异步操作
- 添加加载状态指示器

```jsx
async componentDidMount() {
  this.setState({ loading: true })
  
  try {
    const data = await this.fetchData()
    this.setState({ data, loading: false })
  } catch (error) {
    this.setState({ error, loading: false })
  }
}
```

## 总结

Taro 的生命周期系统是构建可靠应用的基础。通过合理使用应用级别、页面级别和组件级别的生命周期函数，可以有效控制应用的行为和性能。

关键要点：

1. **理解不同级别的生命周期函数及其触发时机**
2. **在合适的生命周期函数中执行相应的操作**
3. **避免常见的生命周期问题，如内存泄漏、重复请求等**
4. **使用生命周期最佳实践优化应用性能和用户体验**

掌握这些知识后，你将能够构建出行为可预测、性能优秀的 Taro 应用。 