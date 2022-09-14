# 【筆記】GraphQL 系列(五) - GraphQL 開發雜記、套件

###### tags: `筆記文章`

**本篇內容會比較零散，主要是紀錄開發過程中碰到的問題以及解決辦法，或是一些 Apollo 功能使用上的紀錄，還請多見諒！！ 🙇**

---

## 使用 apollo-link-token-refresh

最近專案上碰到一個 Bug (**當使用者『停留在網站』時將筆電待機一段時間後，再次打開時會因為 access token 過期而導致 call api 失敗**）。

在開始解釋問題之前先來簡單解釋一下 token 登入機制，一般主要分成 『refresh token』與『access token』這兩種不同的 token，**access token** 主要是用來呼叫 api 取得各種使用者資訊，而 **refresh token** 主要是用來更新 **access token**。

一般來說兩者的『過期』時間也不太一樣，比如說：**access token** 是一小時過期，**refresh token** 是一週過期...等（ps.沒有強制限制一定要多久時間）。

#### 整個登入機制大概會是：

1. **使用者登入** - 給予 **refresh token** 與 **access token**
2. **access token** 過期 - 透過 **refresh token** 再取得一把『新的』**access token**
3. **refresh token** 過期 - 導回登入頁，請使用者重新登入

### 問題發生原因

因為當發現 access token 過期的前提是已經先發送一個 request 過去到後端之後才發現過期，因此希望將整個呼叫 API 的流程調整成：『**在每次發送 request 之前，都先檢查 access token 是否過期，如果過期則先透過 refresh token 取得新的 access token 後，再發送 request**』，那又該如何達成這個需求呢？

這時就需要用到 [**apollo-link-token-refresh**](https://github.com/newsiberian/apollo-link-token-refresh) 這套 Plugin 來產生一個 **refreshLink** (用來做 token 檢查並發送 request 取得新 token)，並且將 **refreshLink** 放在 **httpLink** 之前，讓每次要發送 request 之前都先進入到 **refreshLink** 裡面進行 token 檢查...等項目。

### 理解 JWT

在前面我們一直提到 token，那 token 又是怎麼組出來的呢？一般來說目前專案上的 token 基本上都是以 JSON Web Token（JWT）的格式。

> 推薦文章：[[筆記] 透過 JWT 實作驗證機制 - Mike Huang](https://medium.com/%E9%BA%A5%E5%85%8B%E7%9A%84%E5%8D%8A%E8%B7%AF%E5%87%BA%E5%AE%B6%E7%AD%86%E8%A8%98/%E7%AD%86%E8%A8%98-%E9%80%8F%E9%81%8E-jwt-%E5%AF%A6%E4%BD%9C%E9%A9%97%E8%AD%89%E6%A9%9F%E5%88%B6-2e64d72594f8)

因為我們需要知道這段 token 是否過期，因此我們需要先將 **access token** 解碼(decode)，並透過裡面的參數`exp`來知道這段 token 是何時過期，當然裡面也包含一些其他資訊，如下：

#### 例如：

> iss (Issuer) - jwt 簽發者
> sub (Subject) - jwt 所面向的用戶
> aud (Audience) - 接收 jwt 的一方
> exp (Expiration Time) - jwt 的過期時間，這個過期時間必須要大於簽發時間(秒)
> nbf (Not Before) - 定義在什麼時間之前，該 jwt 都是不可用的
> iat (Issued At) - jwt 的簽發時間
> jti (JWT ID) - jwt 的唯一身份標識，主要用來作為一次性 token,從而迴避重放攻擊

`exp` 時間主要是到『秒』因此要乘 1000 變成毫秒，再跟 new Date().getTime() 進行判斷，例如查詢是否過期，則可以用 `exp * 1000 <= new Date().getTime()` 判斷

- [JWT 官方線上 Debugger](https://jwt.io/)
- [jwt-decode](https://github.com/auth0/jwt-decode)

### apollo-link-token-refresh 實作

剛剛在上面有提到，如果想要在透過 **httpLink** 送出 Request 之前先判斷 accessToken 是否過期，如果過期則先呼叫 API 去取得新的 accessToken 再送出 Request，而這段邏輯可以透過 [**apollo-link-token-refresh**](https://github.com/newsiberian/apollo-link-token-refresh) 來達成，因此先來了解一下這套 Plugin 該如何使用吧！

```javascript=
const refreshLink = new TokenRefreshLink({
     // 欄位名稱需與 handleResponse 的 return 物件匹配,
    accessTokenField: 'newAccessToken'
    // isTokenValidOrUndefined 代表是否有效， return true 代表還有效(將『不會』觸發 fetchAccessToken function)，
    isTokenValidOrUndefined: (operation: Operation) => boolean,
    // fetch Data 取得新 accessToken 等... ,
    fetchAccessToken: ()=>{},
    // handleResponse return 的值會進到這邊
    handleFetch: (accessToken: string, operation: Operation) => void,
    // fetchAccessToken 完會先進到這邊，response 為 fetchAccessToken API 的 responce ，最後如果有 return 則會進 handleFetch
    handleResponse? : (operation: Operation, accessTokenField) => response => any,
    // 整套過程中如果發生錯誤，則會進 handleError
    handleError? : (err: Error, operation: Operation) => void,
});
```

> **注意：**
> 如果有設定 `authLink` 與 `refreshLink` 可能要注意查看 authLink 在 http header authorization 取值的部分。
>
> 如果是 authLink（set accessToken 到 Header ） -> refreshLink(get new accessToken 與 set new accessToken) 的情況，容易造成 authLink 為舊的 accessToken
>
> ##### 推薦改用： `const links = from([refreshLink, authLink, httpLink])` 將 authLink 與 refreshLink 位置調換，讓 refreshLink 在 authLink 前面。

#### 範例：

```typescript=
function formatRefreshLinkParams() {
  const isTokenValidOrUndefined = () => {
    const token = getItem(KEY_ACCESS_TOKEN)
    if (!token) return true // 如果沒有 token 則『不進行』接下來步驟
    const decoded = jwtDecode<JwtPayload>(token)
    // decoded?.exp 是秒 因此要 乘上 1000
    const expireTime = decoded?.exp ? decoded?.exp * 1000 : 0
    // 如果還沒過期，則『不進行』接下來步驟
    if (expireTime - tenMinutesAhead > new Date().getTime()) return true
    return false
  }
  const fetchAccessToken = async () => {
    const response = await fetch(YOUR_GRAPHQL_ENDPOINT, {
      method: 'POST',
      headers: {
        'content-type': 'application/json',
      },
      body: JSON.stringify({
        operationName: YOUR_QRAPHQL_NAME,
        query: `mutation xxx($input: xxx) {
                  user {
                      id
                    }
                  }
                }
                `,
        variables: { input:{/* 如果有參數則.... */} },
      }),
    })
    return response.json()
  }
  const handleFetch = (newAccessToken: string) => {
    // set 新的 token 或是任何 你要處理的邏輯，寫在此
    setItem(KEY_ACCESS_TOKEN, newAccessToken)
  }
  const handleResponse = () => (response) => {
    // format data
    if (!response) return { newAccessToken: null }
    return {
      newAccessToken: response.token/* 從 response 找你需要的值 */ ,
    }
  }
  const handleError = (error) => {
      /* 處理整個流程發生錯誤時 */
      console.log(error)
  }
  // Return to TokenRefreshLink
  return {
    accessTokenField: 'newAccessToken',
    isTokenValidOrUndefined,
    fetchAccessToken,
    handleFetch,
    handleResponse,
    handleError,
  }
}
const refreshLink = new TokenRefreshLink(formatRefreshLinkParams())
```

## 使用 GraphQL Code Generator

以往我們在使用 Apollo Client 撰寫 GraphQL 時，經常會使用 Apollo 所提供的 `useQuery`、`useMutation` 這兩個方法來進行查詢與操作(新增、修改、刪除...等)，它們大致上的用法如下：

```javascript=

/* query */
// GET_DOG_PHOTO 一般來說會放在 queries.ts 等檔案內統一管理每個 query
const GET_DOG_PHOTO = gql`
  query Dog($breed: String!) {
    dog(breed: $breed) {
      id
      displayImage
    }
  }
`;
// 因此會透過 import 來引入需要的 query
import { GET_DOG_PHOTO } from 'queries.ts'
const { loading, error, data } = useQuery(GET_DOG_PHOTO, {
    variables: { breed },
});

/* mutation */
// 與 queries 一樣，基本上會放在一隻 mutations.ts 統一管理
const INCREMENT_COUNTER = gql`
  mutation IncrementCounter {
    currentValue
  }
`;
// 因此需要 import
import { INCREMENT_COUNTER } from 'mutations.ts'
const [mutateFunction, { data, loading, error }] = useMutation(INCREMENT_COUNTER);

```

> 以上節錄自 Apollo 官方範例：[Queries](https://www.apollographql.com/docs/react/data/queries#caching-query-results)、[Mutations](https://www.apollographql.com/docs/react/data/mutations#executing-a-mutation)

### GraphQL 開發過程中的不變之處

從上面範例來看，在使用 Apollo 提供的 `useQuery` 與 `useMutation` 上有幾點比較不直觀的地方，或是在開發過程中的不變之處(ex. TypeScript 搭配)

1. 從 **useQuery** 或 **useMutation** 無法直接從 function 名稱就看出來是要查詢哪個 query 或是操作哪個 mutation，需要透過『第一個參數』才能看出來要做什麼，導致 Debug 看程式碼時比較不直覺。

2. 除了要引入 **useQuery** 與 **useMutation** 之外，還需引入 Query or Mutation (上述所說的統一管理的部分)。

3. **TypeScript 搭配問題！！！(此為最重要的一點)** :star: :star: :star:

   在使用 TypeScript 的專案下必須手動撰寫 **『每個 Schema 對應的 Type』**，雖然我們可以透過 Schema 快速了解每個 Field 是哪個類別(String、Int...等)，**但 Schema Type 與 TypeScript Type 依舊有一些微的差異(ex. Int -> Number)**，如果在有點規模的專案架構下，光是撰寫這些 Type 就會花上不少時間，而且 **『如果 Schema 有變動時，TypeScript 也需要配合調整』**，很容易發生遺漏導致報錯的問題。

4. **TypeScript 架構下 Response 型別註記問題**

   上面舉出了 TypeScript 架構下需要將 Schema 轉換成對應的 TypeScript Type，而另一個在開發過程中常會碰到的問題就是 **Response 型別註記**。相信使用 GraphQL 的人都知道，**『GraphQL 的特色就在於可以自己決定 Response 要哪些資料』，例如：**

   ```graphql=
   # 拿 user 的 name 跟 age
   query{
       user{
           name
           age
       }
   }
   # 拿 user 的 name,age,sex,birthday
   query{
       user{
           name
           age
           sex
           birthday
       }
   }
   ```

   因此 TypeScript 的架構下，我們不就得需要對每一個 Response 寫一個 type 或是 interface，而當 Query 結構換了(ex. name 換成 firstName)我們就需要跟著調整型別註記，這不僅花時間也不好維護。這也是為什麼會使用 [GraphQL Code Generator](https://www.the-guild.dev/graphql/codegen/) 來自動化幫我們產生這些型別與函式。

### GraphQL Code Generator 套件

使用 **GraphQL Code Generator** 能夠自動將 Schema 轉換成 TypeScript，因此每當 Schema 有變更時我們只需要重新 generator 就可以同步更新。

且它還可以幫我們將 Query 與 Mutation 轉換成 Hooks，因此只需要引入像是 `useDogPhotos` 這樣的 hooks 就可以很直覺的使用，不必再引入 `useQuery` 等這些函式了。

#### 安裝 GraphQL Code Generator (React 為例)

> yarn add graphql
> yarn add -D @graphql-codegen/cli
> yarn add -D @graphql-codegen/typescript-react-query
> yarn add -D @graphql-codegen/typescript
> yarn add -D @graphql-codegen/typescript-operations

#### 透過 Cli 快速產生 codegen.yml

> yarn graphql-code-generator init

透過 Cli 的一步一步引導，最後會產生一隻 `codegen.yml` 檔案且也會在 `package.json` 中幫我們把 script 給建立完成，大概會是以下內容：

##### package.json

Cli 中間會有一題是問您要在 `package.json` 的 script 中設定什麼樣的關鍵字來觸發 codegen，這邊是設定 `gen:types` 因此當在終端機輸入 `yarn gen:types` 就會啟動 codegen 幫我們重新產出編譯後的檔案。

```javascript=
"gen:types": "graphql-codegen --config codegen.yml"
```

##### 產生出來的 codegen.yml

```yml=
overwrite: true
schema: "http://localhost:3000/api/graphql"
documents: "graphql/gql/*.ts"
generates:
  src/generated/graphql.tsx:
    plugins:
      - "typescript"
      - "typescript-operations"
      - "typescript-react-apollo"
  ./graphql.schema.json:
    plugins:
      - "introspection"

```

### codegen.yml 參數解釋

這邊只會簡單介紹一下，詳細內容可參考：[Configuration options - GraphQL Code Generator ](https://www.the-guild.dev/graphql/codegen/docs/config-reference/codegen-config#configuration-options)

- **schema**

  > The schema field should point to your GraphQLSchema.

  基本上可以填入 GraphQL Server 的 url endpoint，且 **schema** 參數也可以指向多個 schema，codegen 會自動幫我們將多個 schema 進行合併(merge)。

  **例如：除了 Server Endpoint 之外還有 client-side Schema 的話，則可以將 schema 寫成：**

  ```javascript=
  /*以 js 舉例*/
  schema: ['http://localhost:3000/api/graphql','apollo/localState.ts']

  /* 以 yml 舉例 */
  schema:
   - 'http://localhost:3000/api/graphql'
   - 'apollo/localState.ts'
  ```

  **而如果有需要在 send endpoint 時加入 headers 的話，則可以將 schema 寫成：**

  ```javascript=
  /*以 js 舉例*/
  schema: [
      {
        'http://localhost:3000/graphql': {
          headers: {
            Authorization: 'YOUR-TOKEN-HERE',
          },
        },
      },
    ],

  /* 以 yml 舉例 */
  schema:
       - 'http://localhost:3000/graphql':{headers:Authorization: 'YOUR-TOKEN-HERE'}
  ```

- **documents**

  > The documents field should point to your GraphQL documents: query, mutation, subscription, and fragment.

  基本上就是將需要被 generate 的 query、mutation、fragment 相關的檔案寫到 `documents` 中，一般來說是會直接使用 `**/*.ts` 將資料夾內的所有檔案內容都包含進去(ex. `src/**/*.ts` 代表 src 底下的『不管是資料夾還是檔案』通通都包含)。

  `**/*.ts` 這種寫法如果是在 gql 語法散落在各個 component 的專案就非常方便，就不用將每一隻檔案都寫 `document` 中。

  > **知識補充：** > `**` 代表所有資料夾
  > `*` 代表所有檔案

- **generates**

  > A map where the key represents an output path for the generated code, and the value represents a set of relevant options for that specific file.

  **generates** 就是 generate 後的檔案路徑，且必需要填寫 `generates.plugins` 參數代表要用哪些插件，例如使用 TypeScript、React-Apollo...等套件。

### 成果

這邊將會用實際的 Side Project 當作範例來產出檔案，可以看到 GraphQL-code-generate 自動幫我們將 Schema User 轉換成 TypeScript 型別的 User type。

![](https://i.imgur.com/YUbLcTb.png)

也可以看到 GraphQL-code-generate 自動幫我們將 queries 與 mutations 轉換成各個 Hooks，因此我們只需要引入 `useAllPostQueryQuery` 就可以使用了。

![](https://i.imgur.com/Q5JjK8h.jpg)

## 使用 Apollo Reactive variables

在專案開發上面不免如一定會用到一些管理全域 State 的套件，例如：Redux、Vuex...等，而 Apollo 框架內建就有支援這個部份，它提供了幾個 function 讓我們可以快速的做到『創建』local state 並且與 React 結合，不再需要安裝一堆套件與寫一堆設定。

#### 基本上我們常會用到這兩個 function

1. **makeVar**
   主要用來『創建』與『更新』reactive variable。(可以當成是建立 local state)

   #### Creating:

   ```typescript=
   // localState.ts
   // 『創建』購買彈窗 並 設定初始值
   export const purchaseModalVar = makeVar<{
       isModalOpen: boolean
       price: number
   }>({
       // initial value
       isModalOpen:false,
       price:0
   })
   ```

   #### Reading:

   上面我們透過 `makeVar` 建立了一個 `purchaseModalVar` 的 reactive variable，而我們可以直接透過 `purchaseModalVar()` 來讀取這個 reactive variable 的資料。

   ```typescript=
   // 『讀取』reactive variable 資料
   console.log(purchaseModalVar())
   ```

   #### Modifying:

   有新增一定就有『修改、更新』，而這個部分一樣直接透過 `makeVar` 建立出來的變數 (`purchaseModalVar`) 就可以直接做更新，我們可以將『新的 value』傳到 reactive variable 的第一個參數中。

   ```typescript=
   // 『更新』reactive variable 資料
   purchaseModalVar({
       isModalOpen: true
       price: 100
   })
   ```

2. **useReactiveVar**
   主要用來與 React 連結，看到 `use` 這個關鍵字相信大家應該就知道『它是一個 Reack hook』，還記得當 Redux 的 state 變更的時後，有使用到該 state 的 component 會重新 rerender，因此畫面也會跟著被更新。而 `useReactiveVar` 就是來讓 `reactive variable` 變更時重新 rerender component。

   > The useReactiveVar hook can be used to read from a reactive variable in a way that allows the React component to re-render if/when the variable is next updated.

   ```typescript=
   import {purchaseModalData} from '@/localState'
   export function ShoppingModal() {
     const purchaseModalData = useReactiveVar(purchaseModalVar);
   }
   ```

## Reference

1. [apollo-link-token-refresh](https://github.com/newsiberian/apollo-link-token-refresh)
2. [[筆記] 透過 JWT 實作驗證機制 - Mike Huang](https://medium.com/%E9%BA%A5%E5%85%8B%E7%9A%84%E5%8D%8A%E8%B7%AF%E5%87%BA%E5%AE%B6%E7%AD%86%E8%A8%98/%E7%AD%86%E8%A8%98-%E9%80%8F%E9%81%8E-jwt-%E5%AF%A6%E4%BD%9C%E9%A9%97%E8%AD%89%E6%A9%9F%E5%88%B6-2e64d72594f8)
3. [JWT 官方線上 Debugger](https://jwt.io/)
4. [jwt-decode](https://github.com/auth0/jwt-decode)
5. [Queries - Apollo](https://www.apollographql.com/docs/react/data/queries#caching-query-results)
6. [Mutations - Apollo](https://www.apollographql.com/docs/react/data/mutations#executing-a-mutation)
7. [GraphQL Code Generator](https://www.the-guild.dev/graphql/codegen/)
8. [Reactive variables - Apollo](https://www.apollographql.com/docs/react/local-state/reactive-variables)
