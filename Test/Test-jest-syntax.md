# 【筆記】從 Jest 帶你進入前端測試世界 - 語法篇

###### tags: `筆記文章`

## Jest 套件介紹與安裝

### Jest 介紹

Jest 是一套由 Facebook 開源的 JavaScript 測試框架，源自於 Jasmine 延伸開發，它提供了許多 API 方便我們對 JavaScript 程式碼進行測試。Jest 也提供了 Coverage 測試報告的功能，這讓我們能夠直接知道程式碼中哪些地方沒被測試到。

### Jest 安裝

可以透過 Yarn 或是 NPM 來安裝 Jest

- 使用 **yarn** 安装 Jest︰

```javascript
mkdir jest
cd jest
yarn init
yarn add --dev jest
```

- 使用 **NPM** 安装 Jest︰

```javascript
npm install --save-dev jest
```

接著到 package.json 裡面的 scripts 加入以下設定

```javascript=
{
  "scripts": {
    "test": "jest"
  }
}
```

如果有用到 Babel 則需安裝所需的依賴套件(這邊以 Yarn 舉例)

```javascript=
yarn add --dev babel-jest @babel/core @babel/preset-env
```

新增 babel.config.js 檔案到你的專案下

```javascript=
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        targets: {
          node: 'current',
        },
      },
    ],
  ],
};
```

接下來我們來寫個簡單的測試檔，來確認我們是否安裝成功。

```javascript=
// 專案結構
|- src
|-- utils.js
|- test
|-- sum.test.js
|- package.json
|- babel.congig.js
```

首先創建一個 utils.js 的函式檔並在裡面寫入以下程式碼

```javascript=
// src/utils.js
export function sum(a, b) {
    return a + b;
}
```

接著創建一個 test 資料夾，並且新增一個 sum.test.js 測試檔寫入以下程式碼

```javascript=
// test/sum.test.js
import { sum } from "../src/util";

test("add 3,7 expect 10", () => {
    expect(sum(3, 7)).toBe(10);
});
```

最後在 command line 上執行 yarn test 如果安裝成功的話，就可以看到測試完成後的結果。

```javascript=
$ jest sum.test.js
 PASS  test/sum.test.js
  ✓ add 3,7 expect 10 (1 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        1.142 s
```

**補充：當我們下 yarn test 時，Jest 預設會去尋找專案底下 test 的資料夾，並執行 .test.js 的所有檔案，如果只要測試某一個檔案的話，則在 yarn test 後面加上檔案名稱即可。**

## Jest Common Matchers - 一般測試函式

Jest 有提供許多不同的 **matchers(匹配器)** 函式用來測試我們所攥寫的程式，而這邊將會依依介紹一些常用 matchers 與它的功能

### common matchers

1. **toBe()**
   相當於比較運算子『等號(==)』，使用 Object.is 來做精準相等的判斷，例如： 2+2 == 4 或者 'Hellow'+'World' == 'HellowWorld'。

```javascript=
test('two plus two is four', () => {
  expect(2 + 2).toBe(4); //pass, 2+2 = 4
  expect('Hellow'+'World').toBe('HellowWorld') //pass, HellowWorld
});
```

2. **toEqual()**
   用來做**物件**與**陣列**的匹配，透過遞迴的方式去匹配每一個 field

```javascript=
test('toEqual object', () => {
    const obj = {name: 'Wang'}
    obj.age = 20
    expect(obj).toEqual({ name: 'Wang' , age: 20 })//pass, 對整體物件匹配使用 toEqual
    expect(obj.name).toBe('Wang') //pass, 對物件內屬性的 value 可用 toBe
})
```

3. **toMatch()**
   運用正則表達式來匹配字串

```javascript=
test('use toMatch to test email address', () => {
    const str = 'testJest@gmail.com'
    const emailRegux = /^\w+((-\w+)|(\.\w+))*\@[A-Za-z0-9]+((\.|-)[A-Za-z0-9]+)*\.[A-Za-z]+$/
    expect(str).toMatch(emailRegux) // pass
})
```

