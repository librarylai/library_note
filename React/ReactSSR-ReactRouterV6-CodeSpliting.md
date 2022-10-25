# 【筆記】SSR 系列第二集【React Router v6 & Code Spliting】

###### tags: `筆記文章`

本篇是 SSR 系列的第二篇文章，如果對基本 SSR 還沒有什麼概念的讀者，歡迎看本系列的第一篇文章，本篇將會依舊沿用上一篇所建立的專案繼續實作。

[【筆記】SSR 系列第一集【運用 CRA 做 Server Side Rendering】](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/SyZ4NNNEF)
[【筆記】SSR 系列第三集【Redux Toolkit & GetServerSideProps 實作】](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/r1vn47vcY)

主要會分成以下兩個階段來講。

1. React Router 如何與 Sever-Side Rendering 結合
2. 運用 React Lazy 與 Loadable Component 做到 Code Spliting

**_本篇主要是以筆記的形式，紀錄實作中碰到的問題，如果有哪邊太跳 tone 或解釋錯誤的部分，再麻煩大大們提點。_**

## React Router 如何與 Sever-Side Rendering 結合

![](https://i.imgur.com/t41Il7H.png)

### 前端專案加入 Router 機制

在開始實作 SSR 結合 React Router 之前，首先當然是要先把 React Router 導入到我們的前端專案，因此...讓我們先來安裝 React Router 相關的套件吧！

**_提醒：本篇使用的是 React Router v6 的版本(部分寫法與 V5 不同)_**

```javascript=
yarn add react-router-dom @types/react-router-dom
```

1. **在 index.tsx 引入 BrowserRouter Component**
   `BrowserRouter` 是使用 HTML5 history API (pushState, replaceState and the popstate event) 讓 UI 跟 URL 保持一致

   ```javascript=
   /* index.tsx */
   import { BrowserRouter } from 'react-router-dom'

   ReactDOM.hydrate(
     <React.StrictMode>
       <BrowserRouter> // 這裡～～
         <App />
       </BrowserRouter>  // 這裡～～
     </React.StrictMode>,
     document.getElementById('root')
   )
   ```

2. **在 App.tsx 實作 Route 相關機制**
   這邊要特別注意 v6 以前是用 `<Switch>` 這個 component 在 v6 以後已改用 `<Routes>` 這個 component，而在 `<Route>` 在 props 的參數方面也從 `component` 改成使用 `element` 這個 props。

```jsx=
/* App.tsx */
import {
  Routes, // 這邊需注意 官方文件寫的 Switch 已經在最新版本被改用為 Routes
  Route,
  Link,
} from 'react-router-dom'

function App() {
  return (
    <div>
      <nav>
        <ul>
          <li>
            <Link to='/ExpandingCards'>ExpandingCards</Link>
          </li>
           {/* ...審略... */}
          <li>
            <Link to='/StepsPage'>StepsPage</Link>
          </li>
        </ul>
      </nav>
      <Routes>
        <Route path='/' element={<ExpandingCardsPage />} />
        {/* ...審略... */}
        <Route path='/StepsPage' element={<StepsPage />} />
      </Routes>
    </div>
  )
}
```

### 補充一下 React Route Dom V6 版本的更新步驟

[React Router Dom 官方更新文件](https://github.com/remix-run/react-router/blob/main/docs/upgrading/v5.md)

_如果只想看 v6 版本寫法的結論的話，可以直接往下滑到分隔線的部分。_

**1. Upgrade to React v16.8 or greater**
第一步是更新 React 到 16.8 或以上，因為 V6 版本主要是以 React hooks 的方式實作，所以 React 必須至少要更新到 React 16.8 支援 React hooks 才行。

**2. Upgrade to React Router v5.1**
在更新到 v6 之前，官方建議先更新到 v5.1 方便日後升級到 v6 時會比較順暢，因為 v5.1 與先前的 v4、v5 版本中有部分程式寫法有更改，以及有提供一些 hooks API 可以對 route 做操作( ex. params、location )。
例如 :

> `<Route children>` elements that will help smooth the transition to v6. Instead of using `<Route component>` and `<Route render>` props

```jsx=
// v4 and v5 before 5.1
<Switch>
  <Route path="/about" component={About} />
  <Route
    path="/users/:id"
    render={({ match }) => (
      <User id={match.params.id} />
    )}
  />
</Switch>

// v5.1 preferred style
{/* Can also use a named `children` prop */}
<Route path="/users/:id" children={<User />} />
```

#### Remove \<Redirect>s inside `<Switch>`

> Remove any `<Redirect>` elements that are directly inside a `<Switch>`
> If you want to redirect client-side, move your `<Redirect>` into a `<Route render>` prop.
> `<Switch>` 內請避免直接放 `<Redirect>`
> 如果要使用請將 `<Redirect>`放到 `<Route render>`中。

```jsx=
// Change this:
<Switch>
  <Redirect from="about" to="about-us" />
</Switch>

// to this:
<Switch>
  <Route path="about" render={() => <Redirect to="about-us" />} />
</Switch>
```

**3. Upgrade to React Router v6**
將專案升級到 v6 版本，以下節錄幾個比較大的更新

1. **Upgrade all \<Switch> elements to \<Routes>**

   React Router v6 引入了 `Routes` component 它就像是 `Switch` 一樣，但更加強大且比 `Switch` 有更多的優勢 :

   > - All `<Route>`s and `<Link>`s inside a `<Routes>` are relative. This leads to leaner and more predictable code in `<Route path>` and `<Link to>`
   >   使用 `<Routes>` 可使 `<Route path>` 和 `<Link to>` 的路徑更可被預測。
   > - Routes are chosen based on the best match instead of being traversed in order. This avoids bugs due to unreachable routes because they were defined later in your `<Switch>`
   >   根據最佳匹配選擇路線，而不是按順序匹配，避免了由於無法訪問的路由而導致的錯誤。
   > - Routes may be nested in one place instead of being spread out in different components. In small to medium sized apps, this lets you easily see all your routes at once. In large apps, you can still nest routes in bundles that you load dynamically via React.lazy
   >   Routes 可以統一巢狀的寫在一些地方，而不用分散在不同的 component 中，不管在大專案或小專案中，都可以更快的看到全部的路由且增加可讀性。

   **_為了使用 v6，需要將所有 `<Switch>` 元素轉換為 `<Routes>`。_**

2. **Relative Routes and Links**

   在路由的部分不像 v5 時要明確的寫出路徑，v6 則是以相對路徑的方式，它們會自動建立在 parent route's path 和 URL 上，我們就不用像以前一樣，還要手動的插入 `match.url` 或 `match.path`。如果有子路由，則只需在路徑中最後加上 \* 來表示深度匹配。

   ```jsx=
       // This is a React Router v6 app
   import {
     BrowserRouter,
     Routes,
     Route,
     Link
   } from "react-router-dom";

   function App() {
     return (
       <BrowserRouter>
         <Routes>
           <Route path="/" element={<Home />} />
            {/* 請看這行的 * 字符 */}
           <Route path="users/*" element={<Users />} />
         </Routes>
       </BrowserRouter>
     );
   }

   function Users() {
     return (
       <div>
         <nav>
           <Link to="me">My Profile</Link>
         </nav>
         <Routes>
           {/* 自動匹配子路由 => /users/:id */}
           <Route path=":id" element={<UserProfile />} />
           {/* 自動匹配子路由 => /users/me */}
           <Route path="me" element={<OwnUserProfile />} />
         </Routes>
       </div>
     );
   }
   ```

3. **Advantages of `<Route element>`**

   使用 `<Route element>` 代替 component 可以讓我們不用在提供 `passProps`-style API ，簡單來說，在以前使用 component 的方式要傳遞 props 到組件內，它的寫法是一件比較不直觀且官方也認為是比較不好的方式，因此在 v5.1 時官方就有承諾會討論如何去調整這一塊。

   例如 : 當我們要傳遞一個 props 到 `Profile` 這個組件時，因為 `component` 這個 props 是要傳進一個組件而不是一個 React element，所以需要透過額外設定 props (ex. passProps ) 來傳給組件，或是透過 `render prop` 、 `higher-order component` 來解決

   ```jsx=
   import Profile from './Profile' // 這是一個組件，為了避免與 component props 搞混，這邊先統稱這些為【組件】。

   // React Route dom v5
   <Route component={Profile} passProps={{ animate: true }} />


   // use render prop
   <Route
     path=":userId"
     render={routeProps => (
       <Profile routeProps={routeProps} animate={true} />
     )}
   />

   // What if I want to get access to the route match, or I need
   // to redirect deeper in the tree?
   function DeepComponent(routeStuff) {
     // got routeStuff, phew!
   }
   export default withRouter(DeepComponent);

   ```

   最後一個 `higher-order component` 的例子主要是在 hooks API 提供之前，為了要讓深層組件知道 route 相關的資訊，所以會借助 `withRouter` 這個 `higher-order component` 來幫忙，而現在可以直接透過 `useParams`、`useLocation`等等的 API 來解決。

4. **Use useRoutes instead of react-router-config**
   在 v5 時我們常會用 `react-router-config` 來統一管理 routes，而在新版的 v6 中 react-router 已將 `react-router-config` 這個套件加入到其中。

   _補充 : `react-router-config` 這個套件主要是能簡化我們在設定 router 的過程，透過建立 JavaScript Objects 結構的 routes 與使用套件提供的 `renderRoutes` function 來減少我們手動去一個一個建立 React Elements_

   React Router v6 提供了 `useRoutes` hook 來取代 `react-router-config`，只需將組好的 routes 結構放入到 `useRoutes` 這個 custom hook 中，就可以幫我們直接產出 Route 等相關的 Element，直接附上官方範例。

   ```javascript=
       function App() {
         let element = useRoutes([
           // These are the same as the props you provide to <Route>
           { path: "/", element: <Home /> },
           { path: "dashboard", element: <Dashboard /> },
           {
             path: "invoices",
             element: <Invoices />,
             // Nested routes use a children property, which is also
             // the same as <Route>
             children: [
               { path: ":id", element: <Invoice /> },
               { path: "sent", element: <SentInvoices /> }
             ]
           },
           // Not found routes work as you'd expect
           { path: "*", element: <NotFound /> }
         ]);

         // The returned element will render the entire element
         // hierarchy with all the appropriate context it needs
         return element;
       }
   ```

   ***

   **再講下去就偏離主題了，所以這邊直接總結一下 React Route v6 寫法**

   - **使用 element 取代 component**
   - **Route 恢復使用巢狀 Route**
   - **Route 的相對路徑依賴**

   ```jsx=
   // 1.使用 element 取代 component
   <Route path=":userId" element={<Profile animate={true} />} />

   // 1-2.使用 hooks API 取得參數
   function Profile({ animate }) {
     let params = useParams();
     let location = useLocation();
   }
   function DeepComponent() {
     // But what about components deep in the tree?
     // oh right, same as anywhere else
     let navigate = useNavigate();
   }

   // 2. Route 恢復使用巢狀 Route
   // 3. Route 的相對路徑依賴
    <Route path="users" element={<Users />}>
     // 這邊等於是 /users/me
     <Route path="me" element={<OwnUserProfile />} />
     <Route path=":id" element={<UserProfile />} />
   </Route>
   ```

### Server-Side Rendering 處理 Router 機制

在開始實作 Server 端之前，首先先來分析一下平時我們瀏覽網站時是如何運作的。當使用者在瀏覽器上瀏覽一個網站的時候，Server 端會接收到我們連進來的網址(URL)，並且會去分析路徑然好將對應的畫面、資料回應給我們。簡單來說就是，我們需要在 Server 端處理 URL 並且將對應的 Route 裡面的 Component 顯示給使用者。

在前面的部分，我們已經在前端加入了 Router 相關的機制，現在我們也需要在 Server 端這邊加入 Router 這樣才可以在收到 URL 時進行解析，而這部分 `react-router-dom` 也幫我們處理好了，只需要加上 `<StaticRouter>` 這個 Component 即可。

#### 介紹 StaticRouter

StaticRouter 代表著『位置永遠不會改變』，因為在一開始連進網站的時候，使用者還無法對任何東西進行操作(ex. 按鈕點擊)，想當然爾就不會有跳轉的情況發生，而我們的需求也只是在第一次進入時交給 Server 端處理，後續的切換邏輯就會交還給『前端(BroswerRouter)』接續去執行。官方也說：『如果你只想對一個位置(URL)去返回一個 render output 的話，這是一個簡單的方法』。

**_注意：react-router-dom v6 調整了 `<StaticRouter>` 的引入方式與寫法_**

```jsx=
// 引入改為從 `react-router-dom/server` 來做引入
import { StaticRouter } from 'react-router-dom';

// 寫法上刪除了 context prop
<StaticRouter location={req.url}>
    <App />
</StaticRouter>
```

#### Server 加入 StaticRouter

先直接附上相關程式碼，再來進行解說。

```jsx=
/*...其他 import 省略...*/
import { StaticRouter } from "react-router-dom/server";
app.use(express.static('build',{index: false})) // 指定靜態資源
app.get('*', (req, res) => {
    const sheet = new ServerStyleSheet() // <-- 建立樣式表
    // 將 App 這個 component render 成 HTML string
    const staticHTML = ReactDOMServer.renderToString(
        sheet.collectStyles(
            <StaticRouter location={req.url}>
                <App />
            </StaticRouter>
        )
    )
    /*... 讀取 app.html 等等操作 省略...*/
})
```

以下增加了幾點改動：

1. 引入 `StaticRouter` 來處理靜態路由顯示 Component
2. `app.get` 從 `/` 改成 `*`，因為所有的元件路由都需依靠 `StaticRouter` 來幫忙處理，所以改讓這個處理函式能被任何路由給匹配到。
3. 將 `StaticRouter` 包住 `App` component ，讓 URL 能夠與設定的 Route 匹配，使能夠正常顯示對應的 Component。

#### 畫面結果

![](https://i.imgur.com/1c3vu4T.gif)

> 後端啟動 : yarn dev

## Code Spliting

隨著專案越來越大，我們打包後的 bundle 檔也會越來越肥大，尤其是當我們引入很多的第三方套件時，如果一個不小心就很有可能會導致網站需要很長的時間才能被載入。

**Code Spliting** 這項技術就是用來解決這項問題的解決辦法之一，透過切割 bundle 文件，將檔案分割成數個小型的文件，減少整個 bundle size，再藉由延遲載入 (lazy loading) 的方式來載入所需要的檔案，減少初始載入網站時的時間。

React 官方也有提到關於 Code Spliting 的實作部分，可以透過 dynamic import 和 React.lazy 來動態的載入 Component，比如在設定 Route 的時候我們會在最上方 import 所需要的 Component 進來，但其實我們只有在切換到該頁面時才要顯示該 Component，這時就可以透過 dynamic import 搭配 React.lazy 的方式來達成『延遲載入』效果。

**補充：**

1. 如果是用 CRA 建立專案的話，當碰到 dynamic import 語法時，內建的 webpack 設定已經會自動幫我們處理，不需額外再做設定。
2. 如果有使用 babel 時，則需要安裝 `@babel/plugin-syntax-dynamic-import` 確保 Babel 可以解析 dynamic import 語法。

### 使用 React.lazy 來做前端 Code-Spliting

這邊直接來調整 `App.tsx` import 與設定 Route 的部分

```jsx=
/* App.tsx */
// 原始
import BlurryLoadingPage from '@/containers/blurryLoading/BlurryLoadingPage'
import ExpandingCardsPage from '@/containers/expandingCards/ExpandingCardsPage'

// 調整(Dynamic import & React.lazy)
const BlurryLoadingPage = React.lazy(() => import(/*webpackChunkName:'BlurryLoadingPage'*/ '@/containers/blurryLoading/BlurryLoadingPage'))
const ExpandingCardsPage = React.lazy(() => import(/*webpackChunkName:'ExpandingCardsPage'*/ '@/containers/expandingCards/ExpandingCardsPage'))

```

透過 `webpackChunkName` 來設定打包後的檔案名稱，因為預設會依照動態載入的數量以數字的方式來標示檔案名稱(ex. 1.chunk.js)，這會讓我們難以去查看目前是在讀取哪一隻檔案，透過設定參數後就可以更明顯的呈現檔案名(ex. BlurryLoadingPage.chunk.js)，方便我們在 debug 時快速地找到對應的檔案。

`React.lazy` 可以用來定義需要被動態載入的 component，可以讓我們在初始 render 期間，延遲載入那些當下不會立即被用到的 component，加快畫面的載入速度。

`React.lazy` 必須包覆在擁有 `Suspense` 的 render tree 裡面，最簡單的方式可以直接放在 App 那層，這樣專案內任何地方使用到 `React.lazy` 都會吃到同一個`Suspense fallback`的效果。

**_請注意：如果只有寫 `React.lazy` 沒有`Suspense` 的話將會出現報錯。_**

```jsx=
/* App.tsx  */
<Suspense fallback={<div>Loading...</div>}>
    <Routes>
        <Route path='/' element={<ExpandingCardsPage />} />
        <Route path='/ExpandingCards' element={<ExpandingCardsPage />} />
        <Route path='/ScrollAnimation' element={<ScrollAnimationPage />} />
    </Routes>
</Suspense>
```

`Suspense` 主要用來包覆需要被延遲載入的 component，當 component 尚未載入時可以透過 `fallback` 這個 props 來顯示 loading 畫面或想呈現的資訊(ex.載入中...)，當 component 載入完成後會自動顯示該 component 的畫面。

#### 畫面結果

![](https://i.imgur.com/F7SlNyU.gif)

> 前端啟動 : yarn start

從畫面上可以看到，第一次進入頁面時只讀取了 **ExpandingCards** 這隻 chunk.js 檔，而當我們切換頁面到 **ScrollAnimation** 這個 Component 時，可以發現到畫面『先出現了 loading 之後才出現了畫面』，從 **Network 上也可以發現當我們切換時才載入了 ScrollAnimation.chunk.js 這隻檔案**。
以上這就是 React.lazy、Dynamic Import、Suspense 所搭配出來的延遲載入範例。

### 使用 Loadable Component 來做 SSR Code-Spliting

由於目前 React 17 版本中的 ReactDOMServer 尚未支援 Suspense 且 React.lazy 一定要與 Suspense 搭配使用，這就是為什麼 React.lazy 目前還無法在 Server-Side Rendering 中使用，因此需藉由 [Loadable Components](https://github.com/gregberge/loadable-components) 這套套件來處理 SSR 的部分。**補充：在目前官方所提供的 React 18 更新資訊中，未來 React.lazy 和 Suspense 將會直接支援 SSR 的部分**

#### 前端調整 React.lazy 寫法

首先先安裝 `@loadable/component` 來調整 React.lazy 的部分

> yarn add @loadable/component

如果有使用 TypeScript 的話需額外安裝 `@types/loadable__component` 套件

> yarn add --dev @types/loadable\_\_component

##### 調整 App.tsx 的 React.lazy

接著調整 `App.tsx` 將 `React.lazy` 改成使用 `loadable` 的方式

```jsx=
import loadable from '@loadable/component'
/* 使用 loadable component */
const BlurryLoadingPage = loadable(() => import(/*webpackChunkName:'BlurryLoadingPage'*/ '@/containers/blurryLoading/BlurryLoadingPage'))
const ExpandingCardsPage = loadable(() => import(/*webpackChunkName:'ExpandingCardsPage'*/ '@/containers/expandingCards/ExpandingCardsPage'))
```

##### 移除 Suspense 改用 fallBack prop

因為 Suspense 不支援 SSR，所以使用 loadable component 提供的 `fallback` 選項來達成載入時顯示 loading 效果。

```jsx=
<Routes>
    <Route path='/' element={<ExpandingCardsPage fallback={<LoadingComponent/>}/>} />
    <Route path='/ExpandingCards' element={<ExpandingCardsPage fallback={<LoadingComponent/>}/>} />
    <Route path='/ScrollAnimation' element={<ScrollAnimationPage fallback={<LoadingComponent/>}/>} />
</Routes>
```

##### Add loadableReady client-side

Loadable components 異步載入所有的 scripts 來確保性能最佳化，因此需要使用 `loadableReady` 來等待它們準備完成。

在 `index.tsx` 中使用 `loadableReady` 包住 `hydrate`

```jsx=
loadableReady(() => {
    ReactDOM.hydrate(
        <React.StrictMode>
            <BrowserRouter>
                <App />
            </BrowserRouter>
        </React.StrictMode>,
        document.getElementById('root')
    )
})
```

#### 後端加入 loadable 相關設定

如果要用 Server-Side Rendering 的方式啟動的話，後端也需要再加上以下幾個套件來讓 `@loadable/component` 能在 Server Side 正常運作。

> yarn add @loadable/server && yarn add --dev @loadable/babel-plugin @loadable/webpack-plugin

如果有使用 TypeScript 的話需額外安裝 @types/loadable\_\_server 套件

> yarn add --dev @types/loadable\_\_server

##### babelrc.json

babelrc.json 的 plugin 中加入 `@loadable/babel-plugin` 設定，主要用來轉換 `loadable` 語法，讓我們能在 Server-Side Rendering 的情況下夠正常運作。

```javascript=
{
  "presets": [
    "@babel/preset-env",
    "@babel/preset-react",
    "@babel/preset-typescript"
  ],
  "plugins": ["@loadable/babel-plugin"]
}
```

##### craco.config.js

在前端的 webpack 設定檔中加入 `@loadable/webpack-plugin`，因為本專案是使用 `craco` 來複寫 CRA 的 webpack 設定，所以需將設定加在 `craco.config.js` 當中。

`@loadable/webpack-plugin` 主要是用來產生出 `loadable-stats.json`(預設)檔，用來表示各 loadable component 的 dependencies，例如該組件所要的 css、js 等，主要應用在 Server-Side Rending 中。

```javascript=
const CracoAlias = require('craco-alias')
const LoadablePlugin = require('@loadable/webpack-plugin')

module.exports = {
    plugins: [
        {
            plugin: CracoAlias,
            options: {
                source: 'tsconfig',
                // baseUrl SHOULD be specified
                // plugin does not take it from tsconfig
                baseUrl: '.',
                /* tsConfigPath should point to the file where "baseUrl" and "paths"
             are specified*/
                tsConfigPath: './tsconfig.paths.json',
            },
        },
        // loadable component 套件，讓打包時編譯出 loadable-stats.json file 之後給 Server-Side Rending 使用
        new LoadablePlugin(),
    ],
}
```

#### 後端程式碼調整

讀到這邊基本上我們已經完成了 Loadable Component 的相關 babel、webpack 等設定，且也將前端程式碼的 `React.lazy` 改成 `loadable()` 與進入檔(index.tsx)加上 `loadableReady`，如果對以上敘述沒有印象的話，麻煩先往上滑將缺少的部分補上呦!!! :+1:

現在我們只差最後一步【**將 server 端的進入點加入 loadable/server 相關程式碼**】就可以達成 Server-Side Code Splitting 了，直接先付上程式碼。

##### server/index.tsx

```javascript=
/* index.tsx */
import { ChunkExtractor } from '@loadable/server'
import path from 'path'
const statsFile = path.resolve('build/loadable-stats.json')
app.get('*', (req, res) => {
    const webExtractor = new ChunkExtractor({ statsFile })
    // 將 App 這個 component render 成 HTML string
    const staticHTML = ReactDOMServer.renderToString(
        webExtractor.collectChunks(
            <StaticRouter location={req.url}>
                <App />
            </StaticRouter>
        )
    )
    res.set('content-type', 'text/html')
    res.send(`
        <!DOCTYPE html>
        <html>
          <head>
          ${webExtractor.getLinkTags()}
          ${webExtractor.getStyleTags()}
          </head>
          <body>
            <div id="root">${staticHTML}</div>
            ${webExtractor.getScriptTags()}
          </body>
        </html>
      `)
})
```

這邊幾乎把之前寫的版本全部改掉了，主要移除了讀取 `build/app.html` 以及抓取頁面 styled-compoent 樣式塞進自制的 Html 模板中這段程式碼。因為當我們前端使用 loadable/compoent 進行打包時，它自動會將各元件的樣式、js 等依賴都寫在 `loadable-stats.json`這支檔案中，我們只需要使用 `ChunkExtractor`來讀取 `loadable-stats.json`並透過`getLinkTags`、`getStyleTags`、`getScriptTags`等 method 將相關的 css 檔、 js 檔放到模板中就可以正常的顯示該頁面了。

```javascript=
/* 以下都 remove */
fs.readFile('build/app.html', 'utf8', (err, data) => {
    if (err) {
        console.log('err:', err)
        return
    }
    // 讀取完檔案後，將原本的 root  替換成 靜態 Html 模板
    return res.send(
        data.replace(
            `<div id="root"></div>`,
            `<div id="root">${Html({ body: staticHTML, styles: styles, title: 'SSR' })}</div>`
        )
    )
})


```

#### 畫面結果

## 結論

本次實作了 Router 與 Code Splitting 的部分並且結合 Server-Side Rending，過程中學習到了 React Router v6 的新語法、React.lazy、React.Suspense、Loadable Component 等功能或套件。

不過可能是因為專案規模較小，實作完 SSR Code Splitting 後其實覺得單純在前端使用 `loadable()` 後端讀取 app.html 即可，因為在打包出來的 app.html 檔案中會有一段 code 來處理 lazy load 的部分，當要顯示該組件時才載入該檔案的 JS。

而與 Server 端實作的差別在於因為我們直接透過了 `getLinkTags`、`getStyleTags`、`getScriptTags` 取得該頁面所需要的依賴檔，所以不用再去讀取 app.html，可以直接將這些 tag (ex.`<Link/>`、`<Script/>`)放入基本的 Html 結構中輸出。

兩者目前比較起來讀取檔案所花的時間都差不多，因此如果懶得再去安裝套件設定 Server 端這些 code 的話，其實可以到前端這部分就好。而 Server 端的優點在於，不需額外擔心樣式沒載入的問題，還記得上一篇所提到的 styled component 樣式在 Server-Side Rending 沒有被正確載入的問題，而在目前的實作上還沒有看到這樣這樣的問題發生，或許是在實作中意外的將這個 bug 給修正了也說不定～ 😅

#### 以上就是這次實作的結論，如果有新的見解或新的發現會再繼續更新這篇文章，如果有任何講錯的地方或冒犯的部分也歡迎大家提出來告訴我一聲。

### 謝謝觀看。

#### Github : [https://github.com/librarylai/react-fifty-practice](https://github.com/librarylai/react-fifty-practice)

---

## Update 文章更新 : 解釋樣式載入疑問

還記得我們結論時有說到 styled-component 的問題莫名被解決的了嗎? 其實結論就在正上方可以往上滑一下看一下 :laughing:。
**其實 styled-component 的問題並沒有被解決!!!**，只是因為我們當下的這個例子都是渲染 Route 的頁面，**也就是被 loadable-component 的 `getStyleTags` 給解決掉了**，簡單來想就是 `getStyleTags` 讀取了我們 Route 要顯示的 component 的樣式然後幫我們渲染出來。(PS.為了避免搞混，之後 Route 顯示的 Component 會以 Container 來稱呼)

而當今天我們的 `<App>` component 如果增加了一個 `Navigation` 的 Component 的話，例如以下結構!!!

```jsx=
<div>
    <StickyNavigation /> // 注意這行~~~~~~~
    <Container>
        <Routes>
            <Route path="/" element={<ExpandingCardsPage fallback={<LoadingComponent />} />} />
        {/* 審略其他 Route */}
        </Routes>
    </Container>
</div>
```

Navigation 這個 Component 的樣式就不會正常顯示，因為 `getStyleTags` 只讀取了 Route 裡面的這些 Container 的樣式而已，所以 Navigation 就沒有被正確的讀取到，我們直接來看一下畫面。

![](https://i.imgur.com/RN0j6kf.png)

因此我們需要改寫一下 Server 端的部分，將上一篇[SSR 系列第一集【運用 CRA 做 Server Side Rendering】](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/SyZ4NNNEF) Styled-component 的部分加入上去，透過 `sheet.collectStyles` 收集樣式以及`sheet.getStyleTags()` 取得解析後的樣式並加入到 `<head>` 當中，就會正常顯示樣式了。

```jsx=
/* server/index.tsx */
const sheet = new ServerStyleSheet() // <-- 建立樣式表
const staticHTML = ReactDOMServer.renderToString(
    webExtractor.collectChunks(
        // 主要是這行~~~~~~
        sheet.collectStyles(
            <Provider store={store}>
                <StaticRouter location={req.url}>
                    <App />
                </StaticRouter>
            </Provider>
        )
    )
)
const styles = sheet.getStyleTags() // <-- 從表中獲取所有標籤
res.send(`
    <!DOCTYPE html>
    <html>
        <head>
            ${styles} // 主要是這行~~~~~~
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

#### 成果畫面

![](https://i.imgur.com/RF96SJx.png)

### 結論

剛好想說美化一下 Navigation 的部分，所以做了一個 Menu 的 Component，沒想到就這樣解決的當時覺得疑惑的問題，總額言之就是，styled-component 與 loadable-component 在 Server Side 讀取樣式的部分是缺一不可的，一個是抓取 container 的樣式;一個是抓取 component 的樣式，目前看起來這樣的寫法在抓取樣式的部分應該是不會有問題了，疑惑也通通解開了，真是皆大歡喜啊~~~ :laughing:

---

## 實作問題

1. **what's the diff between react-router-dom & react-router?**
   解答：`react-router-dom` re-exports all of `react-router`'s exports, so you only need to import from `react-router-dom` in your project. [參考連結](https://github.com/remix-run/react-router/issues/4648)
2. **'Switch' is not exported from 'react-router-dom'**
   解答：Looks like the latest version does not provide Switch use Routes instead. [參考連結](https://stackoverflow.com/questions/63124161/attempted-import-error-switch-is-not-exported-from-react-router-dom?rq=1)
3. **react-router and typescript throwing "Can't resolve 'react-router-dom'" error**
   解答：如果專案有使用 typescript 需要加上 `@types/react-router-dom` 讓 typescript 能夠識別 react-router-dom [參考連結](https://stackoverflow.com/questions/56695271/react-router-and-typescript-throwing-cant-resolve-react-router-dom-error)
4. **Could not find a declaration file for module '@loadable/component'.**
   解答：安裝 `@types/loadable__component` 套件，讓 TypeScript 能夠讀懂 @loadable/component [參考連結](https://www.npmjs.com/package/@types/loadable__component)
5. **Loadable Components SSR - chunkNames in Server Stats file different from Client stats file**
   解答：[參考連結](https://stackoverflow.com/questions/59235886/loadable-components-ssr-chunknames-in-server-stats-file-different-from-client)

## Reference

1. [React Router v6 使用指南 - 杜尼卜](https://segmentfault.com/a/1190000023684163)
2. [Upgrade to React Router v6](https://github.com/remix-run/react-router/blob/main/docs/upgrading/v5.md)
3. [React Router 官方 - Server-Side Rendering](https://reactrouter.com/docs/en/v6/guides/ssr)
4. [利用 React Suspense & React Lazy 來優化載入速度 - Lieutenant](https://medium.com/coding-hot-pot/%E5%88%A9%E7%94%A8react-suspense-react-lazy%E4%BE%86%E5%84%AA%E5%8C%96%E8%BC%89%E5%85%A5%E9%80%9F%E5%BA%A6-befe89c1454f)
5. [React | 為太龐大的程式碼做 Lazy Loading 和 Code Splitting -
   神 Q 超人](https://medium.com/starbugs/react-%E7%82%BA%E5%A4%AA%E9%BE%90%E5%A4%A7%E7%9A%84%E7%A8%8B%E5%BC%8F%E7%A2%BC%E5%81%9A-lazy-loading-%E5%92%8C-code-splitting-7384626a6e0d)
6. [How to implement loadable components for bundle splitting with SSR support. - Prasanna Shandilya](https://dev.to/shandilyaprasanna/how-to-implement-loadable-components-for-bundle-splitting-with-ssr-support-3g1j)
