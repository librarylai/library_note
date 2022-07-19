# 【筆記】 Redux Middleware 一步一步帶你看!!!

###### tags: `筆記文章`

## Redux 介紹

> Redux is a predictable state container for JavaScript apps.

**簡單來說：Redux 是用來管理整個應用的狀態，且確保整個應用的資料都來自同一個地方(Store)，讓各元件的溝通更簡單。**

## Redux 三大原則

1. **Single source of truth (唯一資料來源)**
   整個應用程式的資料(State)，儲存在唯一的 store 裡面且是樹狀結構。
2. **State is read-only (State 是唯讀的)**
   改變 state 的唯一的方式是發出一個 action，透過 reducer 去改變 state。
3. **Changes are made with pure functions (用 pure functions 變更)**
   透過 action 來改變指定的 state tree，必須使用 pure function(reducer)。
   **請記得要回傳一個新的 state 物件，而不是去改變先前的 state**

```javascript
function todoReducer(state = initState, action) {
  switch (action.type) {
    case 'add':
      return {
        ...state,
        nowType: 'add',
      }
  }
}
```

![Redux Flow](https://note.pcwu.net/assets/images/2017-03-04-redux-intro-8c335.png)

## Redux Middleware

終於進入本篇的重點 Redux Middleware!!!

> It provides a third-party extension point between dispatching an action, and the moment it reaches the reducer.

Redux Middleware 主要是在 action 到達 reducer 中間對東西(資料)做一些額外處理。(ex. Log、回報當機、非同步 API 溝通、routing...)
![Redux Middleware](https://miro.medium.com/max/760/0*pjOSEDPecBxyfDro)
(Reference: 谷哥 — 詳解 Redux Middleware )

**Middleware 的基本架構:**

```javascript
/* ES6版本: */
const middleware = (store) => (next) => (action) => {
  return next(action)
}

/* ES5版本: */
function middleware(store) {
  return function (next) {
    return function (action) {
      return next(action)
    }
  }
}
```

**接下來將帶大家一步一步看 Middleware 架構是如何運作**

### 1. 第一步，封裝 Log 成一個 function，並將 store、action 當作參數傳入。

```javascript
function logAMiddleware(store, action) {
  console.log('logAMiddleware')
  store.dispatch(action)
}
function logBMiddleware(store, action) {
  console.log('logBMiddleware')
  store.dispatch(action)
}
// 呼叫
logAMiddleware(store, action)
logBMiddleware(store, action)
```

### 2. 第二步，將參數 store 提出

```javascript
let next = store.dispatch
function logAMiddleware(action) {
  console.log('logAMiddleware')
  next(action)
}
function logBMiddleware(action) {
  console.log('logBMiddleware')
  next(action)
}
```

### 3. 第三步，為了能串接多個 Middleware，所以 Middleware 要能夠取得當前的 dispatch 並回傳一個新的 dispatch 的函式

```javascript
function logAMiddlewareStore(store) {
  let next = store.dispatch
  store.dispatch = function logAMiddleware(action) {
    console.log('logAMiddleware')
    let result = next(action)
    return result
  }
}
function logBMiddlewareStore(store) {
  let next = store.dispatch
  store.dispatch = function logBMiddleware(action) {
    console.log('logBMiddleware')
    let result = next(action)
    return result
  }
}
// 呼叫
logAMiddlewareStore(store)
// next = store.dispatch
logBMiddlewareStore(store)
// next = logAMiddleware(action)
```

### 4. 第四步， 將 next = store.dispatch 封裝成函式當參數傳入，且 next 為前一個 middleware

```javascript
function logAMiddlewareStore(store) {
  return function logAMiddlewareNext(next) {
    return function logAMiddleware(action) {
      console.log('logAMiddleware')
      let result = next(action)
      return result
    }
  }
}
function logBMiddlewareStore(store) {
  return function logBMiddlewareNext(next) {
    return function logBMiddleware(action) {
      console.log('logBMiddleware')
      let result = next(action)
      return result
    }
  }
}
// 呼叫
let middlewares = [logAMiddlewareStore, logBMiddlewareStore]
let dispatch = store.dispatch // 初始dispatch
middlewares.forEach((middleware) => {
  dispatch = middleware(store)(dispatch)
  /*  執行流程
        logAMiddlewareStore 流程
        logAMiddlewareStore(store)(dispatch)
      =>logAMiddlewareNext(dispatch)
      =>dispatch = logAMiddleware 
      =>logAMiddleware 內的 next(action) = dispatch(action)
      
        logBMiddlewareStore 流程
      =>logBMiddlewareStore(store)(logAMiddleware)
      =>logBMiddlewareNext(logAMiddleware)
      =>dispatch = logBMiddleware
      =>logBMiddleware 內的 next(action) = logAMiddleware(action) 跟步驟三結果一致
    
    */
})
```

## Redux applyMiddleware 解析!!!

知道 Middleware 怎麼寫了之後，接著要來看在 Redux 該如何使用。

### 1. 首先從 applyMiddleware 的原碼開始看

```javascript
/*applyMiddleware中最主要的部分*/
const middlewareAPI = {
  getState: store.getState,
  dispatch: (...args) => dispatch(...args),
}
const chain = middlewares.map((middleware) => middleware(middlewareAPI))
dispatch = compose(...chain)(store.dispatch)
/* 接續上方例子
  => chain = [logAMiddlewareStore(middlewareAPI) ,logBMiddlewareStore(middlewareAPI) ]
  => chain = [logAMiddlewareNext , logBMiddlewareNext]
  => 由此可知 middlewareAPI 也就是參數 store
 */
```

### 2. 接著看 compose 的原碼

```javascript
export default function compose(...funcs) {
  if (funcs.length === 0) return arg => arg
  if (funcs.length === 1) return funcs[0]
  return funcs.reduce((a, b) => (...args) => a(b(...args)))
  /* 接續上方例子 compose(...chain)(store.dispatch)
   => compose(logAMiddlewareNext,logBMiddlewareNext)(store.dispatch)
   => funcs = [logAMiddlewareNext,logBMiddlewareNext]
   => return [logAMiddlewareNext,logBMiddlewareNext].reduce((a,b)=>(...args)=>a(b(...args)))
  */
```

### 3. 關於 a(b(...args))

```javascript=
/* a(b(...args)) */
=> logAMiddlewareNext(logBMiddlewareNext(store.dispatch))
=> logAMiddlewareNext(logBMiddleware)
```

| a             | b   | a(b(...args))    |
| ------------- | --- | ---------------- |
| A             | B   | A(B(...args))    |
| A(B(...args)) | C   | A(B(C(...args))) |

**注意這邊 (a,b)=>(...args)=>a(b(...args)) 是有多包一層函式 function(...args) ，由此可知 args 就是 store.dispatch**

### 4. 最後我們回來看 (a,b)=>(...args)=>a(b(...args))

```javascript
// 我們來看看 (a,b)=>(...args)=>a(b(...args)) 是如何執行
function reduxReduce(a, b) {
  return function (...args) {
    return logAMiddlewareNext(logBMiddleware)
  }
}
/* 由此可知 applyMiddleware 的 dispatch 為
    dispatch = (...args)=>{ return logAMiddlewareNext(logBMiddlewareNext(...args))}(store.dispatch)
  =>dispatch = logAMiddlewareNext(logBMiddleware) 

而 logBMiddleware 也就是 dispatch 不懂的可以回去看上方架構的第三步

function logBMiddlewareStore(store){
    let next = store.dispatch
    store.dispatch = function logBMiddleware(action){
        console.log('logBMiddleware')
        let result = next(action)
        return result
    }
 }
*/
```

### 5. 以 Call Stack 來看Ｍ iddleWare 的執行

**對 Call Stack 不懂的話可以看看
https://hackmd.io/eQyarYfWQWGnB_mKKsk3nA?both**

```javascript=
// 用下面的 Demo 範例來看，Call Stack 大概長這樣
    |                         |
    |  store.dispatch(action) |
    |  logBMiddleWare         |
    |  logAMiddleWare         |
    |_________________________|
```

## 總結

**根據上面一步一步講解我們可以得知**

1.  每一個 Middleware 中的 next 是下一個 middleware 的 dispatch 部分
2.  最後一個 middleware 的 next 是 redux store 的 dispatch

最後附上簡單的 MiddleWare DEMO 歡迎點進去看看
**Demo:** https://codesandbox.io/s/redux-middleware-example-b5434
**Github:** https://github.com/librarylai/redux-middleware-example

#### 以上是個人看完以下大神的文章後所作的筆記與見解，如有任何錯誤或冒犯的地方還請各位多多指教。

### 謝謝觀看。

## Reference

1. [Redux.org](https://redux.js.org/introduction/three-principles)
2. [chentsulin — 繁體中文 redux 文件翻譯](https://chentsulin.github.io/redux/index.html)
3. [pcwu's TIL Notes — Redex 核心概念筆記](https://note.pcwu.net/2017/03/04/redux-intro/)
4. [谷哥 — 詳解 Redux Middleware](https://medium.com/@max80713/%E8%A9%B3%E8%A7%A3-redux-middleware-efd6a506357e)
5. [magicdawn — redux compose 详解](https://cnodejs.org/topic/5995a7abee602e88524b435e)
