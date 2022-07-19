# 【筆記】演算法 - 廣度優先搜尋法(BFS) 與 深度優先搜尋法(DFS)

###### tags: `筆記文章`

![](https://evatronix.com/images/en/offer/product-design/product-development/Evatronix_Algorithm_development_and_analysis_01_1920x1080.jpg)

廣度優先搜尋法(BFS) 與 深度優先搜尋法(DFS)，不管是在演算法的基礎知識中或是在工作實務方面，都是一門很重要且必須要知道的課題，在實務上我們常常需要與 **樹狀結構(tree)** 打交道，像是查找某一項目是否存在於樹狀結構(tree)中，而如果有這方面演算法知識的話，將會大大減少我們在寫查詢功能的時間且也會更加有效率。

## 廣度優先搜尋法(BFS) 與 深度優先搜尋法(DFS) 介紹

當我們想優化我們的搜尋功能時，不知你有沒有想過，如何搜尋才是最有效率或是哪一種方式才是最符合該此情況的時間複雜度呢？這邊就讓我們先個別介紹一下廣度優先搜尋法(BFS) 與 深度優先搜尋法(DFS)吧！

### 廣度優先搜尋法(BFS)

![](https://i0.wp.com/c-program-example.com/wp-content/uploads/2011/10/bfs.jpg?resize=489%2C464)

**首先，先來看看維基百科怎麼說：**

> 廣度優先搜尋演算法（Breadth-First Search，縮寫為 BFS），是一種圖形搜尋演算法。簡單的說，BFS 是從根節點開始，沿著樹的寬度遍歷樹的節點。如果所有節點均被存取，則演算法中止。

廣度優先搜尋演算法指的是，當我們想從 **某一個節點(vertex)搜尋一個目標節點(target)** 時，會從此節點開始**往相鄰且尚未訪問過**的節點開始尋找，且會將**同一層級裡所有節點查尋完畢後**，再從這些相鄰(同一層)的節點**繼續往下一層尋找**，直到找到目標節點或全部搜尋為止。
所以廣度優先搜尋演算法可以說是用『分層』的概念去進行整個流程搜索。

### 深度優先搜尋法(DFS)

![](https://upload.wikimedia.org/wikipedia/commons/thumb/1/1f/Depth-first-tree.svg/1280px-Depth-first-tree.svg.png)
**維基百科：**

> 是一種用於遍歷或搜尋樹或圖的演算法。這個演算法會儘可能深的搜尋樹的分支。當節點 v 的所在邊都己被探尋過，搜尋將回溯到發現節點 v 的那條邊的起始節點。這一過程一直進行到已發現從源節點可達的所有節點為止。

**深度優先搜尋法指的是：**

1. 決定一個 **初始節點(root)** 並做上記號
2. 接下來探索邊(edge)上，**下一個相連且無記號的節點**，找到後將節點做上記號並繼續往下尋找
3. 重複第二步驟，直到進行到一個**不再與其他節點相連的節點(dead-end)**
4. 則『**回溯**』到『**前一個還有相鄰並且無記號的節點**』然後繼續重複第二步驟
5. 直到**找到目標節點**或**全部搜尋**為止。

## 廣度優先搜尋法(BFS)與 深度優先搜尋法(DFS) 舉例：

**這邊用一個自己在 CodeSandbox 的練習專案當作例子!!!**

![](https://i.imgur.com/HPe7TOr.png)

**題目:找出 constants 底下的 index.js 這隻檔案。**

### 廣度優先搜尋法(BFS)

如果使用廣度優先搜尋法來搜尋的話，將會從第一層開始平面是搜尋，所以會先看 public 資料夾，接著看 src 資料夾，package.json 檔案。發現都沒有找到，則開始往第二層前進，就會看到 components 和 constants 這兩個資料夾。依舊還沒找到則再往第三層搜尋，會先搜尋 components 底下的 DynamicLazy 與 StaicLazy 這兩隻檔案，接著再到 constants 底下的 index.js，因為找到目標所以停止搜尋。

### 深度優先搜尋法(DFS)

如果使用深度優先搜尋法的話，會先看 public 資料夾以及底下的 index.html 檔案，接著看 src -> conponents -> DynamicLazy 和 StaicLazy 這兩隻檔案，會是直線式的深入到底，然後回到上一層相鄰的資料夾繼續深入搜尋，所以接下來會搜尋 constants -> index.js ，因為找到目標，所以不會在繼續往下搜尋了。

## 程式碼範例

**接下來的程式碼範例，將會使用下圖的樹狀結構來舉例！**
程式碼邏輯主要為：當尋找目標為（3BBB）時，**BFS** 與 **DFS** 各自的運作流程，以及碰到已訪問過的節點時將會跳過不再次訪問。

### 廣度優先搜尋法(BFS)

![](https://i.imgur.com/arFQfrE.gif)

```javascript=
/**
 * @param {array | object } root 根節點,也代表全部資料
 * @param {object} target  查詢目標
 * @param {string} targetArr 比對屬性
 */

export default function BFS(root, target, targetArr) {
  let queue = []; // 將根節點放入 queue 中 , 接下來將使用 queue 來進行 bfs 查找ㄝ
  let level = 0; // 層數
  let visitedList = []; // 已訪問節點列表

  if (Array.isArray(root)) queue = [...root];
  // 如果 傳入資料(root) 是陣列則 解構 後放入queue
  else queue.push(root); // 如果 傳入資料(root) 是物件 則 push 進 queue

  while (queue.length) {
    // 當 queue 裡面有值 就繼續進入迴圈
    level += 1; // 層數 +1
    let q_length = queue.length;
    for (let i = 0; i <= q_length; i++) {
      let cur_queue_obj = queue.shift(); // 取出目前 queue 裡面最前面的項目  ex.queue[0]
      // 是否 已訪問過 此節點
      let isVisited = visitedList.find(
        (visitedItem) => visitedItem[targetArr] === cur_queue_obj[targetArr]
      );
      if (isVisited) break; // 如果以訪問過 則跳過
      visitedList.push(cur_queue_obj); // 將節點加入到 已訪問清單
      // 如果 查詢目標屬性的值 等於 取出物件屬性的值 則代表找到目標了, 回傳層數
      if (target[targetArr] === cur_queue_obj[targetArr]) return level;
      // 如果沒找到 且 物件內還有子項目 , 則將 子項目 push 進 queue 列表中, 等待下一層尋找
      if (cur_queue_obj.neighbors && cur_queue_obj.neighbors.length) {
        queue.push(...cur_queue_obj.neighbors);
      }
    }
  }
  return null;
}


```

### 深度優先搜尋法(DFS)

![](https://i.imgur.com/JUpFAQy.gif)

```javascript=
/**
 * @param { object } treeData 根節點,當前資料節點
 * @param { object } target  查詢目標
 * @param { string } targetArr 比對屬性
 * @param { number } level 層數
 * @param { array } visitedList 已訪問節點列表
 */

export default function DFS(
  treeData,
  target,
  targetArr,
  level = 1,
  visitedList = []
) {
  // 是否 已訪問過 此節點
  let isVisited = visitedList.find(
    (visitedItem) => visitedItem[targetArr] === treeData[targetArr]
  );
  if (isVisited) return; // 如果以訪問過 則跳過
  visitedList.push(treeData); // 將節點加入到 已訪問清單
  if (treeData[targetArr] === target[targetArr]) return level; // 如果找到目標了, 則回傳層數
  let neighbors = treeData.neighbors; // 找出節點 的 鄰居節點 或是 子節點
  if (treeData && neighbors && neighbors.length) {
    // 如果有 鄰居節點 或是 子節點 , 則話下繼續尋找
    for (let i = 0; i < neighbors.length; i++) {
      let deepLevel = DFS(
        neighbors[i],
        target,
        targetArr,
        level + 1,
        visitedList
      );
      // 如果有找到目標 level 則 return 結束搜尋
      if (deepLevel) return deepLevel;
    }
  }
  // 玩全沒找到 則 return null
  return null;
}

```

#### CodeSandBox: [BFS&DFS](https://codesandbox.io/s/bfsdfs-7gz3z?file=/src/units/BFS.js:0-949)

<iframe src="https://codesandbox.io/embed/bfsdfs-7gz3z?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="BFS&amp;DFS"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

## 總結

其實在生活中我們常常會用到『廣度優先搜尋法(BFS)』 和『深度優先搜尋法(DFS)』，只是我們沒有發現而已，例如：當要找尋存在於Ｄ槽的某一款遊戲時，會先打開Ｄ槽然後看裡面所有資料夾，再進入到 game 資料夾裡面找到該遊戲的資料夾。這種一層一層往下尋找的方式，就是 **『廣度優先搜尋法(BFS)』**。而 **『深度優先搜尋法(DFS)』** 就像是我們跟著 GPS 的路徑去尋找目的地時，我們會跟導航往前走，而當不小心走錯路時，我們會退回前一個交叉口往另一條前進，直到走到最終的目的地。

**以上就是簡單對這兩種搜尋法的介紹，謝謝大家。**

## Reference

1. [佇列的 JS 實現及廣度優先搜尋（BFS）的實現 - geology](https://segmentfault.com/a/1190000016900956)
2. [深度優先搜尋(DFS)和廣度優先搜尋(BFS)演算法，實用的節點搜尋法 - Magic Len](https://magiclen.org/dfs-bfs/)
3. [深度優先搜尋 - 維基百科](https://zh.wikipedia.org/wiki/%E6%B7%B1%E5%BA%A6%E4%BC%98%E5%85%88%E6%90%9C%E7%B4%A2)
