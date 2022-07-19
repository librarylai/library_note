# 【筆記】SSR 系列第一集【運用 CRA 做 Server Side Rendering】

###### tags: `筆記文章`

最近剛好滑到了不少篇在講 Server-Side Rendering(SSR) 的文章，就想說來找找看與 React 相關的這方面知識，而在使用 React 的過程中，大部分起專案的方式都是透過 Create-React-App(CRA) 來建立專案，所以本篇主要會以透過 CRA 去做到 SSR 效果的方式來介紹與實作。

[【筆記】SSR 系列第二集【React Router v6 & Code Spliting】](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/B186UUGvF)

[【筆記】SSR 系列第三集【Redux Toolkit & GetServerSideProps 實作】](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/r1vn47vcY)

**_內容主要是以筆記的形式，一邊紀錄實作過程 與 一邊紀錄碰到問題，如有不太清楚的地方再麻煩告知一聲。_**

## Client-Side Rendering 與 Server-Side Rendering 介紹

只要有在運用框架 (React、Vue、Angular) 的前端工程師一定就會聽過 Server-Side Rendering (SSR) 與 Client-Side Rendering(CSR) 這兩個名詞，而這兩個得差別究竟是什麼呢 ? 以及該如何選擇要使用 CSR 還是 SSR 呢 ? 相信這應該都是大家曾思考過的問題，因此首先就來分別介紹一下吧。

### Client-Side Rendering

