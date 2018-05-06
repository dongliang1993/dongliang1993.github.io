---
title: react-router 整理
date: 2017-12-29 14:15:54
tags:
---
原文 http://www.zcfy.cc/article/react-router-v4-the-complete-guide-mdash-sitepoint-4448.html

<!-- more -->

## 安装 React Router

React Router 库包含三个包：react-router, react-router-dom, 和 react-router-native。react-router 是路由的核心包，而其他两个是基于特定环境的。如果你在开发一个网站，你应该使用 react-router-dom，如果你在移动应用的开发环境使用 React Native，你应该使用 react-router-native。

使用 npm 安装 react-router-dom：
```js
npm install --save react-router-dom
```

## React Router 基础

下面是路由的例子：
```js
<Router>
  <Route exact path="/" component={Home}/>
  <Route path="/category" component={Category}/>
  <Route path="/login" component={Login}/>
  <Route path="/products" component={Products}/>
</Router>
```

### Router

像上面的例子，你需要一个 <Router> 组件和一些 <Route> 组件来创建一个基本的路由。由于我们创建的是一个基于浏览器的应用，我们可以从 React Router API 中使用这两种类型的路由：
1. <BrowserRouter>
2. <HashRouter>
它们之间主要的区别，可以在它们所创建的 URL 明显看出：
```js
// <BrowserRouter>
http://example.com/about

// <HashRouter>
http://example.com/#/about
```

<BrowserRouter> 在两者中更为常用，原因是它使用了 HTML5 的 history API 来记录你的路由历史。而 <HashRouter> 则使用 URL(window.location.hash) 的 hash 部分来记录。如果你想兼容老式浏览器，你应该使用 <HashRouter>。

### 使用 <BrowserRouter> 组件包裹 App 组件。

index.js
```js
/* Import statements */
import React from 'react'
import ReactDOM from 'react-dom'

/* App is the entry point to the React code.*/
import App from './App'

/* import BrowserRouter from 'react-router-dom' */
import {
  BrowserRouter as Router
} from 'react-router-dom'

ReactDOM.render(
  <Router>
    <App />
  </Router>,
  document.getElementById('root')
)
```

注意：

1. Router 组件只能有一个子元素。子元素可以是 HTML - 例如 div - 也可以是一个 react 组件。
2. <App /> 其实是个根组件，组件想要根据对应的 url 来正确渲染必须要包裹在 Router 组件里面

要让 React Router 工作，你需要从 react-router-dom 库引入相关的 API。这里，我在 index.js 引入了 BrowserRouter，也从 App.js 引入了 App 组件。App.js，如你所猜想的，是 React 组件的入口。

上述代码给我们整个 App 组件创建了一个 history 实例。接下来正式介绍下 history。

### history

history 是一个让你轻松管理所有 Javascript 运行的会话记录的 Javascript 库。history 提供了简洁的 API，让你可以管理 history 堆栈，跳转，确认跳转，以及保持会话之间的状态。 - 来自 React 培训文档

每个 router 组件创建了一个 history 对象，用来记录当前路径 (history.location)，上一步路径也存储在堆栈中。当前路径改变时，视图会重新渲染，给你一种跳转的感觉。当前路径又是如何改变的呢？history 对象有 history.push() 和 history.replace() 这些方法来实现。当你点击 <Link> 组件会触发 history.push()，使用<Redirect> 则会调用 history.replace()。其他方法 - 例如 history.goBack() 和 history.goForward() - 用来根据页面的后退和前进来跳转 history 堆栈。

接下来，我们谈谈 Links 和 Routes

### Links and Routes

<Route> 是 React Router 里最重要的组件。若当前路径匹配 route 的路径，它会渲染对应的 UI。理想来说，<Route> 应该有一个叫 path 的 prop，当路径名跟当前路径匹配才会渲染。

另一方面，<Link> 用来跳转页面。可以类比 HTML 的锚元素。然而，使用锚链接会导致浏览器的刷新，这不是我们想要的。所以，我们可以使用<Link> 来跳转至具体的 URL，并且视图重新渲染不会导致浏览器刷新。

我们已经介绍了创建一个基本的路由需要的所有东西。让我们试一个吧。

Demo 1: 基础路由

