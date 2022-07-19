# 【筆記】Next-Translate 製作網站多國語系

###### tags: `筆記文章`

![](https://i.imgur.com/rDswsZn.png)

在現代的網站中，多國語系已經是不可或缺、非常重要的功能之一，且如果公司想要跨足海外市場的話，勢必就更得支援多國語系不可了，畢竟如果只有支援繁體中文可能連中國市場都難以打入，更何況其他歐美地區呢！？因此多國語系可以說是現今行銷基礎中的基礎。

以往如果使用 [Creat React App](https://create-react-app.dev/) 架構開發網站的話，基本上會使用 [React-intl](https://formatjs.io/docs/react-intl/) 這套多語系的套件來開發，網路上也有許多關於這個套件的文章可以參考，而如果使用 Next.js 架構開發的話，官方文件裡也有一篇叫 [Internationalized Routing](https://nextjs.org/docs/advanced-features/i18n-routing) 的部分在介紹多語系實作。

但今天這篇文章要介紹的是『**Next-Translate**』這套 i18n 套件，個人認為它的優點在於『設定簡單』、『用法簡潔』，如果是以前用過 React-intl 的開發者應該可以很快上手，因為蠻多用法蠻像似的。

#### 廢話不多說直接進入實作環節～～ :writing_hand:

## 大綱

1. **建立 Next.js 專案與安裝 next-translate 套件**
   - 設定 next-translat 相關架構 - nextTranslate
   - 設定 next-translat 相關架構 - i18n.json
   - 設定 next-translat 相關架構 - locales
2. **開始實作 i18n 網頁**
   - 將所有文字搬到 i18n 檔中
   - 將文字替換成透過 i18n 顯示
   - 實作切換語系功能
3. **前端測試 - 各檔案 i18n key 比對**
   - 實作遞迴 - 將 json 中巢狀內的 key 串起來
   - 實作測試 - 比對兩個檔案 format 完後的 key 是否都存在

## 建立 Next.js 專案與安裝 next-translate 套件

因為我們要在 Next 的框架中實作 i18n，所以首先當然是要先建立一個 Next.js 專案，建立步驟如下：

> 1.  npx create-next-app@latest
> 2.  填上專案名稱
> 3.  等待專案建立完成
> 4.  cd {YOUR-PROJECT-NAME}
> 5.  yarn dev

如果有正常看到畫面就代表專案建立成功摟！

### 安裝 next-translate 套件

建立好 Next.js 專案後，再來就是安裝 next-translate 套件。

> **npm :**
> npm install --save next-translate
>
> **yarn :**
> yarn add next-translate

### 設定 next-translat 相關架構 - nextTranslate

現在要開始進行一系列的 i18n 架構設定，首先要先在根目錄的 `next.config.js` 檔中加上 `nextTranslate` 的部分，如果原本專案內就有一些額外設定的話，可以直接寫在 `nextTranslate` 的第一個參數的物件裡面(ex. reactStrictMode)，如果有 webpack 等設定的話則也是直接寫在裡面即可。

**原本：**

```javascript=
const nextConfig = {
  reactStrictMode: true,
}

module.exports = nextConfig
```

**增加 nextTranslate：**

```javascript=
const nextTranslate = require('next-translate')
module.exports = nextTranslate({
  reactStrictMode: true,
  // 如果有 webpack 設定
  webpack: (config, { isServer, webpack }) => {
    return config;
  }
});

```

### 設定 next-translate 相關架構 - i18n.json

接下來需要在專案根目錄內建立一隻 `i18n.js` 或是 `i18n.json` 檔，並且將以下內容到檔案中，這幾個是 next-translate 最基本的設定。

```javascript=
{
    "locales": ["en", "fr", "zh-Hant"], // 專案支援哪些語系
    "defaultLocale": "zh-Hant", // 預設使用語言
    "defaultNS":'common', // 預設使用 common namespace
    "pages": { // 代表每個 route 對應的語言檔
      "*": ["common"],
      "/": ["home", "news"],
      "/about": ["about"]
    }
  }
```

其中要特別介紹 `page` 這個參數，它是一個 key-value 的結構，`key` 代表 route 的位置，value 代表 locales 資料夾內該語言的檔案名稱。

> 注意： value 陣列內的名稱一定要跟 locales 資料夾內的檔案名稱對到。

#### 以上面的例子來解釋：

- `*` 代表所有的 route 都匹配，因此不管在網站的哪一頁都可以拿到 `common` 檔案內的 i18n 文字，基本上 `*` 會拿來放一些共用文字(例如：錯誤提示文字、常用文字...等)。
- `/` 代表首頁，這邊的陣列中填了 `"home", "example"` 這兩個名稱，代表首頁會使用 locales 裡面的 home 與 example 這兩隻檔案的 i18n。

### 設定 next-translat 相關架構 - locales

上面我們 `i18n.js` 檔中已經設定好哪些頁面要用哪些 i18n 檔後，現在要來在專案目錄底下建立一個 locales 資料夾，然後在裡面依照『各個語言來區分資料夾』。

> 語言的命名規範可參考：
>
> - [Language Codes - alchemysoftware](https://www.alchemysoftware.com/livedocs/ezscript/Topics/Catalyst/Language.htm)
> - [地區設定 - wiki](https://zh.wikipedia.org/wiki/%E5%8C%BA%E5%9F%9F%E8%AE%BE%E7%BD%AE)
>
> 這邊是參考 Next.js 官方 [Internationalized Routing](https://nextjs.org/docs/advanced-features/i18n-routing) 的命名規範，也可依照自己的喜好命名，目前測試起來只要 i18n.js 檔的 `locales` 參數內的 value 有跟『資料夾名稱』匹配到即可正常運作。
>
> ![](https://i.imgur.com/grEiO3M.png)

![](https://i.imgur.com/xpMXILa.png)

建立好語系的資料夾後，現在要來建立各語言的 i18n 檔，可以比對一下我們剛剛在上面 `pages` 設定的內容，大概的意思是：全部(`*`)的 route 都可以使用 `common` 這隻 i18n 檔，首頁(`/`)會用到 `home` 跟 `news` 這兩隻 i18n 檔，關於(`/about`)頁會的 `about` 這隻 i18n 檔...以此類推。

#### 建立完檔案後，現在專案架構大概會是這樣：

每個語系資料夾給都有 `common.json`、`home.json`、`news.json`...等檔案，因為 next-translate 會依照現在『網站的語系』來去相對應的資料夾找尋『當下 `route` 所對應的 i18n 檔』。

![](https://i.imgur.com/TXL54sz.png)

> 補充：目前的架構設計是依照官方範例為主，從上圖可以看到每個語系內都有許多隻檔案，這樣的做法雖然是切的比較細，但當專案架構很大的時候會變的不太好找檔案，而且整個 `locales` 資料夾會變得很冗長。
>
> 且一般在做多國語系時一定會將一些『常用到的文字』抽出來到共用區塊中，而當今天有一段內容要加到 i18n 時，如果是在『切分多檔案』的架構上，就會變得要先確認 `common.json` 是否已有存在該文字後再加入到該頁面的 i18n 檔中，因此很容易在快速開發的過程中造成兩個 i18n 檔有同樣文字的問題。
>
> 究竟要『切分多檔案』還是要通通『整合在一隻檔案』中，最後還是要看公司團隊的開發習慣為何，目前與公司前輩們討論過後是全部寫在同一隻檔案中，因為在寫測試檔時也比較方便，因此這邊會在每個語系檔中多增加一隻 `all.json` 檔來模擬全部寫在一起的情況，本文最後也會附上測試範例給大家。

## 開始實作 i18n 網頁

上面幾乎已經將大部分 i18n 的架構都設定完成了，現在可以開始攥寫前端的畫面了，這邊先將等等畫面上會用到的套件列出來，大家也可以自行刻所需要的畫面。

> Material UI：
> yarn add @emotion/react @emotion/styled @mui/material
>
> React 輪播：
> yarn add react-slick slick-carousel
>
> styled-components：
> yarn add styled-components

![](https://i.imgur.com/OOYTnee.png)

畫面上主要分了兩個頁面分別為『首頁』與『關於我們』，分別對應 `i18n.json` 檔中的 `/` 與 `/about` 頁面，在 Navbar 上面做了一個切換語系的 Menu 選單，等等將透過這個按鈕來切換整個專案的語系以及 route 的部分，而『首頁』內則用輪播功能來模擬新聞區塊，用來對應 `i18n.json` 檔中 `news` 的部分。

### 將所有文字搬到 i18n 檔中

上面畫面中先用中文將全部頁面的內容都顯示出來，現在把它們依序搬到各個 i18n 檔中，並且翻譯成其他國家的語言(ex. 英語、法語)。

在寫 i18n 檔時除了以往常用的 `camelCase` 命名之外，也可以用『巢狀』的方式去定義，之後在使用時可以用 `.` 來連接到『巢狀』內的 key，以下圖中的例子來說就會是： `main.news.title`，如果不想用 `.` 來連接的話可以透過 `keySeparator` 來更改成想要的符號。

![](https://i.imgur.com/SneNBW8.png)

### 將文字替換成透過 i18n 顯示

我們可以使用 next-translate 提供的 `useTranslation` hook 來顯示 i18n 文字，因此我們開始將專案內原本寫死的中文換掉吧！
(ps.這邊只以首頁的程式碼當範例，其餘頁面再麻煩自行更換。)

#### 原本：

```jsx=
function Home() {
  return (
    <LayOut>
      <Head>
        <title>查詢首頁</title>
        <meta name='description' content='Next Translate 練習' />
      </Head>
      <Grid container justifyContent={'center'} sx={{ mb: 1 }}>
        <Typography variant='h3' component='div' gutterBottom>
          虛擬貨幣交易網～～～～
        </Typography>
      </Grid>
      <Typography variant='h5' component='p' gutterBottom>
        新聞專區：
      </Typography>
      <CarouselCard />
    </LayOut>
  )
}
```

#### 使用 useTranslation 替換：

```jsx=
function Home() {
  const { t: homeI18n } = useTranslation('home')
  return (
    <LayOut>
      <Head>
        <title>{homeI18n('header.title')}</title>
        <meta name='description' content={homeI18n('header.description')} />
      </Head>
      <Grid container justifyContent={'center'} sx={{ mb: 1 }}>
        <Typography variant='h3' component='div' gutterBottom>
          {homeI18n('main.title')}
        </Typography>
      </Grid>
      <Typography variant='h5' component='p' gutterBottom>
        {homeI18n('main.news.title')}
      </Typography>
      <CarouselCard />
    </LayOut>
  )
}
```

可以看到上面在宣告 i18n function 時，傳入了 `'home'` 字串到 `useTranslation` 的參數中，這是讓我們去告訴 translation 說：麻煩預設請用 `home` 這個 namespace 當作開頭。

可以注意到上面的 `t` function(也就是 `homeI18n`)我們只傳入了 json 檔內的 `key` 進去，它就會自動幫我們 mapping 出文字，但它是怎麼知道是要用哪一個 json 檔案呢？

其實傳入到 `t` function 參數內的 i18nKey 是尤 `namespace:key` 這樣的結構所組成的，而我們在一開始宣告時就已經先將 `home` 這個 namespace 透過 `useTranslation` 傳入到 `homeI18n` 這個 `t` funciotn 中了，所以才可以直接寫 `key` 進去就好。

假設今天要用 `common.json` 裡的 `clickGo` 這個 i18n，我們可以直接組 `common:clickGo` 這樣的字串來去使用。

```javascript=
const { t } = useTranslation() // 注意這邊不用傳入參數
t('common:clickGo') // 麻煩幫我翻譯 common.json 內的 clickGo
```

> 補充：next-translate v1.4.0 版本後，可以透過在 config 加上 `defaultNS` 參數來預設 namespace，像上面我們如果要用 `home` 的 i18n 檔則需要寫 `useTranslation('home')`，而透過設定 `defaultNS:'home'` 這樣就可以直接預設使用了。
>
> ps. 這個在全部 i18n 寫成一隻的情況很好用 (ex.`defaultNS:'all'`)

### 實作切換語系功能

現在所有的文字都已經串上 i18n 了，只差最後一步透過『切換語系』來顯示不同語言的畫面，因此我們需要先做一個切換語系的選單(如下圖)，接著我們可以透果兩種方法來達到切換的功能。

![](https://i.imgur.com/pHmfPrU.png)

1. **使用 next-translate 提供的 `setLanguage` 來達成切換**
   這種方式可以方便在同一頁中切換語系，不必再特別指定要跳轉到哪個 route。
2. **透過 Next.js 本身 [navigation](https://nextjs.org/docs/advanced-features/i18n-routing) 的機制來達成切換 (Link and Router)**
   透過傳遞 `locale` props 在 Next.js 的 `Link` component 上來達成切換，也可以使用 `useRouter` hook 產生的 router object，透過 `router.push({pathName:'xxx',locale:'en'})` 或是 `router.push('xxx',undefiend,{locale:'en'})` 等方式來跳轉頁面並切換語系。

```jsx=
const LanguageMenu = () => {
  const [anchorEl, setAnchorEl] = useState(null)
  // lang 為當前使用 語言
  const { t: commonI18n, lang } = useTranslation('common')
  const isLangOpen = Boolean(anchorEl)
  const handleClick = (event) => {
    setAnchorEl(event.currentTarget)
  }
  const handleClose = () => {
    setAnchorEl(null)
  }
  return (
    <>
      <LanguageButton
        id='basic-button'
        aria-controls={isLangOpen ? 'basic-menu' : undefined}
        aria-haspopup='true'
        aria-expanded={isLangOpen ? 'true' : undefined}
        onClick={handleClick}
      >
        {commonI18n(`language.${lang}`)}
      </LanguageButton>
      <Menu
        id='basic-menu'
        anchorEl={anchorEl}
        open={isLangOpen}
        onClose={handleClose}
        MenuListProps={{
          'aria-labelledby': 'basic-button',
        }}
      >
        {/* 方法一：使用 setLanguage 在同一頁切換語系*/}
        <StyledMenuItem onClick={() => setLanguage('zh-Hant')}>{commonI18n('language.zh-Hant')}</StyledMenuItem>
        {/* 方法二：使用 Link 跳頁並切換語系 */}
        <StyledMenuItem>
          <Link href='/about' locale={'en'}>
            {commonI18n('language.en')}
          </Link>
        </StyledMenuItem>
        {/* 方法三： 使用 useRouter 跳頁並切換語系*/}
        <StyledMenuItem onClick={() => router.push('/about', undefined, { locale: 'fr' })}>{commonI18n('language.fr')}</StyledMenuItem>
      </Menu>
    </>
  )
}

```

![](https://i.imgur.com/OL6WbWI.gif)

從上圖中可以注意到當我們切換不同語系時，網址的部分也會跟著改變，這是 [Next.js - Internationalized Routing](https://nextjs.org/docs/advanced-features/i18n-routing) 的機制，有興趣可以再詳細去理解，上面介紹了三種使用方式(`setLanguage`、`Link`、`router.push`)來切換語系，而它們各自有各自的去用時幾，比如說：『**不想跳頁就切換則可以用`setLanguage`**』，像 Navbar 或 Menu 這種『**設定連結的方式切換則可以用 `Link`**』，而如果要『**動態點擊某些按鈕或是執行到某些條件時就跳轉切換的話則可以用 `router.push`**』的方式。

## 前端測試 - 各檔案 i18n key 比對

其實只要做過 i18n 的人一定有碰過一個問題，那就是在增加 i18n 的途中遺漏了某一個語系，像是在 zh-Hant 中加了 `click:'點擊'` 這個 i18n 但在 en 中則遺漏了，這時當語系切換到 en 時就會沒辦法正確轉換出來，因此畫面就會顯示出原始碼。

而為了防止這樣的問題，最簡單的方法就是去檢查每一個語系中的 i18n 檔是否每個 `key` 都有加到，如果要寫測試的話，這邊會推薦一開始就把全部內容都寫在同一隻 i18n 檔內，因為這樣會比較方便一點，當然也可以各自分開來測，就看團隊如何規劃了。

> 1.  安裝： yarn add jest
> 2.  package.json 中的 scripts 加入："test": "jest"
> 3.  locales 資料夾內增加 `__tests__` 的資料夾
> 4.  `__tests__` 資料夾內創建一隻 `index.test.js`

### 實作遞迴 - 將 json 中巢狀內的 key 串起來

接下來會把剛剛分散在各檔案的 i18n 都集中到 `all.json` 這隻檔案並整理一下命名，然後會以 `zh-Hant.json` 當作基準，讓每一隻檔案都與它做比對。

而因為我們的 json 檔是使用『巢狀』的方式命名，因此我們需要寫一隻遞迴去將每一層的 `key` 串起來，最後產生出一個陣列包含著所有 `key` 的字串。

![](https://i.imgur.com/lqfcG9F.png)

```javascript=
const formatJsonKeyToArr = (json , parentName) => {
  // 先將 json 中第一層的 key 取出
  const jsonKeys = Object.keys(json)
  let formatArr = []
  // 把第一層每個 key 各別跑迴圈
  jsonKeys.forEach((keyString) => {
    // 判斷如果當下這個 key 的 value 是個物件，則代表它是『巢狀』
    if (typeof json[keyString] === 'object') {
      // 將名稱串起來，如果有父層的 key 名稱則將 目前的 key 與父層的 key 串起來。
      // 因為我們最後是想要得到 'homePage.main.title' 這樣的字串
      const childKeys = formatJsonKeyToArr(
        json[keyString], // 就將這個 value 再丟進來跑一次
        parentName ? `${parentName}.${keyString}` : keyString
      )
      // array 合併
      formatArr = formatArr.concat(childKeys)
    } else {
      // 如果當下這個 key 的 value 是個 字串，則就直接丟進 array 中。
      formatArr.push(parentName ? `${parentName}.${keyString}` : keyString)
    }
  })
  // 把整個 array 回傳
  return formatArr
}
```

#### format 完大概會長這樣：

巢狀結構內的 key 都與前一層的 key 串連在一起。

![](https://i.imgur.com/GYPTlwc.png)

### 實作測試 - 比對兩個檔案 format 完後的 key 是否都存在

這邊是直接用 Javascript 的 `some` 跟 `forEach` 去實作兩個 array 內容的比較，如果想要程式碼比較精簡的話，可以考慮裝 [lodash](https://www.lodashjs.com/) 來用，它裡面有提供很多 function 來讓我們更方便的操作陣列，詳細部分再麻煩自行去看摟！

```javascript=
describe('unit test for locales', () => {
  // 以 zh-Hant 為基準
  const zhI18nKeyArr = formatJsonKeyToArr(zhI18n)
  // 需要比對的其他語系 json 檔
  const localesI18nArr = [
    { json: enI18n, language: 'en' },
    { json: frI18n, language: 'fr' },
  ]
  describe(`use zh-Hant to compare other i18n json file  `, () => {
    // 一個一個拿出來比對
    localesI18nArr.forEach((item) => {
      // 將 json 結構 format 成字串
      const i18nKeyArr = formatJsonKeyToArr(item.json)
      describe(`${item.language} i18n`, () => {
        zhI18nKeyArr.forEach((key) => {
          it(`${key}`, () => {
            // 把 zh-Hant format 過後的每一個 key 拿出來比對
            // 看看其他語系檔是否也有存在
            const isExist = i18nKeyArr.some((i18n) => i18n === key)
            // 期待都要是 true
            expect(isExist).toBe(true)
          })
        })
      })
    })
  })
})

```

#### 最後記得在 command line 下：

> yarn test /locales

![](https://i.imgur.com/nh0Vp5X.gif)

## 結語

#### 以上就是這次【 Next-Translate 製作網站多國語系 】的全部內容，希望對正要開發多國語系的人能有一點點幫助，如有任何錯誤或冒犯的地方還請各位多多指教，謝謝您的觀看。

#### Github : [https://github.com/librarylai/next-translate-app](https://github.com/librarylai/next-translate-app)

## Reference

1. [next-translate](https://github.com/vinissimus/next-translate)
1. [Language Codes - alchemysoftware](https://www.alchemysoftware.com/livedocs/ezscript/Topics/Catalyst/Language.htm)
1. [地區設定 - wiki](https://zh.wikipedia.org/wiki/%E5%8C%BA%E5%9F%9F%E8%AE%BE%E7%BD%AE)
1. [React Slick](https://react-slick.neostack.com/docs/example/lazy-load/)
