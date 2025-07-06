# 5. 页面配置与页面间处理函数

在 Taro 应用开发中，页面配置和页面间的数据传递是构建完整应用的关键环节。本章将详细介绍 Taro 中的页面配置项、页面路由机制、页面间数据传递以及页面跳转动画等内容。

## 页面配置项详解

Taro 允许开发者通过配置文件为每个页面设置特定的属性和行为。这些配置既可以在全局 `app.config.js` 中统一设置，也可以在各页面的 `[页面名].config.js` 中单独定义。

### 页面配置文件

每个页面可以拥有自己的配置文件，通常命名为 `index.config.js` (如果页面文件是 `index.jsx`)：

```js
// pages/home/index.config.js
export default {
  navigationBarTitleText: '首页',
  enablePullDownRefresh: true,
  backgroundColor: '#f5f5f5',
  navigationBarBackgroundColor: '#3281ff',
  navigationBarTextStyle: 'white',
  onReachBottomDistance: 50
}
```

### 常用页面配置项

| 配置项 | 类型 | 默认值 | 说明 |
|-------|------|-------|------|
| navigationBarTitleText | String | | 导航栏标题文字 |
| navigationBarBackgroundColor | HexColor | #000000 | 导航栏背景颜色 |
| navigationBarTextStyle | String | white | 导航栏标题颜色，仅支持 black/white |
| backgroundColor | HexColor | #ffffff | 窗口背景色 |
| backgroundTextStyle | String | dark | 下拉背景字体、loading 图的样式，仅支持 dark/light |
| enablePullDownRefresh | Boolean | false | 是否开启下拉刷新 |
| onReachBottomDistance | Number | 50 | 页面上拉触底事件触发时距页面底部距离，单位为 px |
| disableScroll | Boolean | false | 设置为 true 则页面整体不能上下滚动 |
| usingComponents | Object | | 页面自定义组件配置 |

### 页面样式与全局样式

页面样式可以通过 `.scss`、`.less` 或 `.css` 文件定义，并在页面组件中引入：

```jsx
// pages/home/index.jsx
import React, { Component } from 'react'
import { View, Text } from '@tarojs/components'
import './index.scss' // 引入页面样式

export default class HomePage extends Component {
  render() {
    return (
      <View className='home-page'>
        <Text className='title'>首页</Text>
      </View>
    )
  }
}
```

```scss
/* pages/home/index.scss */
.home-page {
  padding: 20px;
  
  .title {
    font-size: 18px;
    font-weight: bold;
    color: #333;
  }
}
```

### 自定义导航栏

除了通过配置项设置导航栏，Taro 还支持自定义导航栏：

```jsx
// 自定义导航栏组件
import React from 'react'
import { View, Text } from '@tarojs/components'
import Taro from '@tarojs/taro'
import './navbar.scss'

function CustomNavBar({ title, showBack = true }) {
  const handleBack = () => {
    Taro.navigateBack()
  }
  
  // 获取系统信息，适配不同机型
  const systemInfo = Taro.getSystemInfoSync()
  const statusBarHeight = systemInfo.statusBarHeight || 20
  const navBarHeight = 44 // 导航栏高度
  
  return (
    <View className='custom-navbar' style={{ paddingTop: `${statusBarHeight}px` }}>
      <View className='navbar-content' style={{ height: `${navBarHeight}px` }}>
        {showBack && (
          <View className='navbar-back' onClick={handleBack}>
            返回
          </View>
        )}
        <Text className='navbar-title'>{title}</Text>
      </View>
    </View>
  )
}

export default CustomNavBar
```

在页面中使用自定义导航栏：

```jsx
// pages/detail/index.jsx
import React, { Component } from 'react'
import { View, Text } from '@tarojs/components'
import CustomNavBar from '../../components/CustomNavBar'
import './index.scss'

export default class DetailPage extends Component {
  // 配置文件中隐藏原生导航栏
  // index.config.js: { navigationStyle: 'custom' }
  
  render() {
    return (
      <View className='detail-page'>
        <CustomNavBar title='详情页' />
        <View className='content'>
          <Text>页面内容</Text>
        </View>
      </View>
    )
  }
}
```

## 页面路由机制

Taro 提供了完整的路由系统，支持页面间的跳转、传参和返回等操作。

### 路由方法

Taro 提供了以下路由方法：

| 方法 | 说明 |
|------|------|
| Taro.navigateTo | 保留当前页面，跳转到应用内的某个页面 |
| Taro.redirectTo | 关闭当前页面，跳转到应用内的某个页面 |
| Taro.switchTab | 跳转到 tabBar 页面，并关闭其他所有非 tabBar 页面 |
| Taro.navigateBack | 关闭当前页面，返回上一页面或多级页面 |
| Taro.reLaunch | 关闭所有页面，打开到应用内的某个页面 |