src/App.js
```js
/* Import statements */
import React, { Component } from 'react'
import {
  Link,
  Route,
  Switch
} from 'react-router-dom'

/* Home component */
const Home = () => (
  <div>
    <h2>Home</h2>
  </div>
)

/* Category component */
const Category = () => (
  <div>
    <h2>Category</h2>
  </div>
)

/* Products component */
const Products = () => (
  <div>
    <h2>Products</h2>
  </div>
)

/* App component */
class App extends React.Component {
  render() {
    return (
      <div>
        <ul className="nav navbar-nav">
          // Link 组件通常被用来链接跳转到其他的页面
          <li><Link to="/">Homes</Link></li>
          <li><Link to="/category">Category</Link></li>
          <li><Link to="/products">Products</Link></li>
        </ul>
        <hr />
        // 如果 Route 组件的 path 属性和当前的 URL 匹配
        // 那么就会渲染对应的 component
        // 这里还有一些问题，下面会说到
        <Route path="/" component={Home}/>
        <Route path="/category" component={Category}/>
        <Route path="/products" component={Products}/>
      </div>
    )
  }
}
```

我们在 App.js 里定义了 Home，Category，和 Products 组件。尽管目前看起来没问题，当组件变得越来越臃肿，最好将每个组件分成单独的文件。根据经验，如果组件代码超过了 10 行，我通常会给它创建一个新的文件。从第二个 demo 开始，我会将 App.js 里面越来越多的组件分成单独的文件。

在 App 组件中，我们写了路由跳转的逻辑。 <Route> 的路径与当前路径匹配，对应组件就会被渲染。对应渲染的组件传给了第二个 prop--component。

在这里，/ 同时匹配 /, /category 和 /products, 因此，所有路由都匹配并被渲染。我们该如何避免呢？应该给 path='/'的路由传递 exact= {true}props：
```js
<Route exact={true} path="/" component={Home}/>
```

若只想要路由在路径完全相同时渲染，你就可以使用 exact props。

### 嵌套路由

创建嵌套路由之前，我们需要更深入的理解 <Route> 如何运行。开始吧。

<Route> 有三个可以用来定义要渲染内容的 props：

1. component. 在上面我们已经看到了。当 URL 匹配时，router 会将传递的组件使用 React.createElement 来生成一个 React 元素。
2. render. 适合行内渲染。在当前路径匹配路由路径时，render prop 期望一个函数返回一个元素。
3. children. children prop 跟 render 很类似，也期望一个函数返回一个 React 元素。然而，不管路径是否匹配，children 都会渲染。

### Path and match

path 用来标识路由匹配的 URL 部分。React Router 使用了 Path-to-RegExp 库将路径字符串转为正则表达式。然后正则表达式会匹配当前路径。

当路由路径和当前路径成功匹配，会生成一个对象，我们叫它 match。match 对象有更多关于 URL 和 path 的信息。这些信息可以通过它的属性获取，如下所示：

1. match.url. 返回 URL 匹配部分的字符串。对于创建嵌套的 <Link>很有用。
2. match.path. 返回路由路径字符串 - 就是 <Route path="">。将用来创建嵌套的<Route>。
3. match.isExact. 返回布尔值，如果准确（没有任何多余字符）匹配则返回 true。
4. match.params. 返回一个对象包含 Path-to-RegExp 包从 URL 解析的键值对。

现在我们完全了解了<Route>，开始创建一个嵌套路由吧。

### Switch 组件

在我们开始示例代码签，我想给你介绍下<Switch>组件。当一起使用多个<Route>时，所有匹配的 routes 都会被渲染。根据 demo1 的代码，我添加一个新的 route 来验证为什么 <Switch> 很有用。
```js
<Route exact path="/" component={Home}/>
<Route path="/products" component={Products}/>
<Route path="/category" component={Category}/>
<Route path="/:id" render = {()=> (<p> I want this text to show up for all routes other than '/', '/products' and '/category' </p>)}/>
```

当 URL 为 /products，所有匹配 /products 路径的 route 都会被渲染。所以，那个 path 为：id 的 <Route> 会跟着 Products 组件一块渲染。设计就是如此。但是，若这不是你想要的结果，你应该给你的 routes 添加 <Switch> 组件。有 <Switch> 组件的话，只有第一个匹配路径的子< Route> 会渲染。

### Demo 2: 嵌套路由

之前，我们给 /, /category and /products 创建了路由。但如果我们想要 /category/shoes 这种形式的 URL 呢？

