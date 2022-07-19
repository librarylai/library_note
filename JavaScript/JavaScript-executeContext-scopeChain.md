# 【筆記】Execute Context & Scope Chain 筆記

###### tags: `筆記文章`

此為看完 **冴羽-深入系列 Blog** 與 **Huli-JavaScript 五講** 後所做的筆記。

_本篇文章中會直翻 context 為【上下文】，如果太難閱讀也可以把它翻譯成【情境】。_

## Execution Context

在進入本篇筆記之前，首先需要了解**何謂 Execution Context？**
Execution Context 就像是一位大樓的管理員，而每一段我們寫下的程式(Code)就像是裡面的住戶，這些 Code 都會產生一個 Lexical Environment，而 Execution Context 則幫助管理哪段 Code 正在被執行。
**Execution Context 基本上包含了以下內容：**

> When control enters an execution context, the scope chain is created and initialized, variable instantiation is performed, and the this value is determined.

- **Variable object，VO**
  Variable object 是執行上下文(情境)中相關資料的作用域，它是一個特殊的物件，存放著上下文中定義的變數和函式宣告，因為執行環境的不同所以 Variable object 所代表的名稱或意思會稍微不同，例如 : 函式上下文中會叫做 Activation Object，全局上下文中會叫 Global Object。

- **Global Object:**
  Global Object 是在進入 execution context 之前就被創建，在 Browser 中其實就是我們常見的 Window 物件。 >節錄自：ECMAScript262 章節 10.1.5 Global Object >There is a unique global object which is created before control enters any execution context. Initially the global.
  Additional host defined properties. This may include a property whose value is the global object itself, for
  example window in HTML.

- **Activation Object:**
  當進入函式所產生的 execution context 後將會創建一個 AO，而之後 AO 便被當作這個函式(function execution context)的 VO 被拿去使用， AO 可以說是函式情境下的變數物件(VO)。

      >節錄自：ECMAScript262 章節 10.1.6 Activation object
      >When control enters an execution context for declared function code, anonymous code or implementation supplied code, an object called the activation object is created and associated with the execution context. The

  activation object is initialized with a property with name arguments and property attributes { DontDelete }.
  The initial value of this property is the arguments object described below.

- **This**
  This 的值會在進入 execution context 被決定，取決於呼叫它的人且是不可變的。 >There is a this value associated with every active execution context. The this value depends on the caller and
  the type of code being executed and is determined when control enters the execution context. The this value
  associated with an execution context is immutable.

- **Scope**
  每當進入一個 Execution Context 時，將會創建關於它的 Scope Chain，而當離開 execution context 時，scope chain 才將會被摧毀。
  ( ps.這個概念有助於之後我們對「閉包」的瞭解。)
  關於 Scope chain 的組成會在下方的內容做進一步的說明，這邊就先簡單帶過，讓大家有個概念。

      ```javascript=
      // scope chain 創建
      fnEC.scope = AO + fn.[[Scope]]
      ```
      >節錄自：ECMAScript262 章節 10.1.4 Scope Chain and Identifier Resolution
      >Every execution context has associated with it a scope chain. This is logically a list of objects that are searched
      when binding an Identifier. When control enters an execution context, the scope chain is created and is populated
      with an initial set of objects, depending on the type of code. When control leaves the execution context, the scope
      chain is destroyed.

### 『補充』📘📗📙

#### 作用域鏈(Scope Chain)

作用域鏈(Scope Chain) 就如同 原型鏈(Prototype Chain)一樣，當在自身的作用域中( 變數物件 VO 或 執行中物件 AO )找不到需要的變數時，就會透過 Scope Chain 往父層的作用域去查找，直到尋找到目標或到最外層(Global Execution Context)才會結束。

## 全域執行環境( Global Execution Context )，

每當我們第一次在 Browser 執行 Javascript 時，預設會創造一個全域執行環境( Global Execution Context )，而它也跟上面一樣包含了相同的內容。

**1. Variable Object：**
GlobalEC 中的 Variable Object 在 Borswer 中指向為 _Window 物件_。

**2. This 指向：**
根據 ECMAScript 規範文件中提到，關於全域執行環境( Global Execution Context )中的 this 指向會 global object 。 >節錄自：ECMAScript® 2020 Language Specification 8.1 Lexical Environments >This global object is the value of a global environment’s this binding.

**3. Scope：**
全域環境中的 Scope 為 Global Object(VO)，因為在全域環境中的 outer environment 的參考為 null。

```
// 我們知道
EC.scope = AO + currentEC.[[scope]]
// GlobalEC
GlobalEC.scope = (AO=VO=Global Object) + null
```

> 節錄自：ECMAScript® 2020 Language Specification 8.1 Lexical Environments
> A global environment is a Lexical Environment which does not have an outer environment. The global environment’s outer environment reference is null.

## 關於 Function Execution Context

函數從創建到執行可以分成三個部分：

1. 函數創建時
2. 函數進入執行上下文時 （Function Execution Context Create）
3. 函數執行時期( Function Execution Context Run Time )

### 函數創建時

首先要先知道 JavaScript 採用的是語彙範疇( Lexiacal Scope )，又稱靜態作用域。