### 路由使用示例

```jsx
// 导航到新页面
Taro.navigateTo({
  url: '/pages/detail/index?id=123&type=product'
})

// 重定向到新页面
Taro.redirectTo({
  url: '/pages/login/index'
})

// 切换到 Tab 页面
Taro.switchTab({
  url: '/pages/home/index'
})

// 返回上一页
Taro.navigateBack()

// 返回多级页面
Taro.navigateBack({
  delta: 2 // 返回的页面数，如果 delta 大于现有页面数，则返回到首页
})

// 重启到某个页面
Taro.reLaunch({
  url: '/pages/index/index'
})
```

### 路由拦截

在某些场景下，需要对页面跳转进行拦截处理，例如检查用户登录状态：

```jsx
// 封装路由拦截方法
const checkLoginAndNavigate = (url) => {
  const isLoggedIn = Taro.getStorageSync('isLoggedIn')
  
  if (isLoggedIn) {
    // 已登录，正常跳转
    Taro.navigateTo({ url })
  } else {
    // 未登录，跳转到登录页
    Taro.navigateTo({
      url: '/pages/login/index?redirect=' + encodeURIComponent(url)
    })
  }
}

// 使用拦截方法
checkLoginAndNavigate('/pages/user/index')
```

## 页面参数传递方式

Taro 提供了多种页面间传递数据的方式，适用于不同的场景。

### URL 参数传递

最基本的参数传递方式是通过 URL 查询字符串：

```jsx
// 页面 A：传递参数
Taro.navigateTo({
  url: '/pages/detail/index?id=123&name=product&category=electronics'
})

// 页面 B：接收参数
class DetailPage extends Component {
  componentDidMount() {
    // 在页面生命周期函数中获取参数
    const { id, name, category } = this.$instance.router.params
    console.log('接收到的参数:', id, name, category)
  }
  
  // 或者在 onLoad 中获取
  onLoad(options) {
    console.log('页面参数:', options) // { id: '123', name: 'product', category: 'electronics' }
  }
}
```

### EventChannel 事件通道

对于需要双向通信的场景，可以使用事件通道：

```jsx
// 页面 A：传递参数并创建事件通道
const eventChannel = Taro.navigateTo({
  url: '/pages/detail/index?id=123',
  events: {
    // 监听从被打开页面传回的数据
    acceptDataFromDetail: function(data) {
      console.log('页面 B 返回的数据:', data)
    }
  },
  success: function(res) {
    // 向被打开页面传送数据
    res.eventChannel.emit('acceptDataFromOpener', { 
      productInfo: { 
        id: 123, 
        name: '商品名称', 
        price: 99.9 
      } 
    })
  }
})

// 页面 B：接收参数并返回数据
class DetailPage extends Component {
  componentDidMount() {
    const eventChannel = this.getOpenerEventChannel()
    
    // 监听从打开页面传来的数据
    eventChannel.on('acceptDataFromOpener', (data) => {
      console.log('接收到的产品信息:', data.productInfo)
      this.setState({ productInfo: data.productInfo })
    })
    
    // 向打开页面传回数据
    setTimeout(() => {
      eventChannel.emit('acceptDataFromDetail', { 
        result: 'success', 
        selectedOptions: ['红色', 'L码'] 
      })
    }, 2000)
  }
}
```

### 全局状态管理

对于复杂应用，推荐使用全局状态管理方案进行数据共享：

```jsx
// 使用 Redux 进行状态管理
import { useSelector, useDispatch } from 'react-redux'
import { setProductDetail, addToCart } from '../store/actions'

// 页面 A：存储数据到全局状态
function ProductListPage() {
  const dispatch = useDispatch()
  
  const handleProductClick = (product) => {
    // 将产品详情存储到全局状态
    dispatch(setProductDetail(product))
    // 导航到详情页
    Taro.navigateTo({ url: '/pages/detail/index' })
  }
  
  // ...
}

// 页面 B：从全局状态获取数据
function DetailPage() {
  // 从全局状态获取产品详情
  const productDetail = useSelector(state => state.products.currentProduct)
  const dispatch = useDispatch()
  
  const handleAddToCart = () => {
    dispatch(addToCart(productDetail))
    Taro.showToast({ title: '已加入购物车' })
  }
  
  // ...
}
```

### 本地存储

对于简单的数据或需要持久化的数据，可以使用本地存储：

```jsx
// 页面 A：存储数据
Taro.setStorageSync('productData', {
  id: 123,
  name: '商品名称',
  price: 99.9,
  details: '...'
})

Taro.navigateTo({ url: '/pages/detail/index' })

// 页面 B：获取数据
componentDidMount() {
  const productData = Taro.getStorageSync('productData')
  this.setState({ productData })
  
  // 使用完后可以清除数据
  // Taro.removeStorageSync('productData')
}
```