不像 React Router 之前的版本，在版本 4 中，嵌套的 <Route> 最好放在父元素里面。所以，Category 组件就是这里的父组件，我们将在父组件中定义 category/:name 路由。
```js
// src/Category.jsx
import React from 'react'
import {
  Link,
  Route,
  Switch
} from 'react-router-dom'

const Category = ({ match }) => {
  return (
    <div>
      <ul>
        <li>
          <Link to={`${match.url}/shoes`}>Shoes</Link>
        </li>
        <li>
          <Link to={`${match.url}/boots`}>Boots</Link>
        </li>
        <li>
          <Link to={`${match.url}/footwear`}>Footwear</Link>
        </li>
      </ul>
      <Route path={`${match.path}/:name`} render= {({match}) =>( <div> <h3> {match.params.name} </h3></div>)}/>
    </div>
  )
}
export default Category;
```

首先，我们给嵌套路由定义了一些 Link。之前提到过，match.url 用来构建嵌套链接，match.path 用来构建嵌套路由。如果你对 match 有不理解的概念，console.log(match) 会提供一些有用的信息来帮助你了解它。
```js
<Route path={`${match.path}/:name`}
  render= {({match}) =>( <div> <h3> {match.params.name} </h3></div>)}/>
```

这是我们首次尝试动态路由。不同于硬编码路由，我们给 pathname 使用了变量。:name 是路径参数，获取 category/ 之后到下一条斜杠之间的所有内容。所以，类似 products/running-shoes 的路径名会生成如下的一个 params 对象：
```js
{
  name: 'running-shoes'
}
```

参数可以通过 match.params 或 props.match.params 来获取，取决于传递哪种 props。另外有趣的是我们使用了 renderprop。render props 非常适合行内函数，这样不需要单独拆分组件。

Demo 3: 带 Path 参数的嵌套路由

我们让事情变得再复杂一些，可以吗？一个真实的路由应该是根据数据，然后动态展示。假设我们获取了从服务端 API 返回的 product 数据，如下所示。
```js
// src/Products.jsx
const productData = [
  {
    id: 1,
    name: 'NIKE Liteforce Blue Sneakers',
    description: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Proin molestie.',
    status: 'Available'

  },
  {
    id: 2,
    name: 'Stylised Flip Flops and Slippers',
    description: 'Mauris finibus, massa eu tempor volutpat, magna dolor euismod dolor.',
    status: 'Out of Stock'

  },
  {
    id: 3,
    name: 'ADIDAS Adispree Running Shoes',
    description: 'Maecenas condimentum porttitor auctor. Maecenas viverra fringilla felis, eu pretium.',
    status: 'Available'
  },
  {
    id: 4,
    name: 'ADIDAS Mid Sneakers',
    description: 'Ut hendrerit venenatis lacus, vel lacinia ipsum fermentum vel. Cras.',
    status: 'Out of Stock'
  },
]
```

我们需要根据下面这些路径创建路由：

1. /products. 这个路径应该展示产品列表。
2. /products/:productId. 如果产品有 :productId，这个页面应该展示该产品的数据，如果没有，就该展示一个错误信息。
```js
src/Products.jsx
/* Import statements have been left out for code brevity */
const Products = ({ match }) => {
   const productsData = [
      {
          id: 1,
          name: 'NIKE Liteforce Blue Sneakers',
          description: 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. Proin molestie.',
          status: 'Available'

      },
      //Rest of the data has been left out for code brevity
  ]
  const linkList = productsData.map(product => {
    return (
      <li>
        <Link to={`${match.url}/${product.id}`}>
          {product.name}
        </Link>
      </li>
    )
  })
  return(
    <div>
      <div>
        <h3> Products</h3>
        <ul> {linkList} </ul>
      </div>
      <Route path={`${match.url}/:productId`}
          render={ (props) => <Product data= {productsData} {...props} />}/>
      <Route exact path={match.url}
          render={() => (
          <div>Please select a product.</div>
          )}
      />
    </div>
  )
}
```

首先，我们通过 productsData.id 创建一列 <Links>，并把它存储在 linkList。路由从路径字符串根据匹配的对应产品 id 获取参数。
```js
<Route path={`${match.url}/:productId`}
  render={ (props) => <Product data= {productsData} {...props} />}/>
```

