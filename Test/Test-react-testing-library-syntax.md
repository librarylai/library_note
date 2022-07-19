# 【筆記】 前端測試！！React-testing-library - 語法篇

###### tags: `筆記文章`

![](https://i.imgur.com/jZI026v.png)

## React-testing-library 介紹

**React-testing-library** 是基於 **DOM Testing Library** 所開發而成的，它增加了一些 API 來幫助我們對 React Component、React Hook 更方便的進行測試。

**DOM Testing Library** 是一個輕量級的套件用來測試 DOM 節點，它提供了一些查找 DOM 節點的方式，就像我們在頁面上查找 DOM 一樣，例如：使用 labal text 去查找 Form 內的表單元素，使用文字(text)來查找連結(Link)與按鈕(button)，也有提供 **data-testid** 來查找不具有特別意義的元素( 這就像是 HTML id Attribute 一樣，我們可以自行在 DOM 上設定 data-testid )。

**DOM Testing Library** 主要鼓勵開發者，將應用程式以更貼近用戶真實操作的方式來進行測試，以達到更高的可信度。

### React-testing-library 安裝

> _如果您是使用 Create React App 建立一包新的專案的讀者，這個部分可以跳過。_
> Create React App 預設將會載入相關的 Testing-library 套件，如下：
> "@testing-library/jest-dom": "^4.2.4",
> "@testing-library/react": "^9.3.2",
> "@testing-library/user-event": "^7.1.2",

我們知道 **React-testing-library** 是基於 **DOM Testing Library** 所開發而成的，所以首先我們需要安裝 **Testing-library**

- **安裝 testing-library**：testing-library 相關主要套件

  > NPM：
  > npm install --save-dev @testing-library/dom
  >
  > (接下來都將使用 Yarn 來舉例)
  > Yarn：
  > yarn add --dev @testing-library/dom

- **安裝 React 測試套件**：主要用來測試 React Component
  > yarn add --dev @testing-library/react
- **安裝 Event 事件測試套件**：主要用來模擬用戶在瀏覽器進行互動時的真實事件
  > yarn add --dev @testing-library/user-event
- **安裝 jest-dom**： 提供 Jest 相關的 DOM matchers(匹配器)
  > yarn add --dev @testing-library/jest-dom

## testing-library 常用 API 介紹

在開始使用 **DOM Testing Library** 時，首先要了解最重要的 『三大查詢方式』 **(Get,Query,Find)**，因為不論是要依照 label、placeholder、text contents、alt text、title、display value、role、test ID 等屬性來查找 DOM node，前面都需要加上 **(Get,Query,Find)** 其中之一的開頭文字，例如：getByTestId, queryByLabelText, findByAltText....等，所以先從這幾個開始介紹吧！

---

### 三大查詢方式

1. **Get：**
   - **getBy 系列：** 將會返回第一個 match 到的 DOM node，如果沒有找到 或是 如果找到超過一個 則會跳出 error。
   - **getAllBy 系列：** 將會返回一個 array 裡面包涵全部 match 到的 DOM node，如果沒有找到則會跳出 error。
2. **Query：**
   - **queryBy 系列：** 將會返回第一個 match 到的 DOM node，如果沒有找到則會返回 null，如果找到超過一個則會跳出 error，適合用來判斷不存在的 node。
   - **queryAllBy 系列：** 將會返回一個 array 裡面包涵全部 match 到的 DOM node，如果沒有找到則會返回 空陣列[]。
3. **Find：**
   - **findBy 系列：** 將會返回 promise 並且在找到 DOM node 後才會呼叫 resolves ，如果沒有找到 或是 如果找到超過一個 或 timeout(default 1000ms) 則會呼叫 rejected。
   - **findAllBy 系列：** 將會返回 promise 並且在找到 DOM node 後才會呼叫 resolves 與 findBy 不同的是返回的是 array 型別，如果沒有找到 或 timeout(default 1000ms) 則會呼叫 rejected。

---

## API 介紹

以下將介紹幾個比較常會使用到的 API 給大家，內容主要為官方的 API 文件與筆者自己下去測試過後的解釋。廢話不多說就趕快來看看吧！

### 1. **Screen :**

screen 是 @testing-library/dom 的 API，它預先綁定了 testing-library 中的查詢語法到 document.body，讓我們能直接透過 screen 來查詢 document.body 內的元素。

**Screen.debug：** 主要是用來方便我們進行除錯或是查看 query 出來的 DOM 是不是我們所期望的內容，可以對 document、單個元件、多個元件 進行查看。

**官方使用範例：**

```javascript
// debug document
screen.debug()
// debug single element
screen.debug(screen.getAllByText('test'))
// debug multiple elements
screen.debug(screen.getAllByText('multi-test'))
```

**舉例（單筆）：**

```javascript
screen.debug(screen.getAllByTestId('jewelryItem'))
// debug log
<div class="sc-dlfnbm PAKvp" data-testid="jewelryItem">
  ...審略...
</div>
```

### 2. **ByLabelText：**

將會查詢 container(ex.document.body) 底下與 **Label 關聯的元素(Element)** ，且會依照我們給定的 TextMatch 來進行匹配(match)。

> **TextMatch：**
> Most of the query APIs take a TextMatch as an argument, which means the argument can be either a string, regex, or a function which returns true for a match and false for a mismatch.

**ByLabelText.jsx**

```javascript=
 {/* 第一種： htmlForm 與 form element id */}
  <label htmlFor='username-input'>Username</label>
  <input id='username-input' />

  {/* 第二種：aria-labelledby 在 form element id 會去引用 id 的值
  參考：https://developers.google.com/web/fundamentals/accessibility/semantics-aria/aria-labels-and-relationships?hl=zh-tw
  */}
  <label id='username-label'>Username</label>
  <input aria-labelledby='username-label' />

  {/* 第三種：放在 label 區塊內 */}
  <label>
    Username <input />
  </label>
  <label>
    <span>Username</span>
    <textarea />
  </label>

  {/* 第四種：aria-label attributes */}
  <input aria-label='Username' />
</div>
```

**ByLabelText.test.js**

```javascript=
describe('ByLabelText', () => {
  render(<ByLabelText />)
  test(' get label test userName ', () => {
    const inputNode = screen.getByLabelText('Username', { selector: 'textarea' })
    screen.debug(inputNode)
  })
})
```

**Selector Option**
當遇到多個 label element 都符合查詢條件時，這時就可以透過 options 參數內的 selector 來做進一步的篩選。 ex.選取上面例子中第三種的 textarea

```javascript=
const node = screen.getByLabelText('Username', { selector: 'textarea' })
```

### 3. **ByText：**

將會查詢所有 text node 中的 textContent，並依照我們給定的 TextMatch 來進行匹配(match)，可以想像成會去尋找呈現在網頁上所有的文字，且匹配有符合查詢條件的項目。

**ByText.jsx**

```html=
<div>
  {/* 第一種 基本款 <p> <span> <div> <h1>....任何能輸入文字的 element */}
  <p>Library react test library </p>
  <span>use span to say something </span>
  {/* 第二種 input type='submit' 所產生按鈕上的文字 */}
  <input type='submit' value='Click me' />
</div>

```

**ByText.test.js**

```javascript=
describe('ByText', () => {
  beforeEach(() => {
    render(<ByText />)
  })
  test(' get text by p  ', () => {
    const pNode = screen.getByText(/Library/gm)
    screen.debug(pNode)
  })
  test(' get text by span  ', () => {
    const spanNode = screen.getByText(/use/gm)
    screen.debug(spanNode)
  })
  test(' get text by input submit value  ', () => {
    const submitNode = screen.getByText(/Click me/gm)
    screen.debug(submitNode)
  })
})
```

#### 補充：

這邊使用了 Jest 生命週期裡的 beforEach 函式，讓每一次執行 test 之前都會先去 render Component，因為在每次執行完測試(test)時 React-testing-library 預設會清除 React tree，這將會導致往下直行其他測試時會查詢不到此元件。詳細了解可參考常見問題（一）。

### 4. **ByTestId**

ByTestId 就像是 JS 上的 getElementById 一樣，只是在 ByTestId 中尋找的是 『**data-testid**』 這個 Attribute，透過『**data-testid**』的值來與我們給定的 TextMatch 來進行匹配(match)。

**ByTestId.jsx**

```javascript=
<div data-testid='dataTestIdContainer'>
  <h1 data-testid='dataTestIdContainer-title'> Title 這是標題 </h1>
  <h4 data-testid='dataTestIdContainer-content'> Content 這是內文 </h4>
</div>
```

**ByTestId.test.js**

```javascript=
describe('ByTestId', () => {
  beforeEach(() => {
    render(<ByTestIdComponent />)
  })
  test(' get testId container', () => {
    const containerNode = screen.getByTestId('dataTestIdContainer')
    screen.debug(containerNode)
  })
  test(' get testId title ', () => {
    const titleNode = screen.getByTestId('dataTestIdContainer-title')
    screen.debug(titleNode)
  })
  test(' get testId content', () => {
    const contentNode = screen.getByTestId('dataTestIdContainer-content')
    screen.debug(contentNode)
  })
})
```

#### 補充：

1. 在使用 ByTestId 上官方還有提供一個簡短的寫法，也能夠抓取到 DOM Element。
   ```javascript=
   container.querySelector(`[data-testid="${yourId}"]`)
   ```
2. 如果今天不想使用官方預設的『data-testid』這個 Attribute 的名字時，官方也有提供方法讓我們去覆蓋掉，使用我們自定義的名字。
   詳細內容可參考： [Configuration Options](https://testing-library.com/docs/dom-testing-library/api-configuration/)
   `javascript configure({testIdAttribute: 'data-my-test-attribute'}) `
3. data-testid 主要是用在無法可靠的依照 Element 原有的屬性去找尋時，或是 Element 經常會因為需求而更改的時候，這時使用 data-testid 就會方便許多。

## Async Methods（處理非同步情況）

大多數的時候，我們可能會碰到需要模擬一些非同步的情況，例如：做一個 Promise 來模擬 fetch mock data 的情況，而這時就需要使用 『Async Methods』來等待 API response 來查看是否符合所期待的內容。

### **waitFor**

[waitFor](https://testing-library.com/docs/dom-testing-library/api-async/#waitfor) 是 React-testing-library 所提供用來『等待』裡面(callback)方法執行完成後再往下測試。在實務方面，它蠻常被用來處理『非同步』的部分，例如：等待某個 Promise 被執行之後再往下測試。

#### 官方範例：

```javascript=
/* 定義 */
function waitFor<T>(
  callback: () => T | Promise<T>,
  options?:{....}
): Promise<T>


// Wait until the callback does not throw an error. In this case, that means
// it'll wait until the mock function has been called once.
// 在這個範例的情況下，waitFor 用來等待 mockAPI 被執行一次。
await waitFor(() => expect(mockAPI).toHaveBeenCalledTimes(1))
```

> 補充：**waitFor** 的 options 參數裡面有兩個比較常會設定到的參數`timeout(多久不再呼叫)`、`interval(每隔多久呼叫一次)`，兩者的預設值分別為`timeout:1000ms`、`interval:50ms`。
>
> 代表 **waitFor** 會『不停的』執行 callback function 直到達到 timeout 時間，因此在設定 timeout 時可能要特別注意一下 interval 的設定，否則可能造成過度呼叫 API 導致被 block...等問題。

## 事件測試

### **user-event**

在開發網站時，我們時常會在 DOM 上監聽一些事件(Event)，像是按鈕的點擊、輸入框內容的變化、滑鼠的移入移出等各種事件，而我們可以透過『**user-event**』這套 Testing Library 的擴充套件，來在測試時模擬使用者真實在瀏覽器上的各種操作(click、hover、mousemove...等)，以提高測試的可靠度並減少錯誤的發生。

#### Install And Import

**Install：**

```javascript=
yarn add @testing-library/user-event @testing-library/dom --dev
```

**Import：**

```javascript=
import userEvent from '@testing-library/user-event'
```

### user-event API 測試

在網頁上經常使用的不外乎就是 onChnage、onClick 事件，像是登入頁就非常符合這兩種事件的條件，不管是輸入帳號密碼或是 Email 信箱都需要使用到 onChnage 事件，而登入按鈕則會使用到 onClick 事件。因此這邊將會分成幾個步驟慢慢帶讀者從登入頁面來模擬使用者輸入帳號、密碼以及點擊登入的流程。

![](https://i.imgur.com/LIIilci.png)

1. 首先先使用 React onChange 合成事件，來監聽 input 輸入內容變化的時候，以及使用 onClick 事件來監聽『登入按鈕』被點擊的時候。

```React=
<InputStyled data-testid='account' type='text' placeholder='account' onChange={handleAccountChange} />
<InputStyled data-testid='password' type='password' placeholder='password' onChange={handlePasswordChange} />
<ButtonStyled onClick={handleLogin}>Login</ButtonStyled>
```

2. 使用 getByTestId 來取得要測試的 DOM 元素（如對 getByTestId 還不了解的讀者可以看 [官方文件-ByTestId](https://testing-library.com/docs/queries/bytestid) ）。

```javascript=
// input DOM
const accountNode = screen.getByTestId('account')
// Button DOM
const loginBtnNode = screen.getByTestId('loginBtn')
```

3. 測試 input 輸入框是否正常運作，使用 user.type(element, text, [options]) 來模擬使用者對輸入框輸入內容。第一個參數代表要測試的 DOM 元素( ex.input、textarea )，第二個參數代表模擬輸入的內容，

```javascript=
// accountNode DOM 輸入 accountInput 文字
userEvent.type(accountNode, 'accountInput')
// 檢查 accountNode DOM 是否有 accountInput 的 value
expect(accountNode).toHaveValue('accountInput')
```

4. 最後模擬使用者點擊登入按鈕的操作，使用 userEvent.click(element, eventInit, options)，這邊 mock 了 onClick 事件內的 console 函式，只是為了讓讀者方便看到測試結果是否符合期待值，如果只是要測試是否有正確被點擊，可以在處理函式內設製 console 並在終端機上查結果。

```javascript=
// Ｍock button onClick 事件內的 console 函式
const logSpy = jest.spyOn(console, 'log')
const loginBtnNode = screen.getByTestId('loginBtn')
userEvent.click(loginBtnNode)
expect(logSpy).toHaveBeenCalled()
```

![](https://i.imgur.com/ljK28d9.png)

## 結語

既前一篇介紹了 Jest 的語法後 ([【筆記】從 Jest 帶你進入前端測試世界 - 語法篇](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/BJuUfqRbv))，這次再來初步瞭解了一下 React-testing-library 是如何幫我們方便測試 React component。最後透過簡單做了一個登入頁面來模擬整個登入流程，從使用 React-testing-library 提供的 API 來抓取 DOM 元素，到使用`user-event` 來模擬 `onClick` 與 `onChange` 的事件操作。

這篇可以算是一個入門篇，主要是先讓大家瞭解一些 React-testing-library 的 API ，所以整體模擬的規模都比較小，之後預計會再寫一篇是【結合 Redux 進行測試】還請各位讀者拭目以待瞜~~~

#### 以上就是這次【React-testing-library - 語法篇】的全部內容，希望對想了解 React-testing-library 的人能有一點點幫助，如有任何錯誤或冒犯的地方還請各位多多指教。

### 謝謝觀看。

#### Github : [https://github.com/librarylai/react-testing-library](https://github.com/librarylai/react-testing-library)

## 常見問題

1. [How do i use a single render across multiple tests in React Testing Library.?](https://stackoverflow.com/questions/62822571/how-do-i-use-a-single-render-across-multiple-tests-in-react-testing-library)
2. [Spying on React functional component method with jest and enzyme; Cannot spyOn on a primitive value](https://stackoverflow.com/questions/58598995/spying-on-react-functional-component-method-with-jest-and-enzyme-cannot-spyon-o)

## Reference

1. [testing-library.com - Queries](https://testing-library.com/docs/dom-testing-library/api-queries/#getby)
2. [user-event - Github](https://github.com/testing-library/user-event)
3. [React Unit Test | 讓你的 UI 測試適應變化（翻譯） - 神 Q 超人](https://medium.com/enjoy-life-enjoy-coding/react-unit-test-%E8%AE%93%E4%BD%A0%E7%9A%84-ui-%E6%B8%AC%E8%A9%A6%E9%81%A9%E6%87%89%E8%AE%8A%E5%8C%96-%E7%BF%BB%E8%AD%AF-b9b2c1c4110f)
4. [[Day29] React Testing Library 的一些實用的小技巧

- pjchender](https://ithelp.ithome.com.tw/articles/10281691)
