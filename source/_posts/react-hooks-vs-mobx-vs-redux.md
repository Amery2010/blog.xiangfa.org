---
title: React Hooks 时代的状态管理库的选择
author: 子丶言
date: 2020-05-26 18:06:00
tags: ['React', 'React Hooks', '状态管理', 'Mobx', 'Redux']
categories: ['React']
---

React 的数据流是自上而下的，从组件外到组件内，从父组件到子组件，且传递下来的 props 是只读的，如果你想更改 props，只能父组件传入一个封装好的 setState 方法。虽然你可以通过一些方案来解决 React 组件间的通信问题，但随着项目业务的增长，组件通信的成本会越来越高！这时候你可能希望有一处专门负责数据状态管理的地方，而这就是我们今天要提到的数据状态管理库的概念。

在 React 项目中常用的数据状态管理主要有 Redux 和 Mobx。而在早期，React 引入 Redux 需要使用大量的“胶水代码”，且遵循 setState 原则。而 Mobx 主张干掉 setState 的机制，它简化了使用成本，但确增加了“依赖收集”的新概念。这两个状态管理库各有优劣，多年相争不下。
<!-- more --> 

## React Hooks 时代

在进入到 React Hooks 时代，Mobx 率先推出了 [Mobx-react-lite](https://mobx-react.js.org/)，让隔壁的 Redux，完全没来得及反应。随着 React Hooks 日渐增长，Redux 也在后来推出了 [React Redux Hooks](https://react-redux.js.org/api/hooks)。~~开启了大航海时代！~~ React Hooks 时代的状态管理之争,由此进入白热化状态。

我将使用 React Hooks 的 useContext 和 useReducer 实现简易的 TodoList，并使用 React Redux Hooks 和 Mobx-react-lite 实现相同的功能。

### React Hooks 原生实现

#### 代码实现

```js
// reducer.js
export const initState = {
  todoList: {},
}

export const reducer = (state, action) => {
  const { payload } = action
  switch (action.type) {
    case 'ADD_TODOLIST': {
      state.todoList[payload.todo] = false
      return {
        todoList: { ...state.todoList },
      }
    }
    case 'TOGGLE_TODOLIST': {
      state.todoList[payload.todo] = !state.todoList[payload.todo]
      return {
        todoList: { ...state.todoList },
      }
    }
    default:
      return state
  }
}

export default {
  reducer,
  initState,
}
```

```jsx
// context.jsx
import React, { useReducer } from 'react'
import { reducer, initState } from './reducer'

const StoreContext = React.createContext(null)

export const useStore = () => {
  const store = React.useContext(StoreContext)
  if (!store) {
    // this is especially useful in TypeScript so you don't need to be checking for null all the time
    throw new Error('You have forgot to use StoreProvider, shame on you.')
  }
  return store
}

export function Provider({ children }) {
  const [state, dispatch] = useReducer(reducer, initState)

  return (
    <StoreContext.Provider value={{ state, dispatch }}>
      {children}
    </StoreContext.Provider>
  )
}
```

```jsx
// main.jsx
import React, { useState, useMemo } from 'react'
import { useStore } from './context'

function Main() {
  const { state, dispatch } = useStore()
  const [todoText, setTodoText] = useState('')
  
  const paddingTodos = useMemo(() => {
    return Object.keys(state.todoList).filter(
      todo => state.todoList[todo] === false
    )
  }, [state.todoList])
  const doneTodos = useMemo(() => {
    return Object.keys(state.todoList).filter(
      todo => state.todoList[todo] === true
    )
  }, [state.todoList])

  const addTodoList = () => {
    dispatch({
      type: 'ADD_TODOLIST',
      payload: { todo: todoText },
    })
    setTodoText('')
  }
  const toggleTodoList = (todo) => {
    dispatch({
      type: 'TOGGLE_TODOLIST',
      payload: { todo },
    })
  }

  return (
    <div>
      <input value={todoText} onChange={ev => setTodoText(ev.target.value)} />
      <button type="button" onClick={() => addTodoList(todoText)}>增加待办事项</button>
      <ul>
        {paddingTodos.map(todo => {
          return <li key={todo} onClick={() => toggleTodoList(todo)}>{todo}</li>
        })}
        {doneTodos.map(todo => {
          return <li key={todo} style={{ textDecoration: 'line-through' }} onClick={() => toggleTodoList(todo)}>{todo}</li>
        })}
      </ul>
    </div>
  )
}

export default Main
```

```jsx
// index.jsx
import React from 'react'
import { Provider } from './context'
import Main from './main'

function TodoList () {
  return (
    <Provider>
      <Main />
    </Provider>
  )
}

export default TodoList
```

#### 优势

原生的 React Hooks 可以实现简单的数据状态管理，你需要引入额外的依赖。这在小型的项目中非常有优势。

#### 劣势

虽然借助原生 React Hooks 可以实现简易的数据状态管理，但官方确只是把这种实现当成一种新的组件间通信的解决方案。主要原因在于数据在不同页面间如果需要同步的话，你需要在不同的页面里引入相同的 reducer.js，当你使用 React Router 之后，你会发现页面切换很容易造成数据的丢失，因此你不得不将 Context 移到组件的最上层，才能解决这类组件注销造成数据丢失的问题。

-----

### React Redux Hooks 实现

#### 安装依赖

```shell
npm install redux react-redux
// or
yarn add redux react-redux
```

#### 代码实现

```js
// reducer.js
const initState = {
  todoList: {},
}

const reducer = (state = initState, action) => {
  const { payload } = action
  switch (action.type) {
    case 'ADD_TODOLIST': {
      state.todoList[payload.todo] = false
      return {
        todoList: { ...state.todoList },
      }
    }
    case 'TOGGLE_TODOLIST': {
      state.todoList[payload.todo] = !state.todoList[payload.todo]
      return {
        todoList: { ...state.todoList },
      }
    }
    default:
      return state
  }
}

export default reducer
```

```jsx
// main.jsx
import React, { useState } from 'react'
import { useSelector, useDispatch } from 'react-redux'

function Main () {
  const [todoText, setTodoText] = useState('')
  const paddingTodos = useSelector(state => {
    return Object.keys(state.todoList).filter(
      todo => state.todoList[todo] === false
    )
  })
  const doneTodos = useSelector(state => {
    return Object.keys(state.todoList).filter(
      todo => state.todoList[todo] === true
    )
  })

  const dispatch = useDispatch()
  const addTodoList = () => {
    dispatch({
      type: 'ADD_TODOLIST',
      payload: { todo: todoText },
    })
    setTodoText('')
  }
  const toggleTodoList = (todo) => {
    dispatch({
      type: 'TOGGLE_TODOLIST',
      payload: { todo },
    })
  }

  return (
    <div>
      <input value={todoText} onChange={ev => setTodoText(ev.target.value)} />
      <button type="button" onClick={() => addTodoList(todoText)}>增加待办事项</button>
      <ul>
        {paddingTodos.map(todo => {
          return <li key={todo} onClick={() => toggleTodoList(todo)}>{todo}</li>
        })}
        {doneTodos.map(todo => {
          return <li key={todo} style={{ textDecoration: 'line-through' }} onClick={() => toggleTodoList(todo)}>{todo}</li>
        })}
      </ul>
    </div>
  );
}

export default Main
```

```jsx
// index.jsx
import React from 'react'
import { createStore } from 'redux'
import { Provider } from 'react-redux'
import Main from './main'
import reducer from './reducer'

const store = createStore(reducer)

function TodoList () {
  return (
    <Provider store={store}>
      <Main />
    </Provider>
  )
}

export default TodoList
```

#### 优势

引入 React Redux Hooks 之后，你会发现页面代码变得更加简洁了。相对于之前的 Redux 实现，React Redux Hooks 更接近于 React Hooks 的原生实现。

#### 劣势

React Redux Hooks 虽然精简了大部分的代码，但依然采用 React Hooks reducer 的实现，你在修改数据状态时需要注意返回全新的 state，不然数据状态可能会不变。 

-----

### Mobx-react-lite 实现

#### 安装依赖

```shell
npm install mobx mobx-react-lite
// or
yarn add mobx mobx-react-lite
```

#### 代码实现

```js
// store.js
import { observable } from 'mobx'

const store = observable({})

export function createStore() {
  return {
    todoList: store,
    get pendingTodos() {
      return Object.keys(this.todoList).filter(
        todo => this.todoList[todo] === false,
      )
    },
    get doneTodos() {
      return Object.keys(this.todoList).filter(
        todo => this.todoList[todo] === true,
      )
    },
    ADD_TODOLIST(todo) {
      this.todoList[todo] = false
    },
    TOGGLE_TODOLIST(todo) {
      this.todoList[todo] = !this.todoList[todo]
    }
  }
}
```

```jsx
// context.jsx
import React from 'react'
import { useLocalStore } from 'mobx-react-lite'
import { createStore } from './store'

const StoreContext = React.createContext(null)

export const useStore = () => {
  const store = React.useContext(StoreContext)
  if (!store) {
    // this is especially useful in TypeScript so you don't need to be checking for null all the time
    throw new Error('You have forgot to use StoreProvider, shame on you.')
  }
  return store
}

export function Provider({ children }) {
  const store = useLocalStore(createStore)

  return (
    <StoreContext.Provider value={store}>
      {children}
    </StoreContext.Provider>
  )
}
```

```jsx
// main.jsx
import React, { useState } from 'react'
import { observer } from 'mobx-react-lite'
import { useStore } from './context'

const Main = observer(() => {
  const store = useStore()
  const [todoText, setTodoText] = useState('')

  const addTodo = (todo) => {
    store.ADD_TODOLIST(todo)
    setTodoText('')
  }
  const toggleTodo = (todo) => {
    store.TOGGLE_TODOLIST(todo)
  }

  return (
    <div>
      <input value={todoText} onChange={ev => setTodoText(ev.target.value)} />
      <button type="button" onClick={() => addTodo(todoText)}>增加待办事项</button>
      <ul>
        {store.pendingTodos.map(todo => {
          return <li key={todo} onClick={() => toggleTodo(todo)}>{todo}</li>
        })}
        {store.doneTodos.map(todo => {
          return <li key={todo} style={{ textDecoration: 'line-through' }} onClick={() => toggleTodo(todo)}>{todo}</li>
        })}
      </ul>
    </div>
  )
})

export default Main
```

```jsx
// index.jsx
import React from 'react'
import { Provider } from './context'
import Main from './main'

function TodoList () {
  return (
    <Provider>
      <Main />
    </Provider>
  )
}

export default TodoList
```

#### 优势

Mobx 中 observer 的实现非常优雅，你可以观察一个对象，然后在需要使用使用到 store 的组件中监听变化（observer）就可以了。这样的写法和 Vue 比较接近。

#### 劣势

Mobx 的实现与 React 的数据不可变思想有些出入，以至于部分只用过 React 开发项目的人无法理解 Mobx 的实现机制。

-----


### 总结

不管是 React Hooks 原生实现，还是借助 Redux 或 Mobx 来实现数据状态管理都是完全可行的。Redux 和 Mobx 的数据状态管理库一哥位置的争夺仍会持续很长一段时间，React Hooks 也可能会继续推出更强大的官方实现方案。但你需要根据实际的业务需求以及你个人对这类框架实现机制的理解来选择最合适的实现。


### 参考资料

- [对 React 状态管理的理解及方案对比](https://github.com/sunyongjian/blog/issues/36)