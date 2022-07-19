# 【筆記】演算法 - 二分搜尋法 (Binary Search)

###### tags: `筆記文章`

![](https://i.stack.imgur.com/R5kJk.gif)

## 基本：二分搜尋法

相信其實不少人在生活中早已使用過『二分搜尋法』這樣的概念在處理事情，像是小時候應該大家都過『終極密碼』吧！簡單說就是猜數字遊戲，不知大家還記不記得當我們在玩終極密碼時，常常第一次一定會喊『50』然後可能會再喊『75』或『25』，直接用對半砍的方式快速的接近目標數，而這樣的方式就是『二分搜尋法』。

### 二分搜尋法概念：

二分搜尋法 (Binary Search) 的執行前提是在『已排序』的陣列資料中，每次都判斷陣列內正中間的值是否為要搜尋的目標值，如果沒有找到，則比較中間值與目標值的大小，如果中間值『大於』目標值則使用陣列左半邊的資料再重複以上的步驟，直到找到為止。反之中間值『小於』目標值則使用右半邊的資料。

### 整體步驟大概是以下：

1. 排序陣列
2. 找出陣列中的『中間值』，如果偶數則取 min-1 位 (Math.floor)
3. 判斷『中間值』是否等於 『目標值』，如果是則回傳 index
4. 比較『中間值』大於小於 『目標值』
5. 中間值『大於』則使用 陣列『左』半邊資料，再重複查詢步驟
6. 中間值『小於』則使用 陣列『右』半邊資料，再重複查詢步驟

### 實作二分搜尋法

#### While 迴圈方式：

```javascript=
// 排序陣列
const arr = [1, 8, 5, 10, 9, 20].sort((a,b)=>a-b);

const binarySearch = (data ,target) =>{
    let left = 0
    let right = data.length - 1
    while(left <= right){

        // [0,5] => min = 3
        const mid = Math.floor((left + right) / 2)
        // 如果 中間值 等於 目標值，則代表找到了(true)
        if(data[mid] === target) return true
        if(data[mid] > target){
            // 中間數已經找過了，所以往左一個
            // [0,2]
            right = mid - 1
        }
        if(data[mid] < target){
            // 中間數已經找過了，所以往右一個
            // [4,5]
            left = mid + 1
        }
    }
    // 沒找到 return false
    return false
}
```

#### 遞迴版本一：

```javascript=
const arr = [1, 8, 5, 10, 9, 20].sort((a, b) => a - b);

function binarySearch(data, target, left = 0, right = data.length) {
    // 取得 陣列內 正中間的 index (如果有小數點則無條件捨去 ex. 3.5 => 3)
    const midIndex = Math.floor((left + right) / 2);
    // 先判斷 如果 中間值 等於 目標值，則代表找到了(true)
    if (data[midIndex] === target) return true;
    // 如果左邊界超過右邊 代表此目標值 不太 陣列中
    // 舉例[0,6] -> [4,6] -> [5,6] -> [6,6] -> [7,6](超過了...不符合)
    if (left > right) return false;
    // 如果沒找到，則比較 中間值 與 目標值 大小
    // 如果 中間值 大於 目標值
    if (data[midIndex] > target) {
        // 因為 中間值 大於 目標值，因此代表目標值會位於 『陣列的前半段』
        // 因此將『右邊界』設定到『中間前一位』的 index
        // 並再次進行 二分搜尋
        nextRight = midIndex - 1;
        return binarySearch(data, target, left, nextRight);
    }
    // 如果 中間值 小於 目標值
    if (data[midIndex] < target) {
        // 因為 中間值 小於 目標值，因此代表目標值會位於 『陣列的後半段』
        // 因此將『左邊界』設定到『中間後一位』的 index
        // 並再次進行 二分搜尋
        nextLeft = midIndex + 1;
        return binarySearch(data, target, nextLeft, right);
    }
}

/* 測試 */
binarySearch(arr, 5)  // 3
binarySearch(arr, 19) // false

```

> 補充：關於邊界的部分： 詳細可參考[二分搜尋法（Binary Search）完整教學（一） - Arthur Lin](https://medium.com/appworks-school/binary-search-%E9%82%A3%E4%BA%9B%E8%97%8F%E5%9C%A8%E7%B4%B0%E7%AF%80%E8%A3%A1%E7%9A%84%E9%AD%94%E9%AC%BC-%E4%B8%80-%E5%9F%BA%E7%A4%8E%E4%BB%8B%E7%B4%B9-dd2cd804aee1) 的文章。
>
> #### 以下補充節錄自該文章內容
>
> 一般定義區間時，會分開區間與閉區間，代表的是有沒有包含兩端點，數學上會分別用中括號和小括號表示。
>
> a. [3, 5]: 閉區間，代表 3, 4, 5 （包含 3、5）
>
> b. (4, 9): 開區間，代表 5, 6, 7, 8 （不包含 4、9）
>
> c. [1, 6): 左閉右開區間，代表 1, 2, 3, 4, 5 （包含 1、卻不包含 6 )

#### 遞迴版本二：

此範例是參考了 `PJCHENder` 大大的 [[演算法] Binary Search：在陣列中尋找特定元素 - PJCHENder](https://pjchender.dev/dsa/algo-binary-search) 文章後，發現也可以用 `slice` 來達成二分搜尋法，因此特別記錄下來。

```javascript=
// 排序陣列
const arr = [1, 8, 5, 10, 9, 20].sort((a,b)=>a-b);

const binarySearch = (data, target) => {
  // 取得 陣列內 正中間的 index (如果有小數點則無條件捨去 ex. 3.5 => 3)
  const midIndex = Math.floor(data.length / 2);
  // 右邊界
  const right = data.length - 1
  // 左邊界
  const left = 0
  // 先判斷 如果 中間值 等於 目標值，則代表找到了(true)
  if (data[midIndex] === target) return true;
  // 如果沒找到 則看一下現在 陣列內容 是否只剩下自己(長度為1)，
  // 因為每次我們都會透過 二分搜尋法 去刪減掉 陣列的長度，
  // 因此到最後 如果都沒找到 必定會剩下最後一個，
  // 如果連最後都沒找到 則代表 目表值不存在於陣列中(false)
  if (data.length === 1) return false;

  // 如果沒找到，則比較 中間值 與 目標值 大小
  // 如果 中間值 大於 目標值
  if (data[midIndex] > target){
    // 因為中間值已經比較過了，因此 取『前一位』的 index
    const midBeforeIndex = midIndex - 1
    // 再次丟進 二分搜尋法中查詢
    return binarySearch(data.slice(left, midBeforeIndex), target);

  }
  // 如果 中間值 小於 目標值
  if (arr[midIndex] < target){
    // 因為中間值已經比較過了，因此 取『後一位』的 index
    const midAfterIndex = midIndex+1
    // 再次丟進 二分搜尋法中查詢
    return binarySearch(data.slice(midAfterIndex,right+1),target);
  }
};

/* 測試 */
binarySearch(arr, 5)  // 3
binarySearch(arr, 19) // false


```

## 進階：多個目標值時的情況（沒有時，取大於目標值且最接近的項目）

如果是有多個答案的情況時該怎麼處理呢？剛好在閱讀二分搜尋法 (Binary Search)時看到了 `Arthur Lin` 大大的[二分搜尋法（Binary Search）完整教學（三）](https://medium.com/@lindingchi/binary-search-%E9%82%A3%E4%BA%9B%E8%97%8F%E5%9C%A8%E7%B4%B0%E7%AF%80%E8%A3%A1%E7%9A%84%E9%AD%94%E9%AC%BC-%E4%B8%89-%E5%BE%88%E5%A4%9A%E7%9B%B8%E5%90%8C%E7%9A%84%E6%83%85%E5%A2%83-c2215d1b9dc7)在講解如果碰到有多個相同答案時的情竟，大致上分成幾種情況：

1. **取最前面 index**
2. **取最後一個 index**
3. **沒找到時，取大於目標值且最接近的 index**

> 舉例來說： [1,3,6,6,6,10,12] (從 0 開始數)
>
> **取最前面 index** 就會為：2
> **取最後一個 index** 就會為：4
> **取大於目標值且最接近 index** 就會為：5

### 這邊以『取最前面 index』來舉例

#### 這邊以上面 `while 迴圈` 的範例來接續討論：

如果要除了要找尋目標值之外，還要再確認是否有陣列還有其他相同的值的話，那我們就不能『一找到時就直接回傳』，需要把整個陣列跑完再輸出結果，因此這邊要先將 ` if(data[mid] === target) return true` 這段先移除。

再來因為我們要取『最前面 index』所以當我們就算『找到』資料時，也要再『往前找看看』是否還有目標資料。

這邊把幾個關鍵字用『』框起來了，要往前找代表我們就算找到了也要把 `right` 往前縮到『當前 index 的前一位』，然後再繼續尋找看看，直到全部搜尋過為止。

> 舉例來說： [1,3,6,6,6,10,12] (從 0 開始數)
>
> **第一次**：目前`[left: 0, right: 6]`，取中間值 6 ，找到 6 但依舊往前確認，因此將 `right` 往前縮到『中間前一位( index: 2 )』。
>
> **第二次**：目前`[left: 0, right: 2]`，取中間值 3，因為小於目標值因此 `left` 往後推到『中間後一位( index: 2)』。
>
> **第三次**：目前`[left: 2, right: 2]`，取中間值 6，找到目標值將`right` 往前縮到『中間前一位( index: 1 )』。
>
> **第四次**：因為 right 小於 index 因此離開 `while 迴圈`，最後`left` 就是最接近(等於)目標值的 index。

#### 注意事項：

> **如果在『有找到』項目的情況下：**
> 不管是要『最後 index』還是『最前 index』都是取 `left` 的值，我們可以從下面的推倒來看，`left` 都會在目標值位置上。
>
> **如果在『沒找到』項目的情況下：**
> 這時 `left` 會是代表『大於目標值』且最靠近目標值的 index。

```javascript=
陣列：[1,3,6,6,6,10,12]
// 最前：
第一次:(l: 0, r: 6, m:4)
第二次:(l: 0, r: 2, m:1)[1,3,6]
第三次:(l: 2 ,r: 2, m:2)[6]
第四次:(l: 2 ,r: 1) ，right 跑到 left 前面，因此取 left:2
// 最前 - 找不到 (4)：
第一次:(l: 0, r: 6, m:4)
第二次:(l: 0, r: 2, m:1)[1,3,6]
第三次:(l: 2 ,r: 2, m:2)[6]
第四次:(l: 2 ,r: 1) ，right 跑到 left 前面，因此取 left:2

// 最後：
第一次:(l: 0, r:6, m:4)
第二次:(l: 4, r:6, m:5)[6,10,12]
第三次:(l: 4, r:4, m:4)[6]
第四次: right 跑到 left 前面，因此取 left: 4
// 最後 - 找不到 (4)：
第一次:(l: 0, r:6, m:4)
第二次:(l: 0, r: 2, m:1)[1,3,6]
第三次:(l: 2 ,r: 2, m:2)[6]
第四次:(l: 2 ,r: 1) ，right 跑到 left 前面，因此取 left:2

```

### 調整程式碼

#### While 迴圈方式(取最前面 index or 大於且最靠近的 index)：

```javascript=
// 排序陣列
const arr = [1,3,6,6,6,10,12]

const binarySearch = (data ,target) =>{
    let left = 0
    let right = data.length - 1
    while(left <= right){
        // [0,6] => min = 4
        const mid = Math.floor((left + right) / 2)
        // 注意這邊多了 等號，代表就算找到了，也繼續往前尋找。
        if(data[mid] >= target){
            // 中間數已經找過了，所以往左一個
            // [0,3]
            right = mid - 1
        }
        // 如果是『取最後一個 index』則是這邊加上 等號
        if(data[mid] < target){
            // 中間數已經找過了，所以往右一個
            // [4,5]
            left = mid + 1
        }
    }
    // 最後都回傳 left
    return left
}
binarySearch(arr,6) // 2
binarySearch(arr,7) // 5 ---> 大於它，且最接近
```

#### 遞迴方式：

```javascript=
// 排序陣列
const arr = [1,3,6,6,6,10,12]

function binarySearch(data, target, left = 0, right = data.length) {
    // 取得 陣列內 正中間的 index (如果有小數點則無條件捨去 ex. 3.5 => 3)
    const midIndex = Math.floor((left + right) / 2);
    // 如果左邊界超過右邊 代表此目標值 不太 陣列中
    // 舉例[0,6] -> [4,6] -> [5,6] -> [6,6] -> [7,6](超過了...不符合)
    // 此時的 left 代表最接近目標值的 index
    if (left > right) return left;
    // 如果 中間值 大於 目標值 或 找到目標值
    // 都繼續往前找看看，因為我們要取得的是『最前面 index』
    if (data[midIndex] >= target) {
        // 因此只要『中間值大於目標值』或『找到目標值』，
        // 都再去檢查『陣列前半段』的內容， 檢查是否還有符合目標值的項目。

        // 因此將『右邊界』設定到『中間前一位』的 index
        // 並再次進行 二分搜尋
        nextRight = midIndex - 1;
        return binarySearch(data, target, left, nextRight);
    }
    // 如果 中間值 小於 目標值
    // PS. 如果是要找 『最後一位 index』則是這邊加上 等號(<=)
    if (data[midIndex] < target) {
        // 因為 中間值 小於 目標值，因此代表目標值會位於 『陣列的後半段』
        // 因此將『左邊界』設定到『中間後一位』的 index
        // 並再次進行 二分搜尋
        nextLeft = midIndex + 1;
        return binarySearch(data, target, nextLeft, right);
    }

}

binarySearch(arr,6) // 2
binarySearch(arr,7) // 5 ---> 大於它，且最接近
```

## 進階： 除了『最前』或『最後』的 index，其他都回傳 null

剛剛上面的情況，當沒有找到值時會回傳『大於且最接近的 index』，但這種情況有時反而會更加難以判斷『到底存在還是不存在』，因此這邊打算將上面的例子改寫成如果沒有就回傳 `null`，其他則一樣取『最前』、『最後』的 index。

#### While 迴圈：

```javascript=
/* 此範例為：取最前面 index */
const arr = [1,3,6,6,6,10,12]

const binarySearch = (data ,target) =>{
    let left = 0
    let right = data.length - 1
    let matchIndex = null
    while(left <= right){
        const mid = Math.floor((left + right) / 2)
        // 如果有匹配到 則將 mid 記錄到 matchIndex
        if(data[mid] === target) matchIndex = mid
        if(data[mid] >= target){
            right = mid - 1

        }
        // 如果取『最後 index』只需將此改成 <=
        if(data[mid] < target){
            left = mid + 1
        }
    }
    // 最後回傳 matchIndex
    return matchIndex
}
binarySearch(arr,6) // 2
binarySearch(arr,7) // null
```

#### 遞迴版本：

```javascript=
/* 此範例為：取最前面 index */
const arr = [1,3,6,6,6,10,12]

function binarySearch(data, target, left = 0, right = data.length, matchIndex = null) {
    // matchIndex 用來記錄有與目標值匹配到的 index
    const midIndex = Math.floor((left + right) / 2);
    // 這邊不再回傳 left，改回傳 matchIndex
    // 如果都沒匹配到，自然會回傳預設值(null)
    if (left > right) return matchIndex;
    if (data[midIndex] >= target) {
        nextRight = midIndex - 1;
        // 注意這邊，
        // 如果 等於目標值 則將 matchIndex 改成現在的 midIndex 並往下繼續查詢
        // 如果 沒有匹配到 則用原本的 matchIndex 繼續查詢
        return binarySearch(
            data,
            target,
            left,
            nextRight,
            data[midIndex] === target ? midIndex : matchIndex
        );
    }
    // ps. 如果是要取 『最後 index』則將『等號』加在這。
    if (data[midIndex] < target) {
        nextLeft = midIndex + 1;
        // 如果是要取 『最後 index』則將 matchIndex 判斷與上面對調。
        return binarySearch(data, target, nextLeft, right, matchIndex);
    }
}
```

## 結語

最近剛好看到 `PJCHENder` 大大的演算法文章，所以想說也來了解一下這些內容並且自己實作一下，再看了許多大大的文章後發現除了最基本的二分搜尋法之外，還有很多其他的變體，像是上面『多個目標值』的情況就是一個很好的例子，而『二分搜尋法』也是『時間複雜度 O(log n)』最常見的例子，因此花了一些時間好好的實作了一遍並詳細寫了一些註解，之後也會再研究其他演算法還請大家多指教。

#### 以上就是這次【 演算法 - 二分搜尋法 (Binary Search) 】的全部內容，希望對有想了解二分搜尋法 (Binary Search)的人能有一些幫助，以上如有任何錯誤或冒犯的地方還請各位多多指教，謝謝您的觀看。

#### GitHub: [https://github.com/librarylai/algorithm/tree/main/pages/binarySearch](https://github.com/librarylai/algorithm/tree/main/pages/binarySearch)

## Reference

1. [[演算法] Binary Search：在陣列中尋找特定元素 - PJCHENder](https://pjchender.dev/dsa/algo-binary-search)
2. [二分搜尋法（Binary Search）完整教學（一） - Arthur Lin](https://medium.com/appworks-school/binary-search-%E9%82%A3%E4%BA%9B%E8%97%8F%E5%9C%A8%E7%B4%B0%E7%AF%80%E8%A3%A1%E7%9A%84%E9%AD%94%E9%AC%BC-%E4%B8%80-%E5%9F%BA%E7%A4%8E%E4%BB%8B%E7%B4%B9-dd2cd804aee1)
3. [二分搜尋法（Binary Search）完整教學（三）- Arthur Lin](https://medium.com/@lindingchi/binary-search-%E9%82%A3%E4%BA%9B%E8%97%8F%E5%9C%A8%E7%B4%B0%E7%AF%80%E8%A3%A1%E7%9A%84%E9%AD%94%E9%AC%BC-%E4%B8%89-%E5%BE%88%E5%A4%9A%E7%9B%B8%E5%90%8C%E7%9A%84%E6%83%85%E5%A2%83-c2215d1b9dc7)
4. [初學者學演算法｜從時間複雜度認識常見演算法 - Cheng-Wei Hu | 胡程維](https://medium.com/appworks-school/%E5%88%9D%E5%AD%B8%E8%80%85%E5%AD%B8%E6%BC%94%E7%AE%97%E6%B3%95-%E5%BE%9E%E6%99%82%E9%96%93%E8%A4%87%E9%9B%9C%E5%BA%A6%E8%AA%8D%E8%AD%98%E5%B8%B8%E8%A6%8B%E6%BC%94%E7%AE%97%E6%B3%95-%E4%B8%80-b46fece65ba5)
