# 运行方式
```
npm i -g create-react-app
cd $this_repo
npm i
npm start
```

# 概述
使用create-react-app作为脚手架，结合React+Redux+React-router，构建一个简单的单页面应用demo。

- create-react-app：脚手架
- react：负责页面组件构建
- react-router：负责单页应用路由部分的控制
- redux：负责管理整个应用的数据流
- react-redux：将react与redux这两部分相结合
- redux-thunk：redux的一个中间件。可以使action creator返回一个`function`（而不仅仅是`object`），并且使得dispatch方法可以接收一个`function`作为参数，通过这种改造使得action支持异步（或延迟）操作
- redux-actions：针对redux的一个FSA工具箱，可以相应简化与标准化action与reducer部分

# 使用create-react-app脚手架
[create-react-app](https://github.com/facebookincubator/create-react-app)是Facebook官方出品的脚手架。有了它，你只需要一行指令即可跳过webpack繁琐的配置、npm繁多的引入等过程，迅速构建react项目。

首先安装create-react-app

```
npm i -g create-react-app
```
安装完成后，就可以使用`create-react-app`指令快速创建一个基于webpack的react应用程序

```
cd $your_dir
create-react-app react-redux-demo
```
这时你可以进入`react-redux-demo`这个目录，运行`npm start`既可启动该应用。

打开访问`localhost:3000`看到下方对应的页面，就说明项目基础框架创建完毕了。
![启动页面](http://upload-images.jianshu.io/upload_images/6476654-ceeeda7ba166a8fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 创建React组件
## 修改目录结构
下面在我们的react-redux-demo项目，查看一下相应的目录结构
```
|--public
    |--index.html
    |-- ……
|--src
    |--App.js
    |--index.js
    |-- ……
|--node_modules
```
其中`public`中存放的内容不会被webpack编译，所以可以放一些静态页面或图片；`src`中存放的内容才会被webpack打包编译，我们主要工作的目录就是在`src`下。

了解react的同学肯定知道，在react中我们通过构建各种`react component`来实现一个新的世界。在我们的项目里，会基于此，将组件分为通用组件部分与页面组件部分。通用组件也就是我们普遍意义上的组件，一些大型项目会维护一个自己的组件库，其中的组件会被整个项目共享；页面组件实际上就是我们项目中所呈现出来的各个页面。因此，我们的目录会变成这样
```
|--public
      |--index.html
      |-- ……
|--src
    |--page
         |--welcome.js
         |--goods.js
    |--component
         |--nav
             |--index.js
             |--index.css
    |--App.js
    |--index.js
    |-- ……
|--node_modules
```
在`src`目录下新建了`page`和`component`两个目录分别用于存放页面组件和通用组件。页面组件包括`welcome.js`和商品列表页`good.js`，通用组件包括了一个导航栏`nav`。

## 两种组件形式
编写页面或组件，类似于静态页的开发。推荐的组件写法有两种：

**1）纯函数形式**：该类组件为无状态组件。由于使用函数来定义，因此不能访问`this`对象，同时也没有生命周期方法，只能访问`props`。这类组件主要是一些纯展示类的小组件，通过将这些小组件进行组合构成更为复杂的组件。例如：

```
const Title = props => (
    <h1>
        {props.title} - {props.subtitle}
    </h1>
)
```

**2）es6形式的组件**：该类组件一般为复杂的或有状态组件。使用es6的class语法进行创建。需要注意的是，在页面/组件中使用`this`注意其指向，必要时需要绑定。绑定方法可以使用`bind`函数或箭头函数。创建方式如下：

```
class Title extends Component {
    constructor(props) {
        super(props);
        this.state = {
            shown: true
        };
    }
    
    render() {
        let style = {
            display: this.state.shown ? 'block' : none
        };
        return (
            <h1 style={style}>
                {props.title} - {props.subtitle}
            </h1>
        );
    }
}
```
下面是这两种组件之间的对比：

   | Presentational Components | Container Components
---|---|---
Purpose | How things look (markup, styles) | How things work (data fetching, state updates)
Aware of Redux | No | Yes
To read data | Read data from props | Subscribe to Redux state
To change data | Invoke callbacks from props | Dispatch Redux actions
Are written | By hand | Usually generated by React Redux

鉴于上面的分析，我们可以将导航栏`nav`编写为无状态组件，而`page`中的部分使用有状态的组件。

导航栏组件`nav`
```
// component/nav/index.css
.nav {
    margin: 30px;
    padding: 0;
}
.nav li {
    border-left: 5px solid sandybrown;
    margin: 15px 0;
    padding: 6px 0;
    color: #333;
    list-style: none;
    background: #bbb;
}

// component/nav/index.js
import React from 'react';
import './index.css';

const Nav = props => (
    <ul className="nav">
        {
            props.list.map((ele, idx) => (
                <li key={idx}>{ele.text}</li>
            ))
        }
    </ul>
);

export default Nav;
```
修改后的`App.js`与`App.css`
```
// App.css
.App {
    text-align: center;
}
.App::after {
    clear: both;
}
.nav_bar {
    float: left;
    width: 300px;
}
.conent {
    margin-left: 300px;
    padding: 30px;
}

// App.js
import React, { Component } from 'react';
import Nav from './component/nav';
import Welcome from './page/welcome';
import Goods from './page/goods';
import './App.css';

const LIST = [{
    text: 'welcome',
    url: '/welcome'
}, {
    text: 'goods',
    url: '/goods'
}];

const GOODS = [{
    name: 'iPhone 7',
    price: '6,888',
    amount: 37
}, {
    name: 'iPad',
    price: '3,488',
    amount: 82
}, {
    name: 'MacBook Pro',
    price: '11,888',
    amount: 15
}];

class App extends Component {
    render() {
        return (
            <div className="App">
                <div className="nav_bar">
                    <Nav list={LIST} />
                </div>
                <div className="conent">
                    <Welcome />
                    <Goods list={GOODS} />
                </div>
            </div>
        );
    }
}

export default App;
```
welcome页面
```
// page/welcome.js
import React from 'react';

const Welcome = props => (
    <h1>Welcome!</h1>
);

export default Welcome;
```
goods页面
```
// page/goods.js
import React, { Component } from 'react';

class Goods extends Component {
    render() {
        return (
            <ul className="goods">
                {
                    this.props.list.map((ele, idx) => (
                        <li key={idx} style={{marginBottom: 20, listStyle: 'none'}}>
                            <span>{ele.name}</span> | 
                            <span>￥ {ele.price}</span> | 
                            <span>剩余 {ele.amount} 件</span>
                        </li>
                    ))
                }
            </ul>
        );
    }
}

export default Goods;
```
现在我们的页面是这样的

![](http://upload-images.jianshu.io/upload_images/6476654-48a1dac919bf9f16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 使用redux来管理数据流

![redux数据流示意图](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016091802.jpg)
redux是flux架构的一种实现。图中展示了，在react+redux框架下，一个点击事件是如何进行交互的。

然而redux并不是完全依附于react的框架，实际上redux是可以和任何UI层框架相结合的。因此，为了更好得结合redux与react，对redux-flow中的`store`有一个更好的全局性管理，我们还需要使用`react-redux`。
```
npm i --save redux
npm i --save react-redux
```
同时，为了更好地创建action和reducer，我们还会在项目中引入`redux-actions`：一个针对redux的一个FSA工具箱，可以相应简化与标准化action与reducer部分。当然，这是可选的
```
npm i --save redux-actions
```
下面我们会以goods页面为例，实现以下场景：goods页面组件渲染完成后，发送请求，获取商品列表。其中获取数据的方法会使用mock数据。

为了实现这些功能，我们需要进一步调整目录结构
```
|--public
      |--index.html
      |-- ……
|--src
    |--page
         |--welcome.js
         |--goods.js
    |--component
         |--nav
             |--index.js
             |--index.css
    |--action
         |--goods.js
    |--reducer
         |--goods.js
         |--index.js
    |--App.js
    |--index.js
    |-- ……
|--node_modules
```

## 首先，创建action
首先，我们要创建对应的action。

action是一个`object`类型，对于action的结构有Flux有相关的标准化建议[FSA](https://github.com/acdlite/flux-standard-action)
一个action必须要包含`type`属性，同时它还有三个可选属性`error`、`payload`和`meta`。
- type属性相当于是action的标识，通过它可以区分不同的action，其类型只能是字符串常量或`Symbol`。
- payload属性是可选的，可以使任何类型。payload可以用来装载数据；在error为true的时候，payload一般是用来装载错误信息。
- error属性是可选的，一般当出现错误时其值为true；如果是其他值，不被理解为出现错误。
- meta属性可以使任何类型，它一般会包括一些不适合在payload中放置的数据。

我们可以创建一个获取goods信息的action：

```
// action/goods.js
const getGoods = goods => {
    return {
        type: 'GET_GOODS',
        payload: goods
    };
}
```
这样，我们就可以得到`GET_GOODS`这个action。

在项目中，使用redux-actions对actions进行创建与管理：

```
createAction(type, payloadCreator = Identity, ?metaCreator)
```
`createAction`相当于对action创建器的一个包装，会返回一个FSA，使用这个返回的FSA可以创建具体的action。

`payloadCreator`是一个`function`，处理并返回需要的payload；如果空缺，会使用默认方法。如果传入一个`Error`对象则会自动将action的error属性设为`true`：

```
example = createAction('EXAMLE', data => data);
// 和下面的使用效果一样
example = createAction('EXAMLE');
```
因此上面的方式可以改写为：

```
// action/goods.js
import {createAction} from 'redux-actions';
export const getGoods = createAction('GET_GOODS'); 
```
\* 此外，还可以使用`createActions`同时创建多个action creators。

## 其次，创建state的处理方法——reducer
针对不同的action，会有不同的reducer对应进行state处理，它们通过type的值相互对应。
reducer是一个处理state的方法（function），该方法接收两个参数，当前状态`state`和对应的`action`。根据`state`与`action`，reducer会进行处理并返回一个新的`state`（同时也是一个新的`object`，而不去修改原`state`）。可以通过简单的switch操作来实现：

```
// reducer/goods.js
const goods = (state, action) => {
    switch (action.type) {
        case 'GET_GOODS':
            return {
                ...state,
                data: action.payload
            };
        // 其他action处理……
    }
}
```
对应`createAction`，`redux-actions`也有相应的reducer方式：

```
handleAction(type, reducer | reducerMap = Identity, defaultState)
```
`type`可以是字符串，也可以是`createAction`返回的action创建器：

```
handleAction('GET_GOODS', {
    next(state, action) {...},
    throw(state, action) {...}
}, defaultState);

//或者可以是
handleAction(getGoods, {
    next(state, action) {...},
    throw(state, action) {...}
}, defaultState);
```
此外，有时候一些操作的一系列action可以在语义和业务逻辑上是有一定联系的，我们希望将他们放在一起便于维护。可以通过`handleActions`方法将多个相关的reducer写在一起，以便于后期维护：

```
handleActions(reducerMap, defaultState)
```
因此，我们使用`redux-actions`来改写我们之前写的reducer

```
// reducer/goods.js
import {handleActions} from 'redux-actions';

export const goods = handleActions({
    GET_GOODS: (state, action) => ({
        ...state,
        data: action.payload
    })
}, {
    data: []
});
```

## 然后，对reducer进行合并
因为在redux中会统一管理一个store，因此，需要将不用的reducer所处理的state进行合并。

redux为我们提供了`combineReducers`方法。当业务逻辑过多时，我们可以将多个reducer进行组合，生成一个统一的reducer。虽然现在我们只有一个reducer，但是为了拓展性和示范性，在这里还是创建了一个`reducer/index.js`文件来进行reducer的合并，生成一个`rootReducer`。

```
// reducer/index.js
import {combineReducers} from 'redux';
import {goods} from './goods';

export const rootReducer = combineReducers({
    goods
});
```

### 之后，将页面组件与数据流相结合
上面的部分已经将redux中的action与reducer创建完毕了，然而，现在的数据流和我们的组件仍然是处于分离状态的，我们需要让全局的`state`，即`store`，的变化能够驱动页面组件的变化，才能完成redux-flow中的最后一环。这就需要将`store`中的各部分`state`映射到组件的`props`上。

解决这个问题就要用到我们之前提到的`react-redux`工具了。

首先，我们需要基于`rootReducer`创建一个全局的`store`。在`src`目录下新建一个`store.js`文件，调用redux的`createStore`方法：
```
// store.js
import {createStore} from 'redux';
import {rootReducer} from './reducer';
export const store = createStore(rootReducer);
```

然后，我们需要让所有的组件都能访问到`store`。最简单的方式就是使用react-redux
提供的`Provider`对整个应用进行包装。这样就可以使所有的子页面、子组件能访问到`store`。因此需要改写`index.js`：
```
// index.js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import {Provider} from 'react-redux';
import {store} from './store';

ReactDOM.render(
    <Provider store={store}>
        <App />
    </Provider>,
document.getElementById('root'));
```
最后，才是进行组件与状态的连接。将store中需要映射的部分connect到我们的组件上。使用其`connect`方法可以做到这一点：
```
connect(mapStateToProps)(component)；
```
redux中存在一个全局的store，其中存储了整个应用的状态，对其进行统一管理。`connect`可以将这个状态中的数据连接到页面组件上。其中，`mapStateToProps`是store中状态到该组件属性的一个映射方式，`component`是需要连接的页面组件。通过`connect`方法，一旦store发生变化，组件也就会相应更新。

我们需要修改原先`page/goods.js`

```
import React, { Component } from 'react';
import {connect} from 'react-redux';

class Goods extends Component {
    render() {
        return (
            <ul className="goods">
                {
                    this.props.list.map((ele, idx) => (
                        <li key={idx} style={{marginBottom: 20, listStyle: 'none'}}>
                            <span>{ele.name}</span> | 
                            <span>￥ {ele.price}</span> | 
                            <span>剩余 {ele.amount} 件</span>
                        </li>
                    ))
                }
            </ul>
        );
    }
}

const mapStateToProps = (state, ownProps) => ({
    goods: state.goods.data
});
// -export default Goods;
export default connect(mapStateToProps)(Goods);
```
此外，也可以为组件中相应的方法映射对应的action的触发：

```
const mapDispatchToProps = dispatch => ({
    onShownClick: () => dispatch($yourAction)
});
```

## 最后，在组件渲染完成后触发整个flow
如果产生了一个需要状态更新的交互，可以通过在组件中相应部分触发action来实现状态更新-->组件更新。触发方式：

```
dispatch($your_action)
```
`connect`后的组件，其`props`里会有一个`dispatch`的属性，就是个`dispatch`方法：

```
let dispatch = this.props.dispatch;
```
因此，最终的`page/goods.js`组件如下：

```
import React, { Component } from 'react';
import {connect} from 'react-redux';
import * as actions from '../action/goods';

const GOODS = [{
    name: 'iPhone 7',
    price: '6,888',
    amount: 37
}, {
    name: 'iPad',
    price: '3,488',
    amount: 82
}, {
    name: 'MacBook Pro',
    price: '11,888',
    amount: 15
}]; 

class Goods extends Component {
    componentDidMount() {
        let dispatch = this.props.dispatch;
        dispatch(actions.getGoods(GOODS));
    }
    render() {
        return (
            <ul className="goods">
                {
                    this.props.goods.map((ele, idx) => (
                        <li key={idx} style={{marginBottom: 20, listStyle: 'none'}}>
                            <span>{ele.name}</span> | 
                            <span>￥ {ele.price}</span> | 
                            <span>剩余 {ele.amount} 件</span>
                        </li>
                    ))
                }
            </ul>
        );
    }
}

const mapStateToProps = (state, ownProps) => ({
    goods: state.goods.data
});

export default connect(mapStateToProps)(Goods);
```
注意到，组件中数据不再是由`App.js`中写入的了，而是经过了完整的redux-flow的过程获取并渲染的。注意同时修改`App.js`
```
import React, { Component } from 'react';
import Nav from './component/nav';
import Welcome from './page/welcome';
import Goods from './page/goods';
import './App.css';

const LIST = [{
    text: 'welcome',
    url: '/'
}, {
    text: 'goods',
    url: '/goods'
}];

class App extends Component {
    render() {
        return (
            <div className="App">
                <div className="nav_bar">
                    <Nav list={LIST} />
                </div>
                <div className="conent">
                    <Welcome />
                    <Goods />
                </div>
            </div>
        );
    }
}

export default App;
```
现在访问页面，虽然效果和之前一致，但是其内部构造和原理已经大不相同了。

# 最后一部分：添加路由系统
单页应用中的重要部分，就是路由系统。由于不同普通的页面跳转刷新，因此单页应用会有一套自己的路由系统需要维护。

我们当然可以手写一个路由系统，但是，为了快速有效地创建于管理我们的应用，我们可以选择一个好用的路由系统。本文选择了react-router 4。这里需要注意，在v4版本里，react-router将WEB部分的路由系统拆分至了`react-router-dom`，因此需要npm`react-router-dom`
```
npm i --save react-router-dom
```
本例中我们使用react-router中的`BrowserRouter`组件包裹整个App应用，在其中使用`Route `组件用于匹配不同的路由时加载不同的页面组件。（也可以使用`HashRouter`，顾名思义，是使用hash来作为路径）react-router推荐使用`BrowserRouter`，`BrowserRouter`需要`history`相关的API支持。

首先，需要在`App.js`中添加`BrowserRouter`组件，并将`Route `组件放在`BrowserRouter`内。其中`Route `组件接收两个属性：`path`和`component`，分别是匹配的路径与加载渲染的组件
```
// App.js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import {Provider} from 'react-redux';
import {store} from './store';
import {BrowserRouter, Route} from 'react-router-dom';

ReactDOM.render(
    <Provider store={store}>
        <BrowserRouter>
            <Route path='/' component={App}/>
        </BrowserRouter>
    </Provider>,
document.getElementById('root'));
```
此时我们启动服务器的效果和之前一直。因为此时路由匹配到了`path='/'`，因此加载了`App`组件。

还记得我们在最开始部分创建的`Nav`导航栏组件么？现在，我们就要实现导航功能：点击对应的导航栏链接，右侧显示不同的区域内容。这需要改造`index.js`中的content部分：我们为其添加两个`Route `组件，分别在不同的路径下加载不同的页面组件（`welcome`与`goods`）
```
// index.js
import React, { Component } from 'react';
import Nav from './component/nav';
import Welcome from './page/welcome';
import Goods from './page/goods';
import './App.css';
import {Route} from 'react-router-dom';

const LIST = [{
    text: 'welcome',
    url: '/welcome'
}, {
    text: 'goods',
    url: '/goods'
}];

class App extends Component {
    render() {
        return (
            <div className="App">
                <div className="nav_bar">
                    <Nav list={LIST} />
                </div>
                <div className="conent">
                    <Route path='/welcome' component={Welcome} />
                    <Route path='/goods' component={Goods} />
                </div>
            </div>
        );
    }
}

export default App;
```
现在，可以尝试在地址栏输入`http://localhost:3000`、`http://localhost:3000/welcome`和`http://localhost:3000/goods`来查看效果。

当然，实际项目里不可能是通过手动修改地址栏来“跳转”页面。所以需要用到`Link`这个组件。通过其中的`to`这个属性来指明“跳转”的地址。这个`Link`组件我们会添加到`Nav`组件中
```
// component/nav/index.js
import React from 'react';
import './index.css';
import {Link} from 'react-router-dom';

const Nav = props => (
    <ul className="nav">
        {
            props.list.map((ele, idx) => (
                <Link to={ele.url} key={idx}>
                    <li>{ele.text}</li>
                </Link>
            ))
        }
    </ul>
);

export default Nav;
```
最终页面效果如下：

![最终效果图welcome页面](http://upload-images.jianshu.io/upload_images/6476654-2a910fb0b5ca7659.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![最终效果图goods页面](http://upload-images.jianshu.io/upload_images/6476654-c1e748c23f8d83cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在在这个demo里，我们点击左侧的导航，右侧内容发生变化，并且页面不会刷新。基于React+Redux+React-router，我们实现了一个最基础版的SPA（单页应用）。
