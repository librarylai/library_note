# 【筆記】Yup 表單驗證套件

###### tags: `筆記文章`

![](https://i.imgur.com/gqU673n.png)

我們在[【筆記】React Formik 表單套件](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/ByYQIBxEq) 這篇介紹 Form 表單時，官方在 Validation 的部分就推薦使用 [Yup](https://github.com/jquense/yup) 這套驗證的 Library 來做表單驗證的部分，當時沒有特別去介紹這套 Library 怕篇幅太長也偏離了主題，因此這次直接整理一篇關於 Yup 的用法以及介紹一些好用的方法，主要會以實務方面會碰到的驗證規則為主。

## Yup

> Yup is a schema builder for runtime value parsing and validation. Define a schema, transform a value to match, assert the shape of an existing value, or both. Yup schema are extremely expressive and allow modeling complex, interdependent validations, or value transformation.

我們可以透過 Yup 提供的方式去定義一套 schema 來驗證 value，或是透過 Yup 幫我們做型別轉換 (ex. 字串 `'0'` 轉 數字 `0`)，這樣就可以很方便的做到許多複雜的驗證，我們直接先接續上一篇 `Formik` 的部分把表單驗證的部分透過 Yup 來實作一下吧！

### 安裝

> npm :
> npm install yup --save
>
> yarn :
> yarn add yup

## 調整上次範例改用 Yup

我們先來回顧一下上次 `Formik` 驗證部分的程式碼，可以看到光是 email 格式驗證的部分就要寫好一段複雜的正規表達式(regular expression)。

```javascript=
const formik = useFormik({
    initialValues: { email: '', password: '' },
    // 重點在此
    validate: (values) => {
      const errors: { email?: String; password?: String } = {}
      if (!values.email) { // email 必填
        errors.email = 'Email 沒有值'
      } else if (!/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i.test(values.email)) {
        // email 必須為 email 格式
        errors.email = 'Email 不正確'
      }
      // 密碼 必填
      if (!values.password) errors.password = '密碼沒有填'
      return errors
    },
    onSubmit: (values, { setSubmitting }) => {
      setTimeout(() => {
        alert(JSON.stringify(values, null, 2))
        setSubmitting(false)
      }, 400)
    },
})
```

而 Yup 直接幫我們將 email 驗證規則封裝成一個 string schema(`string.email(message?: string | function): Schema`)，因此我們可以直接使用 `yup.string().email()` 就完成了 email 的驗證功能。

且在設計錯誤訊息也方便很多，直接在這些判斷 function 上寫入 error message 即可，`Formik` 會自動將錯誤訊息塞入到 `formik.errors` 內，就不用像上面一樣還特別建立一個 `error` 物件來存這些 error message。

```javascript=
/* 使用 yup 來做驗證 */
const validationSchema = Yup.object().shape({
  email: Yup.string().required('Email 沒有值').email('Email 格式不對'),
  password: Yup.string().required('密碼沒有填')
})
```

記得在使用 `Formik` 套件時要將 `validate` 改成 `validationSchema` 這是專門為 Yup 套件設計的參數。然後將 `Yup.object()` 傳進去，且記得物件內的 `key` 必須跟 `Formik` 的 field name 對應到。

```javascript=
 const formik = useFormik({
    initialValues: { email: '', password: '' },
    // 重點在此 validationSchema
    validationSchema: validationSchema,
    onSubmit: (values, { setSubmitting }) => {
      setTimeout(() => {
        alert(JSON.stringify(values, null, 2))
        setSubmitting(false)
      }, 400)
    },
 })
```

#### 先來看一下成果(基本上要跟之前一樣)

![](https://i.imgur.com/lm1qku8.gif)

## 進階內容 - 增加『確認密碼』邏輯

我們已經成功將上次的範例用 Yup 改寫，現在要來增加一些難度，順便多介紹一些 Yup 提供的 function。
先來腦力激盪一下，如果要做『確認密碼』的驗證的話我們需要判斷哪些內容呢？

首先要先確認『密碼』欄位是否已經有填寫了，再來要判斷是否輸入的與密碼相同，因此這次我們要用到 `when()` 與 `oneOf`這兩個 function 來幫助我們達成。

#### `Schema.when(keys: string | string[], builder: object | (values: any[], schema) => Schema): Schema` 介紹：

我們可以透過 `when` 的第一個參數(`keys`) 來去拿到『同層或同層子集』欄位的值，之後可以透過第二個參數對這個值來做一連串操作，它可以是個 builder 物件，也可以是一個 return Schema 的 function。

**先從 builder object 來看：** 主要會用到 `is`、`then`、`otherwise` 這個參數

- `is`： 必須 return true or false，如果為 true 則會進到 `then`，如果為 false 則會進到 `otherwise`。
  舉例： is : (value)=>{return value == true}
- `then`： is 為 true 時進入
- `otherwise`： is 為 false 時進入

```javascript=
let schema = object({
 isBig: boolean(),
 count: number()
   .when('isBig', {
     is: true, // alternatively: (val) => val == true
     // 此範例會進入 then
     then: (schema) => schema.min(5),
     otherwise: (schema) => schema.min(0),
   })
});
```

**再來看 Function 的使用：**

```javascript=
 let schema = object({
  isBig: boolean(),
  count: number()
    .when('isBig', ([isBig], schema) =>
      isBig === true ? schema.max(5) : schema.min(0),
    ),
});
```

#### `Schema.oneOf(arrayOfValues: Array<any>, message?: string | function): Schema Alias: equals` 介紹：

`oneOf` 是用來判斷是否『驗證的值』有包含在 `arrayOfValues` 裡面，如果沒有則透過第二個參數(`message`)來回應錯誤訊息。

> 注意： 因為 `oneOf` 預設是會讓 undefined 通過，因此基本上需要加上 `required()` 來防止沒填時被通過驗證的情況。

`oneOf` 在顯示 message 的時候還有提供兩種注入(interpolation) value 的方式，讓我們顯示『當前這個欄位』目前輸入的值(`${value}`)，與當前 `arrayOfValues` 實際的值(`${resolved}`)。

等等會用範例一次給大家看這兩個 interpolation 的效果。

### 實際範例 - 確認密碼

上面把確認密碼驗證功能會用到的 function 介紹了一下，現在我們實際回到專案中來改寫一下表單與驗證的部分吧！

```jsx=
/* 表單部分 - 增加 確認密碼 輸入框 */
/* email password 審略..... */
/* ps. 這邊故意都把 type 改成 text 方便讓大家看到變化 */
 <div style={{ margin: '20px' }}>
    <label>Confirm Password：</label>
    <input autoComplete='off' type='text' {...formik.getFieldProps('confirmPassword')} />
    {errors.confirmPassword && touched?.confirmPassword && <p style={{ color: 'red' }}>{errors.confirmPassword}</p>}
</div>
/* submit button 審略..... */


/* 驗證部分 - 增加 確認密碼 驗證邏輯 */
const validationSchema = Yup.object().shape({
    email: Yup.string().required('Email 沒有值').email('Email 格式不對'),
    password: Yup.string().required('密碼沒有填'),
    confirmPassword: Yup.string()
        .when('password', (password, schema) => {
            return password ? schema
                .oneOf([password], '需與密碼相同， 當下 confirmPassword 輸入值 ${value} 。 oneOfArray 值 ${resolved}')
                .required('密碼必填')
            : schema
        }),
})
```

#### 成果畫面

從畫面上可以看到 `${value}` 與 `${resolved}` 這兩個的不同，以及當確認密碼與密碼輸入一致時，代表通過驗證(錯誤訊息消失)。

![](https://i.imgur.com/pUkywua.gif)

### 優化確認密碼 驗證邏輯

上面我們用 `when` 與 `oneOf` 的組合完成了確認密碼的功能，觀察一下上面的程式碼可以發現我們使用 `when` 主要是為了取得 password 的值，那有沒有更短的方法去取得 password 的值呢？

當然有！！那就是 『`ref()`』，可以把它當作像是 Object 的『傳址』一樣，創建一個 reference 且直接指定到 password 這個 reference 上面。

> 補充：如果對『傳值』與『傳址』不懂的話，可以參考：[深入探討 JavaScript 中的參數傳遞：call by value 還是 reference？](https://blog.huli.tw/2018/06/23/javascript-call-by-value-or-reference/)

**現在先來看一下 ref 這個參數吧！**

#### `Yup.ref(path: string, options: { contextPrefix: string }): Ref` 介紹：

創建一個 reference 到『同層或同層子集』的欄位上，Refs 會在 `validation` 與 `cast` 的時候被解析(resolved)，且也會在該欄位要被使用時解析(resolved)，直接看一下官方例子來解釋好了。

```javascript=
let schema = object({
  // baz：去 reference foo 子集的 bar，簡單來說就是：baz 的值就是看 bar 是啥就是啥
  baz: ref('foo.bar'),
  foo: object({
    bar: string(),
  }),
  x: ref('$x'),
});

schema.cast({ foo: { bar: 'boom' } }, { context: { x: 5 } });
// 上面說的『解析』指的是當要顯示 baz 的值時，我才去 bar 裡面把他的值抓出來。
// => { baz: 'boom',  x: 5, foo: { bar: 'boom' } }
```

#### 實際範例 - 優化確認密碼

現在我們將上面的 `when` 改成用 `ref` 的形式，來看看程式碼的變化吧！

```javascript=
/*版本2：使用 ref 與 oneOf */
const validationSchema = Yup.object().shape({
  email: Yup.string().required('Email 沒有值').email('Email 格式不對'),
  password: Yup.string().required('密碼沒有填'),
  /*版本2：使用 ref 與 oneOf */
  confirmPassword: Yup.string()
    .oneOf([Yup.ref('password')], '需與密碼相同， 當下 confirmPassword 輸入值 ${value} 。 oneOfArray 值 ${resolved}')
    .required('密碼必填'),
})
```

## 進階再進階 -『密碼』驗證邏輯 複雜化

在上面的部分密碼欄位只有設定『必填』而已，因此現在我們要來讓它變得更複雜一些。

#### 增加規則如下：

1. **開頭必須『大寫』英文**
2. **最少要有 8 碼**
3. **最多只能 10 碼**
4. **結尾必須是數字**

相信這樣應該有一點點複雜了吧，如果要純用 js 去寫的話少說也要好幾個 `if else`，當然我相信許多高手前輩們可以直接用一串 Regular Expression 搞定，但我自己是比較推薦將每個驗證規則各別拆出來，因為這樣也『方便未來功能調整』與『新人接手開發時方便閱讀』，而 Yup 在處理複雜的邏輯方面就非常符合這樣的設計，我們可以各別攥寫每個驗證規則，最後再把它們串連起來。

接下來將會用到 `matches`、`min`、`max` 這幾個 Yup function 來達成新密碼驗證規則，老樣子先來各別介紹一下這幾個 function 吧！

#### `string.matches(regex: Regex, message?: string | function): Schema` 介紹：

Yup 的 `string.matches` 就跟 [JavaScript String match()
](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/String/match)一樣要傳入正則表達式(Regexp)，但 Yup 的 `string.matches`
可以透過第二個參數(`message`)來設定當『不符合 Regexp』時顯示錯誤訊息。

```javascript=
let schema = string().matches(/(hi|bye)/);

await schema.isValid('hi'); // => true
await schema.isValid('nope'); // => false
```

#### `string.min(limit: number | Ref, message?: string | function): Schema` 介紹：

`string.min` 主要是來限制字串的『最小』長度，它跟上面的 `oneOf` 一樣也可以插入(interpolation)`${min}` 在 `message` 參數上，可以顯示出 `limit` 這個參數的 value。

#### `string.max(limit: number | Ref, message?: string | function): Schema` 介紹：

它跟 `min` 一樣，差別在於它代表的是字串的『最大』長度，它一樣也可以插入`${max}` 在 message 參數上，可以顯示出 `limit` 這個參數的 value。

### 實際範例 - 新 密碼驗證規則

上面大概介紹了 `matches`、`min`、`max` 這三個的使用方法，現在直接回到專案來實際改一下 code 吧！

```javascript=
const validationSchema = Yup.object().shape({
  email: Yup.string().required('Email 沒有值').email('Email 格式不對'),
  password: Yup.string()
        .required('密碼沒有填') // 必填項目
        .matches(/^([A-Z])/,'第一個字必須英文大寫') // 開頭大寫
        .min(8,'至少要填寫 8 碼 min: ${min}') // 最少 8
        .max(10,'最多只能填寫 10 碼 max: ${max}') // 最多 10
        .matches(/([0-9])$/,'結尾必須是數字'), // 結尾必須是數字
  /*版本2：使用 ref 與 oneOf */
  confirmPassword: Yup.string()
    .oneOf([Yup.ref('password')], '需與密碼相同， 當下 confirmPassword 輸入值 ${value} 。 oneOfArray 值 ${resolved}')
    .required('密碼必填'),
})
```

我們可以看一下 `password` 的部分，是不是整個驗證規則非常一目瞭然呢？透過用 Yup 這個套件整個連 `if else` 都不用寫，而且在程式碼的呈現方面也能夠條列的很清楚，整個變得好閱讀了很多。

#### 成果畫面

從畫面上可以看到我們整個測試步驟為以下：

1. **是否有填寫**
2. **開頭是否大寫英文**
3. **是否有達到 8 碼**
4. **是否超越 10 碼**
5. **結尾是否為數字**

![](https://i.imgur.com/DUAaSUM.gif)

## 進階再進階 - 客製化驗證

Yup 雖然已經提供很多方便的 function 給我們使用了，但有時還是會碰到一些情況是需要客製化的部分，比如說如果 password 裡面有包含 `'admin'` 字眼的話，就顯示 `不可包含 admin 字眼` 的錯誤提示，而這邊我們就可以使用 `Schema.test` 來實作客製化的部分。

#### `Schema.test(name: string, message: string | function | any, test: function): Schema` 介紹：

`Schema.test` 中的三個參數，第一個 `name` 就像是 key 一樣代表著 unique name，第二個 `message` 就是錯誤提示，第三個`test function` 就是客製化驗證邏輯實作的部分。

### 實際範例 - 增加客製化規則

這邊把剛剛的 password 欄位再增加一條 `不可包含 admin 字眼` 規則。

```javascript=
password: Yup.string()
    .required('密碼沒有填') // 必填項目
    .matches(/^([A-Z])/, '第一個字必須英文大寫') // 開頭大寫
    .min(8, '至少要填寫 8 碼 min: ${min}') // 最少 8
    .max(10, '最多只能填寫 10 碼 max: ${max}') // 最多 10
    .matches(/([0-9])$/, '結尾必須是數字') // 結尾必須是數字
    .test('noAdmin', '不可包含 admin 字眼 ${path}', (value) => {
      // 不可包含 admin 字眼 （ path 為當下欄位的路徑，這邊代表是 password 這個欄位 ）
      return !value?.includes('admin')
    }),
```

#### 成果畫面

![](https://i.imgur.com/7YmhdYO.png)

## 純ＪＳ搭配 Yup 做驗證

整篇文章到目前為止都是以 Formik 搭配 Yup 的部分在討論，但可能不是每間公司在表單方面都是使用 Formik 套件，可能只是用純 HTML form 加上 `useState` 來達成表單輸入，因此這邊將來模擬一下純 JS 如何與 Yup 做搭配。

### 我們先來改寫一下 Html 的部分：

先將前面例子中用到的 Formik 套件拔除，並且加入 `useState` 來紀錄每一個 input 的 value，因此我們要手動寫一下 `onChange` 的部分來將每個 input 的變化寫入到對應的 `setState` 中。

```jsx=
/* 使用 useState */
const [email, setEmail] = useState('');
const [password, setPassword] = useState('');
const [confirmPassword, setConfirmPassword] = useState('');

/* form 表單內容移除 Formik , 使用 useState */
<form onSubmit={handleSubmit}>
    <div style={{ margin: '20px' }}>
      <label>Email：</label>
      <input
        autoComplete="off"
        type="email"
        value={email}
        // 增加這段
        onChange={(e) => setEmail(e.target.value)}
      />
    </div>
    <div style={{ margin: '20px' }}>
      <label>Password：</label>
      <input
        autoComplete="off"
        type="text"
        value={password}
        // 增加這段
        onChange={(e) => setPassword(e.target.value)}
      />
    </div>
    <div style={{ margin: '20px' }}>
      <label>Confirm Password：</label>
      <input
        autoComplete="off"
        type="text"
        value={confirmPassword}
        // 增加這段
        onChange={(e) => setConfirmPassword(e.target.value)}
      />
    </div>
    <div style={{ margin: '20px' }}>
      <button type="submit">Submit</button>
    </div>
</form>
```

### 客製化驗證流程

還記得我們在使用 Formik 時會透過 `validationSchema` 這個參數來處理驗證的部分，而套件自動幫我們將錯誤訊息寫到 `errors` 這個物件裡面。

但因為我們現在改用純 JS 的方式處理，所以就不會有自動將錯誤訊息寫到 `errors` 這個部分，因此我們要來實作這段功能，將驗證結果寫入到 state 中並且判斷是否通過驗證。

```jsx=
/* 宣告一個用來存 錯誤訊息 的 state */
const [errMessages, setErrMessage] = useState({
    email: [],
    password: [],
    confirmPassword: [],
})
const validationSchema = Yup.object().shape({
    /* 驗證規則 跟上面一樣，這邊就先省略.... */
})

/* 客製化驗證流程 */
const customValidation = async () => {
    let isPass = false;
    validationSchema
      .validate({
        email,
        password,
        confirmPassword,
      },{abortEarly: false}) // abortEarly 代表是否要提早結束，預設為 true 因此一碰到有項目驗證錯誤時，就會結束驗證。
      .then((value) => {
        console.log('value',value);
        if(value) isPass = true
      })
      .catch((err: ValidationError) => {
        console.log({err});
      });
  };



```

#### `Schema.validate(value: any, options?: object): Promise<InferType<Schema>`, ValidationError> 介紹：

> Returns the parses and validates an input value, returning the parsed value or throwing an error. This method is asynchronous and returns a Promise object, that is fulfilled with the value, or rejected with a ValidationError.

可以看到 `validate` 返回的是一個 `Promise` 因此當碰到『驗證錯誤的情況時』它會將錯誤內容丟進 `reject` 中，因此這邊要『記得』用 `catch` 才能正確的拿到 `ValidationError` 物件。

![](https://i.imgur.com/ycCuLgI.png)

> 補充：如果是用 `async/await` 的方式的話，要記得用 `try/catch`來包住，不然可能會跳出 Error 畫面且在 `catch` 中才能拿到 error object。

我們可以從 ValidationError `inner` 裡的 `path` 參數中得知是哪個驗證欄位(`key`)錯誤，且可以從 `message` 中拿到錯誤訊息是什麼，有了這些資訊我們就可以將『哪個欄位是哪個錯誤訊息』給記錄起來了。

```javascript=
.catch((err: ValidationError) => {
    console.log({ err });
    const { inner } = err;
    const errObject = {};
    inner.forEach((item) => {
      // 如果 path 或 message 為 undefined 的話，直接 return
      if (!item.path || !item.message) return;
      // 因為有可能一個欄位會有多個驗證錯誤，因此這邊是設計把全部記在一個 array 裡面
      // 例如： password 就會有一次出現兩種 error 的情況（ 開頭必須英文、長度至少 8 碼 ）
      errObject[item.path] = errObject[item.path]
        ? [...errObject[item.path], item.message]
        : [item.message];
    });
    setErrMessage(errObject);
});
```

![](https://i.imgur.com/asoiGhq.png)

### 將錯誤訊息呈現在畫面上

這邊是寫一個共用的 render function 來顯示 Error Message，並且在每個 input 區塊加上此 function。

```jsx=
// 共用 render 錯誤訊息
const renderErrorMessageText = (fieldKey) => {
    return errMessages[fieldKey]?.map((msg, index) => {
      return (
        <p key={index} style={{ color: 'red' }}>
          {msg}
        </p>
      );
    });
};

// input 部分加上 renderErrorMessageText
<div style={{ margin: '20px' }}>
  <label>Email：</label>
  <input
    autoComplete="off"
    type="text"
    value={email}
    onChange={(e) => setEmail(e.target.value)}
  />
  {/* 加上這邊 */}
  {renderErrorMessageText('email')}
</div>

```

#### 成果畫面

![](https://i.imgur.com/S4YRExg.gif)

### 補充： Parsing: Transforms：

Yup 不僅有驗證功能還有轉換的功能，可以依照設定的 Schema 結構來將輸入值轉換成我們要的內容。

> **`Schema.cast` 介紹：** > `cast` 會強制將輸入值依照 schema 需要內容來轉換，例如：輸入是字串 `'5'` 但 schema 是 `number()` 的話，`cast` 會將字串轉成數字 `5` 。

```javascript=
let stringFive = Yup.number().cast('5'); // 5

const obj = Yup.object({
    firstName: Yup.string().lowercase().trim(),
})
.cast('{"firstName": "jAnE "}'); // { firstName: 'jane' }
```

#### 以上就是這次【 Yup 表單驗證套件 】的全部內容，希望對正要處理表單驗證的人能有一點點幫助，關於表單套件的部分也可以參考我的另一篇文章『[【筆記】React Formik 表單套件](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/ByYQIBxEq)』，以上如有任何錯誤或冒犯的地方還請各位多多指教，謝謝您的觀看。

#### Github : [https://github.com/librarylai/react-formik-test](https://github.com/librarylai/react-formik-test)

## Reference

1. [Yup - github](https://github.com/jquense/yup#refpath-string-options--contextprefix-string--ref)
2. [深入探討 JavaScript 中的參數傳遞：call by value 還是 reference？ - Huli](https://blog.huli.tw/2018/06/23/javascript-call-by-value-or-reference/)