## 页面间数据共享

除了直接的参数传递，Taro 还提供了多种页面间共享数据的方式。

### Context API

使用 React 的 Context API 可以在组件树中共享数据：

```jsx
// context/AppContext.js
import React, { createContext, useState } from 'react'

// 创建 Context
export const AppContext = createContext({})

// Context Provider 组件
export function AppContextProvider({ children }) {
  const [userInfo, setUserInfo] = useState(null)
  const [cartItems, setCartItems] = useState([])
  
  // 提供给子组件的方法
  const login = (user) => {
    setUserInfo(user)
  }
  
  const logout = () => {
    setUserInfo(null)
  }
  
  const addToCart = (item) => {
    setCartItems([...cartItems, item])
  }
  
  // 共享的状态和方法
  const contextValue = {
    userInfo,
    cartItems,
    login,
    logout,
    addToCart
  }
  
  return (
    <AppContext.Provider value={contextValue}>
      {children}
    </AppContext.Provider>
  )
}
```

在应用中使用 Context：

```jsx
// app.js
import React, { Component } from 'react'
import { AppContextProvider } from './context/AppContext'

class App extends Component {
  render() {
    return (
      <AppContextProvider>
        {this.props.children}
      </AppContextProvider>
    )
  }
}

export default App
```

在页面中使用共享数据：

```jsx
// pages/user/index.jsx
import React, { useContext } from 'react'
import { View, Text, Button } from '@tarojs/components'
import { AppContext } from '../../context/AppContext'

function UserPage() {
  const { userInfo, logout } = useContext(AppContext)
  
  return (
    <View className='user-page'>
      {userInfo ? (
        <>
          <Text>欢迎，{userInfo.name}</Text>
          <Button onClick={logout}>退出登录</Button>
        </>
      ) : (
        <Text>请先登录</Text>
      )}
    </View>
  )
}

export default UserPage
```

### 自定义事件系统

对于不相关的组件间通信，可以使用自定义事件系统：

```jsx
// utils/eventBus.js
class EventBus {
  constructor() {
    this.events = {}
  }
  
  on(eventName, callback) {
    if (!this.events[eventName]) {
      this.events[eventName] = []
    }
    this.events[eventName].push(callback)
  }
  
  off(eventName, callback) {
    if (this.events[eventName]) {
      if (callback) {
        this.events[eventName] = this.events[eventName].filter(cb => cb !== callback)
      } else {
        delete this.events[eventName]
      }
    }
  }
  
  emit(eventName, ...args) {
    if (this.events[eventName]) {
      this.events[eventName].forEach(callback => {
        callback(...args)
      })
    }
  }
}

export default new EventBus()
```

在不同页面间使用事件总线：

```jsx
// 页面 A
import eventBus from '../../utils/eventBus'

// 发送事件
handleUpdateProfile = (newProfile) => {
  eventBus.emit('profileUpdated', newProfile)
  Taro.navigateBack()
}

// 页面 B
import eventBus from '../../utils/eventBus'

componentDidMount() {
  // 监听事件
  this.profileUpdateListener = (profile) => {
    this.setState({ userProfile: profile })
    Taro.showToast({ title: '资料已更新' })
  }
  
  eventBus.on('profileUpdated', this.profileUpdateListener)
}

componentWillUnmount() {
  // 取消监听
  eventBus.off('profileUpdated', this.profileUpdateListener)
}
```

## 页面跳转动画与交互

Taro 支持自定义页面跳转动画，增强用户体验。

### 基本跳转动画

在小程序环境中，可以通过配置指定跳转动画：

```jsx
Taro.navigateTo({
  url: '/pages/detail/index',
  animationType: 'slide-in-right', // 从右侧滑入
  animationDuration: 300 // 动画持续时间，单位毫秒
})
```

常用的动画类型包括：
- `slide-in-right`：从右侧滑入
- `slide-in-left`：从左侧滑入
- `slide-in-top`：从顶部滑入
- `slide-in-bottom`：从底部滑入
- `fade-in`：淡入
- `zoom-in`：缩放显示
- `zoom-fade-in`：缩放并淡入
- `pop-in`：弹出

### 自定义转场动画

在 H5 环境中，可以使用 CSS 动画实现更丰富的转场效果：

```scss
/* 页面进入动画 */
.page-enter {
  transform: translateX(100%);
}

.page-enter-active {
  transform: translateX(0);
  transition: transform 300ms ease-in-out;
}

/* 页面退出动画 */
.page-exit {
  transform: translateX(0);
}

.page-exit-active {
  transform: translateX(-100%);
  transition: transform 300ms ease-in-out;
}
```

### 手势返回