你可能期望使用 component = { Product } 来替代行内 render 函数。问题是，我们不仅需要 productsData，并顺带把剩余 prop 也传给 Product 组件。尽管你还有其他方法，不过我觉的这是最简单的方法了。{...props} 使用 ES6 的扩展运算符 将所有 prop 传给组件。

这是 Product 组件的代码。
```js
/* Import statements have been left out for code brevity */

const Product = ({match, data}) => {
  const product= data.find(p => p.id == match.params.productId)
  let productData
  if (product) {
    productData = <div>
                    <h3> {product.name} </h3>
                    <p>{product.description}</p>
                    <hr/>
                    <h4>{product.status}</h4>
                  </div>
  } else {
    productData = <h2> Sorry. Product doesnt exist </h2>
  }
  return (
    <div>
      <div>
         {productData}
      </div>
    </div>
  )
```

find 方法用来查找数组中对象的 id 属性等于 match.params.productId。如果 product 存在，productData 就会展示，如果不存在，“Product 不存在”的信息就会被渲染。

### 保护式路由

最后一个 demo，我们将围绕保护式路由的技术进行讨论。那么，如果有人想进入 /admin 页面，他们会被要求先登录。然而，在我们保护路由之前还需要考虑一些事情。

### 重定向

类似服务端重定向，<Redirect> 会将 history 堆栈的当前路径替换为新路径。新路径通过 toprop 传递。这是如何使用<Redirect>：
```js
<Redirect to={{pathname: '/login', state: {from: props.location}}}
```

如果有人已经注销了账户，想进入 /admin 页面，他们会被重定向到 /login 页面。当前路径的信息是通过 state 传递的，若用户信息验证成功，用户会被重定向回初始路径。在子组件中，你可以通过 this.props.location.state 获取 state 的信息。

### 自定义路由

自定义路由最适合描述组件里嵌套的路由。如果我们需要确定一个路由是否应该渲染，最好的方法是写个自定义组件。下面是通过其他路由来定义自定义路由。
```js
// src/App.js
<Switch>
  <Route exact path="/" component={Home} data={data}/>
  <Route path="/category" component={Category}/>
  <Route path="/login" component={Login}/>
  <PrivateRoute authed={fakeAuth.isAuthenticated} path='/products' component = {Products} />
</Switch>
```

若用户已登录，fakeAuth.isAuthenticated 返回 true，反之亦然。

这是 PrivateRoute 的定义。
```js
// src/App.js
const PrivateRoute = ({component: Component, authed, ...rest}) => {
  return (
    <Route
      {...rest}
      render={(props) => authed === true
        ? <Component {...props} />
        : <Redirect to={{pathname: '/login', state: {from: props.location}}} />} />
  )
}
```

如果用户已登录，路由将渲染 Admin 组件。否则，用户将重定义到 /login 登录页面。这样做的好处是，定义更明确，而且 PrivateRoute 可以复用。

最后，下面是 Login 组件的代码：
```js
import React from 'react'
import { Redirect } from 'react-router-dom'

class Login extends React.Component {
  constructor() {
    super()
    this.state = {
      redirectToReferrer: false
    }
  }
  login = () => {
    fakeAuth.authenticate(() => {
      this.setState({ redirectToReferrer: true })
    })
  }
  render() {
    const { from } = this.props.location.state || { from: { pathname: '/' } }
    const { redirectToReferrer } = this.state
    if (redirectToReferrer) {
      return (
        <Redirect to={from} />
      )
    }
    return (
      <div>
        <p>You must log in to view the page at {from.pathname}</p>
        <button onClick={this.login}>Log in</button>
      </div>
    )
  }
}

/* A fake authentication function */
export const fakeAuth = {
  isAuthenticated: false,
  authenticate(cb) {
    this.isAuthenticated = true
     setTimeout(cb, 100)
  },
}
```
## 总结

如你在本文中所看到的，React Router是一个帮助React构建更完美，更声明式的路由库。不像React Router之前的版本，在v4中，一切就“只是组件”。而且，新的设计模式也更完美的使用React的构建方式来实现。

在本次教程中，我们学到了：

如何配置和安装React Router

基础版路由，和一些基础组件，例如<Router>, <Route>和<Link>

如何构建一个有导航功能的极简路由和嵌套路由

如何根据路径参数构建动态路由

最后，我们还学习了一些高级路由技巧，用来创建保护式路由的最终demo。