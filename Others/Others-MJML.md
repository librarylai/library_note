# 【筆記】 Email 模板注意事項 與 MJML Framework 使用介紹

###### tags: `筆記文章`

![](https://i.imgur.com/VXrxpgV.jpg)

最近在工作上碰到需要寄出通知信件的需求，像是申請完帳號、變更密碼等等都需要後端發送信件通知使用者，而身為前端的我們就需要刻出一個通用的 Email 模板來讓後端方便使用。
然而在寫 Email 模板中其實也有很多的限制，並不像單純 html、css 切版這麼的簡單，所以我們找了 [MJML](https://mjml.io/) 這套 Email Framework 來輔助我們在模板上的開發。

## Email 模板攥寫上的坑

在製作 RWD Email 模板中最主要的困難點就是 **『CSS 支援度的問題』** 每個瀏覽器支援度参疵不齊彼此之間都有些微的差異，甚至每個平台的軟體支援度也不相同，例如：Windows Outlook 跟 Mac Outlook 對於某些樣式支援度就一樣。
所以當不知道該不該使用時，我們可以透過 [Can I email](https://www.caniemail.com/) 這裡查詢去是否有支援到該語法，就跟我們常會用 [Can I use](https://caniuse.com/) 來查詢 CSS 語法在瀏覽器上的支援度一樣。

### Email 模板坑總和

以下擷取自各大神文章，把有坑的部分總合一下。

1. CSS 需以 inline-style 方式攥寫
2. 使用 table 佈局，複雜情況下使用 table 嵌套。
3. CSS 不要寫簡寫屬性，比如 `margin:10px 0px;` 應寫 `margin: 10px 0px 10px 0px;`
4. 只能使用 CSS2 語法，因此(`display:flex 、 display:grid、box-sizing、box-shadow` ...等)皆無法使用
5. Email 中只能透過 `<td>` 來使用 `padding`
6. 圖片是唯一可以引用的外部資源，但不支援 svg 圖檔
7. 圖片必須設定高寬，因為不少軟體籲設會將郵件中的圖片遮蔽，使用者需要再點一下才能顯示圖片，所以當圖片沒有設定固定寬高時，將很有可能會導致跑版的問題。
8. 圖片地址請不要寫成本地路徑，收件人將無法正確看到
9. 無法設定滑鼠的相關事件(`onMouseOut、onMouseEnter`)

**\*更多攥寫參考： [考拉波斯貓 - 製作郵件模板的規範](https://iter01.com/542049.html)**
**\*關於 CSS 的部分這邊可以參考這篇：[mailchimp - Email Client CSS Support](https://templates.mailchimp.com/resources/email-client-css-support/)**

然而因為這些支援度的參差不齊與各種限制，倒置往往在開發 Email 模板較會花上更多的時間，而在設計 Email 的功能上也會朝向相對單純的方面去做設計，以避免造成不如預期的呈現效果。

## MJML Email Framework

MJML 是一種標記語言(markup language) 用來減少我們在開發 RWD Email 上的痛苦，透過使用 MJML 設計出來的語法(semantic syntax)讓我們可以 **『不必』** 再一直寫一堆巢狀 Table 或是針對各個 Email 平台、軟體寫一些特別的 CSS，只需要使用 MJML 設計出來的 Tag (`<mj-section>、<mj-column>`...等) 就可以快速達成響應式排版且還能大大的改善程式碼的可讀性。

### 安裝與使用

我們可以透過幾種方法來使用 MJML，以下就來個別介紹介紹～～

1.  **NPM or Yarn**
    透過使用 `npm install mjml` 或是 `yarn install mjml` 在 Node.js 中直接使用 MJML Library 將 MJML 語法直接 compile 成 html code，還可以搭配 [Nodemailer](https://nodemailer.com/about/) 套件來完成寄信功能。
    官方範例：
    ```javascript=
    import mjml2html from 'mjml'

        /*
          Compile an mjml string
        */
        const htmlOutput = mjml2html(`
          <mjml>
            <mj-body>
              <mj-section>
                <mj-column>
                  <mj-text>
                    Hello World!
                  </mj-text>
                </mj-column>
              </mj-section>
            </mj-body>
          </mjml>
        `, options)

        ```

2.  **Online 線上編輯**
    MJML 官網也有提供[線上編輯器](https://mjml.io/try-it-live)方便我們直接編寫 Email，讓我們不需安裝套件或還要再起一包專案來專門處理，大大的減少我們在開發上的前置作業。

    寫完模板後可以透過上方右邊的 `View HTML` 按鈕來查看 MJML 轉換完的 HTML 程式碼，推薦將 `Minify HTML` 也勾起來減少檔案大小，最後如果要使用在開發專案上，則只需要將程式碼複製回專案內即可。

    ![](https://i.imgur.com/8ea10QP.png)

3.  **Visual Studio Code plugin**
    透過官方釋出 Visual Studio Code 的 [MJML Plugin](https://github.com/mjmlio/vscode-mjml) 來直接在 vscode 上操作，套件內提供了許多指令方便我們對程式碼進行 Preview、Export、Beautify 等操作。

    ![](https://i.imgur.com/vHlj0Pf.png)

    如果是在 VSCode 上使用插件開發的話，開發完後可以透過以下指令將 MJML 格式轉成 HTML。

    1. 格式化排版 (Beautify MJML code.)
       指令：MJML: Beautify or Format Document
    2. 輸出檔案成 html 檔 (Export HTML file from MJML.)
       指令：MJML: Export HTML
    3. 開啟 Preview 功能 (Opens a preview in a column alongside the current document.)
       指令：MJML: Open Preview to the Side

    ![](https://i.imgur.com/Poq3qlP.png)

### 主要組件介紹

1.  **mj-body**
    mj-body 代表著整個 email 的全部內容，就像網頁中的`<body>` tag 一樣是屬於最外層的結構之一。
    特別要注意 `mj-body` 的寬度預設為 `600px`，所以當要改變 email 模板大小時記得要透過這邊去修改。

        ![](https://i.imgur.com/Ym0wppn.jpg)

2.  **mj-section**
    mj-section 代表著 email 內容中的每一個區塊，如果有學過 bootstrap 的朋友可以想像成裡面的 `row` 會理解比較快。
    特別要注意 `mj-section` 的寬度預設也是 `600px`，如果想要撐滿整個內容則需加上 `full-width="full-width"` 屬性。

        ![](https://i.imgur.com/9xq2RnU.jpg)

3.  **mj-column**
    mj-column 就如同 bootstrap 網格佈局裡的 col 一樣，水平且平均分配整個 section 的寬度，我們可以在 column 裡面放入 email 的文案、圖片等資訊，任何放在 column 內的元件預設寬度都將與 column 相同(`width:100%`)。

        ![](https://i.imgur.com/fIhBJcx.jpg)

4.  **mj-image**
    mj-image 就像是 html 中的 `<img/>` tag 一樣，且會自動在 email 模板中產生響應式的圖片，預設會自動填滿 `mj-column` 的寬度
    `MJML <mj-image width="300px" src="https://www.online-image-editor.com//styles/2014/images/example_image.png" /> `
5.  **mj-text**
    mj-text 裡面可以放入任何 html tag 與 attributes 例如：p、h1 ～ h6、span...等
    主要用來顯示文字、文案等 email 內文
    `MJML <mj-text> <h1> Hey Title! </h1> </mj-text> `

## 實作碰到問題

1. **Ｑ：如何去 Reset MJML 的預設樣式？**
   Ａ：可以透過 <mj-attributes> 這個 tag 來覆蓋 MJML 裡各組件的預設樣式，請記得這個 tag 是要寫在 <mj-head> 裡面否則將無法如預期使用。
   ```MJML
    <mj-attributes>
      <mj-text padding="0" /> // 將文字預設 padding 設為 0
      <mj-all padding="0"/>//全域設定 將所有組件的 padding 設為 0
    </mj-attributes>
   ```
   相關參考 [Padding inside of columns](https://github.com/mjmlio/mjml/issues/581)
2. **Q：看完官方文件但卻不知如何開頭攥寫 MJML？**
   A：MJML 官方中有提供許多的樣式模板方便讓我們使用，您可以找個與您需求相近的模板來直接使用。[MJML Template](https://mjml.io/templates)

## 結論

剛好這次在工作上碰到了要製作共用 Email 模板的需求，才發現原來在 Email 模板的設計上有這麼多的坑以及需要注意的事項，不僅 css 方面受到限制、每個不同平台的支援度也不盡相同。
透過使用 MJML 這套 Email Framework 大大的減少開發時間，只需使用他們所提供的元件就可以輕鬆的完成 RWD 的 email 模板，在支援度方面也幾乎不必去擔心，攥寫途中也可以透過 plugin 或是[線上編輯器](https://mjml.io/try-it-live)的方式直接看到攥寫的內容，當模板設計完後只需程式碼 export 成 html 檔案就可以了。

#### 以上是這次使用 MJML 的心得與使用流程，簡單介紹了裡面的一些比較重要的元件，希望對剛接觸 Email 模板設計的人或是尚未使用過 MJML 的人能有一些幫助，如有任何錯誤或冒犯的地方還請各位多多指教。

### 謝謝觀看。

## Reference

1. [Gary - 透過 MJML 快速完成 RWD Email 排版](https://matters.news/@gary_chu/%E9%80%8F%E9%81%8E-mjml-%E5%BF%AB%E9%80%9F%E5%AE%8C%E6%88%90-rwd-email-%E6%8E%92%E7%89%88-bafyreiarqgirpi4xpunu5z2bwvb5iygqolrxqxvxvhipcoopttcrbo35ly)
2. [TIN - 從零開始建立一個 Email HTML 模板（上）](https://blog.newsleopard.com/coding-html-emails/)
3. [程式前沿 - html 郵件模板開發注意事項](https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/699733/)
4. [考拉波斯貓 - 製作郵件模板的規範](https://iter01.com/542049.html)
5. [mailchimp - Email Client CSS Support](https://templates.mailchimp.com/resources/email-client-css-support/)