4. **not**
   not 我們可以直接當成是『邏輯運算符』裡的驚嘆號(!)，它經常用來作為判斷不為空值的狀態。

```javascript=
test('two plus two is four', () => {
    expect(2 + 2).not.toBe(5); //pass 2+2 != 5
    expect(2 + 2).not.toBe('') //pass 用來判斷是否不為空
});
```

### Truthiness matchers

1. **toBeNull()** 匹配 null
2. **toBeUndefined()** 匹配 undefined
3. **toBeDefined()** 與 undefined 相反，匹配只要不是 undefined 的值
4. **toBeTruthy()** 匹配 true
5. **toBeFalsy()** 匹配 false

```javascript=
describe('Truthiness matchers test', () => {
    test('null and undefind', () => {
        let str; // undefined
        expect(str).toBeNull(); // failed
        expect(str).toBeUndefined(); // pass
        expect(null).toBeNull(); // pass
    })
    test('toBeDefined', () => {
        let str = 'iron man'; // Defined , type String
        expect(str).toBeDefined(); // pass
        expect(str).toBeUndefined(); // failed
    })
    test('toBeTruthy and toBeFalsy', () => {
        let truthy = true;
        let falsy = false;
        expect(truthy).toBeTruthy() // pass
        expect(falsy).toBeFalsy() // pass
    })
})
```

### Arrays and Iterables

1. **toContain()**
   用來檢查陣列中或可迭代對象是否包含某個值，可以想像成 Javascript 的 Array.prototype.includes

```javascript=
test('use toMatch to test email address', () => {
    const arr = ['apple', 'banana', 'cat', 'dog']
    expect(arr).toContain('apple')//pass
    expect(new Set(arr)).toContain('cat')//pass
})
```

2. **toContainEqual()**
   用來匹配『陣列裡面的物件』是否與期望值相同，將會使用遞迴的方式檢查每一個 filed 是否與期望值相同

```javascript=
describe('toContainEqual test', () => {
    let memberInfo = [{
            name: 'library',
            age: 25
        },
        {
            name: 'Wang',
            age: 23
        },
        {
            name: 'Chen',
            age: 21
        },
    ]
    test('check item in array', () => {
        const myInfo = {
            name: 'library',
            age: 25
        };
        expect(memberInfo).toContainEqual(myInfo); // pass
    });
});
```

### Numbers

1. **toBeGreaterThan()** 匹配『**大於**』某數值
2. **toBeGreaterThanOrEqual()** 匹配『**大於等於**』某數值
3. **toBeLessThan()** 匹配『**小於**』某數值
4. **toBeLessThanOrEqual()** 匹配『**小於等於**』某數值

```javascript=
describe('number matches', () => {
    test('toBeGreaterThan and toBeGreaterThanOrEqual', () => {
        const num = 2 + 3
        expect(num).toBeGreaterThan(3) // pass num > 3
        expect(num).toBeGreaterThanOrEqual(5) //pass num >= 5
    })
    test('toBeGreaterThan and toBeGreaterThanOrEqual', () => {
        const num = 2 + 3
        expect(num).toBeLessThan(6) // pass num < 6
        expect(num).toBeLessThanOrEqual(5) //pass num <= 5
    })
})
```

5. **toBeCloseTo()**
   toBeCloseTo 主要是為了匹配『**浮點數**』相等，在 Javascript 浮點數運算中其實是有陷阱存在的，例如 0.1 + 0.2 我們可能會很直覺的說出是 0.3 ，但在 Javascript 中其實並不等於 0.3 而是 0.30000000000000004。
   例如 1/3 = 0.333333333333 (無窮迴圈)
   而是 1/3 _ 3 = 0.333333333333 (無窮迴圈) _ 3
   就會產生 0.00000000000001 (無窮迴圈) 的誤差
   所以如果使用 toBe(0.3) 來匹配的話將會無法通過測試，我們需要使用 toBeCloseTo(0.3) 來匹配一個接近的數值。

