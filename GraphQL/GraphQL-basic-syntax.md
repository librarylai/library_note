# 【筆記】GraphQL 系列(一) - 基本語法篇

###### tags: `筆記文章`

![](https://i.imgur.com/vPjBcoe.png)

最近因為面試的緣故接觸到了 GraphQL 以及它的相關框架，以前只有聽過它可以讓前端更彈性的拿取想要的資料結構，而不像以往使用 Restful 架構要等到後端回應 Response 後才會知道收到了哪些結構，剛好趁這次機會好好來學習一下 GraphQL 的語法、框架、套件...等相關內容。

> 本篇是直接使用 Next.js 提供的 GraphQL Example 範例下去實作，如果懶的設定環境的朋友可以參考此範例 [Apollo Server and Client Example - Next.js](https://github.com/vercel/next.js/tree/canary/examples/api-routes-apollo-server-and-client)。

> #### 本系列文章列表：
>
> 1.  [【筆記】GraphQL 系列(一) - 基本語法篇](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/HJlKXB0b9)
> 2.  [【筆記】GraphQL 系列(二) - 瞭解 Apollo Client 與 Apollo cache 機制](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/rkJ0kqGQc)
> 3.  [【筆記】GraphQL 系列(三) Scheme-first & Code-first 概念](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/HJv-yX3t9)
> 4.  [【筆記】GraphQL 系列(四) - GraphQL Apollo Testing use react-testing-library](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/S1qcpAIjc)
> 5.  [【筆記】GraphQL 系列(五) - GraphQL 開發雜記、套件](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/BkvyKgo1o)

> #### 本篇內容主要參考以下文件與文章：
>
> - [**GraphQL 官方教學文件**](https://graphql.org/learn/)
> - [**Apollo Server 官方**](https://www.apollographql.com/docs/apollo-server/)
> - [ **_fx777_ 大大的 Think in GraphQL 系列**](https://ithelp.ithome.com.tw/users/20111997/ironman/1878)

**_內容中如有任何錯誤或冒犯的地方還請各位大大們多多提點。_**

## GraphQL 介紹

> GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data.

簡單來說：GraphQL 是一個對於 APIs 而設計出來的資料查詢操作語言，在 runtime 時會使用這些 Queries 去將 存在的 Data 資料查出。

GraphQL 相較於以往 RESTful API 的差別在於擁有各大的靈活性、彈性，以往 RESTful API 的架構下前端要知道後端回應什麼，基本上都要等呼叫完 API 後查看 Response 結構才會得知，當然也可以透過像是 Swagger UI 等工具來查看。

但在使用 GraphQL 的架構下，前端可以使用查詢( Query )語法來指定要拿哪些資料( Field )，因此在呼叫前就可以明確了解整個資料結構以及有哪些欄位，甚至不用像以往取資料時都要拿到整個完整的結構，前端可以依照畫面上的需求更靈活的拿取資料，不用每次要多個欄位就要後端配合調整 API 的 Response，因此大大減少前後端的溝通成本。

#### 以往取得的 Response 結構

```javascript=
{
 User:{
    id: 1,
    name: 'library',
    age: 26,
    sex: 'male',
    posts: [
        {
            title:'今天天氣真好',
            content: '....xxxx'
        },
        {
            title:'好想睡覺',
            content:'....xxx'
        }
    ]
 }
}
```

#### 使用 QraphQL 取得的 Response 結構

```javascript=
/* Query */
query {
    User {
        id
        name
        posts {
            title
        }
    }
}

/* Response */
{
    User:{
        id: 1,
        name: 'library',
        posts:[
            {
                title:'今天天氣真好',
            },
            {
                title:'好想睡覺',
            }
        ]
    }
}
```

## GraphQL 三大架構

GraphQL 的三大架構可以說是：『Schema』、『Resolve』、『Query』，如果簡單形容這三個部分的話，可以說它們分別為：『型別定義』、『後端』、『前端』。

在一開始學習 GraphQL 時常常會不知道要先從哪個部分開始著手，而官方文件則是先從 Query 開始教起，但筆者自己在讀的時候反而會一直去想說『這些資料到底怎麼來的，怎麼填入這幾個 Field 後就拿到資料了』...等等的疑惑，所以這邊打算先從 Schema 型別定義開始說起，再到 Resolve 對每個 Object Type 回傳相對應的資料，最後透過 Query 拿出我們想要的 Field 內容。

## GraphQL Schema

在大部分只聽過 GraphQL 的人中，大概只會知道 QraphQL 就是一個物件然後裡面寫一些想要的 Field name 就會將這些欄位的資料回應給你，像是官方提供的這個例子。

![](https://i.imgur.com/60muQM9.png)
(Reference: https://graphql.org/learn/schema/)

意思大概是：選擇了『hero』這個 Field，並且又從 hero 這個 Object Type 裡面再選擇了『name』、『appearsIn』這兩個 Field。

看到這邊不知您是不是會好奇說：『啊！我要怎麼知道 hero 這個物件裡面有哪些東西是我可以選擇的』，而這就是 Schema 在負責的事，我們透過定義 Schema 來告訴開發者與 QraphQL 說，我們有哪些欄位可以查詢(前端 Query)以及這些欄位需要回應的資料(後端 Resolve)。

### Object Types and Fields

![](https://i.imgur.com/g8MDNmm.png)

在 QraphQL Schema 定義中最常見的兩種 Type 就是 Scalar Type 與 Object Type，**Scalar Type** 可以想像成程式語言的基礎型別(String、Number、Boolean)，而 **Object types** 可以想像成程式語言裡的『物件』，只不過它裡面寫的是各個 field 的型別定義而不是 value，如果對 TypeScript 有點概念的讀者，可以直接想像成定義 [TypeScript - Object Types](https://www.typescriptlang.org/docs/handbook/2/objects.html) 時的 Type Alias 方式。

```javascript=
type User { // ---- Object Type
    id: ID // ---- Scalar Type
    name: String // ---- Scalar Type
    friends: [User]  // ---- Object Type
}
```

#### 上面範例解釋：

1. User 是一個 Object Type，裡面包含了 id、name、friends 這三個 fields。
2. id 與 name 都是 Scalar Type 分別為 『ID』與『String』型別
3. friends 則是 User 這個 Object Type 的陣列型別

---

### Scalar Type

在 QraphQL 裡有五種預設的 Scalar Type 分別為：『Int、Float、String、Boolean、ID』。

> - Int: A signed 32‐bit integer.
> - Float: A signed double-precision floating-point value.
> - String: A UTF‐8 character sequence.
> - Boolean: true or false.
> - ID: The ID scalar type represents a unique identifier, often used to refetch an object or as the key for a cache. The ID type is serialized in the same way as a String; however, defining it as an ID signifies that it is not intended to be human‐readable.

特別要講的是 **ID** 這個 Scalar Type，它代表著一個唯一值，不管是傳 Int 或 String 都可以通過，在實務上主要會傳像是 uuid 等唯一識別碼來當作 ID。

我們也可以去客製化自己的 Scalar Type 並實作這個 Type 的方法，但這有點超過基礎篇的範圍，之後會在找時間實作並分享，如果有興去的讀者可以先看 **_fx777 大大的文章。_** [fx777 - 實作 Custom Scalar Type (Date Scalar Type)](https://ithelp.ithome.com.tw/articles/10206366)

---

### The Query and Mutation types

整個 QraphQL Schema 其實可以說是一個 Object Type，它又被稱為`Root Types`，而 Root Types 裡面又有幾個特殊的 Object Type (ex.『Query』、『Mutation)，它們分別為 Schema 的進入點(entry point)且各自代表著不同的意思。

```typescript=
/*  Schema 定義 */
type Query{
    user: User
}
type Mutation {
    addPost(title: String, content: String, authorId: ID): [Post]
}
```

當我們在前端使用 GraphQL 時，會透過這幾個特殊的 Type 當作進入點去進行操作，像是查詢時我們會用 Query，更新或新增時會用 Mutation。
但其實如果您硬要把 Mutation 裡的內容寫在 Query 內，其實程式也不會有任何問題，這裡感覺比較像是一種規範。

需要注意的是如果將 Mutation 裡的內容寫到 Query 的話，相對應的 Schema 與 Resolve 也要寫在 Query 的區塊內。

```typescript=
/* Query */
query {
    user:{
        id
        name
    }
}
/* Mutation */
mutation AddPost($title: String, $content: String, $authorId: ID) {
    addPost(title: $title, content: $content, authorId: $authorId) {
      id
      title
      content
      author {
        id
        name
      }
    }
  }
```

**以上面的 Query 這個例子來解釋的話步驟為：**

1. GraphQL service 會去 Schema 中尋找 Query 這個 obejct type
2. 找到 user 這個 field(obejct type)
3. 然後再去找 user 裡面的 id 與 name 這兩個 field

---

### Arguments & Variables

![](https://i.imgur.com/eqIXufM.png)

在 QraphQL Schema 定義中，我們也可以對每一個 Field 傳入參數(Arguments)，不管是 Object field 或是 Scalar field 都可以傳入參數。

在 QraphQL Query 查詢定義方面，如果想要傳入參數到 Query 中，則可以使用 `$ + 變數名稱(Variables Name)`的方式在 Query 中增加參數宣告，且如果想將 Variables 傳給 field 的 Arguments，則可以在該 fields 後面加上 `(Argument name : Variables name)`。

```typescript=
/* Schema */
type User {
    id: ID!
    name: String!
}
type Query{
    user(id: ID!): User
}

/* Query */
query getUser($id: ID = 1){
    user(id: $id){
        name
    }
}
```

---

### Enumeration types

上面我們講了 Object Type 與 Scalar Type 這兩個型別，現在要再講一個 Enumeration Types，它跟我們在其他程式語言中的 Enum 很類似，但它主要是寫在 Schema 定義中，用來代表限制該欄位只能出現 Enumeration Types 裡面的值(自動轉為 String 型別)。

```typescript=
/* Enumeration types */
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}
/* Schema */
type query{
   subject: Episode
}
```

---

### Lists and Non-Null

在我們定義 Schema 時其實還有一個特別的關鍵字『**!**』，它代表著 Non-Null (不可為 null) 的意思，常常被用來當作『**必填**』使用，例如我們在定義名稱時如果要欄位為必填項目時，則可以寫成 `name: String!` 的方式，以下簡單舉幾個例子：

```typescript=
type query{
    name: String! // 必填字串
    postA: [Post!] // array 裡面為 Post Type 且不可為 null (ex. null, [], [{id:}])
    postB: [Post]! // postB 欄位本身不能為 null，但陣列裡面可以為 null (ex. [] ,[{id:},null])
    postC: [Post!]! // array 裡面為 Post Type 且不可為 null，且 postC 欄位本身不能為 null (ex. [], [{id:}] )

}
```

另外 Non-Null 也可以使用在 Query 查詢資料時的參數 Variables 上面，用來限制參數的傳入值不可為 null 。

```typescript=
/* 前端 Query 查詢 */
query viewPost($id: ID!){
    post(id: $id){
        title
    }
}
```

---

### Interfaces

> An interface specifies a set of fields that multiple object types can include

GraphQL 跟很多 type systems 一樣也有支援 `interfaces` 的功能，在定義 Schema 的時候如果我們想讓不同的 Object Type 都共享某些 fields 時，則可以用 interface 來將共同的部分抓取出來，之後在各別實作(implementation)自己的 Object Type。

#### 簡單舉例：

當今天資料可能在【不同頁面】或是【不同功能】會不太一樣時，這代表所指的類別就也會不同。

例如：八卦版頁面 Post type 裡面的 User type 指的是 Human，動物版頁面 Post type 裡面的 User type 指的是 Animals。

而這時就可以將 User type 裡面共用的 fields 提出來成 `interface`，並分別做出 Human 與 Animals 這兩個 type，最後再依照流進 user 這個 `interface` 的資料進行分類(ex.有 hairColor 就是 Human type)。

![](https://i.imgur.com/dG0UOJG.png)

```typescript=
/*
 * 原本 user 的 type 只設定為 Human
 * 但...當今天 user 資料可能在不同頁面或是不同情況時會 變成 Humna , Animals, God 等不同資料類型，
 * 需要依造資料的不同去指定給它們不同的 type 時，
 * 就可以用 interface 先將共同的部分取出(Character)，再各自實做
 * */
type query{
    post: Post
}
type User {
    user: Human
}
type Post {
    user: User
}
/* ---------------------------- */
/* 使用 interface */
type query{
   post: Post
}
type User {
    user: Character
}
type Pose {
    user: User
}
/* 定義一個 角色 的 Interface */
interface Character {
  id: ID!
  name: String!
}

/* 實作 Human  這個 type */
type Human implements Character{
    id: ID!
    name: String!
    hairColor: String!
}

/* 實作 Animals  這個 type */
type Animals implements Character{
    id: ID!
    name: String!
    shape: String!
}

/* Query 查詢資料 */
query viewPostUser{
    post{
        id
        name
        ... on Human{ // 這邊用到 query 的 Inline Fragments 寫法
           hairColor
        }
        ... on Animals{ // 這邊用到 query 的 Inline Fragments 寫法
           shape
        }
    }
}

/* resolvers */
const resolvers = {
  SearchResult: {
    __resolveType(user, context, info){
      // 如果 user 資料裡有顏色相關資料,則判斷為 Human tpye
      if(user.color){
        return 'Human';
      }
      // 如果 user 資料裡有形狀相關資料,則判斷為 Animals tpye
      if(user.shape){
        return 'Animals';
      }
      return null; // GraphQLError is thrown
    },
  },
  Query: {
    post: () => { ... }
  },
};
```

上面範例的這段 `... on Human` 是 GraphQL 內`Inline Fragments`的查詢方式，這個方法主要是用在 interfaces 或 union types 上面。

像上面的例子 `user` fileds 回傳的是一個 `Character` 的 interface，而如果我們要取得 Human 與 Animals 這兩個 implements 裡的 fields 的話，則要透過 `Inline Fragments` 的寫法以及根據 `user` fileds 回傳的 type 是 Human 還是 Animals，如果是 Human type 則會去查詢 `hairColor` 這個欄位，反之則是 `shape` 欄位。

詳細內容可參考：[Unions and interfaces - Apollo Server](https://www.apollographql.com/docs/apollo-server/schema/unions-interfaces/)

---

### Union Types

> Unions and interfaces are abstract GraphQL types that enable a schema field to return one of multiple object types.

Union Types 在實作上與 Interface 的方法大致相同，在 Query 與 Resolvers 的實作方面都一樣使用 `Inline Fragments` 與 `__resolveType` 這兩個方法，差別在於 union types 的宣告是使用 `union xxx` 開頭，且 union types 裡的 type 【不必】有共通的 fields，而 interface 的 implements 則是要強制包含該 interface 的 fields。

#### 簡單舉例：

```typescript=
union Environment = Tree | Sea

type Tree {
  title: String!
}

type Sea {
  name: String!
}

type Query {
  search(contains: String): [Environment!]
}

/* Query */
query GetSearchResults {
  search() {
    __typename // 以下主要介紹功能
    ... on Tree {
      title
    }
    ... on Sea {
      name
    }
  }
}

```

這代表 search 這個 field 的陣列裡面，包含了 Tree 或 Sea 這兩個類型的資料，union types 跟 interface 一樣最後都要回傳其中一個 type 的類型。

至於 Query 與 Resolvers 的實作因為跟 Interfaces 一樣所以這邊就不再提了，這邊主要介紹 `__typename` 的功能。

#### The \_\_typename field

當每個 Object Type 在 Schema 中時，會自動產生一個叫 `__typename` 的 field，而 `__typename` 這個 field 會是 Object Type 的名稱(String 類型)，舉例來說：

```typescript=
type Author = {
    id: ID
    name: String
}
// Author 的 __typename 為 'Author' 字串
__typename => 'Author'

```

`__typename` 可以幫助我們在 Query 的回傳內容中知道是哪個 type 所回傳的資料，也可以用來在 `caching results` 方面。

以剛剛上面的 GetSearchResults 為例：

```typescript=
/* Response */
{
  "data": {
    "search": [
      {
        "__typename": "Tree",
        "title": "我是 Title"
      },
      {
        "__typename": "Sea",
        "name": "SEA~~~"
      }
    ]
  }
}
```

## GraphQL Resolves

> A resolver is a function that's responsible for populating the data for a single field in your schema.

上面大致了解 GraphQL 如何寫 Schema、如何定義 Object Type、如何傳入參數 Argument 後，現在可以進入到 GraphQL Resolvers 的部分了，透過 Resolver function 來將資料填入到 Schema 中對應的 fields 內，讓我們直接透過案例來一步一步了解。

#### 案例模擬

透過 [Apollo Server 官方 - Resolver](https://www.apollographql.com/docs/apollo-server/data/resolvers/) 的例子來模擬整個流程(Schema -> Resolvers -> Query)

### 撰寫 Schema：

首先，假設我們有一組 Schema 為：

```typescript=
type User{
    id: ID
    name: String
}
type Query {
    user(id: ID!): User
}
```

可以看到進入點 Query 有一個 `user` 的 field 且需傳入 `id` 參數(Argument)，回傳的資料型態為 `User Object Type`。

`User Object Type` 內又有兩個 fields 分別為 `id` 與 `name`。

### 依照 Schema 撰寫 Resolver

依照上面 Schema Query 中 user 的定義為 User Object，這邊可以理解成 resolver function 的回傳值要是一筆 User 物件(不是陣列)，因此可以看到下面最後是使用 `find` 來回傳 `user.id` 與傳入參數相同的那筆資料。

而 User Type 裡面又有 name field，因此還要再寫 `User:{name()}` 的 resolver function....以此類推，每個 field 都可以依照你的需求來撰寫該 field 的 resolver function。

> 注意：如果 field 沒有定義 resolver function，Apollo Server 預設會用 field name 去自動與 Data 的 key 做 mapping 撈出資料。

```typescript=
/* 假資料 */
const users = [
  {
    id: '1',
    name: 'Elizabeth Bennet'
  },
  {
    id: '2',
    name: 'Fitzwilliam Darcy'
  }
];

/* resolvers */
const resolvers = {
  Query: {
    user(root, args, context, info) { // parent === root
      return users.find(user => user.id === args.id);
    }
  },
  User: {
    name(parent,args,context){
        // parent 就是 user 的 return value
        return parent.name
        /*
         * 這邊其實可以不用 return parent.name
         * 因為 Apollo Server 會自動使用 field name 去撈資料 (parent[fieldName])
        */
    }
  }

}

```

### 前端查詢 Query

上面我們寫完 Resolvers 後，現在就只差前端透過 Query 將資料查詢出來。

#### 假設我們要查詢 user 的 name

```typescript=
query{ // 進入點
    user{ // user field
        name // name field ( name 存在於 user object type )
    }
}
```

這時 Apollo Server 就會一層一層的去將資料查詢出來，首先會進到 `user resolver` 中，接著再進到 `name resolver` 內將資料 return 回來，這時前端就可以拿到所查詢的資料了。

![](https://i.imgur.com/DqApA96.png)

> **以上範例其實就是整個 GraphQL 的最簡單流程!!!**

### Resolvers 注意事項與參數介紹

1. **在撰寫 QraphQL Resolvers 時需要注意『名稱一定要對到 Schema 中 field 的名稱以及 type 的名稱』**

   ```typescript=
   type Author{
       name
   }
   type User{
       author:Author
   }
   type query{
       user(id: ID!):User
   }

   const resolver = {
       Query: {// 需對應 type query
           user(root, args, context, info){} // 需對應 Query -> user field
       },
       User:{ // 需對應 type User
           author(parent, args, context, info){} // 需對應 User -> author field
       }
   }
   /* Query */
   query {
       user{
           author{
               name
           }
       }
   }
   ```

2. **關於 resolver function 的參數 `(parent, args, context, info)` 各自代表的意思**

   ![](https://i.imgur.com/T409Na4.png)

- **Parent：**
  `Parent` 主要代表上一個 Resolve function 所回傳的資料，以上面的例子來說：今天資料是從 Query 的 user field resolver 接收到後再傳給 User 的 author field resolver，所以 author 這裡的 `parent` 指的是 user 的回傳值。

       >user field resolver 因為 Query 已經是最外層的 field，所以 parent 其實也就等於是 `root`，而 `root` 的值是在初始化 Apollo Server 時可以預設的初始值(rootValue 預設為 {})。

  ![](https://i.imgur.com/TiEbF2Z.png)

- **args：**
  `args` 就是我們傳進去的 Arguments，還記得上面範例中在定義 Schema 時我們 user field 是能夠傳入參數(`user(id: ID!):User`)，因此當我們執行 Query 傳入參數時(`query{ user(id:"4") }`)，則 user filed resolver 的 `args` 就會收到傳進來的參數結構(ex.`{id: "4"}`)。

- **context：**
  `context` 跟 `root` 一樣是在初始化 Apollo Server 時可以設定的值，`context`的特點是『它會出現在每一個 resolver function 中』，像是我們可以把要對 dataBase 操作的物件，在初始化 Apollo Server 時就放進到 context 中，這樣在每個 resolver function 內就可以透過 `context` 這個參數去拿到 dataBase 的操作物件。

  ![](https://i.imgur.com/6c4SCzL.png)

- **info：**
  `info` 則是會顯示當前被觸發的這個 field 的資訊，像是它的 `fieldName`、`path` 等資訊。
  ![](https://i.imgur.com/98eMazp.png)

## GraphQL Query

如果是從上面一步一步看到現在的讀者，應該已經對 Query 的寫法有些許的概念了，不外乎就是依照 Schema 一層一層的寫出要查詢的資料結構，然後碰到要傳參數(Argument)的 field 就透過 Variables 傳入，基本上這就是最基本也是最常用的 Query 寫法。

```typescript=
/* 常用 Query 範例 */
query getUser($id: ID){
    user(id: $id){
        name
    }
}
```

接下來會再介紹幾個在寫 Query 時也會使用的(ex. Fragments、Operation Name、Aliases)。

![](https://i.imgur.com/DQJYFcy.png)

### Aliases

> An alternative name provided for a query field to avoid conflicts during data fetching. Use an alias if a query fetches multiple instances of the same field, as shown:

當我們在寫 Query 的時候，如果要對同一個 field Name 抓取多次資料時，可以透過 Aliases 來避免撞名的困擾。

![](https://i.imgur.com/xir3YwM.png)

根據官方範例，當今天一次要對 users 這個 field 依照不同的參數來查詢資料時，因為 GraphQL 不能同時出現相同名稱，所以分別將這兩個 fields 取別名(Alias)為 `admins` 與 `managers`。

---

### Fragments

我們再來看一下上面 Aliases 的範例圖，可以發現 `admins` 跟 `managers` 都要查詢相同欄位，這時我們就可以透過 `Fragment` 這個關鍵字將重複的部分抽取出來，這樣可以使程式碼變得更簡潔，也能更方便的在 Query、Mutation 中重複利用。

> Fragment 的使用方式跟 Javascript 的 [spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) 一樣，在 Fragment 名稱前面使用 `...` 前墜。

![](https://i.imgur.com/qJPajBJ.png)

#### Fragment 實際範例：

前端可以使用官方所提供的 Apollo Client 套件來與 GraphQL Server 互動，像是使用 Apollo Client 提供的 `gql(template literal tag)` 方法，我們可以在方法裡面寫 GraphQL 語法來與 GraphQL Server 互動。

**Apollo Client 開發使用 Fragement 範例**：[Fragement Example-usage](https://www.apollographql.com/docs/react/data/fragments/#example-usage)

---

### Operation Name

我們再來看一下上面範例圖 query 後面的 `AdminsAndManagers` 這個名稱，它就是`Operation Name`，以往我們常用`query{}`的這種寫法其實容易增加 debug 時的難度，且當我們一次執行多筆操作時也無法直觀的知道是哪個 Operator 出現問題。

因此建議在每個 Query, Mutation, or Subscription 後面都加上`Operation Name`來增加程式碼的可讀性與減少 debug 時間。

![](https://i.imgur.com/DbOODIu.png)

## GraphQL Mutations

大部分我們在使用或是討論 GraphQL 時，都是圍繞在資料查詢的部分，如果是從上面看下來的讀者應該會發現，本篇文章到目前為止也是圍繞在如何取得資料，但實務上有『取得』就一定會有『新增、修改、刪除』，而 GraphQL 也有提供一個 `Mutation` 的關鍵字來表示『這是一個會對資料進行更動的 function』，就像 RESTful 架構中我們一看到 `Get` 就知道是要『取得資料』，一看到 `POST` 就知道這是要『更動資料』，而 GraphQL 的 `Query`、`Mutation` 就跟 `Get`、`POST` 是同一個概念。

> 不知道還記不記得，上面在介紹 [The-Query-and-Mutation-type](https://hackmd.io/K8hPjAhORSi_OCUg5nZKIA?view#The-Query-and-Mutation-types) 時有提到 `Query` 跟 `Mutation` 主要是一種規範，如果硬要將 `Mutation` 的內容寫到 `Query` 程式也是能正常運作。

#### 下面我們就用一個『新增 Post』的例子來看 GraphQL Mutation 該如何寫，步驟如下：

> 1.  首先，不外乎當然是要先定義一個 Mutation 的 Schema
> 2.  接著，開始寫 Mutation 的 Resolver function
> 3.  最後，前端透過 mutation 關鍵字來告訴 GraphQL 麻煩幫我執行 Mutation 內的 Resolver function

### 首先，不外乎當然是要先定義一個 Mutation 的 Schema

我們在 `Mutation` 內定義了一個 `addPost` 的 field，且它接收三個參數`title`、`content`、`authorId`，最後回傳 `Post Object Type` 這個類型的陣列。

```typescript=
type Post {
  id: ID!
  title: String! // 文章標題
  content: String! // 文章內容
  author: User // 作者 -> User Object Type
}

type Mutation {
  addPost(title: String, content: String, authorId: ID): [Post]
  deletePost(postId: ID): [Post]
  updatePost(postId: ID, title: String, content: String, authorId: ID): [Post]
}

```

### 接著，開始寫 Mutation 的 Resolver function

這邊要注意我們在寫 Resolver function 時要寫在 `Mutation` 的 key 裡面，而不是寫在 `Query` 裡面，因為我們在 Schema 定義時是寫 Mutation 而不是 Query，且名稱也要與 Schema 中的 `field name` 對到，不然 GraphQL Server 會找不到那個 resolver function。

```typescript=
const resolvers = {
  Query: {/* Query 相關的 Resolver Functions */},
  /* 注意！因為 Schema 的 type 為 Mutation 因此這邊不是寫在 Query 內喔 */
  Mutation: {
    addPost: async (root, arg, context) => {
      try {
        const { title, content, authorId } = arg
        const params = {
          id: shortid.generate(), // 產生一個這篇文章的 uuid
          title, // 文章標題
          content, // 文章內容
          authorId, // 作者 id
        }
        // 塞進 mongoblogDB
        await context.blogDB.insertOne(params)
        return await context.blogDB.find().toArray()
      } catch (error) {
        throw new ApolloError(error)
      }
    },
    /* deletePost 省略～ */
    /* updatePost 省略～ */
  },
}

```

### 最後，前端透過 mutation 關鍵字來告訴 GraphQL 麻煩幫我執行 Mutation 內的 Resolver function

這邊其實就跟 query 的寫法是一樣的，只要 keyword 換成了 mutation 就可以了，因為我們的定義那些都是寫在 mutation 中。

最後這段程式碼是模擬在前端使用 Apollo Client 套件來執行 GraphQL，使用了 Apollo Client 的 `gql` 與 `useMutation` hook 並模擬透過 `Button` 點擊來達成『新增文章』功能。

```typescript=
const ADD_POST_QUERY = gql`
  mutation AddPost($title: String, $content: String, $authorId: ID) {
    addPost(title: $title, content: $content, authorId: $authorId) {
      id
      title
      content
      author {
        id
        name
      }
    }
  }
`
// 前端使用 Apollo Client 提供的 useMutation hooks
const [ addPost ] = useMutation(ADD_POST_QUERY)

// ex. Button 點擊觸發新增
addPost({ variables: { title: '我是 title', content: '我是 content', authorId: 1 } })

```

## 結語

這次剛好因為面試的緣故接觸到了 GraphQL，在過程中雖然成功完成了整個需求，但對語法以及整個流程還是矇矇懂懂，因此打算好好瞭解一下順便將讀的過程筆記下來，本篇主要是 focus 在 GraphQL 的語法以及有哪些東西可以使用，所以沒有特別去提到要如何設定環境，如果有想要玩看看 GraphQL 的讀者可以直接到 [codesandbox](https://codesandbox.io/) 上選擇 Apollo Server 來嘗試。

![](https://i.imgur.com/ey1TITc.png)

至於實作與環境架設的部分預計會在下一篇文章中分享，也會順便介紹一下 Apollo Client 裡面的 hooks 以及 function 等，還請各位讀者拭目以待瞜~~~

#### 以上就是這次【 GraphQL 語法篇 】的全部內容，希望對想了解 Ｇ raphQL 的人能有一點點幫助，如有任何錯誤或冒犯的地方還請各位多多指教，謝謝您的觀看。

#### GitHub : [https://github.com/librarylai/GraphQL-Blog](https://github.com/librarylai/GraphQL-Blog)

## Reference

1. [GraphQL 官方](https://graphql.org/learn/)
2. [Apollo Server 官方](https://www.apollographql.com/docs/apollo-server)
3. [Think in GraphQL 系列 - fx777](https://ithelp.ithome.com.tw/users/20111997/ironman/1878)
