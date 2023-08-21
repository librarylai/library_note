# 【筆記】LINE Bot 系列（二）用程式玩轉 LINE Bot

###### tags: `筆記文章`

> 本篇主要會使用[上一篇 - 【筆記】LINE Bot 系列（一）創建自己的 LINE 官方帳號與相關設定 - 無程式碼篇](https://hackmd.io/@Librarylai/S17rU1T52)建立的 LINE Offical Account 來玩一些 LINE Bot 的功能，例如：自動回覆、廣播通知...等。
>
> 因此如果還沒有建立 LINE Offical Account 等相關設定的讀者，建議可以先看上一篇建立一下呦！因為接下來會需要使用到該官方帳號的 `channelAccessToken` 與 `channelSecret` 參數。

## 專案架構與基本設定

> **溫馨提醒：**
> 這邊因為筆者比較偷懶，因此打算直接用 **Next.js** 這套 Serverless 框架來串接 LINE Bot 的部分，那大家也可以依照官方的教學使用 **Node.js + Express** 來實作 LINE Bot。
> 如果您是用 Nodejs + Express 的話可直接看這邊的官方文件來做設定。 [line-bot-sdk-nodejs](https://line.github.io/line-bot-sdk-nodejs/getting-started/basic-usage.html#configuration)

#### 那現在就讓我們從建立一個 NextJS 專案開始吧！！！

### 建立 Next.js 專案

#### 1. 打開 Terminal 並輸入以下指令

> `npx create-next-app@latest`

#### 2. 接下來它會問一些是否要 ESlint、TypeScript、App Router 等問題

> 這邊都可以依照喜好自己做選擇，基本上本篇不會寫到前端的部分，所以大致上都沒關係，想順便練習 Next13 的寫法的可以用 App Router。
>
> **但等等攥寫 API 的部分會使用 Next12 版本 Pages Router 的 [API Routes](https://nextjs.org/docs/pages/building-your-application/routing/api-routes) 方式下去攥寫（簡單來說就是用 Next12 版本的方式攥寫），因為在目前 Next13 的 Route Handlers 方式中會碰到與 line-bot-sdk-nodejs 這套件不符合的問題！**
>
> 在 SDK 執行 middleware 從 Request Header 取得`x-line-signature` 時會碰到抓取不到的問題，目前初步推測是是 Next13 的 Header 是使用 `new Headers` 的方式與之前用物件形式不同，因此目前的 SDK 在解析上會碰到問題！
>
> ![](https://hackmd.io/_uploads/ryx_KjO3h.png)
>
> ![](https://i.imgur.com/3BkBn9O.png)
>
> **因此如果怕兩種寫法混合麻煩的話，可以直接都選擇沿用『Next12 Pages Router』的選項即可。**

#### 3. 安裝相關依賴套件

> `yarn` // 擇其一
> `npm install` // 擇其一

#### 4. 安裝 [line-bot-sdk-nodejs](https://github.com/line/line-bot-sdk-nodejs)

> `npm install @line/bot-sdk --save` // 擇其一
> `yarn add @line/bot-sdk` // 擇其一
>
> **額外補充：**
> 官方也有推薦一些[開源社群上的第三方套件](<(https://developers.line.biz/en/docs/messaging-api/line-bot-sdk/#community-sdks)>)給大家選擇，比較有名的像是 [linebot](https://github.com/boybundit/linebot)、[bottender](https://github.com/Yoctol/bottender) 這兩個套件，而 bottender 更是串接了許多服務在裡面，像是 Slack、LINE、Messenger...等平台服務，因此如果剛好要串接這些平台的話就可以直接用 bottender 這套。

### 安裝 ngrok

第一次聽到這個名詞的讀者可能會好奇這個東西是什麼呢？因此我們先來問問 ChatGPT 大神看看！它的解釋如下：

> ![](https://hackmd.io/_uploads/SkFsdNP22.png)
>
> 簡單來說：**ngrok** 是一個能『暫時』讓外部的人(同事、客戶)能在『
> 未架站』的情況下，透過一個 『暫時的 https URL』 轉發進本地端(localhost) Server 上。

那這邊因為 LINE 的 Webhook 會需要我們提通一個 https 的 URL，所以等等我們會 ngrok 來產生一個暫時的 https URL 並將它填到 Webhook 上面。因此這邊就需要先請大家下載 ngrok 軟體。

#### 1. [ngrok Download](https://ngrok.com/download)

#### 2. 啟動 ngrok（依照你 Server 的 Port）

> `ngrok http 3000 `
>
> **補充：請記得先將以本地的 Server 跑起來，以這邊為例就是將 Next.js 啟動起來**。
>
> ![](https://hackmd.io/_uploads/HJh_n4w32.png)

## Next.js 專案設定 LINE 相關環境

剛剛前面我們已經將官方的這套 [line-bot-sdk-nodejs](https://line.github.io/line-bot-sdk-nodejs/getting-started/basic-usage.html#configuration) SDK 加入到專案中了，現在我們需要設定一下相關 **config** 的部分>

### 取得 LINE Channel secret & Channel access token

#### 1. 首先，到 [LINE Develop Console](https://developers.line.biz/console/) 中找到你的官方帳號。

![](https://hackmd.io/_uploads/Sy1gbUP32.png)

#### 2. 選到 Basic settings 這個 Tab，並往下滑找到 Channel secret 複製起來。

![](https://hackmd.io/_uploads/ByyIWIw3n.png)

#### 3. 選到隔壁的 Messaging API 這個 Tab，並往下滑找到 Channel access token 並點擊 issue 按鈕它將會產生一組 token，請先將這組 token 複製起來。

![](https://hackmd.io/_uploads/SyEQGIw2h.png)

#### 4. 最後一樣在 Messaging API 這個 Tab 找到 Webhook settings 這個區塊，可以將剛剛 ngrok 產生的 URL 填到 Webhook URL 中，並且將 Use webhook 開啟。

![](https://hackmd.io/_uploads/Sy6WXUP22.png)

### 設定 env 檔與 LINE config

透過剛剛上面的步驟，我們已經拿到了 **Channel secret** 與 **Channel access token** 了，現在要將這些資料設定到 SDK 的 `Client` 與 `middleware` 這兩個 function 中，這樣 LINE 才會知道是要對哪一個官方帳號做廣播、做回覆...等。

#### 1. 建立 `.env` 檔案並將 Channel secret 與 Channel access token

> 貼心提醒： 記得將 `.env` 加入到 `.gitignore` 中呦！！！
>
> ![](https://hackmd.io/_uploads/BkD7IUvn3.png)

#### 2. 創建一個 `LINE_CONFIG` 常數來使用 `.env` 資料。

> 這邊是放在 `constants/line.js` 這隻檔案裡面，讀者們可依照本身專案內的 Coding Style 來存放。
>
> ![](https://hackmd.io/_uploads/ByeT8Iwhn.png)

```javascript=
export const LINE_CONFIG = {
  channelAccessToken: process.env.CHANNEL_ACCESS_TOKEN,
  channelSecret: process.env.CHANNEL_SECRET,
}
```

#### 3. 創建一個 `lineBotService.js` 的 Service 檔案來存放等等要對 LINE Bot 進行操作的所有 function。

![](https://hackmd.io/_uploads/SkYDgvPhh.png)

```javascript=
/* src/services/lineBotService.js */

import { MOCK_SPECIAL_PRODUCTS_DATA } from '@/src/constants/products'
import { LINE_CONFIG } from '@/src/constants/line'
import { Client, middleware } from '@line/bot-sdk'

// client 主要是用來對 LINE Bot 進行操作。ex. 回覆、廣播
export const client = new Client(LINE_CONFIG)
// middleware 是來驗證 webhook 回傳時的 x-line-signature 是否合法。
export const lineMiddleware = middleware(LINE_CONFIG)

```

## LINE Bot 自動回覆(Webhook)

到目前為止我們已經將大致上的設定都完成了，現在要來開始攥寫 API 的部分，那首先先來做 LINE Webhook 的部分，那什麼是 Webhook 呢？相信應該有一些人是第一次聽到 **Webhook** 這個名詞。

簡單來說：『**Webhook 是應用程式之間彼此溝通的一種方式，以往我們想要知道第三方服務是否處理完成時，較常的方式是使用 Polling（輪詢）的方式來進行確認，但如果該第三方服務有提供 Webhook 的方式時，我們就可以透過告知『該服務』我們的 API endpoint，讓『該服務』完成某些操作時主動透過該 API 通知我們，這樣就不用一直去用 Polling 監聽。**』

> **補充：關於 Webhook 詳細說明推薦看：** >[33 成為看起來很強的後端：什麼是 Webhook？ - Web 實驗室](https://www.youtube.com/watch?v=YpgaK1Ho-lI) >[LINE Bot 系列文 — 什麼是 Webhook? - Justin Lee](https://medium.com/@justinlee_78563/line-bot-%E7%B3%BB%E5%88%97%E6%96%87-%E4%BB%80%E9%BA%BC%E6%98%AF-webhook-d0ab0bb192be)

#### 以 LINE Bot 為例的話就是：

當使用者傳送訊息至 LINE App 上的官方帳號時，會使 LINE Platform 透過 Webhook 上的 URL 發送到我們的 Server 端上，而當 Server 端依照收到的訊息進行分析、處理後，可以再透過 LINE SDK 發送事件、訊息到 LINE Platform 上回應給使用者。

![](https://hackmd.io/_uploads/B1qfnddhh.png)

### 自動回覆實作

#### 1. 首先建立一支 `pages/api/lineWebhook.js` 檔

#### 2. 依照 Next.js API Route 格式寫上以下程式碼

在 Webhook 使用上基本都會是使用 POST 的方式進行傳輸，因此這邊先判斷 `req.method` 是否為 `POST`，接這驗證 `x-line-signature` 是否合法，最後才進行 `handleLineEvent` 對每個 `events` 進行分析處理。

```javascript=
/* pages/api/lineWebhook.js */
// 引入 line SDK 的 middlewar 與 getRandomSpecialProductsMessage 取得隨機商品資訊
import { getRandomSpecialProductsMessage, lineMiddleware } from '@/src/services/lineBotService'

export const config = {
  api: {
    bodyParser: false, // line middleware 內會去轉換 req.body，因此這邊不需要再進行 bodyParser
  },
}

/* Reference: https://zenn.dev/pinalto/articles/79dc21060a8c95 */
// 呼叫 LINE SDK 的 middleware fn 來驗證 x-line-signature 是否合法
async function validateLineSignMiddleware(req, res, fn) {
  return new Promise((resolve, reject) => {
    fn(req, res, (result) => {
      return result instanceof Error ? reject(result) : resolve(result)
    })
  })
}
// 依照收到的 message text 訊息，來使用對應的 Service 回應。
function handleLineEvent(event) {
  const messageText = event.message.text
  switch (messageText) {
    // 如果今天收到使用者輸入文字為：`隨機特價商品`
    // 則呼叫 getRandomSpecialProductsMessage 取得商品資訊回覆給使用者。
    case '隨機特價商品':
      return getRandomSpecialProductsMessage(event)
  }
  return Promise.resolve(null)
}


export default async function handler(req, res) {
  if (req.method === 'POST') {
    // Validate request
    await validateLineSignMiddleware(req, res, lineMiddleware)
    // Handle events
    try {
      const events = req?.body?.events
      Promise.all(events?.map(handleLineEvent))
        .then((result) => {
          res.status(200).end()
        })
        .catch((err) => {
          console.log(err)
          res.status(500).end()
        })
    } catch (err) {
      console.error('err', err)
      res.status(500).end()
    }
  } else {
    res.status(405).end()
  }
}
```

#### 3. 攥寫回覆訊息程式碼

上面那段程式碼的最主要部分為 `handleLineEvent` 的段，我們可以透過 `event.message.text` 拿到使用者打在 LINE 官方帳號上的文字，以下圖為例：這邊是判斷『如果收到 `隨機特價商品`』這段文字，則去呼叫 `getRandomSpecialProductsMessage` 取得隨機的商品資訊與圖片。

![](https://hackmd.io/_uploads/rJOYfn_32.png)

> 提醒：這邊 LINE 相關的功能都集中寫在 `lineBotServices` 這隻檔案中。基本上 `client` 與 `middleware` 這邊的程式碼應該在前面的步驟就有寫上了，這邊為了避免疑惑所以將全部程式碼貼上。

```javascript=
/* src/services/lineBotServices.js */
import { MOCK_SPECIAL_PRODUCTS_DATA } from '@/src/constants/products'
import { LINE_CONFIG } from '@/src/constants/line'
import { Client, middleware } from '@line/bot-sdk'
// client 主要是用來對 LINE Bot 進行操作。ex. 回覆、廣播
export const client = new Client(LINE_CONFIG)

// middleware 是來驗證 webhook 回傳時的 x-line-signature 是否合法。
export const lineMiddleware = middleware(LINE_CONFIG)

export const getRandomSpecialProductsMessage = async (event) => {
  const randomProduct = MOCK_SPECIAL_PRODUCTS_DATA[Math.floor(Math.random() * MOCK_SPECIAL_PRODUCTS_DATA.length)]
  return client.replyMessage(event.replyToken, [
    {
      type: 'text',
      text: `特價商品 $ 清單如下：\n\n ${randomProduct.name} 金額＄ ${randomProduct.price}`,
      emojis: [
        {
          index: 5,// 看 $ 這個符號位於文字中的第幾個位子 (index 從 0 開始)
          productId: '5ac21a8c040ab15980c9b43f',
          emojiId: '067',
        },
      ],
    },
    {
      type: 'image',
      originalContentUrl: randomProduct.img,
      previewImageUrl: randomProduct.img,
    },
  ])
}
```

上面這段程式碼的重點在於 `client.replyMessage` 這邊，我們可以透過 `event` 取得 `replyToken` 這也是告訴 LINE Bot 要回應到哪一個使用者上面，而『第二個參數』是我們要『回應的 Message 訊息』，**而如果我們想一次回覆多條訊息時，則可以用一個 Array 來包起每一條回覆訊息**。

以這邊例子來看：我們是回應兩條訊息，一則是商品資訊（以 `text` 形式呈現），另一則是商品圖片(以 `image` 形式呈現 )，寫法再麻煩參考下面補充資訊。

> **補充相關官方文件：**
>
> 1. [SDK Client Method](https://line.github.io/line-bot-sdk-nodejs/api-reference/client.html#methods)
> 2. LINE emojis 相關 ID：[List of available LINE emojis](https://developers.line.biz/en/docs/messaging-api/emoji-list/)
> 3. 回覆訊息 Text Message 寫法：[Text Message ](https://developers.line.biz/en/reference/messaging-api/#text-message)
> 4. 回覆訊息 Image Message 寫法：[Image Message](https://developers.line.biz/en/reference/messaging-api/#image-message)

##### 假資料：MOCK_SPECIAL_PRODUCTS_DATA

```javascript=
// 模擬 特價商品資料
export const MOCK_SPECIAL_PRODUCTS_DATA = [
  {
    id: 1,
    name: '魔方戀柱',
    price: 10000,
    img: 'https://a.rimg.com.tw/s5/e02/636/mirrorlight120/d/4e/3a/22025720854074_859.jpg',
  },
  {
    id: 2,
    name: '魔方善意',
    price: 13000,
    img: 'https://cf.shopee.tw/file/929829979f7589b23fd89c544b31d11d',
  },
  {
    id: 3,
    name: 'Jacksdo 魯夫vs凱多',
    price: 15000,
    img: 'https://wacaimg1.waca.ec/uploads/shops/19315/products/5d/5d9af24b6cf544ca3586884781d64887.jpg',
  },
]

```

### 成果展示

這邊為了方便 DEMO 所以有調整一下官方帳號的選單內容，關於如何設定可以再回去看上一篇呦！！

[https://i.imgur.com/sAbADEZ.gifv](https://i.imgur.com/sAbADEZ.gifv)

## LINE Bot 廣播通知

上面的回覆訊息主要是『回應給單一的使用者』，而有時我們會需要一次『群發訊息給所有加入該官方帳號的使用者』，這時就需要用到 Broadcast(廣播通知) 的方式來通知全部的人。

舉例來說，像是電商相關的網站就時常會使用群發信息來通知一些『優惠事項或是活動項目』...等資訊給客戶，而對於公司內部的操作人員則可以用來通知『商品即將在本月到期』....等資訊來提醒內部人員進行調整。

> **實際案例：**
> 筆者之前負責開發電商相關產品中就有一個功能是，每當月底時就會透過 LINE Bot 來提醒內部人員即將被下架的商品，並且將商品名稱、商品編號(ID)...等資訊一併在群發訊息(Text Message) 中來告知所有的後台操作人員。
>
> **這邊需要注意的是：Text Message 一次最多只能輸入 『5000 個字元』，因此當該月下架商品很多時，可能會碰到『被截斷』的問題，建議在發送前先計算一下字元，如果超過限制則拆成 Array 形式來發送多筆 Message**

### 廣播訊息實作

#### 1. 首先建立一支 `pages/api/lineExpireProductsBroadcast.js` 檔

#### 2. 依照 Next.js API Route 格式寫上以下程式碼

```javascript=
/* pages/api/lineExpireProductsBroadcast.js */
import { getExpireProductsListBroadcast } from '@/src/services/lineBotService'

export default async function handler(req, res) {
  if (req.method === 'POST') {
    /* 透過  broadcast 對群裡所有人進行商品到期通知 */
    getExpireProductsListBroadcast()
      .then((result) => res.status(200).end())
      .catch((err) => {
        console.log(err)
        res.status(500).end()
      })
  } else {
    res.status(405).end()
  }
}

```

#### 攥寫群發訊息程式碼

> 這邊一樣是寫在 `lineBotService` 這隻檔案中，因此相關 `import` 與 `client` ...等宣告程式碼這邊就不再貼上。

這邊主要是用到 `client.broadcast` 這個 SDK 裡的 method 來幫我們做到群發通知的功能，Message 訊息也跟上面 `replyMessage` 一樣可以傳送多筆，只需要改成 Object Array 的方式即可。

```javascript=
/* src/services/lineBotService.js */
export const getExpireProductsListBroadcast = async () => {
  const productString = MOCK_SPECIAL_PRODUCTS_DATA.map(
    (product) => `商品編號: ${product.id} , ${product.name} 金額＄ ${product.price}`
  ).join(`\n`)
  return client.broadcast({
    type: 'text',
    text: `本月 即將到期商品 $ 清單如下：\n\n${productString} \n\n請操作人員協助後續處理，謝謝！`,
    emojis: [
      {
        index: 10, // index 代表 $ 符號所在的的位置，以上面為例：『本月 即將到期商品 $』 前字號位於第 10 個字元
        productId: '5ac2213e040ab15980c9b447', // Doc: https://developers.line.biz/en/docs/messaging-api/emoji-list/
        emojiId: '005',
      },
    ],
  })
}

```

### 成果展示

這邊使用 [Postman](https://www.postman.com/) 來打模擬 Broadcast API 被呼叫時會發生的情況，可以看到當發送 API 後 LINE 官方帳號立馬就收到了我們剛在上面設定的『即將到期商品』訊息。

![Imgur](https://i.imgur.com/cLzj2yj.gif)

#### 以上就是本篇『LINE Bot 系列（二）用程式玩轉 LINE Bot』全部內容，簡單的透過 SDK 來自動發送 Message 到官方帳號中，之後也會再進一步玩一些其他功能，到時再麻煩多多指教，如有任何錯誤的地方也再麻煩告知，謝謝您的觀看。

#### Github : [https://github.com/librarylai/line-bot](https://github.com/librarylai/line-bot)

## Reference

1. [LINE Developers - Messaging API SDKs](https://developers.line.biz/en/docs/messaging-api/line-bot-sdk/)
2. [line-bot-sdk-nodejs](https://line.github.io/line-bot-sdk-nodejs/)
3. [LINE bot 好好玩 30 天玩轉 LINE API - Clarence](https://ithelp.ithome.com.tw/users/20117701/ironman/2634)
4. [LINE Bot を Next.js + TypeScript + Netlify Functions で作る - Zenn](https://zenn.dev/pinalto/articles/79dc21060a8c95)
5. [LINE Bot 系列文 — 什麼是 Webhook?](https://medium.com/@justinlee_78563/line-bot-%E7%B3%BB%E5%88%97%E6%96%87-%E4%BB%80%E9%BA%BC%E6%98%AF-webhook-d0ab0bb192be)