```javascript=
test('toBeCloseTo test float', () => {
    let floatNumber = 0.1 + 0.2
    expect(floatNumber).toBe(0.3) //failed
    expect(floatNumber).toBeCloseTo(0.3) //pass
})
```

## Jest Testing Asynchronous - 非同步測試

在 JavaScript 中執行異步代碼是很常見的。當你以異步方式執行代碼時，Jest 需要知道當前的測試代碼是否已完成，是否可以轉移到另一個測試。
簡單來說：Jest 在異步測試中，需要先等待異步程式執行完成，然後才去執行測試項目，因此這邊會教大家幾種 Jest 處理異步的方法。

### callback

Callback 是最常見的異步模式
假設我們有個獲取資料的函式 fetchData(callback)，函式裡面使用 setTimeout 來模擬呼叫 API 的延遲，獲取資料後透過 callback function 來將資料傳回。

```javascript=
// 獲取資料函式
export function fetchData(callback) {
    let data = '我是資料.........'
    setTimeout(() => {
        callback(data)
    }, 3000)
}
```

有了獲取資料的函式後，接著建立一個 callback.test.js 的測試檔，並引入 fetchData 函式來測試非同步看看。
**一般我們直觀的寫法可能如下：**

```javascript=
import { fetchData } from '../src/fetchData'
describe('Asynchronous callback test', () => {
    const callback = (data) => {
        console.log(data)
        expect(data).toBe('我是資料.........')
    }
    test('callback test', () => {
        fetchData(callback)
    })
})
```

**但測試結果卻是.......**

