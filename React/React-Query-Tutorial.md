# 【筆記】 React Query 教學

###### tags: `筆記文章`

![](https://i.imgur.com/S8tBXzQ.png)

最近在公司的專案上碰到優化列表頁的需求，最後討論使用 React Query 來做前端列表頁的 cache 機制，減少呼叫 API 次數直到過期後才重新呼叫，本篇筆記是紀錄【閱讀 React Query 官方文件】與【套用在專案上】時，所碰到問題的筆記。

## React Query 介紹

React Query 是一套用來方便我們處理 server state 的 Library，它使我們在處理 **fetching, caching, synchronizing and updating server state** 更加簡單。

一般我們使用 React 開發時其實沒有一個很固定的 call api 方式或是更新資料的方式，因此我們常建立自己的呼叫 api 方式，這邊指的不是 function 而是指流程或是習慣。例如: 透過 React hooks 的 useEffect 來呼叫 API 與更新 component 的 state 或透過一些狀態管理工具 (Redux) 來管理整個專案的狀態與非同步獲取資料。

一般大多數【**狀態管理**】的 library 都較擅長處理 client state，而在處理 async 或 server state 方面就不太擅長，因為 server state 較為非同步處理且不能去控制它，**簡單來說就是: 它是會被變動的，且無法保證是最新的。**

#### 補充 : 何謂 server state:

- Is persisted remotely in a location you do not control or own
- Requires asynchronous APIs for fetching and updating
- Implies shared ownership and can be changed by other people without your knowledge
- Can potentially become "out of date" in your applications if you're not careful

我們一般在 React 專案中都會使用 Redux 來管理 Global State，而在較複雜的專案裡面都應該有碰過需要非同步獲取的資料並儲存到 Redux 裡，這時就需要透過其它的套件來處理這類非同步的 state ，例如 redux-thunk、redux-saga...等，而也就造成了 client state 與 server state 都混在同一個 store 裡面。

**我們可以看看作者 Tanner Linsey 在 [reddit](https://www.reddit.com/r/reactjs/comments/ds0hp4/react_query_hooks_for_fetching_caching_and/) 如何解釋 Redux Query 與 Redux 的不同點，以及對於它們的看法：**

> React Query is significantly different from using Redux. Redux is a global state manager with it's own context etc. React Query to some extent is also a global state manager for async data. If you were to integrate the two, you would move all of your async data out of redux into React Query and only use Redux for non-async-data global application state. In fact, this is exactly what we did at Nozzle.io for a while, until we moved away from Redux entirely. Now we just use React Query for all of our remote data and `useReducer` + React Context for global UI state.

簡單來說 React Query 是用來管理 Redux 裡面有關非同步資料的部分，我們可以將 Client State 與 Server State 分離，非同步相關的部分就交給 React Query，而 Redux 只處理同步的全局狀態(Client State)。

## React Query Devtools

當我們要開始使用 React Query 時，不免俗一定要來找找有沒有 Devtools 可以使用，就像我們在使用 Redux 時一樣會安裝 Redux Devtools 來查看目前 store 裡面的資料一樣，而 React Query 很貼心的也有提供這項功能，方便我們看到它內部的運作減少 debug 的時間。

### 啟動 Devtools

要使用 React Query Devtools 很簡單，只需要從 `react-query/devtools` 中引入`ReactQueryDevtools`並放置到根組件底下( App.jsx )，不需要額外安裝任何套件。

```javascript=
import { ReactQueryDevtools } from 'react-query/devtools'

function App() {
   return (
     <QueryClientProvider client={queryClient}>
       {/* The rest of your application */}
       <ReactQueryDevtools initialIsOpen={false} />
     </QueryClientProvider>
   )
 }

```

Devtools 預設只會在 `process.env.NODE_ENV === 'development'` 的情況下才會出現，所以不用擔心它會在 production 環境中出現。

官方還有提供一些 options 讓我們可以額外設定 Devtools，例如上面範例的 `initialIsOpen` 就代表初始時是否要直接打開 Devtools 畫面，詳細內容這邊就不贅述了，直接提供 [React Query Devtools](https://react-query.tanstack.com/devtools) 官方文件給大家

### Devtools 畫面與運作流程

![](https://i.imgur.com/InrM95Y.png)

這邊直接截取官方範例的圖片，我們可以看到上面有 `fresh`、`fetching`、`stale`、`inactive` 四種狀態，這邊分別介紹一下每個狀態代表的意思。

- **fresh** : 代表資料尚未超過 staleTime 還是新鮮狀態，當再次進入頁面時**不會**呼叫 API。
- **fetching** : 代表正在抓取資料中...
- **stale** : 代表資料已經過期，當再次進入頁面時會**再次**呼叫 API 取得新資料。
- **inactive** : 代表目前被 cache 中的資料，當資料超過 cacheTime 時將會從 inactive 中被移除。

## Queries 介紹

Query 主要能將非同步取回來的資料 (responce) 綁定到一個 unique key 上，它可以跟任何基於 Promise 上的方法 (fetch,axios) 一起使用，來從 server 端獲取資料。但如果是想要對 server 端的資料進行操作，官方建議使用 [Mutations](https://react-query.tanstack.com/guides/mutations) 的方式來做更動。

> **簡單來說：Query 主要是代表查詢的操作，我們會傳入一個 unique key 以及一個 query function(必須回傳 prmoise) 當作參數，且 Query 會將非同步回傳的資料與 unique key 綁定(cache)，如果在資料過期之前再次傳送一樣的 unique key 時，Query 會直接先回傳原本的資料，並再背景裡幫我們呼叫 API，讓使用者能先看到畫面而不是白屏，提高使用者體驗。**

### useQuery hook 介紹

> - A unique key for the query
> - A function that returns a promise that:

    * Resolves the data, or
    * Throws an error

useQuery hook 主要有兩個參數，第一個參數是 unique key 用來代表這個 query 的身分證(對於 unique key 介紹可以看下方補充說明)，第二個參數是用來獲取非同步資料的 function 。

**unique key 補充說明:**

> The unique key you provide is used internally for refetching, caching, and sharing your queries throughout your application.

### useQuery hook 使用

在 useQuery 的使用上面，可以使用物件的方式或是參數的形式來呼叫使用。

```javascript=
// first params is query key
// second params is query function

const result = useQuery('todos', fetchTodoList,{...config})
// or using the object syntax
const result = useQuery({
    queryKey:[],
    queryFn:()=>{},
    ...config // 其他 config 設定 ex.
})
```

### Query Keys 說明

React Query 主要是透過 query keys 來管理 query 過後的 caching，query keys 可以是一個**字串、或是陣列(裡面可包含字串、物件、數字...等)**，只要它是讀一無二的就可以使用。

```javascript=
// string
useQuery('todos', ...) // queryKey === ['todos']
// array
useQuery(['todo', 5], ...) // queryKey === ['todo', 5]
// array , object
useQuery(['todo', 5, { preview: true }], ...) // queryKey === ['todo', 5, { preview: true }]
```

#### 官方補充:

1. 物件內的位置互換會被是為【同一種】，陣列內的位置互換則【不會】

```javascript=
// object 位置交換 依舊是 同一種
useQuery(['todos', { status, page }], ...)
useQuery(['todos', { page, status }], ...)
// array 位置交換 就是不同
useQuery(['todos', status, page], ...)
useQuery(['todos', page, status], ...)
```

2. 如果你的 Query function 有依賴變數，你應該要把變數包含在 Query Keys 中

```javascript=
// todoId 傳給了 fetchTodoById(todoId) 當作參數，則應該在 query keys 補上 todoId，這樣才會隨著 todoId 而改變。
 function Todos({ todoId }) {
   const result = useQuery(['todos', todoId], () => fetchTodoById(todoId))
 }
```

### Query Functions 說明

Query Functions 可以是任何返回 Promise 的函式，且返回的 Promise 必須要能 resolve the data 或是 throw an error。

> 簡單來說：這個 Promise 內要有呼叫`resolve()` 或是呼叫 `reject()` 這樣 Query 內部執行到 Promise 時才能夠透過 `.then` 或是`.error` 正確抓出其值。

```javascript=
/* React Query 官方 */
useQuery(['todos', todoId], fetchTodoById)
useQuery(['todos', todoId], () => fetchTodoById(todoId))
useQuery(['todos', todoId], async () => {
const data = await fetchTodoById(todoId)
return data
})
```

#### Query Function Variables

Query keys 不僅僅是當做唯一識別鍵，它預設還會當作 Query Function 的第一個參數。

```javascript=
function Todos({ status, page }) {
   const result = useQuery(['todos', { status, page }], fetchTodoList)
 }

// Access the key, status and page variables in your query function!
function fetchTodoList({ queryKey }) {
    const [_key, { status, page }] = queryKey
    return new Promise()
}
```

### 常用 query options 說明

1. **staleTime** : 代表資料的過期時間(預設為 0)，如果設定為 Infinity 則永遠不會過期。簡單來說 staleTime 代表需要隔多久呼叫 API，預設為 0 代表每一次都還是會 call API，可以透過 Network 查看。
2. **cacheTime** : 代表資料被 garbage collect 回收的時間(預設為 5 分鐘)。簡單來說 cacheTime 就是當我們呼叫完 API 獲取資料後，資料會自動被 cache 直到超過 cacheTime 的時間才會被刪掉。
3. **refetchOnWindowFocus** : 預設為(true)，有以下三種值
   - true : 當視窗被 focus 且資料過期時，則會重新獲取資料(refetch)
   - false : 不管視窗怎麼 focus 都不會重新獲取資料
   - always : 只要視窗被 focus 就重新獲取資料
4. **refetchOnReconnect** : 預設為(true)，有以下三種值
   - true : 當重新連線且資料過期時，則會重新獲取資料(refetch)
   - false : 不管怎麼重新連線，都不會重新獲取資料
   - always : 只要重新連線，就重新獲取資料

### useQuery 回傳值

```javascript=
 const {data, error, status, isLoading, isError, isSuccess,  isFetching } = useQuery('todos', fetchTodoList)
```

useQuery 回傳的 result 主要包含了以下屬性:

> data : 代表 fetch 完後的 response
> error : 代表錯誤訊息
> status : 代表各階段狀態
> isFetching : 代表只要是在呼叫 API 的情況下都是 true ( 包含 background refetching )
> isLoading : 等於 status === 'loading'，代表當還沒有 cache 資料且正在 fetch 當中，可以當作像是 componentDidMount 第一次呼叫時。
> isError : 等於 status === 'error'，代表途中遭遇到錯誤
> isSuccess : 等於 status === 'success'，代表成功獲取資料
> isIdle : 等於 status === 'idle'，代表目前 query 被禁用( 不會執行 query function )

## Dependent Queries

當我們今天要呼叫 Query function 之前要等待一些資訊的話，例如 : 等待先前的 API 完成，或是要符合某些判斷才可以呼叫 API ...等，這時就可以使用 `enabled` 這個 option 來解決這部份的問題。

### 實際舉例

今天我們要有 【啟始時間】 與 【結束時間】後才呼叫 API 的話，我們可能會很直覺的這樣寫 :arrow_down:

```javascript=
/* 這邊舉例為 錯誤寫法 會造曾多次 onSuccess */
const query = useQuery(
    ['TestQuery', {startTime,endTime}],
    () => fetchSomeData,
    {
        onSuccess: () => {
            console.log('inSuccess')
            isFirstRender.current = false
        },
    }
)

/* 一般我們直覺上的寫法 */
function fetchSomeData(){
    if(startTime && endTime){
      // 如果 startTime 與 endTime 有值則進入
        return fetchAPI()
    }
}
```

但這樣寫法會有一個問題，它會造成多次進入 `onSuccess`，雖然它沒有符合條件不會呼叫 API，但它依舊有執行了 `fetchSomeData` function，而【**當 function 沒有寫 return 時，其實等於是 return undefined**】([參考連結](https://stackoverflow.com/questions/51059291/javascript-function-no-return-statement-identical-to-return-undefined))，所以 useQuery 就會以為你已經【完成】了，而將 `status` 變為`success` 進額呼叫了 `onSuccess` function。

![](https://i.imgur.com/XzKocuN.png)

### 解決辦法

會造成這樣的原因是因為 useQuery 一直執行了 Query Function，而我們要解決這樣的問題就得從最根本的地方解決，那就是【**某些條件下直接不要呼叫 Query Function**】，而 [Dependent Queries](https://react-query.tanstack.com/guides/dependent-queries) 這章節就介紹了 `enalbed` 這個 option 讓我們可以告訴 useQuery 在符合某些條件時再執行 query function。

```javascript=
/* 改寫 使用 enabled option */
const query = useQuery(
    ['TestQuery', {startTime,endTime}],
    () => fetchSomeData,
    {
        enabled: Boolean(startTime && endTime),
        onSuccess: () => {
            console.log('inSuccess')
            isFirstRender.current = false
        },
    }
)

function fetchSomeData(){
    // The query will not execute until startTime、endTime exists
    return fetchAPI()
}
// isIdle will be `true` until `enabled` is true and the query begins to fetch.
// It will then go to the `isLoading` stage and hopefully the `isSuccess` stage :)
```

改寫完後為，當符合 `enabled` 的條件時才會執行 `fetchSomeData`，而當 `enabled` 為 false 時 `status` 會變為 `idle`，代表該 query 目前是被 disabled 的狀態(不可執行)，因此就可以防止每次都被呼叫的問題瞜!!!

![](https://i.imgur.com/7HjPdq3.png)

## Mutations 介紹

剛剛上面介紹 useQuery 主要都是在一開始進入頁面或是監聽的參數更改時就重新執行 Query Function 取得新的資料(自動觸發)，但有時我們只是要在點擊按鈕時才需要呼叫 API 去更新資料的情況(手動觸發)，這時就可以用到 useMutation 這個函式去幫我們發起更新(update)。

useMutation 通常用於 create/update/delete 資料或是執行 server side-effects 使用，透過 `mutation.mutate` 去更新資料以及 `mutation.data` 去取得新的資料。

### Mutation 使用

這邊舉一個在專案中碰到的例子，我們有一個 Table 的收益表格，它需要監聽一些條件參數(ex. 起始時間、結束時間、篩選條件、換頁...等)，但它也需要在某些情況【手動的】去更新 Table 內的資料(ex.點擊 Table row 時)，這時我們就不僅要使用到 `useQuery` 還需要使用到 `useMutate` 才可以達成我們的需求。

![](https://i.imgur.com/7pKwGg4.png)

簡單做了張示意圖可能會比較好瞭解，【紅色】圍住的區塊為被監聽的部分，例如當切換分頁時會透過 useQuery 重新取得整個 Table 資料，【藍色】區塊為點擊後會觸發 Mutation 去呼叫 API 取得子項目的資料，【綠色】區塊為呼叫 Mutation 完成後，透過 `onSuccess` 將子項目資料更新進 Table 後的畫面。

#### 範例程式碼

```javascript=
/* 這邊程式碼只是大約模擬實際情況，有些簡陋請見諒 */
/* 使用 useQuery 部分 */
const tableStatementsQuery = useQuery(
    ['tableStatementsQuery',page, startTime, endTime],
    fetchTableStatementsAPI
)
/*
 * 使用 useMutation 部分
 *
 * params 為呼叫 mutate 時傳入的參數
 * onSuccess : 第一個參數為 api responce data ，第二個參數為我們傳入的 params
 *
 * */
const tableStatementsMutation = useMutation((params) => fetchGroupingStatementsAPI(params), {
    onSuccess: (data, params) => {
       /* 更新 Table 資料 */
       setState({
           tableData: data
       })
    },
})



/* 資料顯示 */
function renderTable(){
    return(
        <Table data={tableData} columnClick={handleColumnClick}
    )
}
// table column 點擊
function handleColumnClick(){
    /* column 被點擊時觸發 mutation */
    let params = getSomeParams()
    tableStatementsMutation.mutate(params)
}

/* tableStatementsQuery 更新時會更新  setTableData */
const setTableData = useCallback(() =>{
    setState({
         tableData: tableStatementsQuery.data
     })

},[tableStatementsQuery.data])

// setTableData 更動時，重新觸發 useEffect
useEffect(()=>{
    setTableData()
},[setTableData])


```

### mutation.mutate 使用注意

1. **Mutation 傳入參數**
   在使用 useMutaion 與 mutation.mutate() 時需要特別注意到參數的部分，mutaions function 只可傳入一個參數，所以當多要傳入多個參數時，就要使用 object 的方式包裝起來。

   ```javascript=
   // 創建
   const mutation = useMutation(newTodo => axios.post('/todos', newTodo))

   // button onClick
   <button onClick={() => {
       mutation.mutate({ id: new Date(), title: 'Do Laundry' })
   }}>
    Create Todo
   </button>
   ```

2. **Mutation 抓取 Event**
   如果您的專案是 React v16 或更早的專案可能要注意 [React event pooling](https://reactjs.org/docs/legacy-event-pooling.html) 的問題。
   因為 mutate 是一個非同步的函式，所以不能直接在 event callback 上使用，如果要使用則需要先將 mutate 包在另一個函式內再來使用。

   ```javascript=
   // not work
   <form onSubmit={mutation.mutate}>...</form>

   // work
   const onSubmit = event => {
    event.preventDefault()
    mutation.mutate(new FormData(event.target))
   }

   <form onSubmit={onSubmit}>...</form>
   ```

### Mutation 回傳值

Mutation 的回傳值跟 useQuery 一樣，都有 data, error, status, isLoading, isError, isSuccess, isFetching 屬性可以使用，可以直接參考上方 useQuery 解釋。

### Mutation Side Effects

useMutation 和 mutate 的第二個參數(options)裡也有一些關於 mutation lifecycle 上的操作，像是我們可以在 Mutation 完成時觸發 `onSuccess` 事件，或是對 query 進行 invalidating、refetching 或 updating 等行為。

```javascript=
/* onSuccess 參數解釋
 * data: 為 result data ( Mustation 的回傳值 )
 * variables: 呼叫 Mutation 時傳入的 params
 * */
onSuccess: (data: TData, variables: TVariables, context?: TContext)
```

以上面的實務為例，當 Mutation 執行完後透過 `onSuccess` function 將 responce data(`onSuccess` 的第一個參數) setState 到該 Component 的 State 裡面。

#### useMutation 與 mutate 執行差異

當兩邊都有設置同一個 callback function 時，彼此之間有執行順序之分且執行次數也不同，例如當 useMutation 與 mutate 都設定 `onSuccess` function 時，會先執行 useMutation 的 `onSuccess` function 再執行 mutate 的 `onSuccess` function。

```javascript=
useMutation(addTodo, {
   onSuccess: (data, variables, context) => {
     // I will fire first
   },
 })

 mutate(todo, {
   onSuccess: (data, variables, context) => {
     // I will fire second!
   },
 })
```

而如過是連續觸發多次 `mutate` 的情況，則 `useMutation` 與 `mutate` 的 callback 觸發次數不同，因為每次呼叫 `mutate` 時 mutation observer 都會重新訂閱一次，因此 `mutate` 的 callback 只會以『最後一次』呼叫的 `mutate` 為主，而 `useMutation` 的 callback 則是每一次都會被觸發。

```javascript=
/* 官方範例 */
 useMutation(addTodo, {
   onSuccess: (data, error, variables, context) => {
     // Will be called 3 times
   },
 })

 ['Todo 1', 'Todo 2', 'Todo 3'].forEach((todo) => {
   mutate(todo, {
     onSuccess: (data, error, variables, context) => {
       // Will execute only once, for the last mutation (Todo 3),
       // regardless which mutation resolves first
     },
   })
 })
```

## 實作碰到問題

1. [How to tackle No QueryClient set, use QueryClientProvider to set one in React Query v3](http://5.9.10.113/65972866/how-to-tackle-no-queryclient-set-use-queryclientprovider-to-set-one-in-react-qu)
2. [How to use useQuery hook from ReactQuery to update state?](https://stackoverflow.com/questions/64409464/how-to-use-usequery-hook-from-reactquery-to-update-state)
3. [React Hooks and React Query (useQuery): invoking the API call/ query only when a button is clicked](https://stackoverflow.com/questions/64770279/react-hooks-and-react-query-usequery-invoking-the-api-call-query-only-when-a)
4. 當 useQuery 的 options 都沒有設定時代表 staleTime 為 0，而在切換分頁時容易會碰到第一次點擊不會跳轉畫面的問題，因為 refetchOnWindowFocus 被觸發的緣故導致原本要執行的 function 沒有被正常執行。
   Answer : 設定 staleTime 或者 refetchOnWindowFocus 改為 false。
5. [How to Use Multiple Parameters in useMutation From React Query With TypeScript](https://medium.com/swlh/how-to-use-multiple-parameters-in-usemutation-from-react-query-with-typescript-7e2aeec51446)

## 結論

在實作 React Query 的過程中碰到了不少的問題，從一開始沒有在 root component 加上 QueryClientProvider 而導致報錯，之後在修改檔案上面從原本的使用 useEffect 當作 componentDidMount 或 componentDidUpdate 來呼叫 API 獲取資料，改為用 useQuery 來呼叫，但因為 hooks 只能放在最上層使用，導致在寫法上修改了不少。
我覺得在使用 React Query 上最大的挑戰應該是寫法上的改變、思考方式的切換，之前會放在 useEffect 處理的邏輯，可能在使用 React Query 時，因為寫法上的不同而會有些許的邏輯改變，總之還是得看需求以及專案的複雜度來判斷。

最後也感謝同事 **chihyang** 在讀書會上分享的 [React Query Tutorial](https://chihyang41.github.io/2020/09/07/React-Query-Tutorial/)。

#### 以上純屬個人使用心得與簡單對 React Query 的介紹，如有任何錯誤或冒犯的地方還請各位多多指教。

### 謝謝觀看。

## Reference

1. [React Query 官方](https://react-query.tanstack.com/overview)
1. [React Query Tutorial #1 - Intro & Setup | The Net Ninja ](https://www.youtube.com/watch?v=x1rQ61otgtU)
1. [React Query Tutorial #2 - The useQuery Hook | The Net Ninja](https://www.youtube.com/watch?v=yccbCol546c)
1. [React Query Tutorial #3 - React Query Dev Tools | The Net Ninja](https://www.youtube.com/watch?v=-i3ZQP1yPEs)
1. [React Query Tutorial #4 - Query Variables | The Net Ninja](https://www.youtube.com/watch?v=QwF7-K106eQ)
1. [React Query Tutorial #5 - Pagination | The Net Ninja](https://www.youtube.com/watch?v=K8kgh5C3WP8)
1. [React Query Tutorial - chihyang](https://chihyang41.github.io/2020/09/07/React-Query-Tutorial/)