在 H5 环境中，可以实现类似原生 App 的手势返回效果：

```jsx
import React, { Component } from 'react'
import { View } from '@tarojs/components'
import Taro from '@tarojs/taro'
import './index.scss'

export default class PageWithGesture extends Component {
  constructor(props) {
    super(props)
    this.state = {
      startX: 0,
      moveX: 0,
      isMoving: false
    }
  }
  
  handleTouchStart = (e) => {
    // 只处理左边缘开始的滑动
    if (e.touches[0].clientX < 30) {
      this.setState({
        startX: e.touches[0].clientX,
        isMoving: true
      })
    }
  }
  
  handleTouchMove = (e) => {
    if (this.state.isMoving) {
      this.setState({
        moveX: Math.max(0, e.touches[0].clientX - this.state.startX)
      })
    }
  }
  
  handleTouchEnd = () => {
    if (this.state.isMoving) {
      // 如果滑动距离超过屏幕宽度的 1/3，则返回上一页
      if (this.state.moveX > Taro.getSystemInfoSync().windowWidth / 3) {
        Taro.navigateBack()
      }
      
      this.setState({
        startX: 0,
        moveX: 0,
        isMoving: false
      })
    }
  }
  
  render() {
    const { moveX, isMoving } = this.state
    const pageStyle = isMoving ? { transform: `translateX(${moveX}px)` } : {}
    
    return (
      <View 
        className='page-with-gesture'
        style={pageStyle}
        onTouchStart={this.handleTouchStart}
        onTouchMove={this.handleTouchMove}
        onTouchEnd={this.handleTouchEnd}
      >
        {/* 页面内容 */}
      </View>
    )
  }
}
```

### 页面预加载

为了提升用户体验，可以预先加载下一个可能访问的页面：

```jsx
// 预加载页面
Taro.preloadPage({
  url: '/pages/detail/index?id=123',
  success: () => {
    console.log('页面预加载成功')
  }
})

// 稍后正常跳转
Taro.navigateTo({
  url: '/pages/detail/index?id=123'
})
```

## 最佳实践

### 页面参数传递的最佳实践

1. **简单参数使用 URL 查询字符串**
   - 适用于少量、简单的数据
   - 参数会显示在 URL 中，不适合敏感信息

2. **复杂数据使用全局状态管理**
   - 适用于大量、复杂的数据
   - 适用于多个页面需要共享的数据

3. **临时数据使用本地存储**
   - 适用于一次性传递的大量数据
   - 记得在使用后清除数据，避免内存泄漏

4. **页面返回传参使用事件通道或事件总线**
   - 适用于子页面需要向父页面返回数据的场景

### 页面配置的最佳实践

1. **合理使用全局配置和页面配置**
   - 在 `app.config.js` 中设置通用配置
   - 在页面配置文件中覆盖特定配置

2. **根据页面功能设置合适的导航栏样式**
   - 内容页面使用默认导航栏
   - 特殊页面（如首页、登录页）可以使用自定义导航栏

3. **合理设置页面背景色和下拉刷新样式**
   - 与页面内容保持一致的颜色风格
   - 列表页面开启下拉刷新功能

### 页面路由的最佳实践

1. **合理选择路由方法**
   - 普通页面跳转使用 `navigateTo`
   - 登录成功后使用 `redirectTo` 避免用户返回登录页
   - 完成操作后返回使用 `navigateBack`
   - 需要重置页面栈时使用 `reLaunch`

2. **避免过深的页面栈**
   - 小程序环境通常限制页面栈深度为 10 层
   - 超过限制后，使用 `redirectTo` 替代 `navigateTo`

3. **实现路由守卫**
   - 封装路由方法，统一处理权限验证
   - 对需要登录的页面进行拦截

### 页面间数据共享的最佳实践

1. **选择合适的数据共享方案**
   - 小型应用可以使用 Context API
   - 中大型应用推荐使用 Redux 或 Mobx
   - 不相关组件间通信可以使用事件总线

2. **避免过度共享数据**
   - 只共享必要的数据
   - 注意数据的作用域和生命周期

3. **注意内存管理**
   - 组件卸载时清理事件监听
   - 不再需要的临时数据及时清除

## 总结

本章详细介绍了 Taro 中的页面配置项、页面路由机制、页面间数据传递以及页面跳转动画等内容。通过合理使用这些功能，可以构建出用户体验良好、页面间交互流畅的 Taro 应用。

关键要点：

1. **灵活运用页面配置，为不同页面设置合适的样式和行为**
2. **掌握多种页面间数据传递方式，根据场景选择最合适的方案**
3. **使用页面路由机制实现流畅的页面跳转和返回**
4. **通过页面跳转动画增强用户体验**
5. **遵循最佳实践，构建高质量的 Taro 应用** 