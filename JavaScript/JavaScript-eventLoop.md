# 【筆記】 Event loop、Call Stack 與 Task Queue

###### tags: `筆記文章`

在開始探討主題之前，首先要知道瀏覽器是擁有多個 Process 的，像是每一個分頁都是一個 Process，而今天要講的 Event loop 機制其實跟瀏覽器的 Main Thread 有關， 而 Main Thread 存在於 Rending Process 中， 所以就讓我們先來了解一下 Rending Process 在做什麼吧！？

## Rending Process 渲染程序

> The renderer process’s core job is to turn HTML, CSS, and JavaScript into a web page that the user can interact with.
> Rending Process 主要任務是將 HTML，CSS，以及 JavaScript 轉變為使用者可以互動的網頁內容。

**Rending Process 內包含以下 Thread :**

- Main Thread (負責處理大部分的代碼)
- Worker threads (負責處理 web worker or service worker)
- Compositor thread (負責高效率流暢的渲染畫面)
- Raster thread(負責高效率流暢的渲染畫面)

![Inside look at modern web browser (part 3)](https://developers.google.com/web/updates/images/inside-browser/part3/renderer.png)

而到底什麼是 Main Thread 呢？它又是做什麼來著的？其實很多東西都會在 Main Thread 上執行，像是 Main Thread 中包含了 JS Engine、GUI Render Engine 負責執行 Javscript、執行畫面更新...等，而這些執行大部分都是有明確的先後順序的，不然容易會造成 race condition (同一個 DOM 同時被渲染又被編輯)，所以當如果有一段程式碼需要執行很長一段時間的話，整個畫面就會被 block 住。

正因如此我們需要將這些耗時的東西拉出去執行來解決這件事情，也就是我們常說的『非同步執行』，瀏覽器會『並行』執行這些程式，當執行處理完成後先放回到 task queue 中，之後才回到 Main thread，而 Event loop 就是再協調這一切。

**(推薦看一下這部演講) [Jake Archibald: In The Loop - JSConf.Asia](https://www.youtube.com/watch?v=cCOL7MC4Pl0)**

JavaScript 是單執行緒(單線程 Single Thread )的程式語言，因此一次只能執行一個任務。可以先看看下圖。

![Web browser JS main thread](https://miro.medium.com/max/4564/1*PiFyb7IV8vTDCGEeUOWLVQ.jpeg)
(Reference: Anish Giri — Call Stack, Event Loop, and Task Queue)

### 阻塞（blocking）

因 JS Engine 和 GUI Render Engine 都是在同一條 Main Thread 上，所以當 JS Engine 需要花很長時間在處理一個 task 時，就會造成畫面的卡頓也被稱為阻塞，就如同圖片上的白點一樣，停在黃色的 task 上不停的處理該 task 上的程式，而造成整個循環不再流動了。

![...](https://camo.githubusercontent.com/d42e2b1adb08501999846b740847a9318f816ed190c5170f34e1488f95746b1e/68747470733a2f2f7062732e7477696d672e636f6d2f6d656469612f4466697a744f5f55454141627231772e6a7067)

(Reference: Jake Archibald: In The Loop - JSConf.Asia 2018)

**讀者們可以操作看看下面這個例子**

1. 首先可以先透過按鈕修改 P 標籤的文字看看
2. 再點擊 「Start Blocking JS Engine」 按鈕
3. 然後再修改 P 標籤的文字看看

當「**Start Blocking JS Engine**」 按鈕被點擊後，你將會發現畫面開始有卡頓的現象，且點擊修改 P 標籤的按鈕也沒有反應，可以想像成 Main Thread 正在忙著處理 JS Engine 內的程式，沒空理會 GUI Render Engine 的畫面更新。

[Codepen 連結](https://codepen.io/librarylai/pen/qBbLBpW)

### 同步執行(Synchronous) & Stack(堆疊)

我們一般對程式碼的理解大概都是同步執行(Synchronous)，程式碼由上而下一步一步執行（Ａ完成了才去做Ｂ），且一次執行一個指令或函式。而等待執行的 Task(程式) 會被放入 Stack(堆疊) 中。

#### 何謂 Stack (堆疊)呢？

堆疊是一種後進先出( **Last In, First Out** )的資料結構，它就像是餐飲業的餐盤一樣，最先洗好的餐盤疊在最下面，越後面的會被放在(Push)越上面，而我們使用時也會從最上面的開始抽(Pop)。

有了同步執行(Synchronous) & Stack(堆疊)的概念後，我們透過一個比較複雜一點的例子來說明：

當程式碼執行碰到函式(function)時，且這個函式內又包著其他函式(如下圖)，這時 JavaScript Engine 將會把等待執行的 Task (function) 放入 Call Stack 中，並且會從最後一個函式開始往回執行，且當 Task 執行完畢時將會從 Call Stack 中移除。(如下圖)

![Call Stack, Event Loop, and Task Queue](https://miro.medium.com/max/1200/1*e3qiT_22FbnEep63BJN2_g.gif)(Reference: Francesco — JavaScript main thread.Dissected.)

### 非同步執行(Asynchronous)

因 JavaScript 是單執行序的一次只能做一件事，但可以透過瀏覽器提供的 API (WebAPI)來完成非同步處理。例如當程式執行碰到 setTimeout、Promise、AJAX、XMLHttpRequest…等 WebAPI 時，會將它們拉出來個別去處理。

以 setTimeout 為例的執行流程 :

1. 當執行碰到 setTimeout 時會將 setTimeout 從 JS Engine 拉出來丟到瀏覽器的 WebAPI 中去執行。
2. 當 setTimeout 執行完成時，會將 callback 丟到 task queue 中等待被執行。
3. 等到 call stack 裡面都執行完後，才會執行 task queue 裡的 setTimeout callback function。

### 何謂 Event Loop , Microtasks、Macrotasks

> 簡單來說 Event Loop 就是負責協調 call stack 和 callback queue(task queue、event queue)，當 Call stack 裡沒有需要執行的程式時，則會將 callback queue 的內容丟進來執行。

#### 關於 Microtask、Macrotask

**Macrotask** 就是 task queue，包含了 DOM 操作、事件監聽、事件操作、setTimeout、setInterval、setImmediate、I/O、UI rendering...等

**Microtask** 則包含了 process.nextTick, Promises, Object.observe, MutationObserver

我們可以從圖片中很明顯的看到 Macrotask 與 Microtask 的執行流程，還記得我們上面有提到 task queue(Macrotask) 是要等到 call stack 清空後才會被執行，而每次會將一個 oldest Macrotask 丟回 call stack 中執行，且執行完成後會往下去執行 Microtask。

Microtask 的特性則是會一直執行直到 Microtask queue 裡為空為止，當 Microtask 執行完後才會開始進行 UI rendering。

![Macrotasks 與 Microtasks 差別](https://yeefun.github.io/static/e53e7455f561eeb2a702370fc75b7865/0a151/event-loop-process.jpg)
(Reference: 我知道你懂 Event Loop，但你了解到多深？ - Yeefun )

#### 依照上面解釋的說法想想看這個例子您的答案是什麼？

```javascript
console.log('start')
setTimeout(() => {
  Promise.resolve()
    .then(() => {
      console.log('promise2')
    })
    .then(() => {
      console.log('promise3')
    })
}, 0)
setTimeout(() => {
  console.log('setTimeout')
}, 0)
Promise.resolve().then(() => {
  console.log('promise1')
})
console.log('end')
```

#### 公佈解答

```javascript=
----- log -----
start
end
promise1
promise2 (callback queue 中的第一個callback)
promise3
setTimeout(callback queue 中的第二個callback)
```

**現在你是不是會覺得有點奇怪，依造上面的說法不是要先經過 Macrotask 才會執行 Microtask 嗎？**

答案是沒錯的！整個流程可以分成兩種思考方式：

1. 當 call stack 為空時如果 Macrotask queue 有東西，則會先一個將 Macrotask 加入到 call stack 中執行，當 Macrotask 執行完後 call stack 空了，就會執行 Microtask 到有 Microtask queue 為空為止。

2. 當今天 Macrotask queue 本身就為空時，因此沒有東西能加入到 call stack 中，所以 call stack 也就空了，則就會直接去檢查 Microtask queue 並執行直到有 Microtask queue 為空為止。

兩者都會先查看 Macrotask queue 之後才會去執行 Microtask，而我們時常也會看到『當 call stack 為空時會去執行 Microtask』這句話其實也是對的，因為基本上當 Macrotask 執行完時 call stack 會是空的。

#### 最後回到今天這個例子：

當一開始執行時，碰到了 setimeout 會被丟 WebAPI 中直到時間到了才會被丟到 task queue 裡面，而這時 event loop 其實已經走完了第一次，當它執行完 call stack 裡的 `start`、`end` 後，查看了 task queue(Macrotask queue) 發現裡面是空的就去執行了 Microtask queue ，也就是 `promise1` 的部分，因此當 setimeout 從 WebAPI 回來被丟進 task queue 時，`promise1` 早已被執行完了，才會依序執行 `promise2` 與 `setTimeout`（兩者都是 callback function 會被放進 task queue 裡面依序被執行，因此如果將兩者位置顛倒 log 也會顛倒）。

## 總結

最後總解一下整個 Event Loop 的流程:

1. 執行 Call Stack 內程式
2. 當 Call Stack 為空時，會將最老的 Macrotask 從 Macrotask queue 中放到 Call Stack 中(如果無則跳到步驟 4)
3. 執行 Call Stack 中的 Macrotask
4. 執行完後這時 Call Stack 會是空的才會去檢查 Microtask queue
5. 執行 Microtask queue 裡面的 Microtask 直到為空為止
6. 最後 Microtask 執行完後才開始頁面的渲染

#### 以上是個人看完以下大神的文章後所作的筆記與見解，如有任何錯誤或冒犯的地方還請各位多多指教。

### 謝謝觀看。

## Reference

1. [PJCHENder blog — [筆記] 理解 JavaScript 中的事件循環、堆疊、佇列和併發模式（Learn event loop, stack, queue, and concurrency mode of JavaScript in depth）](https://pjchender.blogspot.com/2017/08/javascript-learn-event-loop-stack-queue.html)
2. [Mariko Kosaka — Inside look at modern web browser (part 3)](https://developers.google.com/web/updates/2018/09/inside-browser-part3)
3. [Hsuan-Ju Lin — 瀏覽器是如何做渲染的?](http://mis101bird.js.org/browserwork/)
4. [Li Mei — JS：详解 Event Loop 运行机制，以及 microtasks 和 macrotask 的执行顺序](https://limeii.github.io/2019/05/js-eventloop/)
5. [keifergu — 理解 JavaScript 中的 macrotask 和 microtask](https://juejin.im/entry/58d4df3b5c497d0057eb99ff)
6. [育能(Harry Xie) — 一次了解 JS 中的 Event loop、Call Stack 與 Task Queue ](https://medium.com/harry-xies-blog/%E4%B8%80%E6%AC%A1%E4%BA%86%E8%A7%A3-js-%E4%B8%AD%E7%9A%84-event-loop-call-stack-%E8%88%87-task-queue-c8041dd8840f)
7. [極客搬運工 — JS 執行機制](https://kknews.cc/code/veqpony.html)
8. [Francesco — JavaScript main thread.Dissected.](https://medium.com/@francesco_rizzi/javascript-main-thread-dissected-43c85fce7e23)
9. [Anish Giri — Call Stack, Event Loop, and Task Queue](https://medium.com/jsninja/call-stack-event-loop-and-task-queue-d49eff2ec7bb)
10. [Lydia Hallie — JavaScript Visualized: Promises & Async/Await ](https://dev.to/lydiahallie/javascript-visualized-promises-async-await-5gke)
11. [Yeefun - 我知道你懂 Event Loop，但你了解到多深？](https://yeefun.github.io/event-loop-in-depth/)
