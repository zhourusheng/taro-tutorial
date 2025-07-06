# 2. 下载与安装

本章将指导你完成 Taro 开发环境的搭建，从零开始配置一个完整的 Taro 开发环境，并创建你的第一个 Taro 项目。

## 2.1 Node.js 环境配置

Taro 是基于 Node.js 开发的，因此在开始使用 Taro 之前，你需要先安装 Node.js 环境。

### 2.1.1 Node.js 版本要求

Taro 对 Node.js 的版本有一定要求，建议安装 **LTS 长期支持版本**：

- Taro 3.x：Node.js >= 12.0.0
- Taro 2.x：Node.js >= 8.0.0

> **提示**：推荐使用 Node.js 14.x 或 16.x 版本，这些版本经过充分测试，与 Taro 的兼容性最好。

### 2.1.2 安装 Node.js

#### Windows 系统安装 Node.js

1. 访问 [Node.js 官网](https://nodejs.org/)，下载适合 Windows 系统的安装包（推荐 LTS 版本）
2. 运行下载的安装包，按照安装向导完成安装
3. 安装完成后，打开命令提示符（CMD）或 PowerShell，输入以下命令验证安装：

```bash
node -v
npm -v
```

如果显示版本号，则表示安装成功。

#### macOS 系统安装 Node.js

1. **方法一：使用安装包**
   - 访问 [Node.js 官网](https://nodejs.org/)，下载适合 macOS 系统的安装包
   - 运行安装包，按照安装向导完成安装

2. **方法二：使用 Homebrew**（推荐）
   - 如果已安装 Homebrew，可以使用以下命令安装 Node.js：

   ```bash
   brew install node
   ```

3. 安装完成后，打开终端，输入以下命令验证安装：

```bash
node -v
npm -v
```

#### Linux 系统安装 Node.js

1. **使用包管理器**（以 Ubuntu 为例）：

```bash
# 使用 apt 安装
sudo apt update
sudo apt install nodejs npm

# 验证安装
node -v
npm -v
```

2. **使用 NVM 安装**（推荐，便于管理多版本）：

```bash
# 安装 NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
# 或使用 wget
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash

# 重新加载配置
source ~/.bashrc  # 或 source ~/.zshrc

# 安装指定版本的 Node.js
nvm install 16  # 安装 Node.js 16.x 版本

# 设置默认版本
nvm use 16
```

### 2.1.3 配置 npm 镜像源

由于网络原因，在中国大陆使用 npm 官方源可能会较慢，建议配置国内镜像源：

```bash
# 使用淘宝 NPM 镜像
npm config set registry https://registry.npmmirror.com

# 验证配置
npm config get registry
```

## 2.2 Taro CLI 工具安装

Taro 命令行工具（CLI）是开发 Taro 项目的核心工具，它提供了项目初始化、编译、打包等功能。

### 2.2.1 全局安装 Taro CLI

```bash
# 使用 npm 安装
npm install -g @tarojs/cli

# 或使用 yarn 安装
yarn global add @tarojs/cli
```

### 2.2.2 验证安装

安装完成后，在命令行中输入以下命令，查看 Taro CLI 版本：

```bash
taro -v
```

如果显示版本信息，则表示安装成功。

### 2.2.3 更新 Taro CLI

如果需要更新 Taro CLI 到最新版本，可以使用以下命令：

```bash
# 使用 npm 更新
npm update -g @tarojs/cli

# 或使用 yarn 更新
yarn global upgrade @tarojs/cli
```

### 2.2.4 Taro CLI 版本与项目版本

需要注意的是，Taro CLI 的版本应该与你项目中使用的 Taro 版本保持一致，否则可能会出现兼容性问题。

如果你需要维护多个不同版本的 Taro 项目，可以考虑使用项目级别的 Taro CLI：

```bash
# 在项目中安装特定版本的 Taro CLI
npm install --save-dev @tarojs/cli@x.x.x

# 然后通过 npx 使用项目级别的 Taro CLI
npx taro build --type weapp
```

## 2.3 创建第一个 Taro 项目

### 2.3.1 使用 Taro CLI 初始化项目

```bash
# 使用命令行创建项目
taro init myTaroApp
```

执行该命令后，Taro CLI 会引导你进行一系列项目配置：

1. **选择框架**：React、Vue2、Vue3 或 Nerv
2. **选择模板**：默认模板、Redux 模板、TypeScript 模板等
3. **选择 CSS 预处理器**：Sass、Less、Stylus 或不使用
4. **选择包管理工具**：npm、yarn 或 pnpm

根据提示完成选择后，Taro CLI 会自动创建项目并安装依赖。

### 2.3.2 项目创建示例

以下是一个完整的项目创建示例：

```bash
$ taro init myTaroApp
👽 Taro v3.6.8

Taro 即将创建一个新项目!
Need help? Go and open issue: https://github.com/NervJS/taro/issues/new

? 请选择框架 React
? 请选择模板源 默认模板
  默认模板
  NutUI React 模板
  Taro UI 模板
  Taro UI Vue3 模板
  React Native 模板
  
? 请选择模板 默认模板
? 请输入项目介绍 My First Taro App
? 请选择 CSS 预处理器 Sass
? 请选择包管理工具 npm
```

### 2.3.3 运行项目

项目创建完成后，可以使用以下命令运行项目：

```bash
# 进入项目目录
cd myTaroApp

# 运行微信小程序
npm run dev:weapp

# 运行 H5 版本
npm run dev:h5

# 运行支付宝小程序
npm run dev:alipay

# 运行字节跳动小程序
npm run dev:tt

# 运行 QQ 小程序
npm run dev:qq
```

运行命令后，Taro 会自动编译项目代码。对于小程序，你需要使用对应的开发者工具打开项目编译后的目录（默认为 `dist` 目录）。

### 2.3.4 微信小程序开发工具配置

以微信小程序为例，运行 `npm run dev:weapp` 后：

1. 打开微信开发者工具
2. 选择"导入项目"
3. 项目路径选择 Taro 项目下的 `dist` 目录
4. AppID 可以使用测试号或填入已申请的小程序 AppID
5. 点击"导入"完成项目导入

## 2.4 项目目录结构详解

Taro 初始化的项目目录结构如下：

```
myTaroApp/
├── config/                 // 项目编译配置目录
│   ├── dev.js              // 开发环境配置
│   ├── index.js            // 默认配置
│   └── prod.js             // 生产环境配置
├── dist/                   // 编译结果目录
├── src/                    // 源码目录
│   ├── app.config.js       // 全局配置
│   ├── app.js              // 项目入口文件
│   ├── app.scss            // 全局样式
│   ├── index.html          // H5 入口 HTML
│   ├── pages/              // 页面文件目录
│   │   └── index/          // index 页面目录
│   │       ├── index.config.js  // 页面配置
│   │       ├── index.jsx    // 页面逻辑
│   │       └── index.scss   // 页面样式
│   └── assets/             // 静态资源目录
├── babel.config.js         // Babel 配置
├── package.json            // 项目依赖
├── project.config.json     // 微信小程序项目配置
└── README.md               // 项目说明
```

### 2.4.1 核心目录和文件详解

#### config 目录

包含项目的编译配置，可以针对不同环境进行定制：

- `index.js`：基础配置，会与环境特定的配置合并
- `dev.js`：开发环境特定配置
- `prod.js`：生产环境特定配置

示例配置：

```javascript
// config/index.js
const config = {
  projectName: 'myTaroApp',
  date: '2023-7-6',
  designWidth: 750,
  deviceRatio: {
    640: 2.34 / 2,
    750: 1,
    828: 1.81 / 2
  },
  sourceRoot: 'src',
  outputRoot: 'dist',
  plugins: [],
  defineConstants: {
  },
  copy: {
    patterns: [
    ],
    options: {
    }
  },
  framework: 'react',
  compiler: 'webpack5',
  cache: {
    enable: false // Webpack 持久化缓存
  },
  mini: {
    postcss: {
      pxtransform: {
        enable: true,
        config: {

        }
      },
      url: {
        enable: true,
        config: {
          limit: 1024 // 设定转换尺寸上限
        }
      },
      cssModules: {
        enable: false, // 默认为 false，如需使用 css modules 功能，则设为 true
        config: {
          namingPattern: 'module', // 转换模式，取值为 global/module
          generateScopedName: '[name]__[local]___[hash:base64:5]'
        }
      }
    }
  },
  h5: {
    publicPath: '/',
    staticDirectory: 'static',
    postcss: {
      autoprefixer: {
        enable: true,
        config: {
        }
      },
      cssModules: {
        enable: false, // 默认为 false，如需使用 css modules 功能，则设为 true
        config: {
          namingPattern: 'module', // 转换模式，取值为 global/module
          generateScopedName: '[name]__[local]___[hash:base64:5]'
        }
      }
    }
  }
}

module.exports = function (merge) {
  if (process.env.NODE_ENV === 'development') {
    return merge({}, config, require('./dev'))
  }
  return merge({}, config, require('./prod'))
}
```

#### src 目录

包含项目的源代码：

- `app.js`：应用入口文件，包含应用生命周期和全局配置
- `app.config.js`：应用全局配置，对应小程序的 app.json
- `app.scss`：全局样式文件
- `pages/`：页面文件目录

示例 app.js：

```javascript
import { Component } from 'react'
import './app.scss'

class App extends Component {
  componentDidMount () {}

  componentDidShow () {}

  componentDidHide () {}

  // 在 App 类中的 render() 函数没有实际作用
  // 请勿修改此函数
  render () {
    return this.props.children
  }
}

export default App
```

示例 app.config.js：

```javascript
export default {
  pages: [
    'pages/index/index'
  ],
  window: {
    backgroundTextStyle: 'light',
    navigationBarBackgroundColor: '#fff',
    navigationBarTitleText: 'WeChat',
    navigationBarTextStyle: 'black'
  }
}
```

#### pages 目录

包含应用的页面文件，每个页面通常包含以下文件：

- `index.jsx`：页面逻辑
- `index.scss`：页面样式
- `index.config.js`：页面配置，对应小程序页面的 json 配置

示例页面文件：

```javascript
// pages/index/index.jsx
import { Component } from 'react'
import { View, Text, Button } from '@tarojs/components'
import './index.scss'

export default class Index extends Component {
  componentWillMount () { }

  componentDidMount () { }

  componentWillUnmount () { }

  componentDidShow () { }

  componentDidHide () { }

  render () {
    return (
      <View className='index'>
        <Text>Hello world!</Text>
        <Button type='primary'>点击按钮</Button>
      </View>
    )
  }
}
```

```javascript
// pages/index/index.config.js
export default {
  navigationBarTitleText: '首页'
}
```

## 2.5 常用命令介绍

Taro CLI 提供了丰富的命令，用于项目的创建、开发、构建等操作。

### 2.5.1 项目初始化

```bash
# 创建新项目
taro init [项目名]
```

### 2.5.2 开发命令

```bash
# 开发微信小程序
taro build --type weapp --watch
# 简写形式
npm run dev:weapp

# 开发 H5
taro build --type h5 --watch
# 简写形式
npm run dev:h5

# 开发支付宝小程序
taro build --type alipay --watch
# 简写形式
npm run dev:alipay

# 开发百度小程序
taro build --type swan --watch
# 简写形式
npm run dev:swan

# 开发字节跳动小程序
taro build --type tt --watch
# 简写形式
npm run dev:tt

# 开发 QQ 小程序
taro build --type qq --watch
# 简写形式
npm run dev:qq

# 开发京东小程序
taro build --type jd --watch
# 简写形式
npm run dev:jd

# 开发快应用
taro build --type quickapp --watch
# 简写形式
npm run dev:quickapp
```

### 2.5.3 生产构建命令

```bash
# 构建微信小程序
taro build --type weapp
# 简写形式
npm run build:weapp

# 构建 H5
taro build --type h5
# 简写形式
npm run build:h5

# 其他平台类似，将 dev 改为 build 即可
```

### 2.5.4 项目信息

```bash
# 查看项目信息
taro info
```

### 2.5.5 更新项目依赖

```bash
# 更新项目中的 Taro 依赖版本
taro update project
```

### 2.5.6 环境及参数配置

```bash
# 指定环境
taro build --type weapp --env [环境名称]

# 指定输出目录
taro build --type weapp --output-root [目录名称]
```

## 2.6 常见问题与解决方案

### 2.6.1 依赖安装失败

**问题**：执行 `npm install` 时，部分依赖安装失败。

**解决方案**：
1. 检查网络连接
2. 尝试使用国内镜像源：`npm config set registry https://registry.npmmirror.com`
3. 清除 npm 缓存：`npm cache clean --force`
4. 尝试使用 yarn 或 pnpm 安装依赖

### 2.6.2 编译错误

**问题**：执行 `npm run dev:weapp` 时出现编译错误。

**解决方案**：
1. 检查 Node.js 版本是否符合要求
2. 检查项目依赖是否正确安装
3. 检查代码语法是否正确
4. 尝试删除 `node_modules` 目录，重新安装依赖

### 2.6.3 小程序开发者工具无法预览

**问题**：编译成功，但在小程序开发者工具中无法正常预览。

**解决方案**：
1. 检查是否正确导入了项目的 `dist` 目录
2. 检查小程序开发者工具是否为最新版本
3. 在小程序开发者工具中，关闭"ES6 转 ES5"、"增强编译"等选项
4. 检查 `project.config.json` 文件配置是否正确

## 2.7 总结

本章我们完成了 Taro 开发环境的搭建，包括 Node.js 环境配置、Taro CLI 工具安装、创建第一个 Taro 项目，并详细介绍了项目目录结构和常用命令。

通过本章的学习，你已经具备了开始 Taro 开发的基础环境和知识。在下一章中，我们将深入了解 Taro 的框架结构和核心概念，为进一步的开发打下基础。

## 2.8 练习

1. 安装 Node.js 和 Taro CLI
2. 创建一个新的 Taro 项目，选择 React 框架和 Sass 预处理器
3. 运行项目，在微信开发者工具中预览
4. 尝试修改 `src/pages/index/index.jsx` 文件，添加一个新的按钮组件
5. 重新编译项目，观察修改效果 