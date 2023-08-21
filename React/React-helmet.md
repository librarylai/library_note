# 【筆記】 Create-React-App 也能動態 Open Graph

###### tags: `筆記文章`

![](https://i.imgur.com/OTtOo6B.png)

最近剛好在處理公司網站 Meta Tag 的部分，其中最重要的就是 Open Graph 的設置，那什麼是 Open Graph 呢？文章標題所說的動態 Open Graph 又能夠達成些什麼？這就讓我們先從 Open Graph 來好好解釋一下吧！

> 本篇主要討論使用 Create-React-App 來達成動態 Open Graph，如果您是 Next.js 的專案，請直接使用 Next 提供的 [next/head](https://nextjs.org/docs/api-reference/next/head) 來實作即可。

## 什麼是 Open Graph

> The Open Graph protocol enables any web page to become a rich object in a social graph. For instance, this is used on Facebook to allow any web page to have the same functionality as any other object on Facebook.

開放社交關係圖 (Open Graph Protocol)，它主要的目是讓網頁在社交軟體或網站呈現時(ex. Line、Facebook、Twitter)，能不僅僅只顯示**網址**還能夠顯其他內容，例如：縮圖、標題、描述...等。

舉例來說：我們將 [蝦皮商品的網址連結](https://shopee.tw/product/264024691/23200782181) 貼到 Facebook 上面，這時我們可以看到不僅僅只有網址，連商品的『標題』、『圖片』都帶了出來，如下圖 :arrow_down:

<img height='600' src='https://i.imgur.com/jUKVUtd.jpg'/>

### Open Graph 設置

看完上面的圖片不知大家好不好奇，究竟該如何設置才能達成這樣的功能呢？
其實方法很簡單，只需要在網站的 `<head>` 標籤中透過 `<meta>` 標籤來設定 `og` 相關的內容即可，大概需要添加以下幾種內容：

> **og:title**：分享網頁時的標題。
> **og:description**：分享網頁時的描述。
> **og:image**：分享網頁時的圖片，例如上面的商品圖。
> **og:type**：內容的媒體類型，例如：website、video.movie...等。
> **og: url**：網站的網址，用來當作 Graph 中的 ID。

**Ps. 詳細的解釋與設定可以看 [Open Graph 官方文件](https://ogp.me/)**

因此如果是在 **Create-React-App** 的架構下要設置這些 Meta Tag 的話，基本上只需要到 `public/index.html` 中將所需要的資訊填寫上去即可，如下圖 :arrow_down:

![](https://i.imgur.com/dwHS4YV.png)

大致的介紹就先到這邊，目前主流的社群平台大部分都有支援『**og 標籤**』的相關設定，因此其他像是 `twitter:card` 等這些 tag 的功能，這邊就不特別介紹了，再麻煩大家自行查詢了。

#### 接下來～讓我們進行本篇的重頭戲『動態 Meta Tag、Open Graph 設置』

## 實作動態 Meta Tag

上面在 `public/index.html` 的這些設置只能說是『**整個專案的預設值**』，如果今天我們想要每一頁都『顯示不同的 Meta Tag』或是像上面的電商網站一樣『分享時動態的帶出商品圖』，這時我們就需要動態的產生這些 Meta Tag。

### 使用 react-helmet-async

在 React 的專案中要做到動態產生 Meta tag 的部分，最常會使用到 [react-helmet-async](https://www.npmjs.com/package/react-helmet-async) 這套 package，而且在設定方面也很簡單，只需要以下幾個步驟：

1. **專案根組件中加入 `<HelmetProvider>`**

```jsx=
/* index.js */
import { HelmetProvider } from 'react-helmet-async'

const app = (
  <HelmetProvider>
    <App/>
  </HelmetProvider>
);
/* 這邊要用 render 或是像官方用 hydrate 都可以 */
/* 官方範例是打算用 renderToString 做 SSR 所以寫 hydrate*/
/* 本篇文章打算用其他的方式，所以這邊 render 就可以。 */
ReactDOM.render(
  app,
  document.getElementById(‘app’)
);
```

2. **使用 `<Helmet>` 元件插入動態 Meta Tag**
   我們可以在頁面中使用 `<Helmet>` 這個 Component 來插入各種 Meta Tag，簡單想像的話，可以把 `<Helmet>` 當作是 HTML 的 `<head>` 標籤一樣，因此任何想動態插在 `<head>` 裡面的東西都可以寫在 `<Helmet>` 裡面，例如 `<link>` 標籤 `title>`、`description` `open graph` 等 Meta Tag。

#### App Page：

```jsx=
/* App.jsx */
import { Helmet } from 'react-helmet-async'
export function App(){
    return (
        <div>
         <Helmet>
            <title>來自 helmet 的 App</title>
            <meta property='og:title' content='來自 helmet 的 App' />
            <meta property='og:description' content='來自 helmet 的 App Desc' />
            <meta property='og:image' content='https://picsum.photos/id/20/200/200' />
          </Helmet>
            /* 其他內容...*/
        </div>
    )
}

```

#### Book Page：

```jsx=
/* Book.jsx */
export function Book() {
  return (
    <div>
      <Helmet>
        <title>來自 Book Page</title>
        <meta property='og:title' content='來自 Book Page' />
        <meta property='og:description' content='來自 Book Page Desc' />
        <meta property='og:image' content='https://picsum.photos/id/10/200/200' />
      </Helmet>
      <h2>Book</h2>
    </div>
  )
}

```

#### Router 相關設定：

這邊只有簡單加上最基本的 router 設定，只是會了能夠模擬出**不同分頁顯示不同 Component 中設定的 Meta Tag** 才增加這些部分，詳細關於 [react-router](https://reactrouter.com/en/main/start/overview) 的使用這邊就不再介紹。

```jsx=
/* index.js */
const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    children: [
      {
        path: 'book',
        element: <Book />,
      },
      {
        path: 'cat',
        element: <Cat />,
      },
      {
        path: 'cat/:id',
        element: <CatBreed />,
      },
    ],
  },
])
/* 將上面提到的 ReactDOM.render 部分『改成』以下內容 */
 ReactDOM.render(
    <HelmetProvider>
      <RouterProvider router={router}></RouterProvider>
    </HelmetProvider>,
    rootElement
 )

```

### 成果畫面

![](https://i.imgur.com/FdaXiSx.gif)

從上圖的 **Elements** 中可以發現到，每當頁面切換時對應的 Component 中所設定的 Meta Tag 也會被動態加入到 `<head>` 標籤內。

如果有像上圖一樣的效果話，那這邊先恭喜您已經能夠『**處理掉工作上 90% 左右的需求**』，除非碰到一些需要先做 pre-rendering (ex.SSR、SSG) 或是提升 SEO(讓爬蟲能夠順利爬取網頁) 等需求，那可能單純設定 `react-helmet` 就還不太足夠，需要再搭配 Server 端或是其他套件(ex. `react-snap`)才能夠達成。

**如果對這部份有興趣，可以再接著往下看。:arrow_down:**

## 實作動態 Open Graph

到目前為止我們已經能在每個頁面中插入不同的 Meta Tag，但**因為 Create-React-App 是屬於 Client Side Rendering（CSR）的架構，網站的內容需要等待 JavaScript 載入後才會顯示**，因此**爬蟲『沒辦法』在第一時間抓到這些動態的 Meta Tag**，也就導致於在 Line、Facebook 等這些社群平台上顯示時，沒辦法符合我們的預期。(ps.依舊顯示預設的標題、圖片...等)

#### 那我們該如何解決這個問題呢？

簡單來說我們需要瞭解 Next.js 的 **pre-rendering** 機制，我們不需要實作像是 `getServerSideProps` 或是 `getStaticProps` 這類的 function，但需要了解一下它們的原理，簡單來說就是『**在顯示畫面前先在 Server 端做一些處理**』或是『**在 build time 時就先將頁面內容 render 出來**』，這也就是現在常常被提到的 Server Side Rendering（SSR）架構。

> **補充：**
> 如果對 CSR、SSR 不太理解的讀者，可以參考我的另一篇文章 [【筆記】SSR 系列第一集【運用 CRA 做 Server Side Rendering】](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/SyZ4NNNEF)

#### 接下來我們將介紹幾種方式簡單的做到 pre-rendering：

- [**react-snap**](https://github.com/stereobooster/react-snap)
- **自己寫個簡單 Server Side Rendering 機制**
- [**prerender.io**](https://prerender.io/)
  - prerender 的部分這邊就不特別講解了，使用方式大概跟步驟二相似，需要自己寫一個 Express server 並且跟著網站步驟安裝 `prerender-node` 套件，大致上就可以了。
  - ![](https://i.imgur.com/UOX0SC7.png)

#### Facebook、Line Open Graph 測試網站

- [Facebook Open Graph 測試網站](https://developers.facebook.com/tools/debug/)
- [Line Open Graph 測試網站](https://poker.line.naver.jp/)

## 使用 react-snap 來做 pre-rendering

[**react-snap**](https://github.com/stereobooster/react-snap) 主要是在 **build time** 時幫我們在背景執行一個 Headless Chrome 來爬取網站中的資料，而它是藉由網站中的每個 links 來連接到各個頁面，因此 **`<a>` 標籤非常重要，因為爬蟲等於是透過它來進到每個頁面並抓取資料**。
最後它會依照 route 的結構在 build 資料夾內建立對應的資料夾與檔案，且會將每個頁面 render 完的 HTML 結構寫在檔案中。

這樣說可能有點難理解，所以我們還是直接實作一下。

### 安裝與設定

**Install:**

> yarn add --dev react-snap

**調整 package.json：**

```javascript=
"scripts": {
    /*...審略...*/
    "postbuild": "react-snap", // 加入這行~~~~~~
  },
```

**調整 src/index.jsx：**

```jsx=
import { hydrate, render } from "react-dom";

const rootElement = document.getElementById('root')
if (rootElement.hasChildNodes()) {
  hydrate(
    <HelmetProvider>
      <RouterProvider router={router}/>
    </HelmetProvider>,
    rootElement
  )
} else {
  render(
    <HelmetProvider>
      <RouterProvider router={router}/>
    </HelmetProvider>,
    rootElement
  )
}
```

**調整 public/index.html：**
相關會透過 **react-helmet** 動態插入的 Meta tag，建議在後面加上 `data-rh="true` 這段程式碼，它會將『原本的 Meta tag 移除』只『留下對應動態插入的 Meta tag』。

```jsx=
<meta name="title" content="預設 title" data-rh="true" />
<meta name="description" content="預設描述" data-rh="true" />
<meta property="og:title" content="預設 og:title" data-rh="true" />
<meta property="og:description" content="預設 og:desc" data-rh="true" />
<meta property="og:image" content="https://picsum.photos/id/2/200/200" data-rh="true" />
```

以上就是 react-snap 的全部設定，記得專案內要有能連進相關頁面的 `<a>` 標籤，不然等等 build 完後不會產出對應的檔案。

```jsx=
const Navbar = () => {
  return (
    <NavbarStyled>
      <Link style={{ color: 'lightblue', marginRight: '16px' }} to='/'>
        Home
      </Link>
      <Link style={{ color: 'lightblue', marginRight: '16px' }} to='/book'>
        Book
      </Link>
      <Link style={{ color: 'lightblue', marginRight: '16px' }} to='/cat'>
        Cat
      </Link>
      <Link style={{ color: 'lightblue', marginRight: '16px' }} to='/cat/1'>
        CatItem
      </Link>
    </NavbarStyled>
  )
}

```

### Build 結果

可以從 build 資料夾裡面發現到，react-snap 在打包時依照我們所提供的 route 結構產生了對應的 `index.html` 檔案。

![](https://i.imgur.com/MM6l4eD.png)

且每隻 `index.html` 檔案都已經預先將內容 render 完成，且可以看到我們在檔案內動態插入的 Meta tag 也已經 render 在檔案內了。

![](https://i.imgur.com/Zwt1d7D.png)

> 補充：
> 如果 Deploy 到 Vercel 後，它在執行 build 時發生錯誤時，可參考該 issue。
> [Deployment fails when Creat React App is added with React Snap - Vercel](https://github.com/vercel/vercel/issues/2357)

```javascript=
"reactSnap": {
    "puppeteerArgs": [
      "--no-sandbox",
      "--disable-setuid-sandbox"
    ]
  },
```

### 成果畫面

最後我們可以在 Line 或 Facebook 上測試看看，或是透過他們的測試網站來測試，這邊就以 Line 來舉例。

> Ps. 關於架站的部分，這邊是直接用 [Vercel](https://vercel.com/) 來架站，因為要測試 Open Graph 您一定得將網站架上去才能測試。

> 建議用相關的測試網站來測試，因為 Line 跟 Facebook 對於 Open Graph 的抓取都有 Cache 的機制，所以有時會發現圖片沒有變更，這邊建議用 **Line 的來測試**，因為 **Line 有提供清掉 cache 的功能**，基本上 Line 成功的話，其他網站基本上應該也是成功的。

![](https://i.imgur.com/YtdE2BS.gif)

![](https://i.imgur.com/OEGvT4w.png)

> **建議**：
>
> 如果要導入到專案的話，可能要思考一下想要做到的範圍『**是否有涉及到權限的部分**』，如果您只是單純公開資料(商品頁)要做到動態 Open Graph 的話，那可能還可以透過 react-snap 來達成。
>
> 因為如果當今天『資料是要透過權限』才能抓取到的話，比如果需要傳入 `Authrization: Bear <token>` 這類的 header 才能抓到資料的話，那代表您在 build time 時就也需要『給爬蟲一個 token 』這樣爬蟲才能抓到資料。
>
> **因此 react-snap 在大型專案中，我個人認為是比較難導入的，所以接下來將介紹另一種方式來解決。**

## 自己寫個簡單 Server Side Rendering 機制

前面我們介紹了使用 react-snap 來達成 pre-rendering，但它還是會受到一些限制，因此最好的方式就是我們自己寫個『**簡易的 Server Side Rendering 機制**』。

延續上面的例子，如果我們要達成 pre-rendering 的話，那就是**每當碰到 `/cat` 這個 route 時，我們就先在 Server 端去呼叫 API 抓取貓咪的圖片後，再把這些資料塞到頁面的 Meta tag 中。**

因此我們要先來在專案中加入 Server 端的相關程式碼，這邊是使用 Express 套件來快速建立一個 server 環境。

### 安裝與設定

**Install:**

> yarn add express axios
> yarn add --dev nodemon

**根目錄下新增 server.js 檔，以及調整 package.json 檔**

```javascript=
 "scripts": {
    "start-server": "nodemon server.js"
 }
```

![](https://i.imgur.com/ntPfxEX.png)

### 攥寫 Server Side 程式碼

在剛剛新增的 `server.js` 內加入以下程式碼：

> **這邊要注意的是：**
>
> 1. 如果是照著文章一路往下實作的話，可能要先檢查一下目前 `build/index.html` 的內容是否有被 `react-snap` 蓋掉( 可能會被 App.js 的內容給蓋掉，看你的 route 如何設置 )。
>
> 2. 保險起見的話，可以先『關掉 `package.json` 中的 `postbuild`』 並且重新 build 一次，也可以順便『把 `react-helmet` 動態插入的那些 `og tag` 通通移除』，因為這些內容等等都會透過 Server 端來插入。
>
> 3. 且 `data-rh="true"` 的部分也可以『刪除』，不然可能會發現從 Server 端的 Commend Line 上面看到有替換，但從網頁的 Element 來看卻看不到，因為 `data-rh="true"` 會把它蓋掉，功能上應該沒有問題但怕造成誤會所以『建議也刪除』。
>
> 4. 因為 `server.js` 中等等會用`build/index.html` 的內容為基準，來做 `replace()` 替換內容。

```javascript=
const express = require('express')
const fs = require('fs')
const axios = require('axios')
const app = express()
const port = 3100

app.use('/', express.static('build'))

// 如果要測試記得先將 react-snap 在 package.json 的 script 那段 postbuild 拿掉，並且重 build
// 不然 replace 時會抓不到對應的字串，因為原本 react-snap 在 build 時就已經把  App.jsx 的 meta tag 給加上去了


/* cat 頁面進來這邊 */
app.use('/cat', async (req, res) => {
  // 呼叫 API 取得 Cat 資料
  let result = await axios.get('https://api.thecatapi.com/v1/images/search?limit=25&page=0&order=desc')
  // 讀取 build/index.html 程式碼
  const htmlCode = await fs.readFileSync(`build/index.html`, 'utf-8')
  // 用 replace 把原本的 title, meta tag 內容替換成從 Server Side 寫入
  const htmlCodeWithMeta = htmlCode
    .replace('React App', `來自 Server Side 產生的 Cat Page`)
    .replace('預設 og:title', `來自 Server Side 產生的 Cat title`)
    .replace('預設 og:desc', `來自 Server Side 產生的 Cat Desc`)
    .replace('https://picsum.photos/id/2/200/200', `${result?.data?.[0]?.url}`)
  // log 看看是否正常
  // console.log('htmlCodeWithMeta', htmlCodeWithMeta)
  // 將調整完的 程式碼 回傳給瀏覽器
  res.send(htmlCodeWithMeta)
})

/* cat breed 頁面進來這邊*/
app.use('/cat/:id', async (req, res) => {
  // 跟上面 Cat 一樣邏輯，上面是直接將符合的文字替換掉。
  // 這邊是模擬比較攏長的寫法（將 整段都寫出來）。
  let result = await axios.get(`https://api.thecatapi.com/v1/images/search?breed_ids=${req.query.id}`)
  const htmlCode = await fs.readFileSync(`build/index.html`, 'utf-8')
  const htmlCodeWithMeta = htmlCode
    .replace('<title>React App</title>', `<title>來自 Server Side 產生的 Cat Breed Page</title>`)
    .replace(
      '<meta property="og:title" content="預設 og:title" data-rh="true" />',
      `<meta property="og:title" content="來自 Server Side 產生的 Cat Breed title" />`
    )
    .replace(
      '<meta property="og:description" content="預設 og:desc" data-rh="true" />',
      `<meta property="og:description" content="來自 Server Side 產生的 Cat Breed Desc" />`
    )
    .replace(
      '<meta property="og:image" content="https://picsum.photos/id/2/200/200" data-rh="true" />',
      `<meta property="og:image" content="${result?.data?.[0]?.url}" />`
    )
  res.send(htmlCodeWithMeta)
})

/* 其他頁面，通通過來這邊 */
app.use('*', (req, res) => {
  res.send(fs.readFileSync(`build/index.html`, 'utf-8'))
})

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`)
})

```

### 執行 Server.js

上面我們已經將程式碼寫到 `server.js` 檔案內了，但我們要怎麼確定『現在是執行在 Server Side 』呢？

首先，我們先在 commend line 執行 `npm run start-server` 並且到 Cat 頁面，如果在 commend line 上看到跟下圖的畫面一樣的 log 內容的話，那代表你目前是從 Server Side 所啟動的。

而從 log 中可以發現到原本在 `public/index.html` 所設定的 Meta Tag 現在都被我們替換成『Server Side』開頭的字眼了。

![](https://i.imgur.com/JqCR75I.jpg)

### 成果畫面

最後本來想部署到 Vercel 後，在 Line 或是 Facebook 上測試看看會不會顯示『**來自 Server Side 產生...xxx**』的文字，但**因為在 Vercel 環境上用 Create-React-App 執行 Server 端碰到了一些困難**，目前還沒有找到方法讓它去 `build` 後再執行 `server.js`，因此打算改用 `Docker + GCP` 的方式來測試，這邊之後也會再寫一篇文章來介紹『用 Docker 部署到 GCP』的部分。

所以這邊我們先改成透過『**重新整理**』的方式來模擬爬蟲讀進來網站時的情況，會不會出現『**來自 Server Side 產生...xxx**』的文字。

![](https://i.imgur.com/8JfN8kD.gif)

> 解釋：
> 一開始為『前端』切換到 `Cat` Page 的畫面，可以看到旁邊還是『預設的 og title、desc』等字眼，接著透過重新整理來模擬『經過 Server 端』後碰到 `replace` 替換成『來自 Server Side』字眼。

## 結語

這次剛好因為工作上碰到要做每個商品的 Open Graph 顯示，因此與公司前輩 **_Ｊ imCat_** 一起討論了幾種解決辦法，這篇文章主要也是將這幾種實作過程記錄下來，最後我個人是比較推薦用第二種方式，也就是『自己建立 Server Side』來處理，因為彈性比較高而且也不會被套件所綁住，當然『如果公司有點資金』的話則推薦就用 [**prerender.io**](https://prerender.io/) 最省時省力。

#### 以上就是【ＣＲＡ 實作動態 Open Graph】的簡單介紹，希望能對一樣使用 Create-React-App 框架的人能有一點幫助，如有任何錯誤或冒犯的地方還請各位多多指教，謝謝您的觀看。

#### GitHub : [https://github.com/librarylai/react-helmet-demo](https://github.com/librarylai/react-helmet-demo)

## Reference

1. [No-code 之旅 — 如何讓網站在分享時看起來漂亮和有吸引力？Open Graph (OG) 簡介 - Jade](https://ithelp.ithome.com.tw/articles/10278469)
2. [Open Graph 介紹：讓網站在社群上被漂亮分享，就靠 OG 標籤！ - Frank Chiu](https://frankchiu.io/seo-open-graph/)
3. [TheCatAPI - Documentation Portal](https://developers.thecatapi.com/view-account/ylX4blBYT9FaoVd6OhvR?report=gpN-ReBkp)
4. [Deployment fails when Creat React App is added with React Snap - Vercel](https://github.com/vercel/vercel/issues/2357)
