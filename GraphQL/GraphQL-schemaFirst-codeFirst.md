# 【筆記】GraphQL Scheme-first & Code-first

###### tags: `筆記文章`

![](https://i.imgur.com/lqJL4Jq.png)

我們都知道 `GraphQL is a query language for APIs`，它的優勢在於可以讓 Client 端自行決定要用哪些資料，不需要後端額外開 API 來處理。

舉例來說：今天網頁(Web)畫面上需要 `name`、`title`、`content` 這三個欄位的資料，而手機(App)是需要 `title`,`description` 這兩個欄位的資料。

在以往 RESTful 的架構下面可能需要額外開一隻 API 給手機端使用，但在 GraphQL 的架構下面只需要從 Schema 中選取手機端所需要的欄位來使用即可，因此不僅提升了前端在開發上的速度，也減少了一些後端額外開發的需求。

## Scheme-first & Code-first

當我們要啟動一個新的專案或是做一個新的服務時，我們可能會在一開始就先規劃整個 GraphQL Service 要選擇哪一種模式進行開發，而目前最常被討論的就是 `scheme-first` 與 `code-first` 這兩種開發方式，這兩種概念簡單來說就是：

> - schema-first：是透過寫 GraphQL Schema Language 方式來開發，就像是官方教學的那樣。
> - code-first：是直接在原本的程式語言中透過一些方法來轉換成 GraphQL schema 、 resolvers 等 。

而不管選擇哪種方式，最後一樣都會是一個完整、功能齊全的 GraphQL Service，它們差別在於架構與攥寫方式的不同。

![](https://i.imgur.com/6wwfEcP.png)

那在開始探討 `scheme-first` 與 `code-first` 之前，我們先來了解一下整個 GraphQL 的發展過程與每個階段碰到問題。

## GraphQL 發展 - GraphQL Release

![](https://i.imgur.com/m2W0rRg.png)

### GraphQL Release

我們都知道 GraphQL 現在支援很多種語言，而對於 Javascript 來說 GraphQL 最核心的套件就是 [graphql-js](https://github.com/graphql/graphql-js)，它也是在當年 GraphQL Release 時官方為 Javascript 所提供的 GraphQL 套件。

我們可以從 [GraphQL Code Libraries(Server / Client / Tools)](https://graphql.org/code/#javascript) 上面觀察到，基本上很多的套件都是基於`graphql-js` 為核心下去開發的，像是`apollo-server`、`graphql-yoga` 都是。

![](https://i.imgur.com/6tgXpjM.png)

而因為當時 GraphQL 的相關套件還很稀少，因此主要都是直接使用 `graphql-js` 來開發。

#### graphql-js

我們可以從 `graphql-js` Github 的範例上面看到如何建立一個 Schema，以這邊的例子來說，創建了一個名為 `RootQueryType` 的 type，裡面有 `hello` 欄位且 `resolve function` 會回傳 `world` 字串。

我們可以從這段程式碼中發現到全部的內容都是寫在一起的，像是從欄位的型別定義(`type`)，到欄位的實作(`resolve function`)都是一層一層包在物件裡面，整體來說的可讀性非常低也很冗長。

因此開始有一些套件出來解決這個問題，也使 GraphQL 進到了下個階段 **『schema-first』**。

```javascript=
import {
  graphql,
  GraphQLSchema,
  GraphQLObjectType,
  GraphQLString,
} from 'graphql';

var schema = new GraphQLSchema({
  query: new GraphQLObjectType({
    name: 'RootQueryType',
    fields: {
      hello: {
        type: GraphQLString,
        resolve() {
          return 'world';
        },
      },
    },
  }),
});
```

> 補充：`graphql-js` 其實就是一種 code-first 開發模式，可以說是 code-first 的原型。

## GraphQL 發展 - schema-first

上面提到 `graphql-js` 的寫法太難以閱讀，且要使用套件提供的方式來定義型別(ex.`GraphQLString`)，無法直接使用 **GraphQL Schema Language** 的方式來定義 Schema。

因此出現了 [graphql-tools](https://github.com/ardatan/graphql-tools) 這套 Library，這邊簡單整理了一下 `graphql-tools` 主要的優點：

> 1.  Use the GraphQL schema language to generate a schema with full support for resolvers, interfaces, unions, and custom scalars. The schema produced is completely compatible with GraphQL.js.
>     (使用 GraphQL schema language 的方式建立 Schema)
>
> 2.  Mock your GraphQL API with fine-grained per-type mocking
>     (快速建立 Mock Data 來測試 UI，不用苦苦等待後端)
>
> 3.  Automatically stitch multiple schemas together into one larger API
>     (可以將多個的 schemas 結合在一起，合成一個大的 Schema)
>
> 4.  Separate business logic from the schema
>     (將商業邏輯與 schema 分開，也就是將 resolve function 與 schema 拆開。)

### graphql-tools

現在我們可以透過 `graphql-tools` 來將 schema 與 resolver 分開來寫，並且最後透過 `makeExecutableSchema` 方法來將 schema 與 resolver 綁定在一起。

因此剛剛上面的範例可以改成以下方式：

```javascript=
/* 定義 schema  */
const typeDefs = `
    type Query {
      hello: String
    }
`
/* 依照 Schema 攥寫商業邏輯 resolvers */
const resolvers = {
   Query: {
    hello: (_, args) => `world`,
  },
}
/* 引入 makeExecutableSchema */
import { makeExecutableSchema } from '@graphql-tools/schema'

/* conbine schema and resolvers */
const executableSchema = makeExecutableSchema({
  typeDefs,
  resolvers
})
```

而這種將定義 (schema) 與邏輯 (resolver) 分開的方式，也慢慢成為現在或是當時較為流行的開發方式 **schema-first**，也被稱作為 **SDL-first** (Schema Definition Language - first) 方式。

以下節錄自 [The Problems of "Schema-First" GraphQL Server Development - Nikolas Burk](https://www.prisma.io/blog/the-problems-of-schema-first-graphql-development-x1mn4cb0tyl3) 整理的 **schema-first** 開發優點：

> 1.  The approach is easy to understand and great for building things quickly
>     （能夠簡單且快速的理解或建立事物。）
>
> 2.  As every new API operation first needs to be manifested in the schema definition, GraphQL schema design is not an after-thought
>     （每當有新的 API 要開發時，都必須先定義 schema。）
>
> 3.  The schema definition can serve as API documentation
>     (Schema 定義可以作為 API 文件。)
>
> 4.  The schema definition can serve as a communication tool between frontend and backend teams — frontend developers are getting empowered and more involved in the API design
>     (透過 schema 能夠讓前後端在溝通上面更順暢，也能讓前端成員更了解整體架構並且參與 API 設計。)
>
> 5.  The schema definition enables quick mocking of an API
>     (scheam 定義出來，因此也能夠快速的先做一些 mock data。)

schema-first 的方式讓 GraphQL 變得更好攥寫也更容易讓人看懂，甚至是連不是工程師的人也能透過 Schema 來了解整個產品的走向。

而 schema-first 並不是全然只有優點，它在開發上也存在了一些問題，因此在 2017/2018 年就出現了許多套件來為了解決這些問題。

## GraphQL 發展 - schema-first workaround

當專案規模越來越大、越來越複雜的時候，不免俗就會開始碰到一些問題或是察覺到哪些部分可以更優化，因此開始有許多套件被開發出來解決或是改善一些 `schema-first` 開發上或碰到的問題。

### 1. schema 與 resolvers 不一致

在`schema-first` 的開發模式下 `schema` 必須與 `resolvers` 匹配到，因此在開發方面就需要確保 `schema` 與 `resolvers` 必須『同步』的調整。

> 補充：基本上如果有對欄位寫 `resolver` 的話就需要在 `schema` 中定義該欄位，但如果是在 `schema` 中定義欄位，就不見得要對該欄位實作`resolver`，預設是會自動將 Response 依照欄位名稱來回傳。

我們可以拿上面的範例來看，假設我們在 schema 的 `query` type 裡面定義了 `hello` 欄位，則也需要在 resolvers 的 `query` 裡面加上 `hello` 的 `resolve function`。

```javascript=

const typeDefs = `
    type Query {
      hello: String
    }
`
const resolvers = {
   Query: {
    hello: (_, args) => `world`,
  },
}
```

當今天規模越來越大時，每當有一個 schema 欄位變更都很容易發生不同步的情況，像是『打錯名字』、『忘記更新』或是『遺漏某部分』等都可能造成問題。

而這類問題不僅僅是在 Server 端的開發上，甚至連前端的開發上也都可能會受到影響，尤其是當前端是使用 TypeScript 做開發的話更容易碰到這些問題。

#### 前端情況

簡單舉例一下：**前端(TypeScript)**自己寫了一個 `Person` type 來對應**後端的 schema Person type**，而當今天 schema 增加了一個 `friends` 欄位時且前端並『不知情』或是『忘記同步調整』的話，可能就會發生某個 Response 裡面 `person` 欄位的型別不一致的問題，這也就造成了前後端的不同步的問題，或者像是 `enum` type 增加了一個項目時，可能也會發生 `TypeScript` 錯誤等問題。

#### 後端情況

如果是在後端開發上面，當 `schema` 與 `resolvers` 沒對應到時，就會無法產生預期的結果，舉例來說：schema 欄位裡取叫 `name` 而在 resolvers 中卻寫成了 `name(s)`，那當 GraphQL 在執行時就會找不到 `name` 的 `resolver function`(因為寫成 `names` 了)，因此無法產生想要的結果。

#### 解決方案

可以透過 [graphql-code-generator](https://github.com/dotansimha/graphql-code-generator) 這個套件來一次解決這個情況。

##### 前端：

在前端方面，我們可以透過 `graphql-code-generator` 套件來依照 GraphQL 的 Schema 直接產生出對應的 TypeScript 檔案，因此前端就可以直接 `import` 檔案內的 `type` 或是 `enum` 來使用，就不需要再自行寫一些型別定義了，也解決了前後端不一致的問題。

![](https://i.imgur.com/2r1Vi1h.png)

##### 後端：

在後端方面，當透過 `graphql-code-generator` 打包產生文件時，如果發生 schema 與 resolver 不一致的情況時基本上會在 `build` 的時後發生錯誤，像是 `resolver` 『增加』或『調整』了欄位名稱，但 `schema` 遺漏時，則會因為找不到對應的 `schema` 欄位而發生錯誤。

![](https://i.imgur.com/z6wMgOJ.png)

因此透過 `graphql-code-generator` 就可以減少一些不一致的問題，當然它還有一些其他好處，像是產生 React Hooks Function、欄位同步更新...等優點。

### 2. Modularization of GraphQL schemas（模組化 GraphQL schemas）

這邊主要在講述說，以往我們可能都是將所有的 Schema 與 Resolvers 都各別寫在一隻檔案中，然而當專案規模越來越大時勢必就需要將 Schema 與 Resolvers 分成一個一個小部分，可能是依照功能區分或是依照模組區分，例如將 Schema 分成：`blog.gql`、`notification.gql`...等檔案，最後再透過 [graphql-import](https://www.graphql-tools.com/docs/migration/migration-from-import) 或是 [graphql-modules](https://www.graphql-modules.com/docs/get-started) 等套件來將分散的檔案組合起來。

> 補充 1：`graphql-import` 目前已經被包含在`graphql-tools`套件裡面，因此不需要額外安裝。
>
> 補充 2：`graphql-modules` 可以將相關的 `schema` 與 `resolver` 直接封裝能一個 `module` 最後再將所有的 `module` 組合成一個完整個 application schema 。

```javascript=
/* 官方範例： */
/* 用 ApolloServer 來舉例使用  graphql-modules */

import { ApolloServer } from 'apollo-server'
import { createModule, gql } from 'graphql-modules'

export const myModule = createModule({
  id: 'my-module',
  dirname: __dirname,
  typeDefs: [
    gql`
      type Query {
        hello: String!
      }
    `
  ],
  resolvers: {
    Query: {
      hello: () => 'world'
    }
  }
})
// This is your application, it contains your GraphQL schema and the implementation of it.
const application = createApplication({
  modules: [myModule,.......] // 放入其他 modules
})

const executor = application.createApolloExecutor()
const schema = application.schema

const server = new ApolloServer({
  schema,
  executor
})

```

### 3. GraphQL 開發體驗不佳 ＆ IDE 支援不足

在使用 GraphQL 開發的過程中，如果沒有在 IDE 裝一些 GraphQL 的相關套件的話，那開發過程會變得非常痛苦，如果是使用 JavaScript 開發 GraphQL 的話一定會用到 `gql` 這個方法，寫法大致如下：

```javascript=
const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    status: String!
    posts: [Post]
    notifications: [Notification]
    sortNotifications: [Notification]
  }`
```

如果是沒有在 IDE 上安裝 GraphQL 套件的話，基本上看到的就是上面這個樣子，通通都會以字串的樣式顯示，非常難閱讀也很難除錯。

如果是使用 VSCode 開發的話，則強烈建議可以安裝 [GraphQL](https://marketplace.visualstudio.com/items?itemName=GraphQL.vscode-graphql) 這個套件，裝完後不僅看到的畫面舒服許多，它也能解析出 `.graphql`, `.gql` file 以及 `gql` tag，且提供我們 autocomplete suggestions 與 validation against schema 等功能。

![](https://i.imgur.com/aW8caik.png)

### 小結 - Schema-first

這邊先來節錄 [Tomek Poniatowicz](https://blog.graphqleditor.com/graphql-schemafirst-codefirst) 對 Schema-first 的優缺點分析：

![](https://i.imgur.com/GQfxUs7.png)

#### 優點：

1. 大多數的情況下，能夠設計出良好的 API 規格(因為是從整個專案的 Schema 為出發點開始設計，像是要哪些類別、哪些欄位...等

2. 遵循依賴倒轉原則(DIP)。
   依賴倒轉原則簡單來說就是將『定義』與『使用』分開，依照每一個定義出來的類別再各別實作其內容，而在 GraphQL 的情況剛好就分別為『Schema』與『Resolver』。

   > 關於『依賴倒轉原則』推薦閱讀：[[CK Patt 設計模式#6] 依賴倒轉原則（DIP）- CK Patt](https://medium.com/@ckpattern35/ck-patt-%E8%A8%AD%E8%A8%88%E6%A8%A1%E5%BC%8F-6-%E4%BE%9D%E8%B3%B4%E5%80%92%E8%BD%89%E5%8E%9F%E5%89%87-dip-e2ed77a0844)

3. 整個團隊(前端、後端、PM)可以在開始開發 API 之前，就可以先對整體架構討論出所需要的 Schema 欄位，因此在 Schema 定義下來的情況下就可以前後『同步』進行開發，減少開發時間。
   > 關於這點，Code-first 開發模式也能夠做到事前討論出所需要的 Schema 欄位。
   >
   > 因此我覺得 Schema-first 模式下的優點在於『可以直接透過 Schema 就能了解整個產品的架構』，甚至連不是工程師背景的人都能透過瞬間理解。

#### 缺點：

1. schema 與 resolvers 不同步時會導致出錯。 (workaround 第一點)

2. 可能會導致代碼冗餘，因為 SDL 定義不容易重用。

3. 不好將分散的 schemas 整合成一個 schemas（需要借助第三方套件）(workaround 第二點)

## GraphQL 發展 - Code-first Framework

在 **schema-first** 的架構下，當專案規模越來越大時，可能 schema 跟 resolvers 都已經在不知不覺中超過了『一萬』行程式碼，而當我們在定義 schema 裡的欄位時『不見得』會需要寫對應的 resolver。
因此假設過了一陣子之後，當有一天想找某個欄位的 resolver 時就會變得很難找尋，因為該欄位不見得有實作 resolver function。

而大多數 **schema-first** 碰到的問題大都來自於我們需要手動寫 SDL schema，比如說：我們在開發過程中時常需要切換攥寫兩種語法，一種『programming language』 (程式語言 ex.JavaScript、PHP、Ruby)，另一種是『GraphQL Schema Definition Language』，因此需要借助一些第三方套件來讓 SDL 能夠支援開發用的程式語言。

正因為這樣，開始有人在思考是否可以直接都使用『programming language』來攥寫，最後直接產生出 schema 呢？這也是 code-first 被發想的初衷。

#### 再來整理一下 schema-first 被提到的缺點

1. schema 與 resolver 不一致時導致無法返回預期結果。
2. schema 與 resolver 龐大時不好尋找 resolver functio。
3. 需要時常切換不同語言攥寫或是要定義不同語言用的 Type ex.TypeScript type、Schema Definition Language、mongoose schemas。

### Code-first Framework/Library - Nexus

![](https://i.imgur.com/VVm7Ex0.png)

[GraphQL Nexus](https://nexusjs.org/) 是一套專門給 JavaScript/TypeScript 的 Code-first 套件，它的特點在於可以直接將 schema 與 resolver 寫在『同一個地方』，並且可以直接用 JavaScript or TypeScript 來攥寫。

```javascript=
import { objectType, queryType, makeSchema } from 'nexus'
const Post = objectType({
  name: 'Post',
  definition(t) {
    t.id('id') // Scalar fields - ID
    t.string('title')// Scalar fields - String
    t.string('body')// Scalar fields - String
    // No resolver means Nexus returns the 'id'、`title`、'body' property from the backing data model
  },
})
const Query = queryType({
  definition(t) {
    t.list.field('posts', {
      type: "Post",
      resolve: () => [
        {
          id: '1',
          title: 'My first GraphQL server',
          body: 'How I wrote my first GraphQL server',
        },
      ],
    })
  },
})
const schema = makeSchema({
  types: [Post, Query],
})
```

#### 從上面的範例看起來，直接解決了『**schema-first 被提到的缺點**』三點問題。

- 『第一點』與『第二點』因為 resolver 直接寫在 schema 下面，因此不會有不一致問題。
- 『第三點』因為可以直接用 JavaScript 來定義 schema，因此也解決了要寫不同語言的問題。

> 補充： GraphQL Nexus 的寫法感覺跟 `graphql-js` 很相似，因為它是基於 `graphql-js` 下去開發，嘗試能以更簡化的方式達成 schema-first 的效果，並且能夠保持使用原本的語言攥寫。

### Code-first Framework/Library - TypeGraphQL and TypeGoose

![](https://i.imgur.com/wUgmFAC.png)
[TypeGraphQL](https://typegraphql.com/docs/introduction.html) 是一套運用 TypeScript classes 與 decorator 的組合方式來產生 GraphQL schema 與 resolver，而 [TypeGoose](https://typegoose.github.io/typegoose/docs/guides/quick-start-guide) 則也是一套藉由 TypeScript decorator 的方式來建立 Mongoose models，因此剛好可以將這兩個套件結合起來使用，達成全部都用同一種語言來開發的。

```typescript=
/* Type */
@ObjectType()
class User {
    @Field(() => ID)
    id: string

    // @Field  代表 schema 欄位 (TypeGraphQL 提供的方法)
    @Field()
    // @prop  代表 Mongoess model 欄位 (TypeGoose 提供的方法)
    @prop({ required: true, unique: true, dropDups: true })
    firstName!: string;{
        /* 如果要對 firstName 寫 resolver 可以直接寫在這裡面
         * 或是透過 @FieldResolver 寫在 UserResolver 裡面*/
       if(this.id === '1') return 'library'
    }

    @Field()
    @prop({ required: true })
    lastName!: string;
}
/* Mongoess User Model */
const UserModel = getModelForClass(User)

/* Resolver */
class UserResolver {
    @Query(returns => [User])
    async users(@Ctx() { }: MyContext): Promise<DocumentType<User>[]> {
        let users = await UserModel.find({})
        return users;
    }
    @FieldResolver()
    firstName() {
        // ......
    }
}
/* 建立 Schema */
const schema = await buildSchema({
  resolvers: [RecipeResolver],
});
```

#### 從上面的範例看起來，TypeGraphQL 與 TypeGoose 也能夠直接解決 schema-first 被提到的三點問題。

> 補充：[十分鐘帶你了解 TypeScript Decorator - 莫力全 Kyle Mo](https://oldmo860617.medium.com/%E5%8D%81%E5%88%86%E9%90%98%E5%B8%B6%E4%BD%A0%E4%BA%86%E8%A7%A3-typescript-decorator-48c2ae9e246d)

### 小結 - Code-first

這邊再來節錄 [Tomek Poniatowicz](https://blog.graphqleditor.com/graphql-schemafirst-codefirst) 對 Code-first 的優缺點分析：

![](https://i.imgur.com/xPQQUQV.png)

#### 優點：

1. schema 與 resolver 可以同時寫在一起。

2. Code-first 可以較輕鬆的解決 Schema-first 碰到的困難。像是：因為都是用同一種語言開發，因此可以減少掉將 GraphQL Language 轉換時可能會碰到的問題。

3. 當 Schema 規模越來越大時比較好管理。目前想到的是：因為 schema 與 resolver 是寫在一起的所以減少在找尋某一隻 resolver 的時間。
   > PS.這點目前持保留意見，因為兩種模式各有利弊。

#### 缺點：

1. 因為 schema 與 resolver 都寫在同一個定義中，很容易造成程式碼難以閱讀。

2. API 設計容易被實作影響。
   > 因為 schema 與 resolver 都寫在一起，假設只有一個人開發且開發前『團隊並未』討論過整體 Schema 的話，就很容易最後設計出來的架構跟 DB 結構相似，而疏忽了整個產品的核心商業模式的架構。
3. 向後兼容更不容易發生。
   > 跟第二點雷同。

## 結論

從上面看下來不管是使用 Schema-first 或是 Code-first 開發，兩種方式都各有其優缺點，站在前端開發者的角度方面，那我個人還是比較習慣 Schema-first 的方式，因為能夠直接透過 Schema 就大概瞭解到有哪些欄位到時會開給前端，這樣在開發上面會比較方便也比較快速。

## Reference

1. [The Problems of "Schema-First" GraphQL Server Development - Nikolas Burk](https://www.prisma.io/blog/the-problems-of-schema-first-graphql-development-x1mn4cb0tyl3)
2. [Code-first vs. schema-first development in GraphQL - Leonardo Losoviz ](https://blog.logrocket.com/code-first-vs-schema-first-development-graphql/)
3. [GraphQL — Code First(Resolver-First) using TypeGraphQL and typegoose - Janitha Tennakoon](https://towardsdatascience.com/graphql-code-first-resolver-first-using-typegraphql-and-typegoose-747616223786)
4. [GraphQL - schema-first vs code-first - Tomek Poniatowicz](https://blog.graphqleditor.com/graphql-schemafirst-codefirst)
5. [GraphQL 初探 - RingCentral 铃盛](https://xie.infoq.cn/article/fa2a4f0d283729b1b66f5f6c0)
6. [[CK Patt 設計模式#6] 依賴倒轉原則（DIP）- CK Patt](https://medium.com/@ckpattern35/ck-patt-%E8%A8%AD%E8%A8%88%E6%A8%A1%E5%BC%8F-6-%E4%BE%9D%E8%B3%B4%E5%80%92%E8%BD%89%E5%8E%9F%E5%89%87-dip-e2ed77a0844)
7. [十分鐘帶你了解 TypeScript Decorator - 莫力全 Kyle Mo](https://oldmo860617.medium.com/%E5%8D%81%E5%88%86%E9%90%98%E5%B8%B6%E4%BD%A0%E4%BA%86%E8%A7%A3-typescript-decorator-48c2ae9e246d)
