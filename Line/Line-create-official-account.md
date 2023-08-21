# 【筆記】LINE Bot 系列（一）創建自己的 LINE 官方帳號與相關設定 - 無程式碼篇

###### tags: `筆記文章`

> **本篇主要教學如何建立『官方帳號』並設定相關回覆語**...等操做，並『**不會**』有任何程式的內容。
> **下一篇才會使用 Next.js 搭配 LINE SDK 與相關套件來實作 LINE Bot 通知的部分**，因此如果想看程式相關的部分，可以直接到這篇 :arrow_right: [【筆記】LINE Bot 系列（二）用程式玩轉 LINE Bot](https://hackmd.io/@Librarylai/BkP8diQnh)

![](https://developers.line.biz/zh-hant/_media/services/bot-designer-main-contents.png)

LINE 可以說是現代生活中不可或缺的通訊軟體，而 LINE Bot 是一個在 LINE 平台上運行的機器人，基本上現在大多數的『商家』或是『企業』都在使用 LINE Bot 來及時處理各種客服的問題，或是宣傳公司的產品、業務...等資訊。

## LINE Offical Account 的特色

相信每個人在生活中應該或多或少都『使用過』或是『加入過』LINE Bot 或是 官方帳號 成為的好友，像是筆者最近去吃**石二鍋**時就發現可以透過他們的『LINE 官方帳號』來進行『預先點餐、線上取號』...等功能，這樣不僅可以省下排隊的時間，還可以更即時的知道當下的狀況、資訊。

<img width='40%' src='https://hackmd.io/_uploads/rktH0kaq3.jpg' />

#### 那究竟 LINE Offical Account 到底有哪些特點能讓現在的創業者幾乎都人手建一個呢？

這邊主有主要挑選以下三點，我認為目前最常被使用到的部分，也可能是草創時期最常被拿來處理的業務。

- **自動回覆訊息**：這是大家最常使用 LINE Bot 來處理的業務，它可以根據預先設定的規則來自動回覆用戶的訊息，例如歡迎訊息、常見問題回答等。
- **互動式功能**：LINE 官方帳號內提供用戶透過回答、或是選擇提供功能選單來進行互動式的服務，例如問卷調查、優惠卷、訂餐、跳轉到公司官網...等。
- **資訊查詢**：用戶可以透過 LINE Bot 來回應用戶查詢出來的各種資訊，例如天氣預報、新聞、股票報價等。

> **補充： LINE Bot 跟 LINE 官方帳號差別**
> 這兩個其實硬要說可以是同一個東西，LINE Bot 可以說是基於 LINE 官方帳號裡面的一個功能，它是一種能夠自動回覆、處理任務的機器人，可以用於提供自動化的客戶服務、回答常見問題、執行特定任務等。
>
> 簡單來說：LINE Bot 是我們可以透過『攥寫程式』或是『透過 LINE 平台內的設定』來自動回答用戶的問題。
>
> 而 LINE 官方帳號則是由企業、品牌、等組織、公司設立的官方帳號，用來與用戶進行互動，提供品牌資訊、推廣活動、服務（ex.優惠卷、預先訂位）等。

## 動手建立一個 LINE 官方帳號吧！

上面大致上介紹了 LINE 官方帳號與 LINE Bot 後，現在就開始先從 LINE 官方帳號開始建立起吧。

#### 1. [前往 LINE Developers 官網](https://developers.line.biz/zh-hant/services/messaging-api/)，並點擊開始使用

![](https://hackmd.io/_uploads/SJ6e1e6ch.png)

#### 2. 選一種登入方式，這邊是直接用 LINE 登入

![](https://hackmd.io/_uploads/Sk571ga93.png)

#### 3. 如果是第一次建立，尚未有 account 時，會進入到這個畫面，那基本上也是寫一些基本訊息後點擊建立即可

![](https://hackmd.io/_uploads/HkV3ogp92.png)

#### 4. 建立 Channel(可以想像成建立服務)

那這邊我們是要建立 LINE Bot 所以 **Channel** 的部分選擇 **Message API**，**Provider** 可以想像成建立不同的角色，筆者這邊懶得定義所以就直接取自己的名字。
![](https://i.imgur.com/vIf8N0Q.gif)

#### 5. 建立完後可以從畫面上找到剛剛所建立的 Message 服務

![](https://hackmd.io/_uploads/rknZWURq3.png)

#### 6. 最後，點進剛剛建立的 Message 服務，並點擊 Message API 這個 Tab，，並透過手機掃描下方的 QR code 就可以將 LINE Bot 加入到手機中摟！！ 🎉🎉🎉

![](https://hackmd.io/_uploads/H1mjN8C92.png)

> **溫馨提醒： QRcode 建議可以儲存起來，方便未來在做一些『行銷推廣』或像一些『實體傳單』都可以附在上面，方便客戶快速加入到其中。**

## 客製化 - 問候語訊息

我們現在已經建立好自己的 LINE Bot 了，如果有跟著上面掃描 QRCode 將 Bot 加入到 Line 中的話，應該會看到像是下面這樣『預設的問候語』，但這可能不太符合我們所需要的情況，因此這邊將一步一步教大家如何去客製化自己的問候語。

<img width='40%' src='https://i.imgur.com/HzDWK2r.png' />

### 調整問候語

#### 1. 首先，一樣到 Message API 這個 Tab 內，並滑到下面的 `Greeting Message` 點擊 `Edit` 按鈕。

![](https://hackmd.io/_uploads/Hy3ELUC93.png)

#### 2. 點擊『開啟「加入好友的歡迎訊息」設定畫面』

![](https://hackmd.io/_uploads/H13f6IAcn.png)

#### 3. 將想要客製化的內容寫在上面，如果想要多段對話則可以點擊下方的『新增』按鈕去建立多個對話訊息。

![](https://hackmd.io/_uploads/ByfR1wCcn.png)

#### 4. 重新掃描 QR code 看一下調整完的畫面。

<img width='40%' src='https://i.imgur.com/XT6XD4p.png' />

## 客製化 － 新增優惠卷

相信大家對於 LINE 官方帳號中的優惠卷應該不陌生，或多或少應該也都有使用過，或是看過店員在您的手機上按來按去操作過，那這邊也要來教大家如何建立自己店家的優惠卷。

#### 1. 首先先到 [LINE Offical Account](https://manager.line.biz/) 頁面

#### 2. 選擇你的 LINE 官方帳號

![](https://hackmd.io/_uploads/rJ47IrRs2.png)

#### 3. 從左邊選單中找到『推廣相關 - 優惠卷』，並點擊建立優惠卷

![](https://i.imgur.com/fIgBfci.png)

#### 4. 填寫『優惠卷名稱』、『有效日期』、『圖片』、『使用說明』，最後點擊建立就完成摟！！！

> 這邊需特別注意所選擇的『時區』是否符合需求，基本上沒意外應該都會是 Asia/Taipei 才對。

![](https://i.imgur.com/voDJhW4.gif)

## 客製化 － 增加圖文選單

如果有加入過一些店家的 LINE 官方帳號的話，基本上點進去都會看到**下面有一些選單可以選擇**，像是查看店家資訊、使用優惠卷、查看優惠卷...等這些選項讓我們能『**即時查詢一些資訊**』，或是讓我們『**跳轉到該公司對應的一些服務頁面，進行一些操作**』。

接續上面的 **_石二鍋_** 範例，可以看到他們 LINE 官方選單上提供了許多功能，像是能讓我們預先點餐、查看優惠卷...等，他們透過使用 LINE 提供的選單功能讓用戶可以更方便的與石二鍋互動。

<img width='40%' src='https://hackmd.io/_uploads/SJTrzSRi2.jpg' />

#### 那我們現在就來實作一下圖文選單的功能吧！！！

#### 1. 首先先到 [LINE Offical Account](https://manager.line.biz/) 頁面

#### 2. 選擇你的 LINE 官方帳號

![](https://hackmd.io/_uploads/rJ47IrRs2.png)

#### 3. 從左邊選單中找到『圖文選單』，並點擊建立圖文選單

![](https://i.imgur.com/Xpr4AKf.png)

#### 4. 填寫基本資訊，方便未來在後台管理，需特別注意『使用期間』為該圖文選單顯示在 LINE 官方帳號上的時間。

![](https://hackmd.io/_uploads/HyBm8D122.png)

#### 5. 設定圖文選單版型。

![](https://i.imgur.com/R9oqpd8.png)

#### 6. 將該版型的每個區塊填上圖片美化一下。

![](https://i.imgur.com/irUpA4v.png)

#### 7. 設定每個區塊要提供的功能，以下圖為例，最左邊的區塊為『優惠卷』，中間區塊為『連結到公司官網』，最右邊為『客服諮詢』

![](https://i.imgur.com/RpuCsdz.png)

#### 8. 設定最下面選單 Bar 的文字，並儲存設定。

![](https://i.imgur.com/kEGzOS5.png)

## 成果展示

最後展示一下目前為止設定完的成果，可以看到當我們將官方帳號加入為好友後，**會收到我們剛在上面設定的『問候語』**，並且點擊選單裡的『客服區塊』會自動顯示出聯繫客服的電話號碼，點擊中間『L's 區塊』會打開 LINE 瀏覽器並連結到官網(ps.這邊是用 google 當範例)，最後點擊左邊的『優惠卷』會看到剛剛我們在前面所設定的優惠卷內容。

![](https://i.imgur.com/LdLnQcZ.gif)

#### 以上就是本篇最基本的『LINE 官方帳號建立與設定』全部內容，下一篇將會進入到本系列的核心內容，依舊會沿用這個官方帳號來進行教學，將會透過程式的方式來自動發送 Message 到官方帳號中，到時再麻煩多多指教，如有任何錯誤的地方也再麻煩告知，謝謝您的觀看。

#### Github : [https://github.com/librarylai/line-bot](https://github.com/librarylai/line-bot)
