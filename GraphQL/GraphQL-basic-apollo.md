# 【筆記】GraphQL 系列(二) - 瞭解 Apollo Client 與 Apollo cache 機制

###### tags: `筆記文章`

![](https://i.imgur.com/MBQLJP6.png)

上一篇介紹了 GraphQL 的語法後，這次要來介紹一下 『Apollo Client』 這個前端的 GraphQL 套件，它的核心 `@apollo/client` 套件甚至支援了 React 框架，裡面提供了許多 hooks 方便 React 開發者對 GraphQL 進行溝通。

上一篇提到如果懶的設定環境想直接試看看的話，可以使用[『Next.js 提供的 GraphQL Example 範例』](https://github.com/vercel/next.js/tree/canary/examples/api-routes-apollo-server-and-client)來實作，它裡面已經幫我們將 Client 端的設定都配置完成 (HttpLink、SchemaLink)。而本篇也會介紹一下範例中 Client 設定以及 HttpLink、SchemaLink 使用時機等等。

**_內容中如有任何錯誤或冒犯的地方還請各位大大們多多提點。_**

## Apollo Client 介紹

> Apollo Client is a comprehensive state management library for JavaScript that enables you to manage both local and remote data with GraphQL. Use it to fetch, cache, and modify application data, all while automatically updating your UI.
> Apollo Client 是一個 JavaScript Library，它讓我們能使用 GraphQL 去管理本地端 (local) 或是遠端 (remote) 資料，像是查詢 (fetch)、緩存 (cache)、更動 (add,update) 等操作。

以往我們寫好了 GraphQL query 語法但並不知道如何在前端中使用，而對於 React 開發者而言 Apollo Client 是一個不錯的選擇，它提供了不少的 hooks 方便我們能進行 query、mutation 等操作，且在查詢 (query) 時也會去追蹤目前的狀態 (isLoading、error)，方便前端在 loading 狀態時顯示一些提示文字。

Apollo Client 也提供 cache 的功能，它會使用 `__typename` 和 `id` 屬性來將 GraphQL 的 result 依照每個 Object 拆分到 Apollo 的 cache 中，且當資料更新時會自動去比對當前 cache 中是否已有該資料 (id) 有的話會自動幫忙更新，因此可以確保資料始終保持同步(**_您如果在實作中仔細觀察的話，會發現當更動資料時尚未重新 query 資料就更新了_**)。

## Next.js 範例 ApolloClient 解讀

首先，我們先來看一下範例中 ApolloClient 設定的部分，可以看到這邊用到了兩個不同的 Link(SchemaLink、HttpLink)，在瞭解這兩個 Link 之前，我們要先知道 `new ApolloClient` 中 `link` 代表什麼意思，因此我們就先從這邊下手理解吧!!!

```javascript=
function createIsomorphLink() {
    // SSR
    if (typeof window === 'undefined') {
        const { SchemaLink } = require('@apollo/client/link/schema')
        const { schema } = require('./schema')
        return new SchemaLink({ schema })
    } else {
    // CSR
        const { HttpLink } = require('@apollo/client/link/http')
        return new HttpLink({
          uri: '/api/graphql',
          credentials: 'same-origin',
        })
    }
}
// 創建 ApolloClient
function createApolloClient() {
    return new ApolloClient({
        ssrMode: typeof window === 'undefined',
        link: createIsomorphLink(),
        cache: new InMemoryCache(),
    })
}
```

### 何謂 ApolloClient 的 link

一般我們在設定 ApolloClient 的時候，可以直接透過 `uri` 參數來設定我們 Server 端的 endpoint，因為在 ApolloClient 中預設是使用 `HttpLink` 來將 GraphQL 操作透過 HTTP 發送到 Server 端，因此我們可以直接透過 `uri` 而不需引入 `HttpLink` 元件。

```javascript=
const client = new ApolloClient({
    // Provide required constructor fields
    cache: new InMemoryCache(),
    uri: 'http://localhost:4000/',//  Server 端 Url
});
```

但當我們想要在發送 HTTP 之前能先有一些額外的檢查或是操作的話，這時就需要透過 ApolloClient 裡 `link` 的參數去客製整個 Apollo Client 的 flow，原本預設是只使用了 `HttpLink` 這一個 link，現在我們可以透過 **Apollo Link** 這個套件來[客製化自己的 Link](https://www.apollographql.com/docs/react/api/link/introduction#creating-a-custom-link)，當然套件內也有一些做好的 Link 讓我們可以直接使用，最後我們需要將這些 Link 串聯起來放到 ApolloClient 的 `link` 參數中，這整個行為被稱為 **link chain**。

![](https://i.imgur.com/esRjtyf.png)

> **Apollo Link** 是就像是 Redux 中的 middleware 一樣，當我們從 Client 端發起操作時(ex.查詢資料)，我們可以去客製化整個 Client 端到 Server 端之間的 flow，像是增加 log 紀錄、權限認證、HTTP header 增加 attrbute ...等。

#### 客製化自己的 Link

```javascript=
/* 客製化自己的 Link */
import { ApolloLink } from '@apollo/client';
/* 增加計入時間 */
const timeStartLink = new ApolloLink((operation, forward) => {
  // operation 是 graphQL operation object(ex.query、variables、operationName.....)
  // https://www.apollographql.com/docs/react/api/link/introduction#the-operation-object
  operation.setContext({ start: new Date() });
  return forward(operation); // 呼叫下一個 link
});
```

#### Apollo Link 提供的 link

```javascript=
/* Apollo link 提供的 link */
import { onError } from "@apollo/client/link/error";
const errorLink = onError(({ graphQLErrors, networkError }) => {
  if (graphQLErrors)
    graphQLErrors.forEach(({ message, locations, path }) =>
      console.log(
        `[GraphQL error]: Message: ${message}, Location: ${locations}, Path: ${path}`,
      ),
    );

  if (networkError) console.log(`[Network error]: ${networkError}`);
});
```

### 透過 ApolloLink.from 串聯 Links

在 ApolloClient 有兩種 link 方式的組成，一種是 [Additive composition](https://www.apollographql.com/docs/react/api/link/introduction#additive-composition) 另一種是 [Directional composition](https://www.apollographql.com/docs/react/api/link/introduction#directional-composition)，兩個的差別在於 Additive composition 只有一條 link chain，而 Directional composition 則像 Git 一樣可以有很多個分支(branch)，並且依照不同的情況去執行不同的 link chain。

> 本篇目前只會介紹 Additive composition，對 Directional composition 有興趣的讀者再麻煩自行閱讀了!!!

#### Additive composition

如果是 Additive composition 的情況則可以用 `ApolloLink.from` 去將每個 link 組合成一條 link chain。

```javascript=
// If you provide a link chain to ApolloClient, you
// don't provide the `uri` option.
const client = new ApolloClient({
  // The `from` function combines an array of individual links
  // into a link chain
  link: from([errorLink, httpLink]),
  cache: new InMemoryCache()
});
```

### Creating a custom link (AuthLink)

上面我們介紹了可以客製化自己的 Apollo Link，而在實務上我們很常需要在 request header 加上 authorization token 等欄位，這時我們就可以透過上面所教的方式去實作一個 『客製化的`AuthLink`』，透過 GraphQL operation 提供的`setContext` function 來為每個 operation 的 header 都加上 authorization 欄位。

#### 範例：客製化一個 AuthLink 來將每個 operation 都加上 headers。

```typescript=
const AuthLink = new ApolloLink((operation,forward)=>{
    operation.setContext(({headers},prevContext) =>({
      headers: {...headers, authorization: "1234"}
    }));
    // 傳給下一個 Link
    return forward(operation)
})

```

## Apollo Client - Query

### Fetch Policy

當我們在使用 Apollo Client 的 `useQuery` hook 時，預設會『先去 Apollo Client cache 裡面檢查我們所要查詢(query)的資料是否都有存在於 cache 裡面，如果有則直接回傳;否則會將 Query 傳給 Apollo Server 請 Server 去查詢該資料』，而以上的流程就是 `cache-first` 的 fetch 策略，這也是`useQuery` 預設的 fetch policy。

#### 以下為官方的 cache 流程圖(cache-first)：

![](https://i.imgur.com/GHkRD97.png)

#### 其他 Fetch Policy 策略

上面講了預設的 fetch 策略後，其實 Apollo Client 還有提供其他的緩存策略可以讓我們選擇，透過 `fetchPolicy` 參數去使用不同的策略

```javascript=
// 使用範例：
const {data} = useQuery(query,{
    fetchPolicy: "network-only",
})
```

| 解釋              | Fetch Policy                                                                                                                              |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| cache-first       | **預設 fetch policy 策略**<br/> 先去 cache 裡面查看，如果有則回傳;如果沒有則發動到 Server 去取的資料並且 cache 與 回傳給 Client。         |
| cache-only        | Apollo Client 只會查看 cache 裡是否有存在，如果沒有則會丟出 error。                                                                       |
| cache-and-network | Apollo Client 會先查看 cache 資料，如果有則先回傳，而不管有沒有 cache 資料都會發送到 Server 去取得最新資料並更新 cache 與回傳給 Client 。 |
| network-only      | Apollo Client 會直接與 Server 溝通，不會先經過 cache 的部分，當取得最新資料時『會更新 cache』與回傳給 Client。                            |
| no-cache          | `no-cache` 跟 `network-only` 很像，差別在於它『不會更新 cache』直接回傳給 Client。                                                        |
| standby           | 它與 `cache-first` 邏輯一樣，差別在於當                                                                                                   |

#### 強烈推薦看：[Understanding Apollo Fetch Policies](https://medium.com/@galen.corey/understanding-apollo-fetch-policies-705b5ad71980)

## Apollo Cache 機制

當透過 GraphQL Query 抓取資料時如果在 query 中有帶上 id 的話，基本上可以在`__APOLLO_CLIENT__.cache.data.data`上看到一個叫 `_ref` 的參數，因為 Apollo 預設會用 `_id` 或 `id` 欄位來組成 cache ID，當然我們也可以客製化 cache ID，這方面會在後續的 `keyFields` 、 `keyArgs` 來介紹。

![](https://i.imgur.com/F83TGCX.png)

從下圖的 cache 結構中可以發現到 `__ref` 的組成『預設』是由該筆資料的 `__typename` 加上 `id` 所組成，且 Apollo 的 cache 機制是使用 normalize 的方式去設計。

> 官方文件位置：[Generate cache IDs](https://www.apollographql.com/docs/react/caching/overview/#2-generate-cache-ids)
> an object's cache ID is the concatenation of the object's \_\_typename and id (or \_id) fields, separated by a colon (:).

![](https://i.imgur.com/bPlqtFh.png)

#### 那如果今天我們想對 cache 的欄位做更動呢？

例如手動增加一些欄位到 cache 裡面、或者 query 新的資料時將『原本的資料』與『現在的資料』進行 merge 的話，又該如何操作呢？

關於這些部分我們需要透過 ApolloClient InMemoryCache 的 `typePolicies` 參數來去對 cache 的內容進行進行設置。

`typePolicies` 的結構是依照 Schema 裡面的每個 Object Type 來定義，用 Object Type 的 name 來當作 key，而 value 則是可以對該 Object Type 裡面的欄位做一些處理(ex.merge、read)。

> **typePolicies：**
> Each key in the object is the \_\_typename of a type to customize, and the corresponding value is a TypePolicy object.

#### 假設我們的 Schema 長相這樣：

Query 裡面有 `user` 與 `product` 欄位，而 User type 裡面又有 `notifications` 這個欄位，Notifications type 裡面又有 content 欄位...以此類推。

```graphql=
# 簡單模擬 Schema
type User {
    id: ID
    address: String
    name: String
    notifications: [Notifications]
}
type Notifications{
    id: ID!
    content: Content # 故意多包一層 Content 的 type 來增加複雜度
}
type Product{
    id: ID
    price: number
}
type Content{ # 故意分出一個 Content 的 type 來增加複雜度
    title: String
    subTitle: String
    createTime: String
    isRead: Boolean
}
type Query {
    user: User
    product: [Product]
}
```

還記得我們上面說 `typePolicies` 要依照 Schema 的 Object Type 去攥寫嗎？因此上面的結構在 `typePolicies` 物件內就會為以下( key 為每個 Object Type ， value 為每個 type 裡面的欄位 )

```javascript=
const typePolicies = {
    Query:{},
    Product:{},
    User:{},
    Notifications:{}
}
```

在 value 的部分，我們可以透過 `keyFields` 來決定這個 type 的 `__ref` 要是怎麼樣的形式，透過 `fields` 這個參數來對這個 type 裡面的欄位進行一些操作，例如：將 User 裡面 `name` 欄位的值後面都加上`from library`的字串。

> 提醒： `__ref` 可以說是 Apollo cache 的 key，Apollo 會依照`__ref` 指向一層一層找到所 cache 的資料。

```javascript=
const typePolicies = {
    User:{
        fields:{
            // 用 address + name 當作 identifying field
            // ex. User:{address:'新北市',name:'library'}
            keyFields:['address','name']
            name:{
                // 當取得 name 欄位的 value 時，會執行 read function，因此會回傳 'xxx from library'
                read: (existing) => {
                    // existing 代表原本這個欄位的值
                    return `${existing} from library`
                }
            }
        }
    },
}

```

## Apollo Cache 機制 - 實際例子：

### 情境 1. 如果我們今天想對 user 的 notifications 做 infinite scroll 的功能話，那我們該如何處理呢？(ps.在 limit,offset 的架構下)

> 解答：前端在取資料時是透過 limit,offset 的架構去每次取得 10 筆資料，而 useQuery 每次回傳的 data 就會是 1\~10(第一次),11\~20(第二次)，而 data 永遠就只會呈現 10 筆資料而已，因此我們需要將每次 query 出來的『新資料』與『舊資料』進行 merge。

```javascript=
const typePolicies = {
    // 情境 1
    User:{
        fields:{
            notification:{
                merge:(existing,incomeing)=>{
                    // 將 原本資料 與 新資料 結合
                    return [...existing,...incomeing]
                }
            }
        }
    },
}
```

畫面的部分使用 [react-infinite-scroll-component](https://www.npmjs.com/package/react-infinite-scroll-component) 來做無限滾動模擬，每當滑到底部時就去呼叫 useQuery 提供的 `fetchMore` function 再去抓取新的資料。

```jsx=
// 大致擷取一下比較重點的部分

const { data: notificationsData, fetchMore } = useQuery(ALL_NOTIFICATIONS_QUERY)
const fetchData = () => {
    if (fetchMore) {
      fetchMore({ query: ALL_NOTIFICATIONS_QUERY })
    }
  }
// render 原始排序
const renderOriginalView = () => {
    return (
      <InfiniteWrapper id='scrollableDiv'>
        <InfiniteScroll dataLength={originNotifications?.length ?? 0} next={fetchData} hasMore={true} scrollableTarget='scrollableDiv'>
          <Grid container>
            {originNotifications?.map((item) => {
              return (
                <Grid item key={item.id}>
                  <Card sx={{ width: '350px', height: '250px' }}>
                    <CardContent>
                      <Typography variant='h5' component='p'>
                        通知：{item?.content?.title}
                      </Typography>
                      <Typography variant='h6' component='p'>
                        副標題：{item?.content?.subTitle}
                      </Typography>
                      <Typography variant='h6' component='p'>
                        發文日期：{moment(Number(item?.content?.creatTime)).format('YYYY/MM/DD')}
                      </Typography>
                    </CardContent>
                  </Card>
                </Grid>
              )
            })}
          </Grid>
        </InfiniteScroll>
      </InfiniteWrapper>
    )
}

```

![](https://i.imgur.com/HJdKTuy.gif)

### 情境 2. 如果有兩個頁面分別都會去取得通知資料，但Ａ頁面只取得 title 欄位，Ｂ頁面只取得 subTitle 欄位，那我們又該如何將同一筆資料整合在一起呢？

> 解答：我們可以透過 `mergeObjects` 這個屬性來將『原本物件』與『這次物件』合併起來。

```javascript=
const cache = new InMemoryCache({
  typePolicies: {
    // 情境 2
    Notification: {
        fields:{
             /* 代表 Notification 這個 Object Type 裡的欄位 */
            content: {
                // keyArgs 代表要用 content 欄位裡的『哪些參數』來組成 __ref 的名稱,
                // 如果設置 false 則會存在 父層的 結構底下，不會特別有一個自己的 `__ref`， 簡單來說：不會被 cache
               keyArgs: false,
               merge:(existing,incoming,{mergeObjects})=>{
                   // existing 代表原本這個欄位的值
                   // incoming 代表這次拿進來的值
                   // mergeObjects 是官方提供我們方便做 merge 用
                   return mergeObjects(existing, incoming)
                    /*
                    以這邊 message 的欄位來看，如果 merge 則會是把原本 message 的 value 與這次的 value 合併
                    ex.
                        原本：message:{ title: 1 }
                        這次：message:{ subTitle: 2 }
                        merge 完： message: { title:1 , subTitle: 2 }
                    */
                }
            }
        }
    }
  },
});

```

> **補充：** `keyFields` 與 `keyArgs` 這兩個參數的設定會改變 `__ref` 的顯示，預設是 `__typename`+`id` 來當作 cache data 物件的 key，如果改成 `keyArgs:['title','content']` 時，則 `__ref` 會變成 `__typename`+ `{title:'YOUR_TITLE',name:'YOUR_NAME'}` 這樣的形式來當作 cache ID。
>
> ![](https://i.imgur.com/LeV7pRH.png)

### 情境 3. 前端是否能在 cache data 中加入客製化的欄位呢？

> 解答：可以的！我們可以任意的在 `fields` 加入想要的欄位，例如：我們在 User Type 裡加入一個 `sortNotifications` 來顯示排序過後的通知列表。

```javascript=
typePolicies: {
    User: {
      fields: {
        // read function
        sortNotifications: (existing, { readField }) => {
          // 注意：這邊我們使用了淺拷貝，因為 readField() 是 immutable
          const notifications = [...readField('notifications')]
          // use lodash orderBy
          const sortData = _.orderBy(
            notifications,
            (item) => {
              // 取出每筆 notification 裡面的 content field
              const content = readField('content', item)
              return content.creatTime
            },
            ['desc']
          )
          return sortData
        },
        /* 上面寫法 與 下面寫法 是一樣的 */
        // sortNotifications: {
        //   read: () =>{}
        // }
      },
    },
  },
```

然後我們需要在 Query 的地方加上 `sortNotifications` 這個 field 以及後面要帶上 `@client` 這個 directive，讓 Apollo 知道這個欄位要從由前端的 typePolices 查找。

```javascript=
export const ALL_NOTIFICATIONS_QUERY = gql`
  query fetchNotifications {
    user {
      # 排序過後
      sortNotifications @client {
        id
        content {
          title
          subTitle
          createTime
          isRead
        }
      }
      # 原始資料
      notifications {
        id
        content {
          title
          subTitle
          createTime
          isRead
        }
      }
    }
  }
`
```

![](https://i.imgur.com/KLhQYdf.gif)

## 透過 Mutation update function 更改 cache data 裡的值

上面的部分主要都是在對 Query 後的資料進行處理，但很常的時候我們會碰到即時更新的部分，例如：點擊刪除貼文後貼文消失、未讀通知訊息上的紅點再點擊後立即消失。

一般最簡單的方式，是在呼叫完 Mutation 後再去呼叫 Query 的 `refetch` function 重新抓一次資料來做更新，這樣的『好處』是程式碼比較精簡(畢竟只需要呼叫一個 function 而已)，但『壞處』是會重新發送一次 request 到後端，而且如果是在 infinite scroll 的架構下可能還會碰到一些問題。

> 舉例來說：當今天每次取 10 筆且是用 `limit,offset` 的架構，使用者滾動到第 30 筆後( offset 可能已經為 30)，突然點擊了第 5 筆的未讀通知，那現在要如何重新 fetch 資料呢？

當然還是有許多方法可以解決，但最方便且又不需重新抓取資料的方法，就是直接去調整 cache 裡的資料，當我們在呼叫 Mutation function 時可以傳入一個 `update` 的參數，這個 `update` function 會在 mutation 執行完後被執行，且它提供了 `cache` 與 `mutationResult` 這兩個參數方便我們對 cache 資料進行調整。

> 關於 Mutation 的相關參數可參考：[官方文件 - useMutation API](https://www.apollographql.com/docs/react/data/mutations#result)

![](https://i.imgur.com/shWNiO5.png)

### Interacting with cached data

官方提供了幾種方便我們與 Cache 資料進行互動的方式，像是 `readQuery` 、 `writeQuery` 就是透過傳入 GraphQL query 的方式來去讀取或更改 cache 裡的資料。

`Fragement` 跟 Query 很像只是它是依照傳入 `Fragement` 去改變 cache data。

`cache.modify` 則跟我們一般在修改物件一樣，直接去對某個 cache data 裡的 `__ref` key 調整它的 value。

![](https://i.imgur.com/vjgHMOJ.png)

接下來將示範使用 `writeQuery` 來處理 Blog 貼文新增、刪除時更改 cache data，以及用 `cache.modify` 來處理通知訊息點擊已讀。

### Blog 新增、刪除貼文 - writeQuery

當我們在新增貼文或刪除貼文時，一定會希望能夠及時看到完成的畫面，如果想要達成這樣的效果，最保險且最簡單的方式就是再去呼叫 API 跟後端拿取更新後的資料，或者前端直接更新 state 裡的資料。

而在 Apollo 中我們可以透過上面介紹的幾種方式去更新 cache 裡的資料來達成畫面更新，因為當透過這幾種方式來更新 cache 時 Apollo 預設會自動重新 re-render 畫面，因此這樣就可以減少發送 request 到 server。

#### 下面將把新增的部分改用 cache.writeQuery 來實作

```javascript=

const [addPost] = useMutation(ADD_POST_QUERY,{
    // 透過 update function 裡的 cache 參數來更新 cache data
    update(cache, { data: { addPost } }) {
      let newData = { viewAllPost: [...addPost] }
      cache.writeQuery({
        query: ALL_POST_QUERY,
        data: newData,
        variables: {} // 如果 query 需要傳入參數的話,可透過 variables 來帶入
      })
    },
})

```

> 補充： `update` function 不只可以定義在 `useMutation`，也可以寫在 Mutate function(addPost) 上面 `addPost({update:()=>{}})`。

#### writeQuery 介紹

> The writeQuery method enables you to write data to your cache in a shape that matches a GraphQL query

簡單來說就是 writeQuery 會依照傳入的 `query` 語法來去查詢出那些資料，並且透過 `data` 參數去將這些資料更新。

如果 `query` 有需要傳入參數的情況，像是只要更新某一筆資料時，則可以透過 `variables` 將所需要的參數傳給 `query`。

#### 成果

![](https://i.imgur.com/RdHe8aV.gif)

### 通知訊息未讀 - cache.modify

上面使用了 `writeQuery` 來更新了 cache data，接下來通知訊息這邊來用 `cache.modify` 更新未讀紅點顯示。

> ps.這邊模擬將 `update` function 寫在 Mutate function 內來更新 cache data。

```javascript=
const [updateMutation] = useMutation(UPDATE_NOTIFICATION)

updateMutation({
  variables: {
    notificationId: item.id,
  },
  update: (cache) => {
    cache.modify({
      id: cache.identify(item), // cache data 的 key , 你也可以自己組 key ex.`${item.__typename}:${item.id}`
      fields: {
        content: (existing) => {
          return {
            // 原本 content 內容
            ...existing,
            // 將 content 裡的 isRead 改成 true
            isRead: true,
          }
        },
      },
    })
  },
})
```

#### cache.modify 介紹

> The modify method of InMemoryCache enables you to directly modify the values of individual cached fields, or even delete fields entirely.

`cache.modify` 與 `writeQuery`、`writeFragment` 不同點在於，它不會觸發 `typePolicies` merge function，因此在修改 cache data 時是會直接『覆蓋掉該筆的 cache 資料』，且不能寫入不存在於 cache 中的欄位。

`cache.modify` 最重要的就是 `id` 與 `fields` 這兩個參數，代表著要對 cache 資料裡的『哪一個 key』裡的『哪一個 fields』做更動，而 fields 裡面也有提供一些方法(ex.`DELETE`方法，讓我們直接把欄位砍掉)

最後官方推薦使用`cache.identify` 的方法去取得 cache ID，因為當 cache ID 是透過多個欄位來組成的話，要自己手動組出就會變得比較複雜且麻煩，因此透過 `cache.identify(想要抓取 ID 的那筆物件)` 就會變得簡單很多。

```javascript=
cache.modify({
    id: cache.identify(item) // cache data key
    fields: {
        comments(existingCommentRefs, { DELETE }) {
          return DELETE; // 刪除 comments 欄位
        },
    },
})
```

#### 成果

![](https://i.imgur.com/EHvlpl1.gif)

## Reference

1. [Understanding Apollo Fetch Policies](https://medium.com/@galen.corey/understanding-apollo-fetch-policies-705b5ad71980)
