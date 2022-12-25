# 【筆記】React PDF 相關套件

###### tags: `筆記文章`

![](https://i.imgur.com/Pra5Qfk.png)

最近剛好與朋友討論到將『前端畫面直接轉成 PDF 下載、列印』的部分，所以就整理了幾套關於這部分的 React 套件。

## 套件介紹： react-to-pdf

> **GitHub**：[**react-to-pdf**](https://github.com/ivmarcos/react-to-pdf#readme) > **安裝：** yarn add react-to-pdf

**react-to-pdf** 提供了 `<ReactToPdf>` 的 component 來將 HTML 內容轉成 PDF 檔，我們可以透過 `targetRef` 屬性來『圈出』需要轉換的區塊。

#### 在設定`targetRef` 方面有兩種模式可以使用：

1. **Using inner target ref**
   透過將 `ReactToPdf` Child Function 的參數 `targetRef` 放到想要轉換的 Element 的 ref 屬性內，就可以將該區塊轉換成 PDF，該方法雖然直觀但缺點是需要將內容都寫在 Child Function 內，因此容易造成該區塊程式碼肥大、不好維護。

```jsx=
<ReactToPdf>
   {({toPdf, targetRef}) =>  (
     <ReactToPDFContainer ref={targetRef}>
       <h1>MUI Table</h1>
       {/* 假資料 Table */}
       {renderMuiTableView()}
     </ReactToPDFContainer>
   )}
</ReactToPdf>
```

2. **Using outer target ref**
   上面的方法是寫在 Child Function 內，而另一種方法是透過 `ref` 來綁定區塊，因此我們就不需要將內容都限制在 Child Function 內，我們可以透過 `useRef` 來創建一個 **ref**，並且將該 `ref` 分別綁定到 `ReactToPdf` 的 `targetRef` Props 與需要轉換的 Element 的 ref 屬性上，就可以將該區塊轉換成 PDF 嘍！

```jsx=
// 透過 useRef Hook 產生一個 custom ref
const pdfRef = useRef()
// render
<div>
  {/* 將 ref 綁定到  targetRef props 上  */}
  <ReactToPdf targetRef={pdfRef} filename='muiTable.pdf'>
    {({ toPdf }) => (
      <Button variant='contained' onClick={toPdf}>
        產生PDF (Generate pdf)
      </Button>
    )}
  </ReactToPdf>
  {/* 將 ref 也綁定到需要轉成 PDF 的內容上  */}
  <ReactToPDFContainer ref={pdfRef}>
    <Title>MUI Table</Title>
    {renderMuiTableView()}
  </ReactToPDFContainer>
</div>
```

### 成果

![](https://i.imgur.com/IdmNRF4.gif)

## 套件介紹：react-to-print

> **GitHub**：[**react-to-print**](https://github.com/gregnb/react-to-print) >**安裝：** yarn add react-to-print

**react-to-print** 主要的功能是幫我們『列印選取的 HTML 內容』，如果去看它的程式碼可以發現作者透過 iframe.contentWindow.print() 來呼叫 browser 的列印功能。

#### 簡單介紹 react-to-print 在使用的幾種方式：

1. **use ReactToPrint component**
   **class component** 的方法跟上面的 `react-to-pdf` 很類似，我們可以透過 `<ReactToPrint>` 來處理想要列印的區塊。

```jsx=
import ReactToPrint from 'react-to-print';
function Print(){
    const printRef = createRef()
    // render 需要列印的相關內容
    function renderUseReactToPrintComponent() {
    return (
      <>
         <ReactToPrint
          // trigger 會在我們傳入的 React Component or Element 上面加上一個 onClick 事件，因此我們不需要自己去寫 onClick 事件
          trigger={() => <Button variant='contained'>列印 or 儲存成 PDF (Print or Save to PDF)</Button>}
          // content 主要回傳我們要列印的內容，我們透過 ref 來 reference 需要列印的區塊(DOM)
          content={() => printRef.current}
        />
        <ReactToPrintContainer ref={printRef}>
          <h1>MUI Table To Print</h1>
          {/* 假資料 Table */}
          {renderMuiTableView()}
        </ReactToPrintContainer>
      </>
    )
  }
    return (
      <main className={styles.main}>
        <h1>將下面內容透過 react-to-print 套件列印內容</h1>
        {/* 使用 component 版本 */}
        {renderUseReactToPrintComponent()}
      </main>
    )
}
```

2. **use useReactToPrint Hook**
   **react-to-print** 也很貼心的提供 `useReactToPrint` Hook 的方式來讓我們用更少的步驟完成列印的功能。
   仔細看可以發現，Hook 與 Component 所需要設定的內容很相似，差別在於一個是在 Hook 的 function argument 上，一個是寫在 Component props 上。而兩種方法沒有誰好誰壞，可以自行依照實際情境來選擇其中一種方式使用。

```jsx=
import { useReactToPrint } from 'react-to-print'
function Print(){
    const printRef = createRef()
    // 產生列印的 trigger function
    const handlePrint = useReactToPrint({
        // 我們透過 ref 來 reference 需要列印的區塊(DOM)
        content: () => printRef.current,
    })
    function renderUseReactToPrintHook() {
        return (
          <div>
            <Button variant='contained' onClick={handlePrint}>
              列印 or 儲存成 PDF (Print or Save to PDF)
            </Button>
            <ReactToPrintContainer ref={printRef}>
              <h1>MUI Table To Print</h1>
              {/* 假資料 Table */}
              {renderMuiTableView()}
            </ReactToPrintContainer>
          </div>
        )
  }
    return(
        <main className={styles.main}>
            <h1>將下面內容透過 react-to-print 套件列印內容</h1>
            {/* 使用 hook 版本 */}
            {renderUseReactToPrintHook()}
        </main>
    )
}
```

### 成果

可以看到當我們按下『列印』按鈕後，自動跳出了 Browser 內建的列印畫面，讀者們可以自行在 console 中用 `window.print()` 嘗試看看，可以發現會跳出一樣的列印畫面，差別在於 **react-to-print** 套件只列出了『我們所選取的部分』而已。( ps.`window.print()` 預設是印整個 window 畫面）

如果是使用 Mac 電腦可以看到有『儲存為 PDF』的選項，因此如果是想要下載 PDF 的話，可以考慮直接使用 **react-to-print** 即可。(ps. up 因為沒 windows 電腦，因此 windows 系統不確定有沒有下載的部分。)

![](https://i.imgur.com/IwywvBu.gif)

## 工作實作補充

### 1. Print 預覽時，最後面永遠多一頁『空白頁』

可以透過在你的 Component 中的最外層加入 `overflow:hidden` 來解決。

```jsx=
const Page = () =>{
    const handlePrint = useReactToPrint({
        content: () => /* 下面那個 Component，詳細設定請回看上面內容*/,
    })
/*
 * 我是一個要來觸發產生 Print 預覽的頁面
 * (ex.帳單頁、說明頁...等)
*/
    return (
        <div>
            <PrintComponent ref={...xxx}/>
        </div>
    )
}

const PrintComponent = () =>{
    /*
     * 這裡是放你要『列印的內容』
     * 簡單舉例如下：*/
    return(
        /* 重點是這邊～～～～
         * 最外層加入  overflow:'hidden' 來防止『無謂的空白頁』*/
        <div style={{overflow:'hidden'}}>
            <p>帳單詳細 PDF 檔</p>
            <p>日期：111/12/23</p>
            <p>內容：......xxxxx</p>
        </div>
    )
}

```

### 2. 透過 `@page` 與 `media @print` 來對 Print 頁面做一些設定

這部分目前測試下來 **Safari 不支援**，因此『**建議推薦使用者在 Chrome 上進行下載**』，直接分享一下實務上用的設定

```css=
 @page {
    size: auto;
    margin: 0mm /* 可依照專案自行調整 */
  }

  @media print {
    body {
      -webkit-print-color-adjust: exact;
    }
    tr {
      /* 為了不要讓 tr 在中間被換頁切掉
       * 此設定代表：每次都以一個 tr 為基準，『禁止在 tr 中間被切割』*/
      page-break-inside: avoid;
      break-inside: avoid;
    }
    .page-break {
      /* 強制該 區塊 後面直接換頁 */
      page-break-after: always;
    }
    .page-break-avoid {
      /* 禁止該 區塊 中間『不可被換頁』 */
      page-break-inside: avoid;
      break-inside: avoid;
    }
  }
```

### 3. `PrintComponent` 只在 Print 預覽時顯示

可以在 **Page** 層使用 `PrintComponent` 時，外面多包一個 DOM 並加上 `display:none` 來隱藏該元件。

```jsx=
<div style={{ display: "none" }}><ComponentToPrint ref={componentRef} /></div>
```

詳細可參考：[How do you make ComponentToPrint show only while printing - react-to-print](https://github.com/gregnb/react-to-print#how-do-you-make-componenttoprint-show-only-while-printing)

## 套件介紹：React-PDF (Render PDF 內容到頁面中)

> **GitHub**：[**React-PDF**](https://github.com/wojtekmaj/react-pdf) > **安裝**： yarn add react-pdf

上面介紹的 **react-to-pdf** 與 **react-to-print** 都是偏向於將 HTML 內容產生成 PDF 檔案，然而在工作上比較常碰到的需求是『將 PDF 內容呈現在網頁上』，例如將『法規』、『使用條款』...等 PDF 內容呈現在網頁上，如果單純使用 Html 來排版可能會花上幾個小時的時間，而且這種客製化的頁面很難必此之間共用、且維運上也比較麻煩，每當內容有調整時就需要去檔案中尋找該行程式碼來修改。

因此如果可以直接將 PDF 檔案 Render 到頁面上，就可以減少很多這方面的功夫，每當內容有調整時，只需要將舊的 PDF 文件覆蓋掉即可，甚至也可以寫一個共用的 `<PdfViewer/>` Component 來專門處理 Render PDF 檔案的需求，是不是感覺很方便啊！！！

### 官方範例

在使用 **react-pdf** 時最核心的 Component 就是：`Document` 與 `Page` 這兩個 Component。
我們主要透過將 PDF 文件路徑傳給 `Document` 的 `file` 屬性中，讓它能夠去讀取這份文件，而 `Page` 主要負責將 PDF 內的頁面渲染到網頁上，因此我們需要透過 `pageNumber` 這個屬性來『告訴 `Page` 要讀取哪一頁』。

#### 我們先來看一下官方範例

```jsx=
// 引入 Document, Page  兩個核心 Component
import { Document, Page } from 'react-pdf';

function MyApp() {
  // 紀錄 PDF 總共多少頁
  const [numPages, setNumPages] = useState(null);
  // 紀錄當前頁數
  const [pageNumber, setPageNumber] = useState(1);
  // 當 PDF 讀取成功時呼叫 cb
  function onDocumentLoadSuccess({ numPages }) {
    setNumPages(numPages);
  }

  return (
    <div>
      {/*
        * file: 檔案路徑、Url
        * onLoadSuccess： 當 PDF 檔案讀取成功時，將該份 PDF 的總頁數存進 useState 中，方便之後『切換頁面』或是『一次 Render 全部』 使用
        *
        * */}
      <Document file="somefile.pdf" onLoadSuccess={onDocumentLoadSuccess}>
        <Page pageNumber={pageNumber} />
      </Document>
    </div>
  );
}
```

### 建立共用 PDFViewer Component

大概知道套件該如何使用後，現在我們來建立一個共用的 PDFViewer Component 吧，而在實作過程中陸陸續續碰到了一些問題，以下為問題相關解法：

1. **Failed to load PDF file.**
   會出現這個問題基本上是因為沒有設定 PDF Worker，因此我們需要從 `pdfjs-dist` 中引入 Worker 並且將 Worker 設定到 `GlobalWorkerOptions.workerSrc` 上即可。[參考文件](https://github.com/mozilla/pdf.js/issues/8305#issuecomment-1004054543)

2. **使用 Nextjs 在 SSR 請況下因為 PDF.js 需要 browser API 的支援因此導致報錯問題。**
   透過 dynamic import 上設定 ssr 為 false 解決 browser api 在 node 環境下不支援問題。[參考文件](https://github.com/wojtekmaj/react-pdf/issues/136#issuecomment-405393139)

#### 完整程式碼

```jsx=
import Pagination from '@mui/material/Pagination'
import Stack from '@mui/material/Stack'
import styled from 'styled-components'
import { Document, Page, pdfjs } from 'react-pdf'
import { useState } from 'react'
import pdfjsWorker from 'pdfjs-dist/build/pdf.worker.entry'
pdfjs.GlobalWorkerOptions.workerSrc = pdfjsWorker

/*
 * isUsePaginationMode : 是否要用分頁模式 (預設一次將 PDF 內容全部渲染出來)
 * pdfUrl： PDF 文件路徑
 * */
export default function PDFViewer({ isUsePaginationMode = false, pdfUrl }) {
  // 紀錄 PDF 總共多少頁
  const [totalPdfPages, setTotalPdfPages] = useState(null)
  // 紀錄當前頁數
  const [pageNumber, setPageNumber] = useState(1)
  // 抓取當前 視窗寬度 (優化方面：可監聽 window.resize 調整)
  const windowWidth = window.innerWidth
  // 當 PDF 讀取成功時呼叫 cb
  function onDocumentLoadSuccess({ numPages }) {
    setTotalPdfPages(numPages)
  }
  // render Pdf 全部頁面內容
  function renderAllPdfPages() {
    if (!totalPdfPages) return
    const mockArray = [...new Array(totalPdfPages)]
    return mockArray.map((_, index) => {
      return <Page width={windowWidth <= 768 ? windowWidth : 1024} key={`page-${index}`} pageNumber={index + 1} />
    })
  }
  // 依照 分頁形式 render Pdf 內容
  function renderPaginationPdf() {
    return (
      <>
        <Page
          width={windowWidth <= 768 ? windowWidth : 1024}
          height={windowWidth <= 768 ? window.innerHeight : 500}
          key={`page-${pageNumber}`}
          pageNumber={pageNumber}
        />
        <PaginationWrapper>
          <Stack spacing={2}>
            <Pagination
              count={totalPdfPages ?? 0}
              variant='outlined'
              color='primary'
              onChange={(event, value) => {
                console.log('value', value)
                setPageNumber(value)
              }}
            />
          </Stack>
          <PaginationSizeWrapper>
            {pageNumber} / {totalPdfPages ?? 0}
          </PaginationSizeWrapper>
        </PaginationWrapper>
      </>
    )
  }
  return (
    <>
      {/*
       * file: 檔案路徑、Url
       * onLoadSuccess： 當 PDF 檔案讀取成功時，將該份 PDF 的總頁數存進 useState 中，方便之後『切換頁面』或是『一次 Render 全部』 使用
       *
       * */}
      <Document file={pdfUrl} onLoadSuccess={onDocumentLoadSuccess}>
        {isUsePaginationMode ? renderPaginationPdf() : renderAllPdfPages()}
      </Document>
    </>
  )
}

```

#### 使用 PDFViewer 範例

```jsx=
import dynamic from 'next/dynamic'
/* 重點在這邊～～～～～ */
const PDFViewer = dynamic(import('../src/components/pdf/PDFViewer'), { ssr: false })

function ReactPdfRender() {
  const [isUsePaginationMode, setIsUsePaginationMode] = useState(false) // 是否要使用分頁模式
  return (
    <div>
      <Head>
        <title>React render PDF</title>
        <meta name='description' content='Generated by create next app' />
        <link rel='icon' href='/favicon.ico' />
      </Head>
      {/* 重點在這邊～～～～～ */}
      <PDFViewer pdfUrl={'/pdf/Monsta-Infinite.pdf'} isUsePaginationMode={isUsePaginationMode} />
    </div>
  )
}
export default ReactPdfRender
```

### 成果

![](https://i.imgur.com/qnD5D6P.gif)

#### 以上為【 React PDF 相關套件 】的簡單介紹，希望對剛好有碰到相關需求的人能有一點點幫助，如有任何錯誤或冒犯的地方還請各位多多指教，謝謝您的觀看。

#### GitHub : [https://github.com/librarylai/react-pdf-print](https://github.com/librarylai/react-pdf-print)

## Reference

1. [react-to-pdf](https://github.com/ivmarcos/react-to-pdf#readme)
2. [react-to-print](https://github.com/gregnb/react-to-print)
3. [react-pdf](https://github.com/wojtekmaj/react-pdf)
