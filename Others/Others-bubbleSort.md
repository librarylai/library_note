# 【筆記】演算法 - 氣泡排序法 (Bubble Sort)

###### tags: `筆記文章`

## 氣泡排序法 (Bubble Sort) 介紹

#### 先來看看維基百科的介紹：

> 泡沫排序（英語：Bubble Sort）又稱為泡式排序，是一種簡單的排序演算法。它重複地走訪過要排序的數列，一次比較兩個元素，如果它們的順序錯誤就把它們交換過來。走訪數列的工作是重複地進行直到沒有再需要交換，也就是說該數列已經排序完成。

#### 簡單來說：

> 氣泡排序法是從陣列的第一筆資料開始，依序比較相鄰兩筆資料的大小，並且將比較大的一方放到右邊，因此當第一輪結束時，最右邊的數必定是陣列裡最大的數，然後再繼續進行第二輪搜尋，以此類推。

### 氣泡排序法執行流程：

1. 陣列內兩兩相鄰的元素(內容)互相比較大小。(ex. `[0,1]` 比較 -> `[1,2]` 比較...以此類推。)
2. 如果『第一個』比『第二個』大，則交換彼此的位置。(ex. `[3,1]` 互相比較：3 比 1 大 因此兩兩交換位置 => `[1,3]`)
3. 當比較完第一輪後會發現，『最大的數』會跑到陣列的最後一位，因此可以排除最後一位。
4. 然後重複上述動作，直到排序完畢。（ex. 如果陣列為 `[3,7,2,5]` 那總共要重複 3 次上述的所有動作，才能確保排序完畢。）

![](https://www.resultswebdev.com/wp-content/themes/results-website-design/uploads/bubble-sort-animation2.gif)

(圖片來源: [Bubble Sort Algorithm - resultswebdev](https://www.resultswebdev.com/bubble-sort-algorithm/))

## 氣泡排序法實作

在開始實作程式碼之前，我們先把上圖例子中的『第一輪』、『第二輪』排序的步驟列出來，放便等等與程式碼 log 內容比較。

```javascript=
// 初始陣列為： [45,30,10,40,20,35,15,25,50]
// 第一輪排序：
    // 比較 45 與 30，發現 45 > 30，因此交換
    [30,45,10,40,20,35,15,25,50]
    // 比較 45 與 10，發現 45 > 10，因此交換
    [30,10,45,40,20,35,15,25,50]
    // 比較 45 與 40，發現 45 > 40，因此交換
    [30,10,40,45,20,35,15,25,50]
    // 比較 45 與 20，發現 45 > 20，因此交換
    [30,10,40,20,45,35,15,25,50]
    // 比較 45 與 35，發現 45 > 35，因此交換
    [30,10,40,20,35,45,15,25,50]
    // 比較 45 與 15，發現 45 > 15，因此交換
    [30,10,40,20,35,15,45,25,50]
    // 比較 45 與 25，發現 45 > 25，因此交換
    [30,10,40,20,35,15,25,45,50]
    // 比較 45 與 50，發現 45 < 50，因此『不』交換
    [30,10,40,20,35,15,25,45,50]

// 第二輪排序： 排除最後一位
    // 比較 30 與 10，發現 30 > 10，因此交換
    [10,30,40,20,35,15,25,45,50]
    // 比較 30 與 10，發現 30 < 40，因此『不』交換
    [10,30,40,20,35,15,25,45,50]
    // 比較 40 與 20，發現 40 > 20，因此交換
    [10,30,20,40,35,15,25,45,50]
    // 比較 40 與 35，發現 40 > 35，因此交換
    [10,30,20,35,40,15,25,45,50]
    // 比較 40 與 15，發現 40 > 15，因此交換
    [10,30,20,35,15,40,25,45,50]
    // 比較 40 與 25，發現 40 > 25，因此交換
    [10,30,20,35,15,25,40,45,50]
    // 比較 40 與 45，發現 40 < 45，因此『不』交換
    [10,30,20,35,15,25,40,45,50] // 提醒： 50 已被排除

// .....以此類推.....
// 第七輪排序：
[10,15,20,25,30,35,40,45,50]

```

### 程式碼實作

#### For 迴圈版本：

```javascript=
const MOCK_DATA = [45, 30, 10, 40, 20, 35, 15, 25, 50];

function bubbleSort(data){
    // 每一輪要執行的最後一位 index
    let lastIndex = data.length - 1
    // 總共要執行 n - 1 輪
    // 以這邊為例：總共要執行 i=0 ..... i=7
    for (let i = 0; i < data.length - 1; i++) {
        console.log(`第${i+1}輪，執行：${lastIndex}次`)
        lastIndex --
        // 兩兩比對，比對到 lastIndex 當輪的最後一位
        // ex.以第一輪來講：lastIndex = 7 則代表需要比到 index7 與 index8 。
        for (let j = 0; j <= lastIndex; j++) {
            if (data[j] > data[j + 1]) {
                // 陣列內元素位置交換
                [data[j], data[j + 1]] = [data[j + 1], data[j]];
            }
            console.log('data',data)
        }
    }
    return data
}

bubbleSort(MOCK_DATA)
```

#### While 迴圈版本：

```javascript=
const MOCK_DATA = [45, 30, 10, 40, 20, 35, 15, 25, 50];

function bubbleSort(data){
    // 每一輪要執行的最後一位 index
    let lastIndex = data.length - 1;
    // 如果 lastIndex > 0 代表還有需要匹配的元素
    // ex. lastIndex = 1 代表至少還有 [0,1] 這兩個需要比對
    while(lastIndex > 0){
        console.log(`執行：${lastIndex}次`)
        lastIndex --
        // 兩兩比對，比對到 lastIndex 當輪的最後一位
        // ex.以第一輪來講：lastIndex = 7 則代表需要比到 index7 與 index8 。
        for (let j = 0; j <= lastIndex; j++) {
            if (data[j] > data[j + 1]) {
                // 陣列內元素位置交換
                [data[j], data[j + 1]] = [data[j + 1], data[j]];
            }
            console.log('data',data)
        }
    }

    return data
}

bubbleSort(MOCK_DATA)
```

### 成果圖片：

![](https://i.imgur.com/NS3Ic4I.png)

## 結語

最近剛好看到 `PJCHENder` 大大的演算法文章，所以想說也來了解一下這些內容並且自己實作一下，從上面的程式碼範例中可以發現到在實作『氣泡排序法(Bubble Sort)』時基本上都需要使用『兩個』迴圈才能達成這個氣泡排序法，因此可以推論出『氣泡排序法(Bubble Sort)』的時間複雜度為『O(n²)』。

#### 以上就是這次【 演算法 - 二分搜尋法 (Binary Search) 】的全部內容，之後預計也會再研究其他演算法，以上如有任何錯誤或冒犯的地方還請各位多多指教，謝謝您的觀看。

#### GitHub: [https://github.com/librarylai/algorithm/tree/main/pages/bubbleSort](https://github.com/librarylai/algorithm/tree/main/pages/bubbleSort)

## Reference

1. [[演算法] Bubble Sort：利用元素兩兩交換達到排序 - PJCHENder](https://pjchender.dev/dsa/algo-bubble-sort)
2. [【Day21】[演算法]-排序 Sort & 氣泡排序法 Bubble Sort - Frank](https://ithelp.ithome.com.tw/articles/10276184)
3. [Bubble Sort Algorithm - resultswebdev](https://www.resultswebdev.com/bubble-sort-algorithm/)