#### Lexiacal Scope 的特性如下：

- **Lexical Scope 代表著區塊間的包裹關係：**
  外層的 Scope 無法取用內層 Scope 的變數，而內層 Scope 可以取用外層的變數
  舉例來說:
  當在 innerScope 中呼叫外層的 name(變數)時，我們可以成功獲取到值(Mozilla)。但我們無法從外層(parentScope)去取得內層(innerScope)的變數數值。
  ```javascript=
  // MDN 語法作用域（Lexical scoping） 範例
  function parentScope() {
      var name = "Mozilla"; // name is a local variable created by init
      function innerScope() { //
          var innerScopeVariable = 'innerScopeVariable'
          console.log(name); // Mozilla
      }
      innerScope();
      console.log(innerScopeVariable) // Uncaught ReferenceError: innerScopeVariable is not defined
  }
  parentScope();
  ```
- **Lexical Scope 是在語法解析時就決定：**
  靜態作用域的變數在**語法解析**時就已經決定了它的作用域，因此不會因執行過程中而改變。
  各位可以試看看下面這個例子，看看是不是跟自己想的一樣

```javascript=
var text = 'globalText'
function scope1(){
    console.log(text)
}
function scope2(){
    var text = 'scope2Text'
    scope1()
}
scope2()
```

由此可知，函數的作用域(scope)在函數定義時就決定了，且函數內部有一個屬性 [[scope]]，當函數(Function)創建的時候，[[scope]]就會保存父層的變數物件(VO)到其中。

#### 函數創建時舉例：

因為 createFunctionScope 創建時是在最外層(Global Execution Context)中，而 Global Execution Context 的 VO 為 Global Object，所以 createFunctionScope 的 [[scope]] 就為 globalExecutionContext.VO(global Object)

```javascript=
function createFunctionScope(){
    let abc = '......'
}

/* Function 內部機制 */
createFunctionScope.[[scope]] = [
    globalExecutionContext.VO
];

// Global Execution Context
GlobalExecutionContext ={
    vo:{
        createFunctionScope: reference to function createFunctionScope(){}
    }.
    scope:[GlobalExecutionContext.vo]
}

```

---

### 函數進入執行上下文時

**_請注意：這個時期還沒有執行代碼。_**
在這個階段會產生 Function Execution Contect，並且將該函式加入到 Execution Context Stack 中，它的結構跟上面介紹的 Execution Contect 大致相同，不一樣的地方是 Execution Contect 中的 Variable object 在 Function Execution Contect 會使用 AO 來代替，且 AO 內還多了參數(Parameter)、Arguments 物件的內容。

```javascript=
// Execution Context Stack
ECStack = [
    nowFnContext,
    globalEC
]
```

**關於( Activation Object, AO )：**

1. 在函數上下文中，我們用執行中物件(Activation Object, AO)來表示函式中的變數物件(VO)。
2. 執行中物件(Activation Object, AO)是函式被呼叫變成執行狀態(Activated)時，產生的一個特殊物件。
3. 執行中物件(AO) 包含了變數宣告、函式宣告、參數(Parameter)、Arguments 物件

### 函數進入執行上下文時的 變數物件( Variable object ) 與 作用域( Scope )

### 變數物件( Variable object = AO)

變數物件包括：

- 函數的所有形參
- 函數宣告
- 變數宣告

**函數的所有形參：**

- **由名稱和對應值組成的一個變數物件的屬性被創建**
- **沒有實參，屬性值設爲 undefined**
  Ps. 如果對 形參 或 實參 不懂的讀者，可以參考此文章 [C 語言中函數的形參與實參是什麼？](https://kknews.cc/zh-tw/code/vvol9qq.html) 、 [yehyeh's Blog - 執行中物件(Activation Object)](http://notepad.yehyeh.net/Content/WebDesign/Javascript/ECMA/Core/JavaScriptCore.php#section6)
  或者可以參考 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions) 上的翻譯 (推薦閱讀英文版，但英文內容未使用到 形參、實參，所以這邊還是介紹一下。 )

```javascript=
調用函數時，傳遞給函數的值被稱爲函數的實參（值傳遞），對應位置的函數參數名叫作形參。

如果實參是一個包含原始型別 (數字、字符串、布林值)的變數( call by value )
則就算函數在內部改變了對應形參的值，返回後，該實參變數的值也不會改變。

如果實參是一個物件引用 ( call by reference )，則對應形參會和該實參指向同一個物件
假如函數在內部改變了對應形參的值，返回後，實參指向的物件的值也會改變
```

