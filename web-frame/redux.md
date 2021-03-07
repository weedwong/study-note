# Redux 使用手册

Redux 是 JavaScript 应用程序的状态容器，提供可预测的状态管理。

## 核心API

1. Store 

存储数据的容器。一般一个应用只有一个Store. Redux 提供`createStore`这个函数，用来生成 Store。

```js
import { createStore } from 'redux';
const store = createStore(fn);
```

2. State 

`Store`对象包含所有数据。如果想得到某个时点的数据，就要对 Store 生成快照。这种时点的数据集合，就叫做 State。

当前时刻的 State，可以通过store.getState()拿到。

```js
import { createStore } from 'redux';
const store = createStore(fn);

const state = store.getState();
```

3. Action

`Action` 是一个描述状态变化的对象。其中的type属性是必须的，表示 Action 的名称。其他属性可以自由设置

```js
const action = {
  type: 'ADD_TODO',
  payload: 'Learn Redux'
};
```

4. Action 创建函数

View 要发送多少种消息，就会有多少种 Action。如果都手写，会很麻烦。可以定义一个函数来生成 Action，这个函数就叫 `Action Creator`。

```js
const ADD_TODO = '添加 TODO';

function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}

const action = addTodo('Learn Redux');
```

5. store.dispatch()

`store.dispatch()`是 View 发出 Action 的唯一方法。

```js
import { createStore } from 'redux';
const store = createStore(fn);

store.dispatch({
  type: 'ADD_TODO',
  payload: 'Learn Redux'
});
```

上面代码中，`store.dispatch`接受一个 Action 对象作为参数，将它发送出去。

结合 Action Creator，这段代码可以改写如下。

```js
store.dispatch(addTodo('Learn Redux'));
```

6. Reducer

> Reducers 指定了应用状态的变化如何响应 actions 并发送到 store 的，记住 actions 只是描述了有事情发生了这一事实，并没有描述应用如何更新 state。

Store 收到 Action 以后，必须给出一个新的 State，这样 View 才会发生变化。这种 State 的计算过程就叫做 `Reducer`。

`Reducer` 是一个函数，它接受 Action 和当前 State 作为参数，返回一个新的 State。

```js
const reducer = function (state, action) {
  // ...
  return new_state;
};
```

整个应用的初始状态，可以作为 State 的默认值。下面是一个实际的例子。

```js
const defaultState = 0;
const reducer = (state = defaultState, action) => {
  switch (action.type) {
    case 'ADD':
      return state + action.payload;
    default: 
      return state;
  }
};

const state = reducer(1, {
  type: 'ADD',
  payload: 2
});
```

上面代码中，`reducer` 函数收到名为 `ADD` 的 Action 以后，就返回一个新的 State，作为加法的计算结果。其他运算的逻辑（比如减法），也可以根据 Action 的不同来实现。

实际应用中，Reducer 函数不用像上面这样手动调用，`store.dispatch`方法会触发 Reducer 的自动执行。为此，Store 需要知道 Reducer 函数，做法就是在生成 Store 的时候，将 Reducer 传入 `createStore` 方法。

```js
import { createStore } from 'redux';
const store = createStore(reducer);
```

上面代码中，`createStore` 接受 Reducer 作为参数，生成一个新的 Store。以后每当 `store.dispatch` 发送过来一个新的 Action，就会自动调用 Reducer，得到新的 State。

> 为什么这个函数叫做 Reducer 呢？因为它可以作为数组的reduce方法的参数。请看下面的例子，一系列 Action 对象按照顺序作为一个数组。

```js
const actions = [
  { type: 'ADD', payload: 0 },
  { type: 'ADD', payload: 1 },
  { type: 'ADD', payload: 2 }
];

const total = actions.reduce(reducer, 0); // 3
```

上面代码中，数组 `actions` 表示依次有三个 Action，分别是加 `0` 、加 `1`和加 `2` 。数组的reduce方法接受 Reducer 函数作为参数，就可以直接得到最终的状态 `3`。

7. 纯函数

Reducer 函数最重要的特征是，它是一个纯函数。也就是说，只要是同样的输入，必定得到同样的输出。

纯函数是函数式编程的概念，必须遵守以下一些约束。

```md
- 不得改写参数
- 不能调用系统 I/O 的API
- 不能调用Date.now()或者Math.random()等不纯的方法，因为每次会得到不一样的结果
```

由于 Reducer 是纯函数，就可以保证同样的State，必定得到同样的 View。但也正因为这一点，Reducer 函数里面不能改变 State，必须返回一个全新的对象，请参考下面的写法。

```js
// State 是一个对象
function reducer(state, action) {
  return Object.assign({}, state, { thingToChange });
  // 或者
  return { ...state, ...newState };
}

// State 是一个数组
function reducer(state, action) {
  return [...state, newItem];
}
```

最好把 State 对象设成只读。你没法改变它，要得到新的 State，唯一办法就是生成一个新对象。这样的好处是，任何时候，与某个 View 对应的 State 总是一个不变的对象。

8. store.subscribe()

Store 允许使用 `store.subscribe` 方法设置监听函数，一旦 State 发生变化，就自动执行这个函数。

```js
import { createStore } from 'redux';
const store = createStore(reducer);

store.subscribe(listener);
```

`store.subscribe` 方法返回一个函数，调用这个函数就可以解除监听。

```js
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
);

unsubscribe();
```

## Store 的实现

我们知道通过 `createStore` 提供了三个方法: 

