# 【筆記】GraphQL 系列(四) - GraphQL Apollo Testing use react-testing-library

###### tags: `筆記文章`

![](https://i.imgur.com/6Y4YpX9.png)

在前幾篇文章中我們已經學會如何使用 Apollo query 與 mutation 的方法來寫了一個『留言板』頁面與『通知中心』頁面，如果忘記內容的話可以回來參考[【筆記】GraphQL 系列(二) - 瞭解 Apollo Client 與 Apollo cache 機制](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/rkJ0kqGQc)。

那在實務上，每當我們要 Release 新版本到 Production 上時，常常都需要檢查一下網站中各頁面、功能是否正常，或是各個 Component 是否有如預期般的渲染，這往往都需要花上一些時間來檢查整個 flow，甚至可能常會有遺漏的情況發生或是 QA 人員無法檢查的情況(ex.邏輯、樣式...等)

因此我們是不是能在開發過程中就先簡單寫個測試，並且在每次 push code 之前就先跑一次單元測試，確保『邏輯』與『畫面(HTML 結構)』是如我們所想像的，最後再將程式碼 push 上去，減少在人工測試的過程中有遺漏的情況。

#### 那在 Apollo 的架構中，我們又該如何測試元件呢？就讓我們繼續看下去～～

## Apollo 測試概述

![](https://i.imgur.com/iz5Tdgf.png)

在 Apollo Client 的架構中不外乎就是需要測試 Query 與 Mutation 這兩個操作情境中的各種情況，並且檢查結果是不是跟我們預期的一樣，因此這邊各別列出幾個測試情境：

- **Query 的部分，主要測試以下『三種』條件下畫面是否正常渲染：**

  1. **『讀取中』的情況下畫面是否正常顯示 Loading 內容**
  2. **『發生錯誤』的情況下畫面是否正常顯示 Error 內容**
  3. **『正確取得到資料』的情況下畫面是否正常顯示『部落格』內容**

- **Mutation 的部分，主要測試以下功能是否正常：**
  1. 『**新增』功能是否正常，新增後比數是否增加，且是否為該筆資料。**
  2. **『刪除』功能是否正常，刪除後比數是否減少。**

> 補充：這邊將使用專案中的『部落格』頁面來當作範例進行測試，並且使用 react-testing-library 來將整個頁面的結構記錄成 snapshots，之後將透過比對整個 snapshots 檔案來檢查元件是否被更新且是否符合預期...等判斷。
>
> ![](https://i.imgur.com/boudVeU.png)

## 安裝測試套件 & 環境設定

```javascript=
yarn add @testing-library/react @testing-library/user-event jest
```

> 補充：在安裝 @testing-library/react 時可能會碰到專案 React 版本太舊的問題，因為目前 @testing-library/react v13 的版本不支援 React<=17 的專案，因此如果是小於 18 的專案麻煩請先安裝`12.1.2`。
>
> 相關文章可參考 [stackoverflow - Cannot find module 'react-dom/client' from 'node_modules/@testing-library/react/dist/pure.js'](https://stackoverflow.com/questions/71713405/cannot-find-module-react-dom-client-from-node-modules-testing-library-react)

#### package.json 增加測試 script

```javascript=
/* package.json */
  "scripts": {
    "dev": "next",
    "generate": "graphql-codegen",
    "build": "next build",
    "start": "next start"
    "test": "jest" // 多增加這行，方便之後直接透過 yarn test 來進行測試。
  },
  "jest": {
    "testEnvironment": "jsdom", // 解決問題二
    "moduleNameMapper": {
      "^@/(.*)": "<rootDir>/$1" // 解決 testing alias 找不到問題
    }
  },
```

### 環境問題

1. **Consider using the "jsdom" test environment**
   解答：[Consider using the "jsdom" test environment](https://stackoverflow.com/questions/69227566/consider-using-the-jsdom-test-environment)
2. **Test environment jest-environment-jsdom cannot be found. Make sure the testEnvironment configuration option points to an existing node module.**
   解答：安裝 `jest-environment-jsdom` 套件。`yarn add --dev jest-environment-jsdom`
3. **專案設定 alias 導致 Jest 找不到 module 問題**
   解答：如果專案內有設定 alias 的話，可能會導致 Jest 在執行過程中找不到 module 的問題。如果碰到的話，可以透過上面的`moduleNameMapper`設定來解決。[參考文章 - Testing with Jest and Webpack aliases](https://stackoverflow.com/questions/42629925/testing-with-jest-and-webpack-aliases)

## 使用 Apollo MockedProvider 測試

還記得我們在建置 Apollo 環境的時候會在專案中最外層 (ex. `app.js` or`index.js` or `_app.js`) 加上 **ApolloProvider** 這個 Component 嗎？它可以說是 Apollo 架構中最核心的部分，透過 ApolloProvider 來讓 Apollo Client 與 React 彼此之間互相溝通，這也就是為什麼我們在前端能夠使用一些 GraphQL 語法、方法...等功能。

> 補充：**Apollo Client** 上可以設定不少功能，像是**透過 HTTP Link 來發送 GraphQL 結構的 HTTP Request**、**cache 機制**、**localState**(可以想像成取代 Redux 做全局 State)，可以依照實際情況去擴充。

但現在我們只需要『測試元件功能是否正常』，或是『有沒有正常 render 出預想的結果』，所以並不需要實際去發送 HTTP Request。但『如果不發送 Request 我要如何模擬這些資料、這些情境呢？』，因此 Apollo 官方就提供了 [**MockProvider**](https://www.apollographql.com/docs/react/development-testing/testing/#the-mockedprovider-component) 這個 Component 來方便我們『模擬』整個情境。

### MockedProvider 介紹

要使用 MockedProvider 很簡單就是直接包在你想要測試的 Component 外層即可，仔細觀察可以發現它有個『**mocks**』的參數，它主要是用來告訴 Apollo 說：『當今天碰到 query 是 **xxx 時**，就回傳這些 **假** 資料給我』

#### 老樣子直接先看官方範例吧！

```jsx=
const mocks = [
  {
    request: {
      query: GET_DOG_QUERY,
      variables: {
        name: "Buck"
      }
    },
    result: {
      data: {
        dog: { id: "1", name: "Buck", breed: "bulldog" }
      }
    }
  }
];

 <MockedProvider mocks={mocks} addTypename={false}>
    <Dog name="Buck" />
 </MockedProvider>
```

#### 以上面的例子來解釋大概就是：

當今天 `Dog` 元件裡面碰到 `GET_DOG_QUERY` 這個 Query 且參數為`{name:'Buck'}` 時，就幫我回傳 `data: {dog: { id: "1", name: "Buck", breed: "bulldog" }}` 這些資料給我。

## 部落格 Query 測試

上面大致上介紹了 **MockedProvider** 該如何使用以及核心參數後，現在我們該回歸本次的主題，測試一下整個『部落格』頁面是否正常了，頁面的測試不外乎就是測試『畫面(query)』與『各項操作(mutation)』是否符合預期，那我們就先來測試『畫面(query)』的部分吧。

#### 以下將測試 3 種情況：

1. 讀取中 Loading Snapshots 是否符合預期結構
2. 錯誤 Error Snapshots 是否符合預期結構
3. 收到資料後 Snapshots 是否符合預期結構

> **個人見解：**
>
> 對於畫面上的測試，我個人是覺得可以直接測試該情況的 **Snapshot**，好處是可以直接確認整體 render 出來的結構是不是我們預設的情況，且當未來有結構被更動時也可以直接察覺。
>
> 舉例來說：如果今天只測試某個『Button 的文字』是不是等於『送出』，我們可能的做法會用 `expect(Button.text).toBe('送出')` 來比對，但當今天突然更改了 attribute 或是增加了某個 class 時，這時在測試方面就會因為只比對『文字』所以依舊會通過測試，但如果是使用 Snapshot 來做判斷的話就會因為這些變化而產生錯誤提醒。
>
> 當然這兩種各有好有壞，至於要使用哪個測試方式來實作，可能還是需要看公司專案的情況與碰到事情來決定，那本篇主要會以 **Snapshot** 的方式進行。

### 測試 - 讀取中 Loading Snapshots

在開始測試之前，先來教大家一個比較快的方式去做假資料，那就是直接先把專案跑起來，然後直接把 Browser -> Network 裡該 Query 的 response 內容貼到 `result` 的結構裡面。

![](https://i.imgur.com/SA83WHJ.gif)

那這邊為了防止整個貼上後程式碼變得很長、不好閱讀，所以寫了一個 `createPostData` function 來簡化一下資料長度，才不會整隻檔案『肉肉ㄉㄥ ˊ』。

```javascript=
// createPostData 用來產每一筆 post 假資料
function createPostData(id, title, content, authorName) {
  return {
    id,
    title,
    content,
    author: {
      id,
      name: authorName,
    },
    __typename: 'Post',
  }
}
```

#### 所以整個『部落格』的 Mock Data 大概就會這樣 :arrow_down:

```javascript=

const mockBlogData = [
  {
    request: {
      query: ALL_POST_QUERY,
      // variables : 如果輸入參數為 {} 則 result 為以下....
      variables: {},
    },
    result: {
      data: {
        viewAllPost: [
          createPostData('1', '好想出去玩', '天氣真好', 'library'),
          createPostData('2', '好想買遊戲王卡', '青眼白龍勒～～', 'Zack'),
          createPostData('3', '好想打遊戲', '殺～～～', 'Wang'),
        ],
      },
    },
  },
]
```

#### 整個測試的程式碼大概會長 :arrow_down:

透過使用 react-testing-library 的 `render` 來解析整個 `MockedProvider` 與需要測試的元件(`Index`)，這邊我們把解析完的 container 內容 Snapshot 起來並進行比對。

```jsx=
// 測試 Loading 情況
it('1. Test Blog Loading Situation', async () => {
  const { container } = render(
    <MockedProvider mocks={mockBlogData}>
      <Index />
    </MockedProvider>
  )
  expect(container).toMatchSnapshot()
})
```

#### Snapshot 結果 :arrow_down:

仔細看會發現，顯示的是 **Loading** 狀態的內容，因為『我們在操作 GraphQL 時，是屬於非同步的狀態(Promise-based)』，因此如果我們直接 `expect(container).toMatchSnapshot()` 就只會拿到 Loading 狀態的內容，而如果要取得資料的話則需要『做一個 Promise 去等待操作完成』。這部分牽扯到『event queue、call stack』概念這邊就不多說了，可以參考[【筆記】Event loop、Call Stack 與 Task Queue](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/S1X5WxOSU)。

![](https://i.imgur.com/IpAtlFz.png)

### 測試 - 收到資料後 Snapshots

上面測試了 Loading 狀態也講到了『如果要取得資料需要**做一個 Promise 去等待操作完成**』，因此這邊我們就把 Promise 的部分加上然後再跑一次 Snapshots 看看畫面變成怎樣吧！

#### 測試程式碼 - 增加 Promise code

```jsx=
// 測試資料顯示情況
it('3. Test Blog Data Situation', async () => {
  const { container } = render(
    <MockedProvider mocks={mockBlogData}>
      <Index />
    </MockedProvider>
  )
  // 主要新增這行～～～～～
  // 因為我們在操作 GraphQL 是屬於非同步的狀態(Promise-based)，因此需要做一個 Promise 去等待操作完成
  // 這邊有額外使用 react-testing-library 的 waitFor 方法，讓整個測試必須整個 callback 完成後再進行下去(專門處理複雜情況，例如：等待資料回來後再額外進行其他判斷)。
  // 但這邊情境比較單存，因此可以直接用 await new Promise((resolve) => setTimeout(resolve, 0))
  await waitFor(() => new Promise((resolve) => setTimeout(resolve, 0)))
  expect(container).toMatchSnapshot()
})
```

#### Snapshot 結果 :arrow_down:

從 Snapshot 上可以看到，剛剛所做的那些『假資料』都正常 render 出來了。

![](https://i.imgur.com/RHCsGIt.gif)

### 測試 - 錯誤 Error Snapshots

如果要測試 Error 的情況，我們只需要將原本 Mock Data 的 `result` 改成使用 `error` 並且丟出一個 `new Error` 就可以觸發錯誤狀態。

#### 測試程式碼 - 調整 Mock Data 回傳 Error

```jsx=
const mockBlogErrorData = [
  {
    request: {
      query: ALL_POST_QUERY,
      variables: {},
    },
    error: new Error('系統發生錯誤，請再重試一次'), // 這邊改成使用 error 來觸發 Error 狀態。
  },
]
// 測試 Error 情況
it('2. Test Blog Error Situation', async () => {
  const { container } = render(
     // 這邊替換成使用  mockBlogErrorData
    <MockedProvider mocks={mockBlogErrorData}>
      <Index />
    </MockedProvider>
  )
  await waitFor(() => new Promise((resolve) => setTimeout(resolve, 0)))
  expect(container).toMatchSnapshot()
})
```

#### Snapshot 結果 :arrow_down:

仔細觀察可以看到，畫面顯示了我們丟出的錯誤訊息『系統發生錯誤，請再重試一次』。

![](https://i.imgur.com/BPepC2N.png)

## 部落格 Mutation 測試

剛剛在上面的內容主要都是在介紹關於『取資料(Query)』的部分，而現在要進入到『使用者操作的環節』，例如：新增文章、刪除文章...等操作。

### 測試 - 新增文章

在開始測試『新增』功能之前，我們先來看一下新增文章的 Demo 流程。

![](https://i.imgur.com/PAj9kik.gif)

整體步驟大致為『點擊新增按紐』->『顯示彈窗』->『輸入表題(Title)』->『輸入內容(Content)』->『點擊確認按鈕』->『列表顯示內容』。

> **提醒：** Mutation 跟 Query 比較不一樣的地方在於我們需要對元件進行一些操作，例如：『找到按鈕』並『觸發點擊事件』，因此這邊會用到 `queryByText` 與 `userEvent` 這兩個方法來達成這部分。

#### 測試新增的 Mock Data :arrow_down:

```javascript
const mockAddPostData = [
  {
    request: {
      query: ADD_POST_QUERY,
      // variables 可以想像是 我們去預設當輸入參數為 多少時(variables)，則回傳值會是怎樣...
      // 例如： 當輸入參數為 {title:'',content:'',authorId:1} 的話，則回傳 createPostData('1', '測試新增文章', '測試內容', 'library')
      // 因此我們可以設計多個 variables 對應 result 的情況。
      variables: {
        title: '測試新增文章',
        content: '測試內容',
        authorId: 1,
      },
    },
    result: {
      data: {
        addPost: [createPostData('1', '測試新增文章', '測試內容', 'library')],
      },
    },
  },
]
```

#### 測試程式碼 :arrow_down:

```jsx=
// 測試新增 mutation
it('4. Test Add new Blog Situation', async () => {
  const user = userEvent.setup()
  const { container } = render(
    <MockedProvider mocks={mockAddPostData} addTypename={false}>
      <Index />
    </MockedProvider>
  )
  // 找到 新增文章 按鈕
  const addPostButton = await screen.queryByText('新增文章')
  // 等待 addPostButton 被點擊後，打開 PostDialog 彈窗
  await user.click(addPostButton)
  // 找到 PostDialog 標題 input
  const postDialogTitleInput = await screen.getByLabelText('postDialog-title').querySelector('input')
  // 找到 PostDialog 內容 textbox
  const postDialogContentInput = await screen.getByLabelText('postDialog-content')
  // 找到 PostDialog 彈窗裡的『送出』按鈕
  const addPostSubmitButton = await screen.queryByText('送出')
  // 標題 填入『測試新增文章』
  await user.type(postDialogTitleInput, '測試新增文章')
  // 內容 填入『測試內容』
  await user.type(postDialogContentInput, '測試內容')
  // 等待 送出按鈕 被點擊
  await user.click(addPostSubmitButton)
  // 等待 mockProvider 非同步執行
  await waitFor(() => new Promise((resolve) => setTimeout(resolve, 0)))
  // 執行 Snapshot
  expect(container).toMatchSnapshot()
})
```

> **補充：檢測看看是否有正確找到元件**
>
> 在寫測試時，如果想確認是否有正確的抓到 DOM 的話，可以使用 console 搭配 **prettyDOM** 來檢測看看是否有正確找到。
>
> 像是想抓取 **Material UI** 的 `<TextField>` 這個輸入框時，如果透過 prettyDOM 來看會發現抓到的一個 `div` 裡面包著 `input`，因此需要再往內查詢，才會正確抓取 **input**。
>
> ```javascript=
> console.log('addPostButton', prettyDOM(addPostButton))
> ```

#### Snapshot 結果 :arrow_down:

從 Snapshop 檔案上可以看到 `mockAddPostData` 的 Result Data 顯示在 Snapshop 上，代表新增功能正常執行。

![](https://i.imgur.com/eH6cRGh.png)

## 結語

這次透過 Apollo 官方提供的 `MockProvider` 來對 Query 與 Mutation 等操作進行單元測試，當然這不是唯一的測試方式，像是 [Testing Apollo Components using react-testing-library - Andrew Hansen](https://www.arahansen.com/testing-apollo-components-using-react-testing-library/) 這篇文章就是依舊使用專案內的 `ApolloProvider`，透過創建新的 `ApolloClient` 並且在 `resolvers` 中依照 Schema 結構塞入『假資料』來進行測試。

而這兩種測試方式都各有優缺點，且在實作方面也不太相同，因此建議可以兩種類型的文章都看一下，再決定要導入哪種方案到專案中。

#### 以上就是這次【 GraphQL Apollo Testing 】的全部內容，如有任何錯誤或冒犯的地方還請各位多多指教，謝謝您的觀看。

#### Github : [https://github.com/librarylai/GraphQL-Blog](https://github.com/librarylai/GraphQL-Blog)

## Reference

1. [Mocking Apollo GraphQL Queries in React Testing - Nick Scialli](https://typeofnan.dev/mocking-apollo-graphql-queries-in-react-testing/)
2. [Testing Apollo Components using react-testing-library - Andrew Hansen](https://www.arahansen.com/testing-apollo-components-using-react-testing-library/)
3. [Testing React components - Apollo](https://www.apollographql.com/docs/react/development-testing/testing/#example-1)