![](https://i.imgur.com/Q9ykwbM.png)

雖然看起來好像通過了測試，但其實根本沒有測試到 expect(data).toBe('我是資料.........') 這段碼程式。這是因為在默認情況下，Jest 一旦執行到尾端就會當作完成測試，因此測試就會在還沒調用回調函數前結束了。
**從下方圖片中可以看到 callback(data) 這句是沒有被執行到的**
![](https://i.imgur.com/hy5PVKT.png)

#### 解決辦法：

我們可以透過傳入 **done** 這個參數來告訴 Jest 要等待回調函數執行結束後，再結束測試。

```javascript=
describe('Asynchronous callback test', () => {
    test('callback test', (done) => { // 注意這裡！！！
        const callback = (data) => {
            expect(data).toBe('我是資料.........')
            done() // 注意這裡！！！
        }
        fetchData(callback)
    })
})
```

加入 done()之後再跑一次測試看看，就會看到 **覆蓋率 100% 的完美測試** 結果了。🎉🎉🎉
![](https://i.imgur.com/39MqmnB.png)

#### 注意事項

1. 如果加入了 **done 參數**但卻未使用它，將會出現測試失敗(failed)的情況
   ![](https://i.imgur.com/lbS0Puz.png)

2. 如果在 expect 時測試失敗，則會拋出錯誤並不會往下執行 done()，因此如果想知道為何失敗，則可以使用 try/catch 來包住 expect 函式與將 error 傳遞給 done(error)。

### promise

如果在非同步方面是使用 Promise 來處理的話，對於測試方面就比較簡單一些，可以直接使用 .then() 來處理 resolve 的結果。

```javascript=
export function promiseFetchData() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve('testData')
        }, 3000)
    })
}
```

```javascript=
import { promiseFetchData } from '../src/promise'
describe('Asynchronous promise test', () => {
    test('promise test', () => {
        return promiseFetchData().then((data) => { //注意這段！！！
            expect(data).toBe('testData')
        })
    })
})
```

**要特別注意的是：在對 Promise 進行測試時，要記得加上 return ，如果忘記的話將會導致 Promise 在進入 .then()、resolve 之前就已被視為測試完成。**

#### .resolves / .rejects

Jest 也有提供用來針對 promise 的測試匹配器，可以在 expect 中使用 .resolves/.rejects 屬性。
以 .resolves 為例，Jest 將會等待 promise resolve 後並匹配測試內容是否與期望值相同。

```javascript=
describe('Asynchronous promise test use resolves', () => {
    test('promise test', () => {
        return expect(promiseFetchData()).resolves.toBe('testData')
        // 記得要有 return
    })
})
```

### async/await

我們也可以使用 ES7 的 async/await 來針對 promise 做測試，可以在 test() 的 callback function 中加上 async 關鍵字，並在 promise 函式前加上 await 關鍵字。
**如果對 async/await 還不懂的讀者，可以看 MDN 的文件：** [MDN async function](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Statements/async_function)

```javascript=
describe('Asynchronous async/await test', () => {
    test('promise test', async () => { // 注意這段！！
        const data = await promiseFetchData() // 注意這段！！
        expect(data).toBe('testData')
    })
})
```

**async/await 也可以結合 .resolves / .rejects 匹配器一起使用。**

```javascript=
describe('Asynchronous async/await test use resolves/rejects', () => {
    test('promise test', async () => {
        await expect(promiseFetchData()).resolves.toBe('testData')
    })
})
```

## Jest LifyCircle - Setup and Teardown

在開始講解 Jest 的**作用域**之前，我們要先來了解一下 describe 是什麼，因為 describe 會與作用域的 beforeAll,beforeEach,afterAll,afterEach 等函式有相關，所以就讓我們先從 describe 開始講起吧！

### describe

describe 從英文上翻譯為『描述、形容』，所以我們大概可以推測出，它是負責描述這個區塊內所有測試項目的**主題**，簡單來說可以想像成將測試項目**分區塊、分類**...等。

```javascript=
 $ jest
 PASS  test/numberMatchers.test.js
 PASS  test/toMatch.test.js
 PASS  test/toEqual.test.js
 PASS  test/toContain.test.js
 FAIL  test/toBeCloseTo.test.js
```

還記得當我們下 **yarn** **test** 時，會執行所有檔案的測試項目，因此全部的測試內容都會被條列顯示，就像上方這樣。
如果沒有使用 describe 來區分這兩個測試，我們將無法知道這些項目是來自哪個分類，只能回去從程式碼中來瞭解，因此 describe 在這時就顯得格外重要。

```javascript=
$ jest --coverage describe.test.js
 PASS  test/describe.test.js
  元件測試 // describe
    ✓ Testing  (1 ms) // test
  函數測試 // describe
    ✓ Testing  (1 ms) // test
```

**上方則是將測試項目用 describe 分類**
透過 describe 將測試項目分類，測試者就可以一目瞭然知道我們正在做哪個區塊的測試，大大減少了回去查程式碼的時間。

### 作用域 Scoping

每個 .test.js 的測試檔就像是一個 JavaScript 的檔案，每個 Function 都有它的作用域，而整個 test 檔可以當作是一個全域的 Function。
在測試中，執行範圍（全域、區域）除了我們熟知的變數影響外，還會影響以下 Jest 函式的執行順序。

1. beforeAll ：所在區域內會第一個執行。
2. beforeEach ：每一次的測試執行前會先執行。
3. afterAll ：所在區域內會最後一個執行。
4. afterEach ：每一次的測試執行完後會馬上執行。

**直接透過下面這個例子來瞭解執行順序**

```javascript=
beforeAll(() => console.log('全域 - beforeAll：我一定是第一個執行'));
afterAll(() => console.log('全域 - afterAll：我一定最後一個執行'));
beforeEach(() => console.log('全域 - beforeEach ： 我每次都會執行，且會比區域的 beforeEach 快'));
afterEach(() => console.log('全域 - afterEach ： 我每次都會執行，且會比區域的 afterEach 慢'));
test('', () => console.log('全域 - test'));
describe('Scoped / Nested block', () => {
    beforeAll(() => console.log('區域 - beforeAll'));
    afterAll(() => console.log('區域 - afterAll'));
    beforeEach(() => console.log('區域 - beforeEach'));
    afterEach(() => console.log('區域 - afterEach'));
    test('', () => console.log('區域 - test'));
});
/*
 * console.log 全域 - beforeAll：我一定是第一個執行
 * console.log 全域 - beforeEach ： 我每次都會執行，且會比區域的 beforeEach 快
 * console.log 全域 - test
 * console.log 全域 - afterEach ： 我每次都會執行，且會比區域的 afterEach 慢
 * console.log 區域 - beforeAll
 * console.log 全域 - beforeEach ： 我每次都會執行，且會比區域的 beforeEach 快
 * console.log 區域 - beforeEach
 * console.log 區域 - test
 * console.log 區域 - afterEach
 * console.log 全域 - afterEach ： 我每次都會執行，且會比區域的 afterEach 慢
 * console.log 區域 - afterAll
 * console.log 全域 - afterAll：我一定最後一個執
 * */
```

從上面我們可以知道：

1. 全域 beforeAll 一定第一個執行
2. 全域 afterAll 一定最後一個執行
3. beforeAll 與 afterAll 在作用域中只會執行一次
4. beforeEach 與 afterEach 會包住 test 分別在『測試前』與『測試後』執行各一次

這些函式在測試中，經常會使用 beforeEach(每次) 或 beforeAll(一次) 在測試之前呼叫方法，像是加入**初始值**或**呼叫 API**等，並在測試結束後呼叫 afterEach(每次) 或 afterAll(一次) 將資料清空還原等。

#### Scoping 常見問題補充：

> If you have a test that often fails when it's run as part of a larger suite, but doesn't fail when you run it alone, it's a good bet that something from a different test is interfering with this one. You can often fix this by clearing some shared state with beforeEach. If you're not sure whether some shared state is being modified, you can also try a beforeEach that logs data.
>
> 如果你有一個測試，它是在大項測試中的一小部分，且經常執行失敗，但當它單獨運行時卻是正常運作時，通常可以通過修改 beforeEach 來清除一些共享狀態來解決這種問題。
> 例如：我們在一個 describe 中有許多的測試項目 test 且都共用同一個 data 進行測試，這時就可以使用 beforeEach 在每一次要開始測試前將 data 設回初始值。

```javascript=
describe('describe block', () => {
    let initData = {
        name:'Library',
        age: 25
    }
    let data = initData
    beforeEach(() => {
        console.log('每一次都將data設回初始值')
        data = initData
    });
    // 修改了 data 的值
    test('update data name ', () =>{
        data.name = 'Wang'
        expect(data.name).toBe('Wang') // pass
    });
    // 上面測試完成後接續測試此項目，因測試前會先執行 beforeEach 所以 data 依舊為初始值
    test('data object ', () =>{
        data.name = 'Wang'
        expect(data).toEqual({name:'Library',age: 25}) // pass
    });
});
```

## Jest Mock Function - 模擬函式

**Mock** 是在前端測試中一個很重要的部分，在前端測試中除了測試『DOM 元素』以外就是測試『函式 Function』。相信大家應該都有過在一個函式(SUT)上面呼叫其他函式(DOC)的經驗，它(DOC)可能是幫忙做一些『數學運算』或是『呼叫 API』取值...等，而在測試中會希望測試『SUT 函式』且不希望被函式內的『其他 DOC』所干擾，這時就可以透過 Mock 函式可以讓我們更集中在測試某個函數上面，接下來將透過一些例子教大家如何使用 Ｍ ock 函式。

#### 名詞解釋

1. SUT : System Under Test。主要測試函式。
2. DOC : Depended On Component。SUT 所依賴的東西（函式、物件）。

### Mock functions

首先假設有個收銀系統功能如下：

1. 計算項目總額功能
2. 折扣優惠功能

```javascript=
// 計算購物總金額
export function calculateShopTotal(items, discountFunction) {
    return items.reduce((acl, cur) => {
        let itemTotal = (cur.price * cur.count)
        if (discountFunction(cur.name)) {
            itemTotal = itemTotal * 0.8 // 如果是打折商品，就打8折
        }
        return acl + itemTotal
    }, 0)
}
// 判斷是否為打折品項
function checkIncludeDiscountItemList(name) {
    let discountList = ['衣服', '鞋子']
    return discountList.includes(name)
}
```

先寫段測試來測看看我們寫的是否正確，程式碼如下

```javascript=
import { calculateShopTotal,checkIncludeDiscountItemList } from '../src/calculateShop'
let shopData = [{
        name: '衣服',
        count: 3,
        price: 20
    },
    {
        name: '褲子',
        count: 6,
        price: 23
    },
    {
        name: '鞋子',
        count: 2,
        price: 60
    },
]
describe('Mock functions', () => {
    test('price total', () => {
        expect(calculateShopTotal(shopData, checkIncludeDiscountItemList)).toBe(282)
        // test pass
    })
})
```

還記得我們一開始所説的**希望測試『SUT 函式』且不希望被函式內的『其他 DOC』所干擾**』，所以這邊我們可以透過 jest.fn() 來創建一個 mock function。

#### jest.fn()

先來介紹一下 Jest 在 Mock 方面提供了一些屬性讓我們能更方便的測試。

1. mockReturnValueOnce / mockReturnValue 函式

- mockReturnValueOnce 代表**下一次** return 的值
- mockReturnValue 代表**每一次** retrun 的值

所以在使用上會像下面這樣：

```javascript=
const mockFn = jest.fn()
mockFn.mockReturnValueOnce(1)
mockFn.mockReturnValueOnce(2)
mockFn.mockReturnValue(true)
/*
 * 呼叫
 * 第一次：1
 * 第二次：2
 * 第三次：true
 * 第四次：true
 * .....
 * 第n次：true
```

3. .mock 属性
   所有的 mock 函數都有這個特殊的 .mock 屬性，它可以紀錄調用次數、參數、返回值...等狀態。

```javascript=
const mockFn = jest.fn()
expect(mockFn.mock.calls.length).toBe(1); // 紀錄被呼叫幾次
mockFn.mock.calls[0][0] // 紀錄第一次被呼叫時的 第一個參數
mockFn.mock.calls[0][2] // 紀錄第一次被呼叫時的 第二個參數
mockFn.mock.results[0].value // 紀錄第一次的回傳值
mockFn.mock.results[1].value // 紀錄第二次的回傳值
```

最後回到收銀系統這個例子，這邊改用 mock 函式來模擬 DOC 函式邏輯的部分，這樣如果測試結果有誤的話，就代表 SUT 函式的邏輯是有錯誤的，這也呼應了『單元測試』所代表的意義（以程式碼的最小單位進行測試）。

```javascript=
describe('Mock functions', () => {
    const mockFn = jest.fn()
    mockFn.mockReturnValueOnce(true)
    mockFn.mockReturnValueOnce(false)
    mockFn.mockReturnValue(true)
    test('price total', () => {
        expect(calculateShopTotal(shopData, mockFn)).toBe(282)
    })
    test('mock called length', () => {
        expect(mockFn.mock.calls.length).toBe(3)
    })
    test('mock first arg of first call', () => {
        expect(mockFn.mock.calls.[0][0]).toBe('衣服')
    })
     test('mock result of second call ', () => {
        expect(mockFn.mock.results[1].value).toBe(false)
    })
})
```

![](https://i.imgur.com/3OGNtHK.png)

### Mock Modules

> 如果說 jest.fn 能夠作為一個 Function 的替身，那麼 jest.mock 就是能模擬整個模組的 Mock！ – 神Ｑ超人

Jest.mock 可以讓我們用來模擬整個模組，它常被用在模擬第三方套件，但這邊比較像是模擬了整個第三方套件內部函式的名稱，但『**沒有模擬業務邏輯**』。
舉例來說：我們有個獲取資料的函式(fetchAPI)，裡面使用了 axios 這個第三方套件來做呼叫 API 的動作，但為了不要真的發送 API 且又不能修改 axios 的情況下，就可以用 jest.mock(axios) 來直接模擬函式的行為。

```javascript=
import  axios  from 'axios'
export function fetchAPI() {
    return axios.get('shopList').then((resp) => {
        return resp.data
    })
}
```

使用 jest.mock 模擬 axios 後，axios.get 就變成了 mock function 如同 jest.fn() 一樣，而我們就可以透過 mockResolvedValue 來返回假的資料。

```javascript=
import { fetchAPI } from '../src/axiosFetch'
import axios from 'axios';
// 模擬整個 axios 模組
jest.mock('axios')
describe('mock modules test', () => {
    test('axios test', () => {
        let result = 'mockAxiosData'
        axios.get.mockResolvedValue(result)
        // mockResolvedValue 相等於 axios.get.mockImplementation(() => Promise.resolve(resp))
        return expect(fetchAPI()).resolves.toBe('mockAxiosData')
    })
})
```

這麼一來就不會真的執行到 axios.get 而是透過 Mock 回傳設定好的結果而已。

### jest.spyOn

從上面的 jest.mock 我們可以知道它是模擬一個 Module 的『空殼』，
而這邊所介紹的 **jest.spyOn** 則是模擬一個 Module 內的 Function 且會完整重現 DOC 的邏輯。

#### jest.spyOn 用法

在使用 jest.spyOn 之前先來了解一下它的用法。

> jest.spyOn(object, methodName)
> 第一個參數為：『物件』、『模組』，將要模擬的函式包進『**函式或模組**』中
> 第二個參數為： 要模擬的函式名稱

#### 舉例：計算總金額函式內直接使用折扣函式，不當作參數傳入。

來修改一下上面 jest.fn 的例子，『**計算總金額函式**』，這次將不傳入 discountFunction ，而是直接把折扣函式包在計算總金額的含式內。

```javascript=
// 計算總金額版本二
export function calculateShopTotalUseMock(items) {
    return items.reduce((acl, cur) => {
        let itemTotal = (cur.price * cur.count)
        if (checkIncludeDiscountItemList(cur.name)) {
           // 注意 checkIncludeDiscountItemList 函式，它直接包在 calculateShopTotalUseMock 函式中被呼叫
           itemTotal = itemTotal * 0.8
        }
        return acl + itemTotal
    }, 0)
}
```

接著將 checkIncludeDiscountItemList 函式包裝成一個 Module，並使用 import 將函式引入。

```javascript=
import { checkIncludeDiscountItemList } from './checkIncludeDiscountItemList'
```

現在我們就可以透過使用 spyOn 來模擬折扣函式( inner function )的返回值，來達到想要的測試結果。

```javascript=
import * as checkIncludeDiscountItemListModule from '../src/checkIncludeDiscountItemList';
describe('use jest.mock ', () => {
    const checkIncludeDiscountItemListMock = jest.spyOn(checkIncludeDiscountItemListModule, 'checkIncludeDiscountItemList')
    checkIncludeDiscountItemListMock.mockReturnValue(false)
    test('price total', () => {
        expect(calculateShopTotalUseMock(shopData)).toBe(318)
    })
})
```

## 範例練習

```javascript=
/**
 * 依據傳入的物件和路徑傳回找到的結果
 * 若傳入路徑中包含array，且路徑後一位並非指定array index，則會遞迴執行每一元素
 * 若傳入路徑中包含array，且路徑後一位為array index(e.g. '0', 2 , 3 etc..)
 *
 * @export
 * @param {*} obj 物件
 * @param {*} path 路徑
 * @returns array
 */
export function getObjectPropertyValueByKey(obj, path) {
    let result = []
    let forEachPath

    function takeDeep(currentObject, currentPath) {
        currentPath = [...currentPath]
        let cur = currentPath[0]
        if (Array.isArray(currentObject)) {
            // 設定取哪一筆時，ex. [languages, 0, name] 當走到 0 時會進入
            if (+cur >= 0) {
                currentPath.shift()
                return takeDeep(currentObject[cur], currentPath)
            } else {
                forEachPath = currentPath
                currentObject.forEach((item) => takeDeep(item, forEachPath))
            }
        } else {
            if (currentObject) {
                if (typeof currentObject[cur] === 'object') {
                    currentPath.shift()
                    return takeDeep(currentObject[cur], currentPath)
                } else {
                    result.push(currentObject[cur])
                }
            } else {
                return result
            }
        }
    }
    takeDeep(obj, path)
    return result.join(' , ')
}
```

```javascript=
describe('getObjectPropertyValueByKey test', () => {
    let testObj = {
        languages: [{
                name: '大家好',
                lang: 'tw'
            },
            {
                name: 'Hellow everyone',
                lang: 'en'
            },
        ],
        contract: {
            contractId: '9987123',
            contractInfo: {
                languages: [{
                        name: '這是一份超級龐大合約',
                        lang: 'tw'
                    },
                    {
                        name: 'this is a big contract',
                        lang: 'en'
                    }
                ]
            }
        }
    }
    // 取得物件中某一個值
    test('test get one value', () => {
        return expect(getObjectPropertyValueByKey(testObj, ['languages', '0', 'name'])).toBe('大家好')
    })
    //取得物件中某一陣列內的所有值,會 return 字串
    test('test get list value', () => {
        return expect(getObjectPropertyValueByKey(testObj, ['languages', 'name'])).toBe('大家好 , Hellow everyone')
    })
    // 當輸入為 null undefinede 空時,直接return傳入值
    test('test first argument is null or undefined', () => {
        return expect(getObjectPropertyValueByKey(null, ['languages', 'name'])).toBeFalsy()
    })
})
```

## Reference

1. [jestjs.io](https://jestjs.io/docs/en/getting-started)
2. [ReactJSDay 2019](https://noriste.github.io/reactjsday-2019-testing-course/book/jest-101/what-is-jest.html)
3. [神 Q 超人 - Jest | 讓 Jest 為你的 Code 做測試-基礎用法教學](https://medium.com/enjoy-life-enjoy-coding/%E8%AE%93-jest-%E7%82%BA%E4%BD%A0%E7%9A%84-code-%E5%81%9A%E5%96%AE%E5%85%83%E6%B8%AC%E8%A9%A6-%E5%9F%BA%E7%A4%8E%E7%94%A8%E6%B3%95%E6%95%99%E5%AD%B8-d898f11d9a23)
4. [神 Q 超人 - Jest | 跨越同步執行的 Jest 測試](https://medium.com/enjoy-life-enjoy-coding/unit-test-%E8%B7%A8%E8%B6%8A%E5%90%8C%E6%AD%A5%E5%9F%B7%E8%A1%8C%E7%9A%84-jest-%E6%B8%AC%E8%A9%A6-93bda140dcaa)
5. [神 Q 超人 - Jest | 測試設置分類（describe）及作用域（scoping)](https://medium.com/enjoy-life-enjoy-coding/unit-test-%E6%9B%BF%E6%B8%AC%E8%A9%A6%E8%A8%AD%E7%BD%AE%E5%88%86%E9%A1%9E-describe-%E5%8F%8A%E4%BD%9C%E7%94%A8%E5%9F%9F-scoping-2c5082266ca)
6. [神 Q 超人 - Jest | JOJO 是你？我的替身能力是 Mock ！](https://medium.com/enjoy-life-enjoy-coding/jest-jojo%E6%98%AF%E4%BD%A0-%E6%88%91%E7%9A%84%E6%9B%BF%E8%BA%AB%E8%83%BD%E5%8A%9B%E6%98%AF-mock-4de73596ea6e)
7. [jyt0532 - 測試替身(上篇)](https://blog.techbridge.cc/2018/03/02/test-double-1/)
