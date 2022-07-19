# 【筆記】演算法 - 費波那契數列 (Fibonacci)

###### tags: `筆記文章`

費波那契數列(Fibonacci)其實就是我們小時候常聽到的『費氏數列』，如果印象中沒記錯的話，大概是在國中學習『等差級數』、『等比級數』的時候會聽到這個名詞，讓我們先來複習一下『費氏數列』究竟是什麼意思？

## 費氏數列

『費氏數列』是指一串數字且每一個數字都是前兩個數字的『和』，除了最前的兩個數字之外，最前面的兩個數字基本上會是以 `0,1` 或是 `1,1` 開始，當然也可以是任何數字，只要數列中後續的數字以這兩個數字為基礎相加而成的話，它就是一個『費氏數列』。

### 費氏數列源由

費氏數列是由一位義大利數學家[費波那契(Leonardo Fibonacci)](https://zh.m.wikipedia.org/zh-tw/%E6%96%90%E6%B3%A2%E9%82%A3%E5%A5%91)，又稱比薩的列奧納多，他描述兔子生長的數目時數目時用上了這個數列：『**假設每隻小兔子要花一個月長成一隻大兔子，而每一對大兔子在牠們永恆的生命中每個月都會生出一對新的小兔子**』，這邊舉例一對小兔子為`r`，一對大兔子為`R` 來模擬一下整個流程。

> - 第一個月有一對小兔子(r\)
> - 第二個月，小兔子長大了(R\)
> - 第三個月，大兔子生出小兔子(R->Rr) => (Rr)
> - 第四個月，大兔子生出小兔子，小兔子長大了(R->Rr , r->R) => (RrR)
> - 第五個月，大兔子生出小兔子，小兔子長大了(R->Rr, r->R, R->Rr) => (RrRRr)
>
> ![](https://i.imgur.com/vISkQxp.png)

#### 如果想更詳細理解可參考：[費氏數列的魔術 - 康軒教師網](https://www.945enet.com.tw/main/ha/teach_d.asp?pno=175&dt=A3)

### 程式碼實作

在開始實作之前，先再來複習一下費氏數列的公式：

![](https://i.imgur.com/XoJGNdG.png)

當前的費氏數`(n)`為前兩位費氏數`(n-1)、(n-2)`的和。

**_提醒：以下的 `F(n)` 裡面 n 所代表的『不是』`index`，而是口語的第一個、第二個，如果要當作是 index 就麻煩思考方面要記得 `-1`_**

#### 遞迴方式：

我們把上面的 `F` 當作是一個 function 來看的話那就會像是：

> 以 F(4) 為例且初始兩位為 F(2)、F(1)
>
> => F(4) = F(3) + F(2)
> => F(4) = F(2) + F(1) + F(2)

```javascript=
function Fibonacci(originArr, n) {
    if (n <= 2) {
        return originArr[n - 1];
    } else {
        return Fibonacci(originArr, n - 1) + Fibonacci(originArr, n - 2);
    }
}

const f4 = Fibonacci([1, 2], 4);

```

#### for 迴圈方式：

我們把上面的 `F` 當作是一個 `陣列(Array)` 來看的話那就會像是：
`F(4)` 等於是 `F(3) + F(2)`，也就是等於是陣列內『第三位(index:2)』的值加上陣列的『第二位(index:1)』的值。

而要達成這樣的方式，就等於是我們要先一個一個算出陣列內每一位的數值，假如今天是 `F(9)`，那就得從`F(1)`算到`F(8)`最後把`F(8)` 加上`F(7)` 得到 `F(9)` 的數值，這邊就跟上面『遞迴方式』會有一點思考上的不同。

```javascript=
function Fibonacci(originArr, n) {
    const arr = [...originArr]
    for(let i = 2 ; i < n ; i++){
        arr[i] = arr[i - 1] + arr[i - 2];
    }
    return arr[n];
}

const f4 = Fibonacci([1, 2], 4);

```

#### 『遞迴』與『迴圈』思考方式差異：

> **遞迴**：比較偏向說，將目標數(n)解析出到底有幾個 F(1) 與 F(2)，然後最後將全部的 F(1) 與 F(2) 通通加總起來，以 `F(4)` 為例，就是有『 1 個 F(1) 』與『 2 個 F(2) 』。
>
> **迴圈**：比較偏向說，抓取陣列前兩位相加起來，因此取得目標數(n)之前，要先將前面的內容都限計算過一片，以 `F(9)` 為例，就會將`F(1)`到`F(8)` 通通算出來，丟到陣列裡面。

## 結語

我們可以透過『遞迴』或『迴圈』的方式來達成『費波那契數列』，而這兩者所耗費的時間複雜度則不太一樣，像是『遞迴』的方式因為每一層都會分成『兩個』(`f(4) = f(3),f(2)`)且每一項都還會再繼續往下『分裂』到底部，因此時間複雜度會是『**O(2^n)**』，而『迴圈』的方式則是直接將陣列前兩位加總算出下一位，因此時間複雜度為『**O(n)**』，由此可知在『費波那契數列』的實作上使用『迴圈』的方式會比較節省時間。

#### 以上就是這次【 演算法 - 費波那契數列 (Fibonacci) 】的全部內容，之後預計也會再研究其他演算法，以上如有任何錯誤或冒犯的地方還請各位多多指教，謝謝您的觀看。

#### GitHub: [https://github.com/librarylai/algorithm/tree/main/pages/fibonacci](https://github.com/librarylai/algorithm/tree/main/pages/fibonacci)

## Reference

1. [初學者學演算法｜從費氏數列認識何謂遞迴 - Cheng-Wei Hu | 胡程維](https://medium.com/appworks-school/%E5%88%9D%E5%AD%B8%E8%80%85%E5%AD%B8%E6%BC%94%E7%AE%97%E6%B3%95-%E5%BE%9E%E8%B2%BB%E6%B0%8F%E6%95%B8%E5%88%97%E8%AA%8D%E8%AD%98%E4%BD%95%E8%AC%82%E9%81%9E%E8%BF%B4-dea15d2808a3)
2. [費氏數列的魔術 - 康軒教師網](https://www.945enet.com.tw/main/ha/teach_d.asp?pno=175&dt=A3)
3. [費式數列 - openhome.cc](https://openhome.cc/zh-tw/algorithm/basics/fibonacci/)
4. [【 JavaScript 】JavaScript 費氏數列的 3 種解法 - Jimmy](https://jimmyswebnote.com/javascript-fibonacci-numbers/)
