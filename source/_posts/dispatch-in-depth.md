title: Dispatch in Depth
date: 2017-06-08 15:30:33
category: Redux
---

<img src="/images/redux-dispatch/landing.jpeg" alt="Landing" width="80%"/>

[Redux](http://cn.redux.js.org/index.html) 是 JavaScript 状态容器，提供可预测化的状态管理。它支持 React、Angular、Ember、jQuery 甚至纯 JavaScript。

## 知识准备

阅读本文时，你需要提前了解一下以下内容：

- [redux](http://cn.redux.js.org/index.html)
- [react-redux](https://github.com/reactjs/react-redux)
- [redux-thunk](https://github.com/gaearon/redux-thunk)

这三个库中都提供了 dispatch 方法。
本文以这三种不同的 dispatch 为突破口来了解 redux， react-redux 和 redux-thunk 的工作原理。

<!-- more -->

## 这篇文章对你是否会有帮助？

首先请阅读以下三段代码：

1. `./src/index.js`

{% codeblock lang:javascript %}
import { createStore } from 'redux'
import reducer from './someReducer'

let store = createStore(reducer)

function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}

store.dispatch(addTodo('Read the docs'))

{% endcodeblock %}

2 `./src/containers/LinkContainer.js`

{% codeblock lang:javascript %}
const mapDispatchToProps = (dispatch, ownProps) => ({
  onClick: () => {
    dispatch(setVisibilityFilter(ownProps.filter))
  }
})
{% endcodeblock %}

3 `./src/actions/index.js`

{% codeblock lang:javascript %}
export const getAllProducts = () => dispatch => {
  // Call API to retrieve products data
  shop.getProducts(products => {
    dispatch(receiveProducts(products))
  })
}

{% endcodeblock %}

请注意这三段代码中都使用了 `dispatch`。现在请思考以下几个问题：

1. 以上三段代码中出现的 dispatch 分别是哪个库提供的？
2. 以上三种 dispatch 有什么不同？
3. 以上三段 dispatch 的使用，在 React 中分别适用于什么场景？

如果以上三个问题，你并不是全部明白，请继续；反之，你可以就此停止阅读这篇文章。

## 原生 Redux 中的 dispatch

在 Redux 中，对 dispatch 的介绍如下：

>Dispatches an action. This is the only way to trigger a state change.

>The store's reducing function will be called with the current getState() result and the given action synchronously. Its return value will be considered the next state. It will be returned from getState() from now on, and the change listeners will immediately be notified.

Redux 中的 [store](http://cn.redux.js.org/docs/basics/Store.html) 是一个对象：

1. 每个 Redux 应用中， store 只有一个
2. store 是用来维持 state 树的
3. Redux 中数据流是单向的，想要使 state 发生变化，只能通过调用 dispatch 方法来分发一个 action

在“纯正的” redux 中，想要改变 store 中的 state， 需要调用：

{% codeblock lang:javascript %}
store.dispath({
  type: 'TODO',
  text: 'New todo text'
})
{% endcodeblock %}

然后 store 接收到传入的 action 之后，会调用 reduce 函数，返回值作为新的 state, 至此，state 得到了更新。

### 在 React 中使用“纯正的” Redux 

在 React 中 使用 “纯正的” Redux，一共需要以下几个步骤：

1. 在组件中初始化 store
2. 调用 store.subscribe(reactRenderMethod) 将 React 组件的 render 方法注册为 listener（当调用 store.dispatch 分发了一个 action 之后，就会执行组件的 render 方法来重新渲染 UI ）
3. 在组件内调用 store.dispatch(actionCreator) 来改变 store 中的 state（比如：点击了 addTodo 按钮之后，调用 store.dispatch(addTodo)）
4. store.dispatch 执行之后，触发注册了的 listener，重新执行 render 方法
5. 在 render 组件时，调用 store.getState() 获取最新的 state 更新到 UI 上


### 局限 -- 用“纯正的” Redux 管理多个组件的状态 

在只有一个组件的场景下，使用上述“纯正的” Redux 不失为一个简单灵活的方案。

但是如果组件数量大于一个，并且也需要用 Redux 来管理状态时，怎么办呢？Redux 提供了[bindActionCreators](http://redux.js.org/docs/api/bindActionCreators.html) 方法来解决这种问题。

`bindActionCreators` 方法的 [源码](https://github.com/reactjs/redux/blob/master/src/bindActionCreators.js) 如下：

{% codeblock lang:javascript %}
function bindActionCreator(actionCreator, dispatch) {
  return (...args) => dispatch(actionCreator(...args))
}
{% endcodeblock %}

从源码中可以看出，`bindActionCreator`的返回值为一个 function ，这个 function 执行时，会调用 store 的 dispatch 来分发相应的 action creator。


所以，我们可以用 `bindActionCreators` 来修改一下 [redux counter example](https://github.com/reactjs/redux/blob/master/examples/counter) 中的 [index.js](https://github.com/reactjs/redux/blob/master/examples/counter/src/index.js) 样例代码：

Before:

{% codeblock lang:javascript %}
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore } from 'redux'
import Counter from './components/Counter'
import counter from './reducers'

const store = createStore(counter)
const rootEl = document.getElementById('root')

const render = () => ReactDOM.render(
  <Counter
    value={store.getState()}
    onIncrement={() => store.dispatch({ type: 'INCREMENT' })}
    onDecrement={() => store.dispatch({ type: 'DECREMENT' })}
  />,
  rootEl
)

render()
store.subscribe(render)
{% endcodeblock %}

After:

{% codeblock lang:javascript %}
import React from 'react'
import ReactDOM from 'react-dom'
import { createStore, bindActionCreators } from 'redux'
import Counter from './components/Counter'
import counter from './reducers'

// Define action creators
const increment = () => {
  return { type: 'INCREMENT' }
}

const decrement = () => {
  return { type: 'DECREMENT' }
}

// Create store
const store = createStore(counter)
const rootEl = document.getElementById('root')

// Use bindActionCreators to pass action creator wrapped into a dispatch call to a component
const boundIncrement = bindActionCreators({ increment }, store.dispatch)
const boundDecrement = bindActionCreators({ decrement }, store.dispatch)

const render = () => ReactDOM.render(
  <Counter
    value={store.getState()}
    onIncrement={() => boundIncrement.increment()}
    onDecrement={() => boundDecrement.decrement()}
  />,
  rootEl
)

render()
store.subscribe(render)
{% endcodeblock %}

以上的例子中，我们用 `bindActionCreactor` 把方法 `dispatch(actionCreator())` 往下传到了组件 Counter 上。但是，在实际项目中，组件的嵌套结构千变万化，比如请考虑下面这种情况：

1. 组件`Counter` 中有一个子组件 `A`
2. 组件`A` 中有一个子组件 `B`
3. 组件`A` 和 组件`B` 都需要用 Redux 做状态管理

此时用这种 “纯正”的 Redux 方案的话，此时我们需要把 store 传入 A 和 B 中，并且需要完成 Redux 中的 state 和 action creator 与组件之间的绑定。即以下两步：

1. 每次 store 中的状态发生改变之后，都要更新数据到 UI 上
2. 需要调用 store.dispatch 去使 store 中的数据发生改变。

为了解决这个问题，[react-redux](https://github.com/reactjs/react-redux) 便应运而生了。

## react-redux 中的 dispatch

回到们上面讨论的第二个代码段。（代码来自 [redux todos example](https://github.com/reactjs/redux/blob/master/examples/todos/src/containers/FilterLink.js) ）

{% codeblock lang:javascript %}
// FilterLink.js

import { connect } from 'react-redux'
import { setVisibilityFilter } from '../actions'
import Link from '../components/Link'

const mapStateToProps = (state, ownProps) => ({
  active: ownProps.filter === state.visibilityFilter
})

const mapDispatchToProps = (dispatch, ownProps) => ({
  onClick: () => {
    dispatch(setVisibilityFilter(ownProps.filter))
  }
})

const FilterLink = connect(
  mapStateToProps,
  mapDispatchToProps
)(Link)

export default FilterLink

{% endcodeblock %}

mapDispatchToProps 是一个入参包含了 dispatch 的 function ，因此想要了解 dispatch 是哪里来的，我们需要追踪一下 dispatch 在什么时机被谁调用了。所以我们需要把 connect 作为突破口，看看 mapDispatchToProps 在 connect 方法中进行了怎么样的处理。

### connect是怎样工作的？

在 [connect 方法的源码](https://github.com/reactjs/react-redux/blob/master/src/connect/connect.js) 中：

{% codeblock lang:javascript %}
// connect.js
import connectAdvanced from '../components/connectAdvanced'
import defaultSelectorFactory from './selectorFactory'

export function createConnect({connectHOC = connectAdvanced, ...otherProps, selectorFactory = defaultSelectorFactory}) {
	return function connect(mapStateToProps, mapDispatchToProps,{...props}) {
		const initMapDispatchToProps = match(mapDispatchToProps, mapDispatchToPropsFactories, 'mapDispatchToProps')
		// Blah blah...
		return connectHOC(selectorFactory, {
			initMapDispatchToProps，
			...otherProps
		})
	}
}

export default createConnect()

{% endcodeblock %}

可以看出，connect方法最终把 initMapDispatchProps 作为参数传入了 connectHOC 方法中，并将其返回。下面我们来看看 connectHOC 方法中，都做了些什么事情。

### connectAdvanced -- 绑定 state 和 action creator 到组件中

{% codeblock lang:javascript %}
// connectAdvanced.js
import hoistStatics from 'hoist-non-react-statics'

export default function connectAdvanced(selectorFactory, { initMapDispatchToProps, ...otherOptions }) {
	return function wrapWithConnect(WrappedComponent) {
		class Connect extends Component {
		  constructor(props, context) {
		    super(props, context)
		    this.store = props[storeKey] || context[storeKey]
		    // Blah blah...
		    this.initSelector()
			}
			
			const selectorFactoryOptions = { {initMapDispatchToProps, ...connectOptions}, ...otherOptions }
			
			initSelector() {
		    const sourceSelector = selectorFactory(this.store.dispatch, selectorFactoryOptions)
		    this.selector = makeSelectorStateful(sourceSelector, this.store)
		    this.selector.run(this.props)
		  }	
		}		
		return hoistStatics(Connect, WrappedComponent)
	}
}

{% endcodeblock %}

从上述代码中我们可以了解到，`connect` 调用之后，会返回一个 `function`，且此 `function`的入参为一个 `Component`。

最终返回为：

{% codeblock lang:javascript %}

hoistStatics(Connect, WrappedComponent)

{% endcodeblock %}

[hoist-non-react-statics](https://github.com/mridgway/hoist-non-react-statics) 这个库是用来： 

>Copies non-react specific statics from a child component to a parent component. Similar to Object.assign

所以当我们调用:

{% codeblock lang:javascript %}

connect(mapStateToProps, mapDispatchToProps)(SomeComponent)

{% endcodeblock %}

之后，最终的返回值是一个复制了 SomeComponent 属性的新的 Componenent，即为我们常说的: container  component。

而在构造这个 container  component 时，执行了 selectorFactory 方法，将 `this.store.dispatch` 和 initMapDispatchToProps 作为参数传入。

此时，selectorFactory 方法中便同时可以访问到 store.dispatch 和 `mapDispatchToProps`了！让我们一起了解一下，这个方法在做什么事情吧！

### selectorFactory -- mapping action creator 和 state 到组件的 props 中

从 [selectorFactory.js源码](https://github.com/reactjs/react-redux/blob/master/src/connect/selectorFactory.js) 中我们可以知道：

1. `selectorFactory` 的入参有两个：dispatch 和其他初始化属性（包括 mapDispatchToProps）
2. `selectorFactory` 会根据 `store.state` 计算出新的 `props`，并把所有的`actionCreator` 包在 dispatch 中，最终调用 `mergedProps` 把这些 props 合并后返回

现在我们明白了，connect 方法会返回一个新的 container  component，并且初始化这个 container  component 时，会用 this.store.dispatch 把需要绑定的 action creator 包起来，放在 container  component 的 props 中。

现在又有一个新问题了，我们在这个组件中使用了 `this.store.dispatch`。 但是此前只有在 Provider 组件中传入了 store，这个 Provider 的子组件是怎么能拿到 store 的呢？

在生成新的container component时，构造函数中是这样为 this.store赋值的：

{% codeblock lang:javascript %}

  this.store = props[storeKey] || context[storeKey]
  
{% endcodeblock %}

调试的截图如下：
<img src="/images/redux-dispatch/debug.png" alt="screenshot" width="80%"/>

可知此时的 store 来自于 `context`。

### context -- 传递 store 到组件树中

在应用的入口，我们将 store 传入了 react-redux 中的 Provider 组件中(代码来自 [redux todos example](https://github.com/reactjs/redux/blob/master/examples/todos/src/index.js) )。

{% codeblock lang:javascript %}
import React from 'react'
import { render } from 'react-dom'
import { createStore } from 'redux'
import { Provider } from 'react-redux'
import App from './components/App'
import reducer from './reducers'

const store = createStore(reducer)

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)

{% endcodeblock %}

在 [Provider的源码](https://github.com/reactjs/react-redux/blob/master/src/components/Provider.js) 中：

{% codeblock lang:javascript %}
class Provider extends Component {
  getChildContext() {
    return { [storeKey]: this[storeKey], [subscriptionKey]: null }
  }
  constructor(props, context) {
	 super(props, context)
	 this[storeKey] = props.store;
  }
  // Blah blah...
}
{% endcodeblock %}

Provider组件的构造函数中，将 `store` 放在了 [context](https://facebook.github.io/react/docs/context.html) 中，context 在 React 中的用法为：

> In some cases, you want to pass data through the component tree without having to pass the props down manually at every level. You can do this directly in React with the powerful "context" API.

因此，Provider 组件通过 context 把 store 传到了以它为根节点的所有子组件中。


### react-redux 的工作原理

通过以上分析，我们可以知道：

1. 在应用入口将 store 传入 Provider 组件 (用 Redux 做状态管理的组件树的根节点)
2. Provider 将 store 通过 context 传递给以它为根节点的组件树中的所有子组件
3. 以 Provider 组件为根节点的组件树中的子 container  component 是通过调用 connect 之后生成的。 调用 connect 方法后，`react-redux` 会根据 PresentationalComponent 和传入的 state & actionCreator 计算出新的 container  component。 在新的 container  component 中：从 context 中获取根节点组件传入的 store，包装传入的 actionCreator 为 store.dispatch(actionCreator) ，并将其绑定到组件的 props 中；将 state 绑定到组件的 props中；最终返回  container  component 。

## react-thunk 中的 dispatch

代码段一中使用的是：“纯正”的 Redux 。
代码段二中使用的是： react-redux 。
掌握了以上两种使用方法，我们就可以基本 handle 住大部分的 redux 应用了。

### 异步的 action creator

在实际项目中，异步操作的处理是很常见的，比如说：

1. 在某个页面读取文件时显示 loading 或者进度条，当文件读取成功后，展示文件。
2. 发送 API 请求，当请求成功之后，展示数据。请求失败之后，显示错误消息。

下面我们就用从代码段一和代码段二中学到的知识来实现下面的需求：

>根据 API server 返回的 products 数据渲染列表。

这个处理中一共包含两个操作：
1. 发送 API 请求来获取 products 数据。
2. 拿到 API 请求的返回值以后，dispatch action 来更新 store 中的 products 数据。

为了把数据处理和 UI 展示分离，我们把这两个操作都放在 action 中去处理：

{% codeblock lang:javascript %}
// ProductsContainer.js

function mapStateToProps(state) {
  return {
    products: state.products.list,
  }
}

const mapDispatchToProps = dispatch => ({
  getAllProducts,
  dispatch,
})

export default connect(mapStateToProps, mapDispatchToProps)(Products)

{% endcodeblock %}

{% codeblock lang:javascript %}
// Products.js
//...
  componentDidMount() {
    this.props.getAllProducts(this.props.dispatch)
  }
//...
{% endcodeblock %}

{% codeblock lang:javascript %}
// action.js
import shop from '../utils/apiHelper'

function receiveProducts(products) {
	return {
    type: 'PRODUCTS/RECEIVE_PRODUCTS',
    payload: products,
	}
}

export const getAllProducts(dispatch) {
	shop.getProducts(products => {
		dispatch(receiveProducts(products))
	})
}
{% endcodeblock %}

以上代码中，dispatch 总共出现了三次：

1. `action.js` 中，`getAllProducts` 方法的入参为 `dispatch`。
2. 在 `mapDispatchToProps` 方法中，把 `dispatch` 也传入了 `Products` 组件中。
3. 在 `Products` 组件在调用 `getAllProducts` 方法时，把 `this.props.dispatch` 作为参数传了进去。

在 “纯正的” Redux 和 “react-redux” 中，调用 dispatch 去分发一个 action creator 时，[action creator](http://redux.js.org/docs/Glossary.html#action-creator) 的返回值必须是一个 plain object。当 dispatch 某一个 action creator 时，react-redux 会默认帮我们执行 dispatch(actionCreator())，因此分发 action creator 时不用手动传入 dispatch 参数。

但是我们现在的 action creator 是一个包含了两个操作的 function。如果不手动传入 dispatch 的话, react-redux 就会在这个function 外面包上一个 dispatch 去分发 action，于是我们会得到错误：
`Action can only be plain object`

因此，我们可以类推出：action creator 中的异步操作将都会需要在以上三个地方加上关于 dispatch 参数传递的处理。如果你的应用足够简单，那么这样的做法就够用了。但是如果你的应用中存在很多的异步操作，恐怕就会出现大量的重复代码了。

那么怎样在 action creator 中处理异步的 action 呢？

异步的 action creator 并不会直接执行某一个函数并返回 plain action object ，而是需要在执行某个方法之后，才会返回 plain action object。

答案就是：[redux-thunk](https://github.com/gaearon/redux-thunk)

### 用 redux-thunk 处理异步 action creator

redux-thunk 是 Redux 中解决异步 action creator 的标准方案。代码段三就是使用的这种方案。

redux-thunk 是一个 [redux middleware](http://redux.js.org/docs/advanced/Middleware.html) 。 这里有一篇关于 redux middleware 的文章 [中间件与异步操作](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_two_async_operations.html)，大家感兴趣的话可以查看。


> It provides a third-party extension point between dispatching an action, and the moment it reaches the reducer. People use Redux middleware for logging, crash reporting, talking to an asynchronous API, routing, and more.


代码段三：（代码来自 [redux shopping-cart example](https://github.com/reactjs/redux/blob/master/examples/shopping-cart/src/actions/index.js) ）

{% codeblock lang:javascript %}
// ./src/actions/index.js

export const getAllProducts = () => dispatch => {
  // Call API to retrieve products data
  shop.getProducts(products => {
    dispatch(receiveProducts(products))
  })
}

{% endcodeblock %}

这段代码的功能是：

请求API，当获取了 products 数据之后，dispatch 一个 receiveProducts 的 action 。

在 Redux 中，action creator 的返回值只能是 `plain action object`， 但是当用了 redux-thunk 之后，我们就可以在 action creator 中返回一个 function 了。那么这个 function 是怎样执行的呢？

从 [redux-thunk 源码](https://github.com/gaearon/redux-thunk/blob/master/src/index.js) 来看：

{% codeblock lang:javascript %}
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
{% endcodeblock %}

当 dispatch(getAllProducts()) 时，middleware 会拿到下面的 action :

{% codeblock lang:javascript %}
function(dispatch) {
  shop.getProducts(products => {
    dispatch(receiveProducts(products))
  })
}
{% endcodeblock %}

当判断这个 action 的类型是 function 时，会把 store 中的 `dispatch`, `getState`, `extraArgument` 一起传给这个 function (此时 dispatch 被赋值为 `store.dispatch`)，然后执行它。所以当 API 请求成功之后，可以调用 `dispatch(receiveProducts(products))`。

middleware 的工作原理和如何写一个 middleware 并不是本文的重点，所以暂时不在此做详细介绍。

## 总结

读到这里，你现在知道文章开头提到的三个问题的答案了吗？

在 React 中使用 Redux 时，`react-redux` 和 `redux-thunk` 都是官方推荐的，所以我们经常会在项目中看到各式各样的 dispatch。在定义 `action creator` 或者 调用 `mapDispatchToProps` 时有的需要把 dispatch 传入 function 中，有的却可以省略。希望本文可以让大家 `redux`，`react-redux` 和 `redux-thunk` 有更好的理解。



