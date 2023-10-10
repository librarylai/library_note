# 【筆記】Next13 使用 Next-auth 實作第三方登入

###### tags: `筆記文章`

![](https://hackmd.io/_uploads/Hky2Scnyp.png)

## Next-auth 介紹

> NextAuth.js is a complete open-source authentication solution for Next.js applications.

NextAuth 是一套在 Next.js 上的身份驗證套件，除了**常見的 email、password 登入之外**，它還整合了許多第三方登入，**像是：Facebook、Google、Line...等第三方登入都包含在其中**。
以往最早的時候我們要串接第三方登入時都需要依照每個平台安裝相對應的 SDK，並且使用各平台規定的方式、函示來取得登入者資訊，大概會像是下面這樣（ps.下面範例應該有點歷史了是使用 SDK 的方式，現在官方都有對應的 npm package 可以直接安裝）

#### 舉例 - Facebook SDK 登入

```javascript=
FB.login(function(response){
  // handle the response
});
```

資料來源：[https://developers.facebook.com/docs/facebook-login/web/](https://developers.facebook.com/docs/facebook-login/web/)

#### 舉例 - Google SDK 登入

```javascript=
<script>
  window.onload = function () {
    google.accounts.id.initialize({
      client_id: 'YOUR_GOOGLE_CLIENT_ID',
      callback: handleCredentialResponse,
      login_url: ....
    });
    google.accounts.id.prompt();
     // render login button
     google.accounts.id.renderButton(
      /** @type{!HTMLElement} */ parent,
      /** @type{!GsiButtonConfiguration} */ options
    )
  };


</script>
```

資料來源：[https://developers.google.com/identity/gsi/web/reference/js-reference?hl=zh-tw](https://developers.google.com/identity/gsi/web/reference/js-reference?hl=zh-tw)

如果每一個平台就要分別使用對應個 SDK 與對應的語法，當今天網站要支援很多個第三方平台登入時，就會變得比較複雜，**且依照 OAuth2.0 的流程可能還需要再起一個後端 Server 來做 Authorization Code 交換 Access Token 的部分**，如下圖：

![](https://www.technice.com.tw/wp-content/uploads/2022/08/7.png)
(Reference：[認識 OAuth 2.0：一次了解各角色、各類型流程的差異](https://www.technice.com.tw/experience/12520/))

**而 Next-auth 就是幫我們整合了這些流程，任何的第三方登入、登出只需要透過 `singIn`、`singOut` 這兩個 Function 就可以達成，且 NextAuth 默認會將登入後資訊儲存在 session/cookie 中，如果有提供 `NEXTAUTH_SECRET` 的話，則會使用它來進行加密，廢話不多說直接來實作一下吧！！**

## 安裝

### 1. 建立一個 Next 13 專案

```
npx create-next-app@latest
```

Ps. 這邊是都選擇 『**Yes**』 (包含：Typescript、tailwindcss、src folder、eslint....等)

### 2. 安裝 Next-auth

```
npm install next-auth
```

## 建立 Google OAuth Client

### 1. 前往 Google APIs & Services 頁

![](https://hackmd.io/_uploads/SJxJIZ2kT.png)

### 2. 建立一個 OAuth Client Id

![](https://hackmd.io/_uploads/HkOGLZhJa.png)

### 3. 填寫名稱、URIs

> **origins：** 未來上線後需填寫成你的 domain name，在開發模式下可以填寫 `localhost:3000<你的 Server PORT>`
>
> **redirect URIs：** NextAuth 這邊有規定填寫成 `https://{YOUR_DOMAIN}/api/auth/callback/google`，一樣開發模式可將 domain 改寫 `localhost:3000`。
> [參考文件：https://next-auth.js.org/providers/google](https://next-auth.js.org/providers/google)

![](https://hackmd.io/_uploads/SknPI-hya.png)

### 4. 建立成功後，請複製 `Client Id`、`Client secret`

這邊麻煩先將 `Client Id`、`Client secret` 複製下來並寫到 `.env` 檔中，等等在使用 NextAuth 的 `GoogleProvider` 時會需要填入。
![](https://hackmd.io/_uploads/BJcldW21a.png)

## 建立 LINE Develop Login Channel

### 1. 前往 [LINE Developer Console](https://developers.line.biz/console/) 頁面

### 2. 選擇 or 建立一個 Provider

![](https://hackmd.io/_uploads/rJUw6uTyT.png)

### 3. 選擇建立 LINE Login

![](https://hackmd.io/_uploads/BJrspdaJ6.png)

### 4. 填寫 Channel 基本資訊

![](https://i.imgur.com/PhSA1Od.gif)

### 5. 建立完成後，進入選單 LINE Login 內

進入選單 LINE Login 內，將 Callback URL 填寫上 `
http://<你的 Domain>/api/auth/callback/line`

> 詳細可參考：[NextAuth Line Provider 官方文件](https://next-auth.js.org/providers/line#configuration-1)

![](https://hackmd.io/_uploads/Bkk0kt616.png)

### 6. 複製 `Channel ID` 與 `Channel Secret`

進入選單 Basic settings 頁，並且將 `Channel ID` 與 `Channel Secret` 記錄起來，等等需填入到 `.env` 中。

## 實作第三方登入 － NextAuth 架構

### 1. 建立 `.env.local` 檔設定環境變數

#### 1-1. 將 Google OAuth ClientID 與 Secret 填入

```env=
GOOGLE_CLIENT_ID='959402876660-o2h2iqs258fbs8xxxxxxxxx'
GOOGLE_CLIENT_SECRET='GOCSPX-xxxxxxxxxxxxxx'
```

#### 1-2. 將 LINE login ChannelID 與 Secret 填入

```env=
LINE_CHANNEL_ID='200xxxxxxx'
LINE_CHANNEL_SECRET ='f63dxxxxxxxxx'
```

#### 1-3. 建立 `NEXTAUTH_URL` 與`NEXTAUTH_SECRET` 並填入 env 檔中

```env=
# 當 Production 將 NEXTAUTH_URL 改成正式網址
NEXTAUTH_URL='http://localhost:3000'
NEXTAUTH_SECRET = 'XEHMpKpp3STxPnSnExxxxxx='
```

`NEXTAUTH_SECRET` 可以透過終端指令 `openssl rand -base64 32` 來建立或是 [https://generate-secret.vercel.app/32](https://generate-secret.vercel.app/32) 該網站來建立一個隨機的 secret。

### 2. 建立 `/api/auth/[...nextauth]/route.ts` 檔

首先根據 NextAuth 的官方教學，需要在 `/api/auth/[...nextauth]/` 路徑底下建立一支 `route.ts` 檔，並且寫入以下內容：

[官方文件：https://next-auth.js.org/configuration/initialization#route-handlers-app](https://next-auth.js.org/configuration/initialization#route-handlers-app)

```typescript=
import NextAuth from 'next-auth'
import { authConfigs } from '@/lib/loginAuth'

const handler = NextAuth(authConfigs)

export { handler as GET, handler as POST }
```

### 3. 建立 `lib/loginAuth.ts` 檔

這邊統一將 NextAuth 相關 config 與會用到的 function 統一集合在 `lib/loginAuth.ts` 檔案中。

### 4. 攥寫 authConfigs 內容

> **補充：取得登入 LINE 時 email 資訊**
> 如果要取得 email 則需要**重新定義 `scope` 選項**，預設只有`openid`、`profile`，因此要再加上 `email`，且也**需在 LINE Developer 中將 OpenID Connect 裡的 Email address permission 打開。**

```typescript=
/* lib/loginAuth.ts */
import GoogleProvider from 'next-auth/providers/google'
import LineProvider from 'next-auth/providers/line'
import { NextAuthOptions } from 'next-auth'
import { getServerSession } from 'next-auth/next'

export const authConfigs: NextAuthOptions = {
  // 設定 secret 讓 session cookie 加密
  secret: process.env.NEXTAUTH_SECRET,
  // 存放各種要使用的 第三方登入 或是 客製化自己的登入方式（ex. email, password）
  providers: [
    GoogleProvider({
      // 填上 Google OAuth 建立的 client id & secret
      clientId: process.env.GOOGLE_CLIENT_ID as string,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET as string,
    }),
    LineProvider({
      // 填上 Line login 的 Channel id & secret
      clientId: process.env.LINE_CHANNEL_ID as string,
      clientSecret: process.env.LINE_CHANNEL_SECRET as string,
      // 如果要取得 email 則需要重新定義 scope 選項，預設只有`openid`、`profile`，因此要再加上 `email`
      // 且也需在 LINE Developer 中將 OpenID Connect 裡的 Email address permission  打開
      authorization: { params: { scope: 'openid email profile' } },
    }),
  ],
  callbacks: {
    async jwt({ token, account, profile }) {
      // 這邊可以將，我們從第三方登入取得的資料存入 token，並且讓 token 帶入 session 給前端存取
      // 可以依照自己的需求，例如透過 email, name, 去建立一個該專案的 access_token 與 refresh_token 給前端存取，且將第三方取得的資料存到 ＤＢ 中
      // 舉例：將 access_token 存入 token 中 （該 access_token 是透過第三方登入取得的）
      if (account && account.access_token) {
        token.accessToken = account.access_token
      }
      return token
    },
    async session({ session, token, user }) {
      // Send properties to the client, like an access_token from a provider.
      // 一般預設 session 只會存放 user 資料（ex. name , email , image），如果要存放其他資料，可以透過這邊存入
      // 舉例：將 token 中的 access_token 存入 session 中
      session.accessToken = token.accessToken as string
      return session
    },
  },
}

export const getSessionData = async () => await getServerSession(authConfigs)

```

### 5. 攥寫 `SessionAuthProvider.tsx` 檔

這邊因為 Next13 Server Component 的緣故，無法在 `RootLayout` 中直接使用 NextAuth 的 `SessionProvider`，因為 `SessionProvider` 是需要在 Client Component 中 render，因此這邊需要額外封裝出一個 Provider，如下：

> 詳細可參考：[Authentication with Next.js 13 and Next Auth - Praveen Jayakody](https://medium.com/ascentic-technology/authentication-with-next-js-13-and-next-auth-9c69d55d6bfd)

```tsx=
/* src/context/providers/SessionAuthProvider.tsx */
'use client'
// 需單獨開一隻來封裝 SessionProvider 在 use client 中使用，因為 layout.tsx 是 server component
import { SessionProvider } from 'next-auth/react'

export default function Provider({ children, session }: { children: React.ReactNode; session: any }) {
  return <SessionProvider session={session}>{children}</SessionProvider>
}

```

### 6. 將 `SessionAuthProvider` 放到 RootLayout 中

這邊會用到 NextAuth 的 `getServerSession` 來取得 session 資料，並將 session 資料傳到剛剛建好的 `SessionAuthProvider` 中，**這樣在底下的任何 Client Component 才能透過 NextAuth 提供的 `useSession` 來取得 session 資料。**

> 補充：
> 簡單來說，NextAuth 提供了兩種取的 Session 資料的方式，一種是在 Server Component 上的 `getServerSession`，另一種是 Client Component 上的 `useSession`。

```jsx=
/* app/layout.tsx */
// 這邊需要將 getServerSession、authConfigs、SessionAuthProvider 都 import 近來
import { getServerSession } from 'next-auth/next'
import { authConfigs } from '@/lib/loginAuth'
import SessionAuthProvider from '@/context/providers/SessionAuthProvider'

export default async function RootLayout({ children }: { children: React.ReactNode }) {
  const session = await getServerSession(authConfigs)

  return (
    <html lang='en'>
      <body className={inter.className}>
        <SessionAuthProvider session={session}>{children}</SessionAuthProvider>
      </body>
    </html>
  )
}
```

## 實作第三方登入 － 前端登入畫面

上面已經完成了 NextAuth 基本的架構了，現在來實作前端的登入畫面，並測試一下整個第三方登入的功能。

### 1. 實作第三方登入共用按鈕 `ThirdPartyLoginButton`

這邊主要的重點在透過 NextAuth 提供的 `signIn` function 來選擇要用哪種方式登入，會自動執行 CSRF token 的相關步驟，來防範 CSRF 問題。

> 詳細可參考：[https://next-auth.js.org/getting-started/client#signin](https://next-auth.js.org/getting-started/client#signin)

```jsx=
/* components/buttons/ThirdPartyLoginButton.tsx */
'use client'
import Image from 'next/image'
import { signIn } from 'next-auth/react'

type Props = {
  type: 'google' | 'line'
  icon: string
  buttonText: string
}

export default function ThirdPartyLoginButton({ type, icon, buttonText }: Props) {
  const handleLogin = () => {
    signIn(type) // 重點在這邊～～～～～
  }
  return (
    <button
      onClick={handleLogin}
      className='w-60 bg-white hover:bg-slate-200 border-2 text-center font-bold flex  items-center gap-3 rounded-full border-slate-950 hover:border-cyan-300 px-3 py-2 '
    >
      <Image src={icon} alt='Logo' width={24} height={24} priority />
      <p className='text-slate-950'>{buttonText}</p>
    </button>
  )
}

```

### 2. 實作登出按鈕 `LogoutButton`

使用 `signOut` function 來進行登出，這邊也會自動執行 CSRF token 的相關步驟，來防範 CSRF 問題。

> 詳細可參考：[https://next-auth.js.org/getting-started/client#signout](https://next-auth.js.org/getting-started/client#signout)

```jsx=
/* components/button/LogoutButton.tsx */
'use client'
import { signOut } from 'next-auth/react'

export default function LogoutButton() {
  const handleLogin = () => {
    signOut() // 重點在這～～～～
  }
  return (
    <button
      onClick={handleLogin}
      className='w-60 bg-white hover:bg-slate-200 border-2 text-center font-bold flex justify-center items-center gap-3 rounded-full border-slate-950 hover:border-cyan-300 py-2'
    >
      <p className='text-slate-950'>Logout</p>
    </button>
  )
}

```

### 3. 實作登入畫面 - 引入`ThirdPartyLoginButton` 與 `LogoutButton`

> 這邊直接調整 `app/page.tsx` 檔來方便 Demo，大家可依照專案需求來建立。

這邊將剛剛實作的登入、登出按鈕引入到畫面中，且因為是 Server Component 的關係，所以使用 `getSessionData` 來取得 session 資料，當第三方登入成功後將 session 資料顯示出來。

```jsx=
import ThirdPartyLoginButton from '@/components/button/ThirdPartyLoginButton'
import LogoutButton from '@/components/button/LogoutButton'
import { getSessionData } from '@/lib/loginAuth'
import Image from 'next/image'
export default async function Home() {
  const session = await getSessionData()
  return (
    <main className='flex min-h-screen flex-col items-center  p-24'>
      {session && (
        <div className='mb-4'>
          <Image width={60} height={60} alt={'profile'} src={session.user.image} />
          <div>{session.user.email}</div>
        </div>
      )}
      <div className='flex gap-3 flex-col bg-white rounded-2xl  p-12'>
        <ThirdPartyLoginButton type={'google'} icon={'/icon-google.svg'} buttonText={'Continue with Google'} />
        <ThirdPartyLoginButton type={'line'} icon={'/icon-line.svg'} buttonText={'Continue with LINE'} />
        <LogoutButton />
      </div>
    </main>
  )
}

```

## 成果展示

![](https://i.imgur.com/dgMhHvc.gif)

#### Github：[https://github.com/librarylai/next-auth-login](https://github.com/librarylai/next-auth-login)

## Reference

1. [How to Build a Fullstack App with Next.js, Prisma, and Vercel Postgres - Vercel blog](https://vercel.com/guides/nextjs-prisma-postgres)
2. [用 NextAuth 實作第三方登入 - Ruofan](https://blog.errorbaker.tw/posts/ruofan/next-auth/)
3. [Add Authentication to Next.js in 10 mins with OAuth - CoderOne](https://www.youtube.com/watch?v=AbUVY16P4Ys)
4. [Authentication with Next.js 13 and Next Auth - Praveen Jayakody](https://medium.com/ascentic-technology/authentication-with-next-js-13-and-next-auth-9c69d55d6bfd)
5. [使用 Next-auth 給 Next-js 應用添加鑑權與認證 - readfog](https://www.readfog.com/a/1690069039624327168)
6. [使用 NextAuth.js 來串接 Line Login 時可能遇到的問題 - Wildsky F.](https://blog.wildsky.cc/posts/line-nextauth-problem)