```md
store.getState()
store.dispatch()
store.subscribe()
```

```js
import { createStore } from 'redux';
let { subscribe, dispatch, getState } = createStore(reducer);
```

`createStore` 方法还可以接受第二个参数，表示 State 的最初状态。

下面是 `createStore` 方法的一个简单实现，可以了解一下 Store 是怎么生成的。

```js
const createStore = (reducer) => {
  let state;
  let listeners = [];

  const getState = () => state;

  const dispatch = (action) => {
    state = reducer(state, action);
    listeners.forEach(listener => listener());
  };

  const subscribe = (listener) => {
    listeners.push(listener);
    // 注销监听函数
    return () => {
      listeners = listeners.filter(l => l !== listener);
    }
  };

  dispatch({});

  return { getState, dispatch, subscribe };
};
```

## Reducer 的拆分

Reducer 函数负责生成 State。由于整个应用只有一个 State 对象，包含所有数据，对于大型应用来说，这个 State 必然十分庞大，导致 Reducer 函数也十分庞大。

```js
const chatReducer = (state = defaultState, action = {}) => {
  const { type, payload } = action;
  switch (type) {
    case ADD_CHAT:
      return Object.assign({}, state, {
        chatLog: state.chatLog.concat(payload)
      });
    case CHANGE_STATUS:
      return Object.assign({}, state, {
        statusMessage: payload
      });
    case CHANGE_USERNAME:
      return Object.assign({}, state, {
        userName: payload
      });
    default: return state;
  }
};
```

上面代码中，三种 Action 分别改变 State 的三个属性。这样我们就可以将 Reducer 函数被拆成三个小函数，每一个负责生成对应的属性。

```js
const chatReducer = (state = defaultState, action = {}) => {
  return {
    chatLog: chatLog(state.chatLog, action),
    statusMessage: statusMessage(state.statusMessage, action),
    userName: userName(state.userName, action)
  }
};
```

Redux 提供了一个combineReducers方法，用于 Reducer 的拆分。你只要定义各个子 Reducer 函数，然后用这个方法，将它们合成一个大的 Reducer。

```js
import { combineReducers } from 'redux';

const chatReducer = combineReducers({
  chatLog,
  statusMessage,
  userName
})

export default todoApp;
```

下面是combineReducer的简单实现。

```js
const combineReducers = reducers => {
  return (state = {}, action) => {
    return Object.keys(reducers).reduce(
      (nextState, key) => {
        nextState[key] = reducers[key](state[key], action);
        return nextState;
      },
      {} 
    );
  };
};
```

## 数据流

![数据流指示图](../public/images/redux_flow.jpg)

首先，用户发出 Action。

```js
store.dispatch(action);
```

然后，Store 自动调用 Reducer，并且传入两个参数：当前 State 和收到的 Action。 Reducer 会返回新的 State 。

```js
let nextState = todoApp(previousState, action);
```

State 一旦有变化，Store 就会调用监听函数。

```js
// 设置监听函数
store.subscribe(listener);
```

`listener`可以通过`store.getState()`得到当前状态。如果使用的是 React，这时可以触发重新渲染 View。

```js
function listerner() {
  let newState = store.getState();
  component.setState(newState);   
}
```

## 中间件概念

它提供的是位于 action 被发起之后，到达 reducer 之前的扩展点。

举例来说，要添加日志功能，把 Action 和 State 打印出来，可以对store.dispatch进行如下改造。

```js
let next = store.dispatch;
store.dispatch = function dispatchAndLog(action) {
  console.log('dispatching', action);
  next(action);
  console.log('next state', store.getState());
}
```

上面代码中，对store.dispatch进行了重定义，在发送 Action 前后添加了打印功能。这就是中间件的雏形。

## 中间件的用法

```js
import { applyMiddleware, createStore } from 'redux';
import createLogger from 'redux-logger';
const logger = createLogger();

const store = createStore(
  reducer,
  applyMiddleware(logger)
);
```

上面代码中，redux-logger提供一个生成器createLogger，可以生成日志中间件logger。然后，将它放在applyMiddleware方法之中，传入createStore方法，就完成了store.dispatch()的功能增强。

主要：

1. `createStore`方法可以接受整个应用的初始状态作为参数，那样的话，`applyMiddleware`就是第三个参数了。

```js
const store = createStore(
  reducer,
  initial_state,
  applyMiddleware(logger)
);
```
2. 中间件的次序有讲究。

```js
const store = createStore(
  reducer,
  applyMiddleware(thunk, promise, logger)
);
```

上面代码中，`applyMiddleware`方法的三个参数，就是三个中间件。有的中间件有次序要求，使用前要查一下文档。比如，`logger`就一定要放在最后，否则输出结果会不正确。

## applyMiddleware 实现

```js
export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    var store = createStore(reducer, preloadedState, enhancer);
    var dispatch = store.dispatch;
    var chain = [];

    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    };
    chain = middlewares.map(middleware => middleware(middlewareAPI));
    dispatch = compose(...chain)(store.dispatch);

    return {...store, dispatch}
  }
}
```

上面代码中，所有中间件被放进了一个数组chain，然后嵌套执行，最后执行store.dispatch。可以看到，中间件内部（middlewareAPI）可以拿到getState和dispatch这两个方法。

参考 

1. [Redux 入门教程（一）](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html)
2. [Redux 入门教程（二）](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_two_async_operations.html)
3. [Redux 入门教程（三）](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_three_react-redux.html)