# 【筆記】SSR 系列第三集【Redux Toolkit & GetServerSideProps 實作】

###### tags: `筆記文章`

![](https://i.imgur.com/BxFW0W5.png)

這陣子常看到大大們在介紹 Redux ToolKit 這套 Redux 的 library，就連 Redux 官方自己也強烈建議使用這套件來編寫 redux 的邏輯，既然官方這麼推薦，我們就來一探究竟一下 Redux ToolKit 的強大之處，也順便比較一下究竟與原本 Redux 的寫法差在哪呢 !?

**如果您對 Redux 完全沒概念，或對 store、action、reducer 等名詞完全沒概念的話，這篇可能會對您有一點點吃力。不好意思!!**

提醒 : 本篇文章部分程式碼是接續前兩篇內容繼續實作，如果沒看過前兩篇的話建議觀看一下

[【筆記】SSR 系列第一集【運用 CRA 做 Server Side Rendering】](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/SyZ4NNNEF)
[【筆記】SSR 系列第二集【React Router v6 & Code Spliting】](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/B186UUGvF)

**_本篇主要是以筆記的形式，紀錄實作中碰到的問題，希望盡量以口語的方式去解說內容，如果有哪邊太跳 tone 或解釋錯誤的部分，再麻煩大大們提點。_**

## 介紹 Redux Toolkit

**_以下將會介紹 Redux Toolkit 套件與常用 API，如果已了解該套件的讀者，可以直接跳過這部分_**

Redux Toolkit 主要是用來解決 Redux 的三個常見問題

> - Configuring a Redux store is too complicated
> - I have to add a lot of packages to get Redux to do anything useful
> - Redux requires too much boilerplate code

以往我們在建立 Redux 的 store 時，大部分都會開一隻 action 檔(ex. loginAction) 一隻 reducer 檔(ex. loginReducer) ，而在 Redux Toolkit 中則只需要創建一個 slice 並將 action 與 reducer 等相關程式碼直接寫在 slice 裡面即可。

在非同步的方面，以往需要額外載入 `redux-thunk` 或 `redux-saga` 等套件來處理這部分，而 Redux Toolkit 默認就已經包含了 `redux-thunk` 來處理 data fetching 與 caching 等功能。

簡單來說，Redux Toolkit 這套 library 它包含了 Redux 的核心與一些我們使用 Redux 常會用到的套件在裡面(ex. Redux Thunk、Reselect )，使我們可以減少安裝一些套件與相關設定，也可以讓我們在攥寫 Redux 的程式碼可以更精簡。

### Redux Toolkit 核心 API

以下將會介紹 Redux Toolkit 常用的 API，主要以翻譯官方文檔的內容為主，會盡量以簡單的方式去說明，希望能讓對 Redux 沒經驗的人能夠看懂。

**如果已經了解 Redux Toolkit 的讀者，可以直接跳過這個部分。**

- [**configureStore()**](https://redux-toolkit.js.org/api/configureStore) : `configureStore` 包裝了原本 Redux 的`createStore` 簡化了設定的流程，它可以自動結合 slice reducers、加入任何 Redux Middleware 與啟動 Redux DevTools 擴充套件。

```javascript=
import { configureStore } from '@reduxjs/toolkit'
import rootReducer from './reducers'
const store = configureStore({
    reducer,
    middleware: (getDefaultMiddleware) => getDefaultMiddleware().concat(logger),
    devTools: process.env.NODE_ENV !== 'production',
    preloadedState,
    enhancers: [reduxBatch],
})
```

- [**createReducer()**](https://redux-toolkit.js.org/api/createReducer) :`createReducer` 是簡化創建 Redux reducer 的 function，內部使用 [Immer](https://immerjs.github.io/immer/) 這套 library 方便我們攥寫 immutable 的資料內容。

  **以往 Redux Reducer 的寫法**
  基本上偏向使用 `switch case` 的方式去處理每一個 action type。

  ```javascript=
  const initialState = { value: 0 }
  function counterReducer(state = initialState, action) {
     switch (action.type) {
     case 'increment':
       return { ...state, value: state.value + 1 }
     case 'decrement':
       return { ...state, value: state.value - 1 }
     case 'incrementByAmount':
       return { ...state, value: state.value + action.payload }
     default:
       return state
     }
  }
  ```

  createReducer 主要有兩個參數，第一個是 `initialState` 代表 reducer 的初始值，第二個是 callback function`(builder: Builder) => void` 透過 Builder 物件裡面的 `addCase` 去定義 reducer 裡面的每一個 case。

  ```javascript=
  const reducer = createReducer(
    {
      counter: 0,
      sumOfNumberPayloads: 0,
      unhandledActions: 0,
    },
    (builder) => {
       /*
        * 補充 addCase 參數解釋
        * 第一個參數: 可以為 string 或透過 createAction 產生出來的 action creator (因為內部會自動呼叫 toString 函式取得 action type )
        * 第二個參數: 就是 redux 的 reducer function (簡單解釋: 當前 state , new Data)
        * */
      builder
        .addCase(increment, (state, action) => {
          // action is inferred correctly here
          state.counter += action.payload
        })
        // You can chain calls, or have separate `builder.addCase()` lines each time
        .addCase(decrement, (state, action) => {
          state.counter -= action.payload
        })
        // You can apply a "matcher function" to incoming actions
        .addMatcher(isActionWithNumberPayload, (state, action) => {})
        // and provide a default case if no other handlers matched
        .addDefaultCase((state, action) => {})
    }
  )
  ```

  #### Usage with the "Map Object" Notation

  createReducer 也可以使用物件的方式來定義 reducer 的 case，物件的 key 代表 action type，value 則代表處理該 action type 的 function。
  render 還有提供一個 `getInitialState` function 用來讓我們取得初始值。

  ```javascript=
  const counterReducer = createReducer(0, {
      increment: (state, action) => state + action.payload,
      decrement: (state, action) => state - action.payload,
  })
  console.log(counterReducer.getInitialState()) // 0
  ```

- [**createAction()**](https://redux-toolkit.js.org/api/createAction) : createAction 結合了 action type 與 action creator function，當我們使用 `createAction` 創建 action 時我們只需要給定 action type 它預設就會根據 action type 自動去建立對應的 action creator function。以往我們在使用 Redux 建立 action 時都要對每一個 action type 都去建立一個 function，而`createAction` 整個簡化了這些步驟，它本身還有內建 `toString` 這個函式方便我們取得 action type。

  **以往 Redux Action 的寫法**
  每個 action type 都需要寫一個對應的 function。

  ```javascript=
  const INCREMENT = 'counter/increment'
  function increment(amount: number) {
    return {
      type: INCREMENT,
      payload: amount,
    }
  }

  const action = increment(3)
  // { type: 'counter/increment', payload: 3 }
  ```

  **使用 createAction 的寫法**
  依照建立時傳進的 action type (ex.counter/increment) 直接建立好對應的 action creator function。

  ```javascript=
  const increment = createAction<number | undefined>('counter/increment')

  let action = increment()
  // { type: 'counter/increment' }

  action = increment(3)
  // returns { type: 'counter/increment', payload: 3 }

  console.log(increment.toString())
  // 'counter/increment'
  ```

  #### Using Prepare Callbacks to Customize Action Contents

  預設創建 action creator 只需要傳入一個參數(action type)，但如果你想要接收多個參數或客制化 function 的邏輯的話，可以透過傳入一個 function 到 `createAction` 的第二個參數中來客制化整個函式。

  ```javascript=
  const addTodo = createAction('todos/add', function prepare(text: string) {
    return {
      payload: {
        text,
        id: nanoid(),
        createdAt: new Date().toISOString(),
      },
    }
  })
  ```

- [**createSlice()**](https://redux-toolkit.js.org/api/createSlice) : 可以說是整合了 `createReducer` 與 `createAction` 直接透過 `createSlice` 自動產生 reducers 與該 reducers 對應的 action creators。

  **原本使用 createReducer、createAction 作法**

  ```javascript=
  const increment = createAction('increment')
  const counterReducer = createReducer(0,
    (builder) => {
      builder
        .addCase(increment, (state, action) => {
          state = state + action.payload
        })
      }
  )
  const store = configureStore({
    reducer: reducer,
  });
  store.dispatch(increment(3));

  ```

  **使用 createSlice 作法**
  createSlice 接受一個物件參數來設定相關配置，配置參數如下

  - **name :** 接受一個字串，將成為 action type 的前綴(ex. counter/increment)，可以當成 slice 的 key。
  - **initialState :** reducer 的初始 state。(當初始值傳入為 function 時則會為`lazy initializer` function，主要用在 localStorage 讀取初始值使用。)
  - **reducers :** 主要是以物件的形式，可以參考上面的 createReducer。
  - **extraReducers :** 它跟 reducers 很像，一樣可以傳入 object 或 function(builder callback)，兩者的差別在於 `extraReducers` 可以讓我們去使用其他已經定義好的 slice 的 action 。[參考連結](https://stackoverflow.com/questions/66425645/what-is-difference-between-reducers-and-extrareducers-in-redux-toolkit)

  ```javascript=
  const counterSlice = createSlice({
    name: 'counter',
    initialState: 0,
    reducers: {
      increment: (state, action) => state + action.payload,
    },
      /*
       * auth.actions.logout 是額外已經被定義的 slice 裡面的 action
       * ex.
       * auth = createSlice({
       *   reducer: {
       *    logout: ()=>{....}
       *   }
       * })
       *  */
    extraReducers:(builder) => {
      builder.addCase(auth.actions.logout, state => {
        state.isLoggedIn = false;
      });
    }
  });

  const { increment } = counterSlice.actions
  const store = configureStore({
    reducer: counterSlice.reducer,
  })
  store.dispatch(increment(3))
  ```

- [**createAsyncThunk()**](https://redux-toolkit.js.org/api/createAsyncThunk) : `createAsyncThunk` 主要是用來處理非同步的操作，它需要傳入一個 action type string ( 可以想像成這個 action 的 key )和一個 payloadCreator callback (用來處理非同步的邏輯 ex.fetch api)，且 payloadCreator callback 需回傳一個帶有 Data 的 Promise。

  `createAsyncThunk` 會根據我們傳入的 action type string 去建立三個不同的 action creators，它會使用 `createAction` 去建立 `pending`,`fulfilled`, `rejected` 這三個 action creator ( 又可以稱做為 lifecycle action creator )，我們可以在 extraReducer 中對每一個 action creator 去做相對應的操作。

  **舉例 : 如果 action type string 為 users/auth 則 lifecycle action type 則為以下 :**

  > pending : users/auth/pending
  > fulfilled : users/auth/fulfilled
  > rejected : users/auth/rejected

  ```javascript=
  /*
   * userId : 是當我們呼叫 fetchUserById 時傳入的參數，可以是一個物件。以這邊的例子來看的話就是 24 行的 123
   * thunkAPI : 是一些通常傳給 Redux thunk 的參數，例如裡面有 getState 可以用來取得 redux store 裡面的資料。
   * */
  const fetchUserById = createAsyncThunk(
    'users/fetchByIdStatus',
    async (userId, thunkAPI) => {
      const response = await userAPI.fetchById(userId)
      return response.data
    }
  )
  // Then, handle actions in your reducers:
  const usersSlice = createSlice({
      // 審略 name,reducer 等參數...
    extraReducers: (builder) => {
      // 當 fetchUserById 非同步處理完成時
      builder.addCase(fetchUserById.fulfilled, (state, action) => {
        // Add user to the state array
        state.entities.push(action.payload)
      }),
      // 當 fetchUserById 非同步處理失敗時
      builder.addCase(fetchUserById.rejected, (state, action) => {
        // Add user to the state array
        state.entities = []
      })
    },
  })

  // Later, dispatch the thunk as needed in the app
  dispatch(fetchUserById(123))
  ```

- [**createSelector()**](https://redux-toolkit.js.org/api/createSelector) : `createSelector` 是使用 [Reselect](https://github.com/reduxjs/reselect) 這套 library，可以用它來創建一個可記憶、可組合的 selector，它會 cache 我們的輸入 (input selector) 與輸出 (resultFunc)，當傳入的 input 不變時 selector 會回傳先前計算的結果，而不會呼叫 resultFunc 去重新計算，這樣可以避免一些不必要的重複運算，減少性能的消耗。

  > - Selectors can compute derived data, allowing Redux to store the minimal possible state.
  > - Selectors are efficient. A selector is not recomputed unless one of its arguments changes.
  > - Selectors are composable. They can be used as input to other selectors

  這邊直接透過程式碼來簡單介紹一下這個套件!!!

  ```javascript=
  /* createSelector 定義
   * createSelector(...inputSelectors | [inputSelectors], resultFunc, selectorOptions?)
   * */

   /* createSelector 可以是多個 inputSelectors 或是 用陣列包住多個 inputSelectors 也可以組合多個 Selectors */
  // 多個 inputSelectors
  const selectA = createSelector(
    state => state.shop.value1,
    state => state.shop.value2,
    (value1, value2) => value1 + value2
  )
  // 用陣列包住多個 inputSelectors
  const selectB = createSelector(
    [state => state.shop.value1, state => state.shop.value2],
    (value1, value2) => value1 + value2
  )
  // 組合多個 Selectors
  const selectC = createSelector(
      selectA,
      selectB,
      (aResultValue,bResultValue)=>{
          /* aResultValue 為 selectA 的 result
           * bResultValue 為 selectB 的 result */
          return aResultValue + bResultValue
      }
  )
  const exampleState = {
    shop: {
      value1:10,
      value2:20
    }
  }
  console.log(selectB(exampleState)) // 30
  console.log(selectC(exampleState)) // 60
  /* inputSelectors 參數 與 呼叫 Selector 時傳入的參數 相對應
   *  ex1. selectB(exampleState) 為例 (state)=>{ state 為 exampleState  }
   *  ex2. selectB(store,props) 為例 (state,p)=>{ state 為 store ; p 為 props }
   * 在 Redux 使用上，通常會將 Redux 的 State 當作 呼叫 Selector 的第一個參數; 額外的 props 會當作第二個參數傳入。
  */
  ```

## 前端 Redux Tooklit 實作

#### 接下來的內容大致可分為以下幾個步驟:

1. **Redux 套件安裝**
2. **建立一支 store.ts 並使用 configureStore 來建立一個 Redux store**
3. **將 store 綁定到整個專案中**
4. **建立 React State Slice**
5. **將 Slice 的 reducer 加入到 Store**
6. **component 使用 redux 接取資料**

### **1. Redux 套件安裝**

安裝 Redux Toolkit 與 React Redux

> yarn add @reduxjs/toolkit react-redux

**補充: React Redux 套件**

> React Redux 是 Redux 官方提供與 React UI 綁定的套件，它可以讓 React component 讀取 Redux store 裡面的資料(data)以及操作 actions 去更新 store 裡的資料(data)

如果專案有使用 Typescript 且 React Redux 小於 v7.2.3 版的話，則需額外安裝 `@types/react-redux` 套件。

> yarn add @types/react-redux

---

安裝好套件後現在我們就來將 Redux 設定到專案內吧!!

### **2. 建立一支 store.ts 並使用 configureStore 來建立一個 Redux store**

```javascript=
/* store/store.ts */
import { configureStore } from '@reduxjs/toolkit'
import { useDispatch } from 'react-redux'

// ...

export const store = configureStore({
  /* 第四步再來介紹~~ */
})

// Infer the `RootState` and `AppDispatch` types from the store itself
export type RootState = ReturnType<typeof store.getState>
// Inferred type: {posts: PostsState, comments: CommentsState, users: UsersState}
export type AppDispatch = typeof store.dispatch
export const useAppDispatch = () => useDispatch<AppDispatch>() // Export a hook that can be reused to resolve types

```

這邊 export 了 `RootState` 與 `AppDispatch` 這兩個 type，與`useAppDispatch` 這個 hook，主要是方便之後在使用上可以快速加入型別，`useAppDispatch` 這個 hook 直接幫我們將型別套用在 `useDispatch`上，這樣就不用每一次使用時都還要在設定一次型別。

**補充: 如果對 ReturnType 不了解的讀者，可以看 _pjchender_ 大大的 [[Day10] TS：什麼！Conditional Types 中還能建立型別？使用 infer 來實作 ReturnType 和 Parameters](https://ithelp.ithome.com.tw/articles/10272213) 文章**

### **3. 將 store 綁定到整個專案中**

通過在 `src/index.tsx` 中加入 React Redux 的 `<Provider>` component，讓 Redux store 裡面的資料能透過 props 傳入到整個專案中。

```javascript=
import { store } from '@/store/store'
import { Provider } from 'react-redux'
//...
ReactDOM.render(
    <React.StrictMode>
        <Provider store={store}>
            <BrowserRouter>
                <App />
            </BrowserRouter>
        </Provider>
    </React.StrictMode>,
    document.getElementById('root')
)
```

### **4. 建立 React State Slice**

建立一隻 `slice/newsSlice` 檔來直接模擬抓取資料後塞進 redux store 中的情境。
API：新聞資料是使用 [NewsAPI](https://newsapi.org/) 這個網站去抓取的公開資料，使用方法很簡單，只要點擊官網右上角的`Get API key`(如圖一)並註冊完成後就可以拿到一組 API key(如圖二)，透過該 API key 就可以抓取各種官網上所提供的資料了。

(圖一)
![](https://i.imgur.com/bdRxB8a.png)
(圖二)
![](https://i.imgur.com/N3l2z3b.png)

有了 API 後現在我們就要來實作 Slice 的部分了，首先因為呼叫 API 是屬於非同步的部分所以可以使用 `createAsyncThunk` 來完成，這邊記得要將這個 action 給 `export` 出來，這樣到時才能方便 dispatch 它。

接著使用 `createSlice`創建一個 Slice 並且在 `extraReducers` 中去設定我們剛剛那個 async action，當它 `fulfilled` 時將資料寫進該 Slice 的 State 中。

```javascript=
/* store/slice/counterSlice.tsx */
import { createAsyncThunk, PayloadAction, createSlice } from '@reduxjs/toolkit'
import axios from 'axios'
// 透過 createAsyncThunk 來實作一個 非同步的 action
// 特別記得要 export ，才可方便之後 dispatch 使用
export const fetchNewsAPI = createAsyncThunk('news/fetchNewsAPI', async (_: void, thunkId) => {
  const response = await axios.get('https://newsapi.org/v2/top-headlines?country=tw&category=business&apiKey=41a1d4035b60422a931ed0f23b95e320')
  return response.data.articles
})
interface Item {
  title: string
  description: string
}
export interface NewsState {
  newsData: Item[]
}
const initState: NewsState = {
  newsData: [],
}

const newsSlice = createSlice({
  name: 'news',
  initialState: initState,
  reducers: {
    // 清空 newsData
    cleanNewsData: (state) => {
        state.newsData = []
    },
  },
  extraReducers:{
    [fetchNewsAPI.fulfilled.toString()]:(state, { payload }) => {
          state.newsData = payload
    }
  }
  // 另一種寫法： 使用 (builder) =>{}
  // (builder) => {
  //   builder.addCase(fetchNewsAPI.fulfilled, (state, { payload }) => {
  //     state.newsData = payload.newsData
  //   })
  // },
})
// 每個定義的 reducers 會被建立成各個 action creators
export const { cleanNewsData }  = newsSlice.actions

export default newsSlice.reducer


```

### **5. 將 Slice 的 reducer 加入到 Store**

還記得在上面的步驟中我們建立了一個【空的】 Redux store，而剛剛又建立了一個 newsSlice 並且 export 該 slice 的 action 與 reducer，現在需要將 slice 的 reducer 加入到 store 中，讓 store 知道能透過該 reducer function 來更新 store 的 state。

```javascript=
/* store/store.tsx */
import newsReducer from '@/store/slice/newsSlice'
export const store = configureStore({
  reducer:{
    news: newsReducer
  }
})
```

### **6. component 使用 redux 接取資料**

到目前為止我們已經將整個 Redux 架構設定完成，現在就來在 component 中實際使用一下，透過 `useDispatch` 來執行 newsSlice 裡面的 `fetchNewsAPI` 這個 action creator 來呼叫 API 獲取資料，並透過 `useSelector` 來拿取 store 中的 `newsData`，就讓我們先來看一下成果畫面。(圖片有點不清晰，可以放大倍率來看 😵)

![](https://i.imgur.com/wpV72qN.gif)

從畫面上可以看到，當進入 `ScrollAnimation` 時 Redux 會觸發了 `news/fetchNewsAPI/pending` 與 `news/fetchNewsAPI/fulfilled` 這兩個 action，還記得我們在上面 `newsSelice` 中設定了 `fulfilled` 的情況嗎 ? 忘記了可以往上滑看看程式碼，大概意思是:【當 `fulfilled` 時將 api 的 responce data 寫進 newsData 中】，所以這邊可以看到在 `fulfilled` 時 newsData 裡面有了資料。最後在離開時觸發 `news/cleanNewsData` 將 newsData 清空。

```javascript=
/* ScrollAnimationPage.tsx */
const ScrollAnimationPage = (props: Props) => {
    /* ...審略...  */
    const dispatch = useAppDispatch()
    const newsData = useSelector((state: RootState) => state.news.newsData)
    // componentDidMount 與 componentWillUnMount
    useEffect(() => {
        dispatch(fetchNewsAPI())
        return ()=>{
            dispatch(cleanNewsData())
        }
    }, [dispatch])
    // render ScrollBox
    function renderScrollBox() {
        if(!newsData.length) return
        return newsData.map((item, index) => {
          return (
            <ScrollableBox ref={(el: HTMLDivElement) => (boxRefs.current[index] = el)} isShow={checkShowBox(index)} key={index}>
              {item.title}
            </ScrollableBox>
          )
        })
    }
    return <ScrollAnimationPageContainer>{renderScrollBox()}</ScrollAnimationPageContainer>
}

export default ScrollAnimationPage
```

這邊只留下與 Redux 相關的程式碼部分，主要模擬進入時(componentDidMount)呼叫新聞 API 獲取資料，離開時(componentWillUnMount)清空則 `newsData`，裡面使用了 `useAppDispatch` 而不是使用 `useDispatch` 主要是 `useAppDispatch` 封裝了 typescript 型別的部分，詳細程式碼可以看【第一點】的部分，以上就是整個模擬【執行 Redux 獲取資料與清空資料】的部分。

**最後讓我們快速複習一下 :**

1. **先建立一個空的 Redux Store**
2. **前端 root component 加入 React Redux 的 `<Provide>` component 並將 Store 丟進去 component 中**
3. **使用 `createSlice` 建立 slice 並 export action、reducer**
4. **將 slice 的 reducer 加入到 Redux Store 中，完成整個 Redux 設定。**
5. **component 中使用 `useSelector` 與 `useDispatch` 來抓取 store 中的資料與執行 action。**

---

## Server-Side 使用 Redux 與實作 getServerSideProps

上面的部分都是單純在前端執行，但不要忘了....我們這個系列是 SSR 系列，所以怎麼能少了 Redux 與 Server-Side Rending 搭配使用的部分呢，所以首先我們一樣要先思考一下，在 Server-Side Rending 的情況中我們要達成哪寫事情呢?

**這邊先來複習一下何謂 Server Side Rendering！！!**

> **Server Side Rendering 主要是當使用者進入到網站時，就預先將該頁面所需的內容在 Server Side 處理好並建立要回傳的 HTML 結構，讓 broswer 在一開始就有完整的 HTML 結構，而不是只有 `<div id='root'>` 這個 DOM 元素。**

還記得上面的部分我們在 `componentDidMount` 時去執行了 Store 的 action 取得新聞資料(call API)嗎，而我們如果要讓 Server Side 在一開始吐出來的 HTML 就包含這些內容的話，就需要先 preload data 並塞到 store 或 component 中，讓這些 component 能夠在 `renderToString` 的時候正確的解析出『包含內容』的 HTML。

#### 我們先直接看個對照圖!!

兩張都是經過 Server Side Rednering 後回傳的 HTML 結構，可以很明顯的看出結構上的不同，一個是有『包含內容』的 HTML 結構，一個則是純 Component 的 HTML 結構。

PS. 在後端就預先載入資料的會抓取英文版資料，而如果是回到前端才抓取資料的會是中文版資料。

![](https://i.imgur.com/kxABr4y.png)
( 純 Component 的 HTML 結構)

![](https://i.imgur.com/aOKhUxK.png)
( 包含內容的 HTML 結構 )

#### 接下來的內容大致可分為以下幾個步驟:

1. **Store 綁定到 Server 端**
2. **了解 getServerSideProps 功能**
3. **前端 Component 實作 getServerSideProps**
4. **增加 router config 檔**
5. **後端實作呼叫 getServerSideProps 邏輯**
6. **Responce HTML 模板注入 解析完的 HTML 結構**
7. **Responce HTML 模板注入 PRELOADED_STATE**
8. **Responce HTML 模板注入 SERVER_SIDE_PROPS**

### Store 綁定到 Server 端

要在 Server-Side 中也能使用 Redux 不外乎就是要把 `<Provider>` 也加到 Server 中，Store 的部分就直接引入上面所建立的 Store 即可。

```javascript=
import { store } from '@/store/store'
import { Provider } from 'react-redux'
/* 部分程式碼可看上一章  */
const staticHTML = ReactDOMServer.renderToString(
    webExtractor.collectChunks(
        sheet.collectStyles(
            // 主要是 Provider 這段
            <Provider store={store}>
                <StaticRouter location={req.url}>
                    <App />
                </StaticRouter>
            </Provider>
        )
    )
)
```

### 了解 getServerSideProps 功能

這部分參考了 **_Airwaves 大大_** [React SSR | 從零開始實作 SSR — 載入資料篇 - Airwaves](https://medium.com/%E6%89%8B%E5%AF%AB%E7%AD%86%E8%A8%98/server-side-rendering-ssr-in-reactjs-part4-38649606d384) 的文章。

因為每個 Component 所需的內容都不同，所以當我們要在 Server Side 去取得內容時就會產生問題，當然我們可以將每個 Component 預先取得資料的方法都寫在 Server 端，但這樣當規格一有變化時不僅前端要調整，後端也需要跟著調整，而且邏輯判斷也會被分散在前端與後端之中，難道不能都在前端就設定好嗎??。

因此在 Next.js 上它們就有提供幾個方法( ex.getInitialProps、getServerSideProps、getStaticProps)來達成事先載入資料，而我們這次的目的就是來實作 `getServerSideProps` 的整個流程，所以讓我們先來了解一下這個 function 在做什麼吧!!

`getServerSideProps` function 主要設定在每個 Component 中，讓我們在 Server Side Rendering 時可以透過呼叫該 Component 的 `getServerSideProps` 去做一些預處理的操作，例如當從 Server Side 渲染畫面時，先呼叫 `getServerSideProps` (裡面可能是一些呼叫 API 的邏輯)來取得資料並塞入到該 Component 當作 Props，讓 Server Side 在輸出 HTML 結構給 broswer 時就會有了這些資料內容。

我們先來看看 [Next.js 官方 - getServerSideProps](https://nextjs.org/docs/basic-features/data-fetching#getserversideprops-server-side-rendering) 的介紹，我們可以看到這個 `getserversideprops` 是一個 Promise 的 Function 並且【**回傳要給該 Component 的 Props 參數**】，而 `Context` 則是從 Server 端傳過來的參數(ex. store、req、res...等)。

```javascript=
export async function getServerSideProps(context) {
  return {
    props: {}, // will be passed to the page component as props
  }
}

```

### 前端 Component 實作 getServerSideProps

現在我們要來改寫一下`ScrollAnimationPage` 這隻檔案，將它加入 `getServerSideProps` function，為了讓之後比較好看出變化，這邊刻意在取資料的部分做了一點區別，如果是從 Client Side 進入 Component 的話會出叫 `fetchNewsAPI` 取得【中文】的新聞資料，如果是從 Server Side 預先載入的話會呼叫 `fetchEnNewsAPI` 取得【英文】的資料。

```javascript=
const ScrollAnimationPage: IReactComponent<Props> = ({ serverSideProps }) => {
    // componentDidMount 與 componentWillUnMount
    useEffect(() => {
    // 簡單判斷如果是來自於 前端 切換近來的話，就呼叫ＡＰＩ
    // 如果來自於 Server Side， newsData 會在 Server 端 preload 時就有值
    if (newsData.length === 0) {
      dispatch(fetchNewsAPI())
    }
    // 離開 component 時清除 ( 做 componentWillUnMount 效果 )
    return () => {
      dispatch(cleanNewsData())
    }
    }, [])

    /* render 內容審略...  */
}
// 給 Server Side 呼叫
async function getServerSideProps(context: IServerSideContext) {
  // context 是來自 server 端傳進來的 ex. req ,res, redux store....
  const { store } = context
  // 等待 獲取資料 塞到 store
  await store.dispatch(fetchEnNewsAPI())
  return {
    props: {
      test: '我我來自 server side',
    },
  }
}

ScrollAnimationPage.getServerSideProps = getServerSideProps
export default ScrollAnimationPage
```

這邊我們透過在 `getServerSideProps` 設定呼叫 `fetchEnNewsAPI` ，讓我們在 Server Side 時能夠透過執行 `getServerSideProps` 來呼叫 store 的 async action 預先取得英文版新聞資料，這樣 `ScrollAnimationPage` 就能夠在 Server Side 時透過 store 裡的 newsData 這個 state 取得這些資料。

```javascript=
/* store/slice/newsSlice.ts */
// 取得英文版 Data
export const fetchEnNewsAPI = createAsyncThunk('news/fetchEnNewsAPI', async (_: void, thunkId) => {
  const response = await axios.get('https://newsapi.org/v2/top-headlines?country=us&category=business&apiKey=41a1d4035b60422a931ed0f23b95e320')
  return response.data.articles
})
```

### 增加 router config 檔

為了要知道使用者第一次進入網站時應該要顯示哪個 component，所以我們要先來製作一下 route 與 component 的照表，這樣我們才會知道哪個 route 要顯示哪個 component，以前需要額外安裝 `react-route-config` 這個套件，但現在 `react-router v6` 已經內建就包含了 `react-route-config`，所以我們可以直接使用 `matchRoutes` 去解析使用者進入的 `path`，再去對該 `path` 的 component 進行操作 (ex. 呼叫 getServerSideProps)

```javascript=
/* route/routes.tsx */

/* 使用 loadable component */
const ExpandingCardsPageLoadable = loadable(
	() => import(/*webpackChunkName:'ExpandingCardsPage'*/ '@/containers/expandingCards/ExpandingCardsPage')
)
/* ...審略其他 loadable compoent 宣告...  */

// 這邊多了一個  component 的參數，主要是給 Server Side 能夠呼叫 component 的 getServerSideProps
const serverSideRoutes: IRouteItem[] = [
    { path: '/', component: ExpandingCardsPage, element: <ExpandingCardsPageLoadable /> },
    { path: '/ExpandingCards', component: ExpandingCardsPage, element: <ExpandingCardsPageLoadable /> },
    { path: '/ScrollAnimation', component: ScrollAnimationPage, element: <ScrollAnimationPageLoadable /> },
    { path: '/RotatingNavigation', component: RotatingNavigationPage, element: <RotatingNavigationPageLoadable /> },
    { path: '/BlurryLoading', component: BlurryLoadingPage, element: <BlurryLoadingPageLoadable /> },
    { path: '/HiddenSearchWidget', component: HiddenSearchWidgetPage, element: <HiddenSearchWidgetPageLoadable /> },
    { path: '/StepsPage', component: StepsPage, element: <StepsPageLoadable /> },
]

export default serverSideRoutes

```

這邊的 `serverSideRoutes` 中最重要的就是 `path` 與 `component` 這兩個參數，Server Side 可以透過這個 router config 去找到符合 `path` 的 `component`，而 `element` 參數則是給前端渲染 `<Route>` 使用。

#### 補充: 在 `react-router v6` 中，我們也可以使用 `useRoutes` 加上 router config 來代替一個一個手寫 `<Route .../>` 元件。

```javascript=
// 原版
<Container>
    <Route path="/" element={<ExpandingCardsPage />}/>
</Container>

/*使用 router config 方式*/
<Container>
    {useRoutes(routes)}
</Container>
```

### 後端實作呼叫 getServerSideProps 邏輯

**現在要進入到這一系列最終要的部分了!!**

還記得我們剛剛在上面已經做到能夠將找到該 `path` 是哪個 `component` 了吧，現在我們就要來處理呼叫 `component` 的 `getServerSideProps` 這部分的邏輯。先上程式碼~~~

```javascript=
/* server/index.tsx */

// 實作處理 getServerSideProps
function getServerSidePropsPromise(req: express.Request) {
  // 比對 route ，抓出符合目前 request 的 route 項目
  let matchResult = matchRoutes(routes, req.path)
  // 如果沒有 match 到就直接 return
  if (!matchResult) return
  // 將 match 到的 route 裡面 component 的 getServerSideProps 執行
  let serverSidePropsPromise = matchResult.map(async (routeItem) => {
    let route: IRouteItem = routeItem.route
    let component = route.component
    if (!component) return null
    // getServerSideProps is a promise function ( getServerSideProps 是一個 promise function )
    if (component.getServerSideProps) {
      return component.getServerSideProps({ store })
    }
    return null
  })
  // 這邊透過 filter 去濾掉 null 的項目
  return serverSidePropsPromise.filter((hasPromise) => hasPromise)
}
```

這邊我們定義了一個 `getServerSidePropsPromise` 的 function，裡面會先去 match 有符合 route 的 component 們，然後再去執行這些 component 裡面定義的 `getServerSideProps`。

不知大家還記不記得我們在 `component` 中定義 `getServerSideProps` 時是 export 一個 Promise，所以我們這邊呼叫完 component 的 `getServerSideProps` 時會【**收到的是一個 Promise 而不是裡面 return 的 object。**】

因此我們在呼叫完 `getServerSidePropsPromise` 後還需要對這些 Promise 再進行一次處理，才會取得最後要傳給 component 的 props。

```javascript=
app.get('*', async (req, res) => {
    let serverSidePropsList: Array<IServerSideProps | null> = []
    let serverSidePropsPromise = getServerSidePropsPromise(req)
    // 因為 getServerSideProps 會回傳 promise  所以要再 await 一次拿到裡面得 props
    if (serverSidePropsPromise) {
        serverSidePropsList = await Promise.all(serverSidePropsPromise)
    }
    const webExtractor = new ChunkExtractor({ statsFile })
    const sheet = new ServerStyleSheet() // <-- 建立樣式表
    // 將 App 這個 component render 成 HTML string
    const staticHTML = ReactDOMServer.renderToString(
        webExtractor.collectChunks(
          sheet.collectStyles(
            <Provider store={store}>
              <StaticRouter location={req.url}>
                <App serverSideProps={serverSidePropsList} />
              </StaticRouter>
            </Provider>
          )
        )
    )
})
```

最後我們將這個 `serverSidePropsList` 塞進 `<App>` component 中，讓它一路傳給所需要的 component，這樣 component 就能夠在回傳 HTML 給 broswer 之前，就預先 render 出含有內容的 HTML 結構了。

```javascript=
const App:React.FC<IApp> = ({ serverSideProps }) => {
    // 審略其餘內容
    reutrn(
        <Route
            path="/ScrollAnimation"
            element={
                <ScrollAnimationPage serverSideProps={serverSideProps} fallback={<LoadingComponent />} />
            }
        />

    )
}
```

### Responce HTML 模板注入 解析完的 HTML 結構

到上面為止我們已經已經能將含有資料內容的 HTML 結構給產出了，現在只需要將這些 HTML 放進我們的模板中。這邊模板的程式碼不需做任何更動，主要的差別在於 `staticHTML` 所產生的內容，所以這段 code 跟上一集是一模一樣不需額外改寫。

補充 : `staticHTML` 是透過 React 提供的 `renderToString` 將 React element render 成 HTML 並回傳 HTML String。以這邊的例子來說 : `staticHTML` 就是 React 將含有資料的 `ScrollAnimatePage` 解析成 HTML String 的回傳值。

```javascript=
 res.send(`
    <!DOCTYPE html>
    <html>
      <head>
        ${styles}
        ${webExtractor.getLinkTags()}
        ${webExtractor.getStyleTags()}
      </head>
    <body>
        <div id="root">${staticHTML}</div>
        ${webExtractor.getScriptTags()}
      </body>
    </html>
`)

```

### Responce HTML 模板注入 PRELOADED_STATE

到目前為止我們可以說是已經將 Server Side Rending 的部分完成了，如果大家一步一步跟著實作到這段的話，應該可以從瀏覽器上的 `Source` 看到【含有內容的 HTML】，但這樣還沒有結束!!!

不知您有沒有發現，我們在 Server Side 呼叫完 API 的資料並沒有出現在 Client Side 中，依照上面的例子，我們在 Server Side 抓取了【**新聞資料**】並存到了 Redux 的 Store 中，但 Client Side 的 Store 裡並未有那些資料，導致目前畫面 render 出來是不正確的，所以我們要將這些資料在 Client Side 建立 `configureStore` 時當作 initial state 塞進 Store 中，這樣畫面才可以正確的被 render 出來，而不是直接白屏。

**可參考 Redux 官網的 [Inject Initial Component HTML and State](https://redux.js.org/usage/server-rendering#inject-initial-component-html-and-state)**

#### 這邊是筆者自己的解讀，不見得正確但可以給大家參考 :

> 我們在 Server Side 所使用的 component 並不等於是回到 Client component 時就會沿用這些 component。
>
> 可以想像成 : 我們在 Server Side 只是透過這些 Component 來去 render 出要給 broswer 的靜態 HTML，所以當我們在 Server Side 傳入一堆 Props 給元件時，Client Side 的該元件也不會有那些 Props。
>
> 當網頁讀取時 `ReactDOM.hydrate()` 將會 reuse 我們從 Server Side render 出來的 HTML，簡單來說就是 React 會開始與這些 HTML 結合(ex.綁定 Event 事件、創建 virtual DOM...等)，而因為我們將 initial state 給到 Redux 中，所以解析出來的結構與我們在 Server Side render 的相同，因此最後才會是相同的 real DOM。
>
> 原文 :
> When the page loads, the bundle file will be started up and ReactDOM.hydrate() will reuse the server-rendered HTML. This will connect our newly-started React instance to the virtual DOM used on the server. Since we have the same initial state for our Redux store and used the same code for all our view components, the result will be the same real DOM.

#### 回到範例這邊

所以我們可以透過在模板上增加一個 Script tag 來將這些資料傳回到 Client Side。

```javascript=
/* server/index.tsx */
res.send(`
    <!DOCTYPE html>
    <html>
      <head>
           ...審略...
      </head>
    <body>
        <div id="root">${staticHTML}</div>
        ${webExtractor.getScriptTags()}
        <Script>
            // WARNING: See the following for security issues around embedding JSON in HTML:
            // https://redux.js.org/usage/server-rendering#security-considerations

            // 增加這邊~~~~~~
             window.__PRELOADED_STATE__ = ${JSON.stringify(preloadedState).replace(
            /</g,
            '\\u003c'
          )}

        </Script>
      </body>
    </html>
`)

```

Client Side 在建立 Store 時將 `__PRELOADED_STATE__` 的資料當作初始值，最後再將`window.__PRELOADED_STATE__` 刪除掉。

```javascript=
/* store/store.ts */
let commonOptions: IStroeOption = {
  reducer: {
    news: newsReducer,
  },
}
if (typeof window !== 'undefined') {
  // 將從 Server side 取得到的 store 資料注入到 client side store 內
  commonOptions.preloadedState = window?.__PRELOADED_STATE__
  //  Allow the passed state to be garbage-collected
  delete window.__PRELOADED_STATE__
}
export const store = configureStore({...commonOptions})
```

### Responce HTML 模板注入 SERVER_SIDE_PROPS

剛剛上面是將 Redux 的資料傳回到 Client Side，而我們也可以將 Props 的資料也一併用一樣的方法將資料傳回來，所以我們可以在模板中再增加以下這段程式碼。

```javascript=
/* server/index.tsx */
 window.__SERVER_SIDE_PROPS__ = ${JSON.stringify(serverSidePropsList).replace(/</g,'\\u003')}
```

還記得我們在 Server Side 時我們將 `getServerSideProps` 的回傳值傳到 `<App>` 元件中(`<App serverSideProps={serverSidePropsList} />`)，現在我們也將這些資料傳到
Client Side 的 `index.tsx` 中，再透過 Props 傳入到 `<App>` 元件裡，這樣前後端就可以達到資料一致、畫面一致。

```javascript=
/* src/index.tsx */
loadableReady(() => {
    ReactDOM.hydrate(
        <React.StrictMode>
            <Provider store={store}>
                <BrowserRouter>
                    <App serverSideProps={ window.__SERVER_SIDE_PROPS__}/>
                </BrowserRouter>
            </Provider>
        </React.StrictMode>,
        document.getElementById('root')
    )
    delete window.__SERVER_SIDE_PROPS__
})

```

### 最後附上成果畫面

![](https://i.imgur.com/IC6LAx6.gif)
(SSR : 英文版，CSR : 中文版)

## 結論

這次玩了一下 Redux-Toolkit 與簡單模擬了如何在 Server Side Rending 時就 render 出包含 API 資料的 HTML 結構，過程中參考了許多大大們的文章，才瞭解到原來 Next.js 有提供 getServerSideProps 這些函式來幫助我們達成這些需求。

個人認為如果是小型專案要直接自己實作 Server Side Rending 可能還可以，但如果是在大型專案的話，我建議還是直接使用 Next.js 會比較好，不管在效能上或是功能性上 Next.js 都更加的完善，況且 Next.js 還可以直接區分哪個 Component 要 SSR；哪個 Component 要 SSG 或者要 CSR，光是在這方面要自己實作就會比現在的版本更加複雜許多，因此還是建議直接使用 Next.js 的架構進行開發，以上只是筆者自己的淺見給您參考。

#### 以上就是這次實作的結論，如果有新的見解或新的發現會再繼續更新這篇文章，如果有任何講錯的地方或冒犯的部分也歡迎大家提出來告訴我一聲。

### 謝謝觀看。

#### Github : [https://github.com/librarylai/react-fifty-practice](https://github.com/librarylai/react-fifty-practice)

## 實作碰到問題

1. **ceateSlice 的 reducer 與 extraReducer 差別在於?**
   解答：[參考連結](https://stackoverflow.com/questions/66425645/what-is-difference-between-reducers-and-extrareducers-in-redux-toolkit)
2. **A computed property name must be of type 'string', 'number', 'symbol', or 'any' with extraReducers**
   解答：[參考連結](https://github.com/reduxjs/redux-toolkit/issues/478)
3. **Uncaught ReferenceError: regeneratorRuntime is not defined in React**
   解答： 安裝 `@babel/plugin-transform-runtime` [參考連結](https://stackoverflow.com/questions/61755999/uncaught-referenceerror-regeneratorruntime-is-not-defined-in-react)
4. **ReferenceError: fetch is not defined**
   解答：`The fetch API is not implemented in Node.`，推薦改使用 axios 套件。
5. **Node.js 中的错误 `window not defined`**
   解答： [參考連結](https://www.coder.work/article/105874)
6. **Explicitly Specifying types for Express' "application, request, response..."**
   解答:[參考連結](https://stackoverflow.com/questions/27676884/explicitly-specifying-types-for-express-application-request-response)

## Reference

1. [Redux 官網](https://redux.js.org/introduction/installation)
2. [Redux Toolkit 官網](https://redux-toolkit.js.org/introduction/getting-started)
3. [[Day10] TS：什麼！Conditional Types 中還能建立型別？使用 infer 來實作 ReturnType 和 Parameters - pjchender](https://ithelp.ithome.com.tw/articles/10272213)
4. [React SSR | 從零開始實作 SSR — Redux 篇 - Airwaves](https://medium.com/%E6%89%8B%E5%AF%AB%E7%AD%86%E8%A8%98/server-side-rendering-ssr-in-reactjs-part3-7f2097963754)
5. [React SSR | 從零開始實作 SSR — 載入資料篇 - Airwaves](https://medium.com/starbugs/react-%E7%94%A8%E5%AF%A6%E4%BD%9C%E4%BA%86%E8%A7%A3-server-side-rendering-%E7%9A%84%E9%81%8B%E4%BD%9C%E5%8E%9F%E7%90%86-c6133d9fb30d)