Ps. 如果對 call by value、call by reference 不懂的讀者，可參考 **Huli 大大** 的 [深入探討 JavaScript 中的參數傳遞：call by value 還是 reference？](https://blog.techbridge.cc/2018/06/23/javascript-call-by-value-or-reference/)

**以下節錄自 yehyeh 大大的解說：**

> Argument(實參) & Parameter(形參)
> 引數(Argument) = 實際參數(Actual Parameter )
> 參數(Parameter) = 形式參數(Parameter)

```javascript=
function createFunctionScope(param1, param2){     // param1, param2 = Parameter
   return param1 + param2;
}
var a = 10 , b=20
var result = createFunctionScope(a, b);  // a,b = Argument
```

(Reference: [yehyeh's Blog - 執行中物件(Activation Object)](http://notepad.yehyeh.net/Content/WebDesign/Javascript/ECMA/Core/JavaScriptCore.php#section6) )

**函數宣告：**

- 由名稱(key)和對應值(value)（函數物件(function-object)）組成一個變數物件
- 如果變數物件已經存在相同名稱的屬性，則完全替換這個屬性

**變數宣告：**

- 由名稱(key)和對應值(value) undefined 組成一個變數物件的
- 如果變數名稱跟已經宣告的形式參數或函數相同，則變數宣告不會干擾已經存在的這類屬性

### 作用域( Scope )

剛剛說過函式創建時內部會有一個 [[Scope]] 的屬性來記住父層的 Scope，當函式被呼叫時(也就是現在這個階段)會產生一個 Execution Context，並且將自己的 AO 與 [[Scope]] 一起放入到這個 Execution Context 的 Scope 屬性中。

```javascript=
// 函式創建時
fn.[[scope]] = 父層的作用域(Scope) = [父層AO,[[父層scope]]]
// 函數執行上下文
fn.scope = [fn.AO,fn.[[scope]]]

```

### **函數進入執行上下文 舉例說明：**

這個時期是函式被加入到 Execution Context Stack 中等待被執行時。

```javascript=
/*  Execution Context Stack */
ECStack = [
    createFunctionScope,
    globalEC
]

/* function */
function createFunctionScope(paramA) {
  var b = 2;
  function c() {}
  var d = function() {};
  b = 3;
  c()
}

createFunctionScope(1); // 注意這裡！！
```

這邊可以看到 createFunctionScope 被呼叫，建立了 createFunctionScope 的 Execution Context，裡面包含了執行中物件(Activation Object, AO)、作用域(Scope)，這個階段還尚未賦值只是先將( 函數宣告、變數宣告、參數、arguments )加入 AO 當中。

```javascript=

createFunctionScope={
    AO:{
        arguments:{
        '0':1
        }, // Argument
        paramA: 1, // Parameter
        b: undefined, // 由名稱(key)和對應值(value) undefined組成一個變數物件的
        c: reference to function c(){}, //由名稱(key)和對應值(value)（函數物件(function-object)）組成一個變數物件
        d: undefined
    }
    Scope:[AO,[[Scope]]]
}
```

### 函數執行時期

在函數執行時期會依序執行程式碼，根據程式碼修改變數物件（Activation Object, AO）的值
這邊接續上面函數上下文的範例，在執行時期的 AO 將會變成這樣：

```javascript=
// 此為執行時期 AO
createFunctionScope={
    AO:{
        arguments:{
        '0':1
        },
        paramA: 1,
        b: 2,
        c: reference to function c(){},
        d: reference to FunctionExpression "d"
    }
    Scope:[AO,[[Scope]]]

}
```

當函式執行完成後，會將該函式從 Execution Context Stack 中移除。

```javascript=
// Execution Context Stack
ECStack = [
    // nowFnContext, 函式執行完成，已被移除
    globalEC
]
```

## 總結

這篇文章主要為讀完 **冴羽-深入系列 Blog** 與 **Huli-JavaScript 五講** 後的心得筆記，上面講了 Execttion Context 的組成與 Function 從創建到執行完成中每個階段各自做了些什麼事情，以及帶入了 Scope Chain 的一些概念。
有了上面的概念後，最後再回來看一下『閉包』的定義「由函式和與其相關的參照環境組合而成的實體」，是不是呼應了函式的 [[scope]] 屬性，[[scope]] 在函式創建時就記錄著父層的 Scope，而父層的 Scope 不就代表可以獲取到它的 Variable object，這就是為什麼當父層函式執行完後，子層函式還是可以取到父層的原因，也能解釋『閉包可以把值鎖住函式(Function)裡面』這句話了。

#### 以上是個人看完以下大神的文章後所作的筆記與見解，如有任何錯誤或冒犯的地方還請各位多多指教。

### 謝謝觀看。

## Reference

1. [冴羽 - JavaScript 深入之作用域链](https://github.com/mqyqingfeng/Blog/issues/6)
2. [冴羽 - JavaScript 深入之变量对象](https://github.com/mqyqingfeng/Blog/issues/5)
3. [Huli - 所有的函式都是閉包：談 JS 中的作用域與 Closure](https://blog.huli.tw/2018/12/08/javascript-closure/)
4. [Huli - 深入探討 JavaScript 中的參數傳遞：call by value 還是 reference？](https://blog.techbridge.cc/2018/06/23/javascript-call-by-value-or-reference/)
5. [yehyeh's Blog - 執行中物件(Activation Object)](http://notepad.yehyeh.net/Content/WebDesign/Javascript/ECMA/Core/JavaScriptCore.php#section6) )
6. [C 語言中函數的形參與實參是什麼？](https://kknews.cc/zh-tw/code/vvol9qq.html)