![](https://i.imgur.com/wkAqtVe.png)
( Reference: [Client-side rendering vs. server-side rendering: which one is better](https://laptrinhx.com/client-side-rendering-vs-server-side-rendering-which-one-is-better-44494895/) )

**_這邊的 Client-Side Rendering 主要是以 React 的 SPA 架構來解釋。。_**

Client-Side Rendering 可以說是目前較常用的一種 render 模式，頁面完整的內容是透過載入 JavaScript 後產生，一開始 Server 端只會回傳一個簡單的 Html 結構，在讀完 JavaScript 後才會將完整的內容產生到畫面上，而目前主流的框架大部分都是以 Client-Side Rendering 的形式渲染。

簡單以 React 來舉例，檔案內只有一個 id 為 root 的根節點，也可以說它是一個容器(container)，而我們在專案內所寫的 React 程式碼都會在 boundle.js 裡面，透過讀取 boundle.js 後動態的將所要呈現的內容塞進【容器】裡面。

```javascript=
<html>
    <head>...省略...</head>
    <body>
        <div id="root"></div>
        <script src="boundle.js"></script>
    </body>
</html>
```

Client-Side Rendering 的優點是可以減少與 Server 端的互動，就以往的話，每個頁面都有一個 Html 檔，所以每當我們切換頁面時就得再跟 Server 去要一次新頁面的 Html 檔，如果是以前有寫過 PHP 的讀者應該不陌生。而現在 React 框架所做的 SPA 頁面就只有一個 Html 檔，在切換頁面時是透過 JavaScript 的方式去做到切換顯示不同的畫面，大大的減少了與 Server 端溝通所需的時間。

Client-Side Rendering 也有一些缺點，當使用者第一次進入到網站時，因為還沒有載入 JavaScript 所以會無法呈現畫面，而當專案越來越大時，代表 JS 檔也就會越來越肥大，這時所需要載入的時間就會越來越久(當然也有其他辦法可以解決，這邊留在之後的 Code Splitting 再來討論)，就容易讓使用者懷疑網站是不是壞掉，造成 UX 的不理想。

Client-Side Rendering 最主要的問題就是 SEO 評分過低的問題，因為 Server 端回應的 Html 檔只有 `<div id="root"/>` 這個 Tag 而沒有其他的內容，導致瀏覽器的**_爬蟲_** 無法正確的爬取網站的內容，這也是導致分數低落的主要原因。

### Server-Side Rendering

![](https://i.imgur.com/FfuGvCs.png)
( Reference: [Client-side rendering vs. server-side rendering: which one is better](https://laptrinhx.com/client-side-rendering-vs-server-side-rendering-which-one-is-better-44494895/) )

**_這邊的 Server-Side Rendering 是單純對名詞的解釋，與接下來所介紹的內容有些微差別 ex.換頁（ CRA 實作 SSR 較像是一個 混合框架 ）。_**

Server-Side Rendering 就是當使用者進入網站時，Server 端就回應了包含完整內容的 Html 結構到瀏覽器中，之後才去載入 JavaScript 的部分，其實以前每切換一個頁面就去跟 Server 端要一隻 Html 檔來渲染的方式就是最典型的 Server-Side Rendering 的例子。

Server-Side Rendering 的優點在於，使用者可以提早看到畫面，因為在 Server 端回應時就已經將畫面帶回來了，而也是因為這點所以在 SEO 的分數方面就能比在 CSR 上增加許多，因為瀏覽器的爬蟲就能夠正確的爬取到網頁的資訊。

Server-Side Rendering 的缺點在於，Server 端的負荷會比 CSR 來的嚴重，就以往早期的網頁模式來看，每切換一個頁面就會去跟 Server 端要新頁面的資料，而當使用者一爆多的時候就會容易造成 Server 的負荷過重而導致延遲等問題。

## Client 端架構與基本設定

看完上面對於 Client-Side Rendering 與 Server-Side Rendering 的介紹後，現在我們就先來將 Client Side 的部分先建立起來吧。

本篇主要以 Create React App 來實做 Server Side Rendering，CRA 可以說是一個 React 的 CLI，它能讓開發者們快速的建立起一個 React 專案，且預設就會幫我們將 webpack、eslint、typescript、babel...等設定完成，所以我們可以直接使用不需額外再去重新設定這些配置，非常方便。

廢話不多說，直接先建一個 React 專案!!!

```javascript=
npx create-react-app cra-ssr
cd cra-ssr
npm start
```

筆者這邊是直接沿用一個之前的練習專案來做調整，該專案一樣是以 create-react-app 的方式起的請大家放心～

#### 使用 craco 設定路徑別名

因為筆者習慣用`@`來配置路徑別名，所以這邊額外安裝 [craco](https://www.npmjs.com/package/@craco/craco) 與 [craco-alias](https://github.com/risenforces/craco-alias) 來設定路徑別名。使用 `craco` 還有一個優點就是方便我們去『複寫』或『增加』webpack 的設定，這樣就不需要去 `eject` 整個專案。

##### craco.config.js

```javascript=
/* craco.config.js */
const CracoAlias = require('craco-alias')
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
    ],
}
```

這邊是透過 `tsconfig.paths.json` 來設定路徑別名，相關內容可以往下滑到 TypeScript 的部分，這邊就不再額外贅述。

##### package.json

```javascript=
/* mac 寫法 */
"start": "craco start",
"build": "craco build && mv build/index.html build/app.html",
"test": "craco test",

/* windows 寫法 */
"build": "craco build && move \"build\\index.html\" \"build\\app.html\"
```

**:warning: 請注意這段 :** `mv build/index.html build/app.html`

這邊調整了打包後的 `index.html` 檔名改為 `app.html`，這是防止在之後的 Server-Side Redndering 中提供靜態資源檔時 `app.use(express.static('build'))` 自動去讀取了 `index.html`，所以這邊將它改名。

**額外解法 : 透過 `express.static('build',{index: false})` 的第二個參數設定 index 為 false。**

**:warning: 這邊的範例程式碼會將兩種方式一同使用，大家可以擇一使用即可。**

## Server 端架構與基本設置

完成上面的基本畫面之後，現在要開始建立 Server 的架構與編譯上的設定，Server 端的部分主要採用 Node.js 的 Express 框架，所以要先在專案中加入 Express 套件。

### 安裝 Express Server

using NPM:

> npm install express

using yarn:

> yarn add express

#### 創建 server 資料夾與檔案

在專案目錄裡面新增一個 server 資料夾並在裡面增加 index.ts 檔案來當啟動 Express

> mkdir server
> touch index.ts

#### 建立 Express Server

首先先產生一個基本的 Express Server，我們可以跟著[官方教學文件](https://expressjs.com/zh-tw/starter/hello-world.html)實作，如下：

```javascript=
const express = require('express')
const app = express()
const port = 3000

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`)
})
```

### 安裝 webpack

![](https://i.imgur.com/aJKRzF4.png)
( Reference: webpack.js.org )

webpack 是一個 JavaScript 的模組打包器，可以透過一個或多個進入點將各個不同的模組組合成一個或多個的包。

> webpack is a static module bundler for modern JavaScript applications.
> it internally builds a dependency graph from one or more entry points and then combines every module your project needs into one or more bundles, which are static assets to serve your content from.

而現在很多 CLI 工具在建立的時後就已經幫忙把 webpack 的相關設定建立完成了，不需要開發者們從零開始的對 webpack 進行設定，只需要額外去新增部分所需要使用到的套件即可，也可以使用類似 [CRACO](https://github.com/gsoft-inc/craco) 去覆寫 CRA 的基本設定 ，，，，)。

---

**推薦 : 對 webpack 有興趣的讀者，可以去讀 Peter 大大的 webpack 系列文章。**[ 傳送門:door:](https://ithelp.ithome.com.tw/users/20107789/ironman/3332)

---

#### 安裝 webpack 相關套件

> yarn add webpack@4.44.2 webpack-cli@3.3.12 webpack-node-externals@1.7.2 --dev

> 目前透過 CRA 創建出來的專案 react-script 對 webpack 的支援目前只到 v4.44.2，因此如果直接安裝 latest 版本的話將會有版本不相容的問題，當然也可以透過創建 webpack.client.js 去設定關於 client 端的 webpack config，不過這邊就有點超出範圍所以暫時不往下討論。
>
> 這邊額外安裝了 webpack-node-externals 套件，主要是用來排除 node_modules 目錄中的所有模塊。

### 使用 Babel 來撰寫 ES6

![](https://i.imgur.com/fTi2s04.png)

> Babel is a tool that helps you write code in the latest version of JavaScript. When your supported environments don't support certain features natively, Babel will help you compile those features down to a supported version.
> Babel 是一個幫助你寫最新版本的工具，當你的環境沒有支援某些功能時，它將會幫助你將這些功能編譯成可以支援的版本。

因為各家瀏覽器對語法的支援度不盡相同，且每年都會有新版本的 ECMAScript 出現 (ex. ES2015、ES2016、ES2017...等)，所以某些舊瀏覽器會因尚未支援新語法而導致功能無法正常使用。
因此可以藉由使用 Bable 以及各種 Polyfill 來使舊瀏覽器將尚未支援的新語法轉換為可以支援的舊語法。

而現在的專案大多會使用 ES6 的語法來寫，所以這邊將導入 Babel 來處理 ES6 的部分

**_補充：Polyfill 的準確意思為：用於實現瀏覽器並不支援的原生 API 的程式碼。參考 : [前端“黑話”polyfill](https://codertw.com/%E5%89%8D%E7%AB%AF%E9%96%8B%E7%99%BC/29473/)_**

#### 安裝 babel 相關套件

> yarn add babel-loader @babel/core @babel/preset-env --dev
>
> Webpack 會透過 babel-loader 調用 Babel，同時也必須安裝以下套件 :

1. @babel/core Babel 的核心功能
2. @babel/preset-env 預設在目標環境中使用最新的 JavaScript 語法
3. @babel/preset-react 支持 JSX 語法格式

#### 創建 .babelrc.json 設定檔

在專案目錄下新增一個 .babelrc.json 設定檔，並將相關需要使用的套件加入到 presets 中

```javascript=
{
  "presets": [
    "@babel/preset-env",
    "@babel/preset-react"
  ]
}
```

接下來創建一支 webpack.server.js 並在 roules 增加使用 balbel-loader 的條件，我們可以直接在 webpack 中直接對 balbel-loader 做設定，不過這邊採用將設定拉到 babel 設定檔中(.babelrc.json ) 去處理，這樣 webpack 比較不會這麼冗長也比較乾淨。

```javascript=
const path = require('path');
const nodeExternals = require('webpack-node-externals');

module.exports ={
    entry:'./server/index.ts',
    target:'node',
    mode: 'development',
    externals:[nodeExternals()],
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].js',
    },
    watch: true,
    resolve: {
        extensions: ['.tsx', '.ts', '.js']
    },
    module:{
        rules:[
            /* babel 設定 */
            {
                test: /\.(js|jsx|tsx|ts)$/,
                exclude: /node_modules/,
                use: 'babel-loader'
            }
        ]
    }
}
```

### 安裝 TypeScript

![](https://i.imgur.com/RxHvuhO.png)

> TypeScript is a strongly typed programming language which builds on JavaScript giving you better tooling at any scale.

TypeScript 是一個包裝在 JavaScript 之上的語言，也可以說是 JavaScript 的超集，它預設也支援了 ECMAScript6 的語法，它可以編譯成簡潔的 JavaScript 代碼，並方便開發者們在瀏覽器、伺服器或任何 JavaScript 引擎上運作。

因為 JavaScript 是屬於弱型別語言( 變數的型別會依照給的值而改變 )，所以程式碼中容易有一些潛在的 Bug，例如 : 在傳遞 API 參數時，將所需要的 Number 型別 (數字 1)，傳成 String 型別 (字串'1') 而導致沒有返回預期的結果。

而使用 TypeScript 的好處在於，它提供了靜態型別檢查系統，讓我們可以在編譯期間就可以發現錯誤，也可以利用它強型別的特性去約束開發者們的習慣，讓開發者在開發上能更嚴謹的撰寫程式碼，減少 Debug 的時間。

---

**推薦 : 對 Typescript 有興趣的讀者，可以去讀 Kira 大大的 Kira 系列文章。**[傳送門:door:](https://ithelp.ithome.com.tw/users/20120053/ironman/2273)

---

#### 安裝 typescript 套件

> yarn add --dev @types/node @types/express

#### 創建 tsconfig 設定檔

在專案目錄下新增一隻 tsconfig.server.js 檔，來處理在 server 時的 typescripte 設定

```javascript=
/* tsconfig.server.js */
{
    "include": ["server"], // compile TS files in server directory
    "extends": "./tsconfig.paths.json",
    "compilerOptions": {
    "target": "es5",
    "lib": [
      "dom",
      "dom.iterable",
      "esnext"
    ],
    "allowJs": true, // allow import of JS modules
    "skipLibCheck": true, // only check types we refer to from our code
    "esModuleInterop": true,  // allow imports of modules in ES format
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,// prevents cross-OS problems
    "noFallthroughCasesInSwitch": true,
    "module": "commonjs",
    "moduleResolution": "node",
    "resolveJsonModule": true, // enable import of JSON files
    "jsx": "react-jsx"// compile JSX to React.createElement statements for SSR
    },
}


```

**注意： 這邊不能設定 "noEmit" 為 true 否則將無法成功打包，因為 noEmit 的意思為『不生成输出文件』。相關問題可參考：[Webpack with typescript getting TypeScript emitted no output error](https://stackoverflow.com/questions/55304436/webpack-with-typescript-getting-typescript-emitted-no-output-error)**

#### webpack 增加 typescript 設定

在 webpack 設定檔的 rules 中增加關於 typescript 的設定

```javascript=
/* webpack.server.js */
  {
    test: /\.ts(x?)$/,
    exclude: /node_modules/,
    use: [
        {
            loader: 'ts-loader',
            options: {
                configFile: 'tsconfig.server.json',
            },
        },
    ],
  },
```

---

**推薦 : 如果對 use、loader、opeion 不太熟得話，推薦看看 [Peter Chen - 尋覓 webpack - 14 - 配置 webpack - 模組 Module 的處理 ] [傳送門:door:](https://ithelp.ithome.com.tw/articles/10246047)**

---

#### package.json 增加設定

在 package.json 的 script 中加入 server 打包的相關指令。

```javascript=
"dev:build-server": "webpack --config webpack.server.js -w",
"dev:build-client": "craco build && mv build/index.html build/app.html",
"dev:server": "nodemon ./dist/main.js",
"dev": "npm-run-all --parallel dev:server dev:build-*",
```

接下來就可以在終端機嘗試下看看指令，看看能不能正常的打包出 dist 資料夾與檔案，如果有成功打包出檔案的話，那目前的對於 webpack、babel、typescript 的設定就大致告一段落了。

![](https://i.imgur.com/yEr94Zn.png)

### 額外補充 :blue_book:

- **特別針對 paths 做別名(alias)的設置**

一般我們引入檔案時都會使用相對路徑的方式去一層一層找尋該檔案位置，這樣的缺點是每次都要去數它是我的上幾層，例如 `../../../scss/index.scss` 就代表是，目前檔案上上上層的 scss 資料夾內的 index.scss，這樣的方式非常麻煩而且還很容易出錯，因此我們可以透過設定路徑的別名 alias 來簡化 import 的路徑。

**1. 透過 typescript 提供的 baseUrl 與 path mapping 來簡化路徑**

```javascript=
/* tsconfig.paths.json */
{
    "compilerOptions": {
        "baseUrl": ".", // 路徑的基礎位置
        "paths": {
           "@/*" : ["./src/*"] // path mapping 可設定多個 => @/ 代表指向基礎路徑下 src 底下位置
         }
    }
}
```

**2. 引入到 tsconfig.json 中**

路徑這段程式碼是寫在 `tsconfig.paths.json` 中，所以要把它 extends 到 `tsconfig.json` 中

```javascript=
/* tsconfig.json */
{
    /* ...省略... */
    "extends": "./tsconfig.paths.json",// 加入這段.....
    "compilerOptions":{/*省略*/},
    "include": /*省略*/
}
```

**3. webpack 中設定別名**

最後不要忘記在 `webpack.server.js` 也要加入別名的設定，不然在打包時有可能會碰到 webpack 認不出來而爆錯的問題。我們可以在 webpack 中插入 [tsconfig-paths-webpack-plugin](https://www.npmjs.com/package/tsconfig-paths-webpack-plugin) 這個 plugin 直接幫我們將 tsconfig 所設定的 paths 加入到 webpack 中。

> yarn add --dev tsconfig-paths-webpack-plugin

```javascript=
/* webpack.server.js */

// 引入 tsconfig-paths-webpack-plugin 插件
const TsconfigPathsPlugin = require('tsconfig-paths-webpack-plugin');

module.exports = {
{/* ...省略... */}
    resolve: {
        extensions: ['.tsx', '.ts', '.js'],
        plugins:[ new TsconfigPathsPlugin() ] // 將 tsconfig.json 的 paths 規則套入到 webpack 中 ex. alias
    },
    module: {
        {/* ...省略... */}
    },
}
```

## Server Side Rendering

在完成上面的基本架構與設定後，現在要開始撰寫 server side rending 的相關程式碼了。

### 1. Client 端改用 ReactDOM.hydrate()

一般我們透過 CRA 去建立 React 專案時，預設進入檔( index.ts )會使用 ReactDOM.render 去渲染整個專案，而在 server-render container 時，則需要將 ReactDOM.render 改成 ReactDOM.hydrate 來使用。

#### hydrate()

> Same as render(), but is used to hydrate a container whose HTML contents were rendered by ReactDOMServer.

hydrate() 其實與 render() 是很相似的，而 hydrate 主要用在 ReactDOMServer 也就是 Server 端上，React 將會嘗試附加 event listener 到現有的 markup。

簡單來說就是在 SSR 的情況下，Server 端渲染出靜態頁面後 boundle.js 會開始載入，當 boundle.js 載入完成後 React 並不會重新去產生所有的組件，而是透過 hydrate 的方式嘗試將現有的 DOM 節點加入 Event 事件等等，用來提高初次載入的效能。

```javascript=
/* index.ts */
// 一般 CRA 創建時的進入點 ( SSR時請改用下面 )
ReactDOM.render(
    <React.StrictMode>
        <App />
    </React.StrictMode>,
    document.getElementById('root')
)

// SSR 改用 ReactDOM.hydrate()
ReactDOM.hydrate(
    <React.StrictMode>
        <App />
    </React.StrictMode>,
    document.getElementById('root')
)
```

### 2. Server 端引入 ReactDOMServer

> The ReactDOMServer object enables you to render components to static markup. Typically, it’s used on a Node server:

在 Server 端的進入檔裡引入 ReactDOMServer 並使用 `renderToString` 這個函式將 Client 端的 `<App/>` component render 成靜態 HTML string，並拿來當作 Server-Side Rendering 載入時使用，讓搜尋引擎在爬取頁面時或是使用者在進入網站時，就可以先看到`<App/>`的畫面。

#### renderToString

根據官方文件所描述 : 【用來將 React element render 成初始的 HTML 並回傳 HTML string，主要是用在伺服端來產生 HTML，如果 Client 端有使用 React.hydrate() 的方法的話，React 將會保留這個 node 並只附上事件處理，這使你能有一個高效能的初次載入體驗。】

```javascript=
/* server/index.tsx */
import App from '../src/App' // 引入 Root Component
import React from "react";
import ReactDOMServer from 'react-dom/server'; // 引入 ReactDOMServer 將 component 轉成 static HTML string
import express from 'express'

const port: string | number = process.env.PORT || 3001
const app = express()
app.get('/', (req, res) => {
  const staticHTML = ReactDOMServer.renderToString(<App />);
  res.send(staticHTML)
})

app.listen(port, () => console.log('Example app is listening on port 3001.'))

```

#### server 端處理 styled-component

因為前端專案使用 styled-component 的方式來寫樣式，所以當 Server 端在 render 時需要將 App 的 style 取出並嵌入到 HTML 文件中的 head 裡面。
**參考文章 : [Dennis Brotzky - The simple guide to server-side rendering React with styled-components](https://medium.com/styled-components/the-simple-guide-to-server-side-rendering-react-with-styled-components-d31c6b2b8fbf)**

```javascript=
/* server/template/html.js */

type HtmlParams = {
	body: string
	styles: string
	title: string
}
const Html = ({ body, styles, title }: HtmlParams): string => `
  <!DOCTYPE html>
  <html>
    <head>
      <title>${title}</title>
      ${styles}
    </head>
    <body style="margin:0">
      <div id="app">${body}</div>
    </body>
  </html>
`
export default Html
```

並將 ReactDOMServer 解析完的靜態 App Component 套上我們客製化的 style 模板，讓我們第一次讀取網站時能把 styled-component 的樣式也套入進去。

```javascript=
app.get('/', (req, res) => {
    const sheet = new ServerStyleSheet() // 建立樣式表
    const staticHTML = ReactDOMServer.renderToString(sheet.collectStyles(<App />))
    const styles = sheet.getStyleTags() // 從表中獲取所有標籤
    // 將 App 這個 component render 成 HTML string
    res.send(Html({ body: staticHTML, styles: styles, title: 'SSR' }))
})
```

### 3. 讀取前端打包後的 JS 檔(簡易版)

到目前為止我們已經完成在 server 端 render 出 react component 並且套入 CSS 樣式，但這樣只是單純的畫面而已，沒辦法與使用者進行互動（按鈕點擊）等相關操作，所以我們需要讀入打包後的 js 檔並透過 React.hydrate() 來將事件監聽附加到靜態 Html ( renderToString 產出的靜態 App component ) 上，這樣 JS 操作的部分就能正常的執行了，而基本能動的、最陽春的 SSR 就大功告成了。

**這邊拿上面 HTML Template 來舉個例子 :**

```javascript=
/* server/template/html.js */
 <!DOCTYPE html>
  <html>
    <head>
      <title>${title}</title>
      ${styles}
    </head>
    <body style="margin:0">
      <div id="root">${body}</div> // 這邊 id 改為 root，對應 React 根節點 id
      <script src="bundle.js"/> // 引入打包後的 JS 檔
    </body>
  </html>
```

### 4.讀取 build/app.html 來模擬真實情況

上面的部分我們是直接去載入`<App/>`這個 Component，但在實務上我們應該是要去讀取打包後的檔案，而不是某一個元件，因此這邊將會來模擬真實情況。

那在開始時實做之前讓我們回頭想想為什麼要做 Server-Side-Rendering ? 這是因為我們打包出來的 app.html 只會有一個 id 為 root 的 div HTML Tag，其餘的程式碼是透過讀取 JS 檔後動態產生的，導致搜索引擎爬蟲在一開始爬取網站時無法正確的抓到該頁面的程式碼，使得 SEO 分數低落。

所以要改善的部分最主要的就是【**讓程式碼能在 Server 端回應給瀏覽器時就產生出來**】，因此讓我們重新改寫一下 Server 檔。

```javascript=
/* server/index.ts */
/* ...上面省略... */
app.use(express.static('build',{index: false})) // 指定靜態資源
app.get('/', (req, res) => {
    const sheet = new ServerStyleSheet() // <-- 建立樣式表
    // 將 App 這個 component render 成 HTML string
    const staticHTML = ReactDOMServer.renderToString(sheet.collectStyles(<App />))
    const styles = sheet.getStyleTags() // <-- 從表中獲取所有標籤
    fs.readFile('build/app.html', 'utf8', (err, data) => {
        if (err) {
            console.log('err:', err)
            return
        }
        // 讀取完檔案後，將原本的 root  替換掉 靜態Html
        return res.send(
            data.replace(
                `<div id="root"></div>`,
                `<div id="root">${Html({ body: staticHTML, styles: styles, title: 'SSR' })}</div>`
            )
        )
    })
})
```

透過 `readFile` 讀取完檔案後，將檔案內容裡的 `<div id="root"></div>` 替換成我們上面處理完得 `靜態 HTML`，這樣就在第一時間將程式碼回應給瀏覽器了，讓我們看看差別吧!!!

**前端打包後檔案 build/app.html ( CSR )**

![](https://i.imgur.com/gYRjUJD.png)

**後端預先產生靜態 HTML 後塞進 root 內 ( SSR )**

![](https://i.imgur.com/lLjqubo.png)

從上面兩張圖就可以看出明顯的差異，一張是我們平時透過 Create-React-App 打包後產生的 index.html；一張是我們透過 Server 端實作 Server-Side-Rending 效果，可以很明顯的看到我們將 component 的內容產生進 root DOM 裡面了。 :smile:

## 結論

這次簡單的實作了如何透過 Create-React-App 來達成 Server-Side-Rendering 的效果，實作過程中碰到了不少問題，尤其是在設定 webpack、typescript 等過程中踩了不少坑，以往都依賴 react-script 幫我們把 webpack 等這些設定好，所以不常從零開始去做一系列的設定，剛好透過這次的機會把整個再學習了一次。
本篇可以算是 **SSR 系列的入門**，之後還打算再繼續研究【Router 與 Code Splitting 】、【 Redux 】...等內容，還請各位讀者拭目以待瞜~~~

#### 以上就是這次【透過 CRA 實作 SSR 】的全部內容，希望對想了解 Server-Side-Rendering 的人能有一點點幫助，如有任何錯誤或冒犯的地方還請各位多多指教。

### 謝謝觀看。

#### Github : [https://github.com/librarylai/react-fifty-practice](https://github.com/librarylai/react-fifty-practice)

## 實作問題

1. **How to setup TypeScript + Babel + Webpack?**
   解答：You compilation setup should have TS output fed to Babel. [參考連結](https://stackoverflow.com/questions/38320220/how-to-setup-typescript-babel-webpack)
2. **TypeError: loaderContext.getOptions is not a function**
   解答：[vue: 使用 ts-loader 引入 ts 文件](https://blog.csdn.net/mouday/article/details/116653235)
3. **Blank page after running build on create-react-app**
   解答：[Parameter "homepage" in package.json](https://github.com/facebook/create-react-app/issues/1487)
4. **SSR with styled-component**
   解答：[[譯] 使用 styled-components 的 React 服務端渲染極簡指南](https://iter01.com/37781.html)
5. **window is not defined when the server run**
   解答: [USING WINDOW IN REACT SSR: THE COMPLETE GUIDE](https://stephencook.dev/blog/using-window-in-react-ssr/)
6. **'Cannot parse tsconfig.path.json' error**
   解答：在 json 文件中不能存在註解。[參考連結](https://github.com/risenforces/craco-alias/issues/23)
7. **如何修改 Create React App 打包後的 index.html 名稱**
   解答：[How can I rename index.html in a create-react-app project?](https://stackoverflow.com/questions/50780983/how-can-i-rename-index-html-in-a-create-react-app-project)
8. **windows 中 package.json 設定 script 來移動檔案找不到路徑問題**
   解答：在 mac 上是使用 `mv` 且路徑正常使用斜線(`/`) 即可，但在 windows 上請將 `mv` 改為`move` 且路徑應使用反斜線(`\`)。[參考連結](https://www.codenong.com/38858718/)
9. **Express 靜態資源自動讀取 index.html 檔案問題**
   解答：可以透過 `express.static('build',{index:false})` 第二個參數去額外設定將自動讀取 index.html 功能關閉。[Express Static Middleware serves index.html automatically](https://stackoverflow.com/questions/42226397/express-static-middleware-serves-index-html-automatically)

## Reference

1. [React Day12 - Babel 介紹](https://ithelp.ithome.com.tw/articles/10185730?sc=rss.qu)
2. [Webpack 前端打包工具 - 使用 babel-loader 編譯並轉換 ES6+ 代碼](https://awdr74100.github.io/2020-03-16-webpack-babelloader/)
3. [How to Enable Server-Side Rendering for a React App](https://www.digitalocean.com/community/tutorials/react-server-side-rendering)
4. [Fullstack TypeScript: Node.js + React SSR](https://nils-mehlhorn.de/posts/typescript-nodejs-react-ssr)
5. [React SSR | 從零開始實作 SSR — 基礎篇](https://medium.com/%E6%89%8B%E5%AF%AB%E7%AD%86%E8%A8%98/server-side-rendering-ssr-in-reactjs-part1-d2a11890abfc)
6. [[Day 14] Server-Side-Rendering - (1)](https://ithelp.ithome.com.tw/articles/10244948)
7. [Github - dzianisbohush/cra-with-ssr](https://github.com/dzianisbohush/cra-with-ssr)
8. [LaptrinhX — Client-side rendering vs. server-side rendering: which one is better](https://laptrinhx.com/client-side-rendering-vs-server-side-rendering-which-one-is-better-44494895/)
