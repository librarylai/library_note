# 【筆記】JSDC - Web3.js Demo 程式碼解讀

###### tags: `筆記文章`

最近剛開始接觸到區塊鏈到東西，從透過幣安買幣開始到慢慢去了解何謂以太坊、何謂虛擬錢包...等，剛好這次的 JSDC 2021 有關於區塊鏈的講談，其中 **_李婷婷_** 講者的現場 DEMO 更是讓我耳目一新，所以打算詳細了解一下**_李婷婷_** 講者在 JSDC 上的程式碼。

**本篇文章主要是一步一步去瞭解整個程式碼的運作，並跟著講者的 JSDC 影片實作，如有任何冒犯或錯誤的地方，再麻煩大大們告知。**

這邊也有我與朋友共同整理的一些筆記，內容會不定時慢慢增加，有點亂的地方還請見諒。[【筆記】區塊練相關 - 基礎知識](https://hackmd.io/@9iEIv7CwQuKe2LizHnDhaQ/ryr4kkbDK)

### 前置作業

1. Fork 或 Clone 講者的 Demo 專案到自己的 Github 或電腦中

   - Github: https://github.com/tina1998612
   - web3-demo: https://github.com/z-institute/Web3-Demo/blob/main/index.js

2. yarn install 載入所需要的 package

## 錢包設定、測試網設定

本文章是使用 『**2021JSDC 大會 李婷婷 講者**』的 Demo 程式碼

### 使用 infura 與區塊鏈互動

infura 可以快速的讓我們跟區塊鏈的節點互動，而不用自己去建立一個節點( Geth )來跟以太坊溝通，並且我們可以透過 infura 提供的 API 來對以太坊進行互動。

#### 進入 infura 官網創建帳號

首先先進入到 [infura 官網](https://infura.io/)並申辦帳號(官網右上角)。

1. **輸入您的 Email 帳號以及您要在 infura 使用的密碼。**

<img src='https://i.imgur.com/sfBPCHg.png' width='400' height='400'/>

2. **前往 Email 點選確認信裡面的按鈕**

<img src='https://i.imgur.com/OWShqQi.png' width='400' height='400'/>

3. **點擊 Create New Project 去創建一個專案(選擇 Ethereum)**

<img src='https://i.imgur.com/DNdkBqg.png' width='400' height='400'/>

#### 進入到 project setting 裡面

接著進入到 project setting 裡面將 `PROJECT ID` 與選擇 `ENDPOINTS` 後的 url 記到 `.env` 裡面，因為我們這邊是使用 Kovan testnet 所以選擇 Kovan。

![](https://i.imgur.com/1dkMNGd.png)

```javascript=
/* .env */
projectID = YOUR-PROJECT-ID
testnetURL = https://kovan.infura.io/v3/
```

#### 最後回來看一下程式碼

```javascript=
const Web3 = require("web3"); // 引入 web3.js
// 將 testnetURL 與 projectID 組成 （這也就是 infura 創建完 project id 後的 ENDPOINTS URL）
const rpcURL = `${process.env.testnetURL}${process.env.projectID}`;
```

### Wallet 設定錢包

接下來看一下錢包的部分，這邊我們要先建立一個錢包並將錢包的 private key 給到我們的套件中 `@truffle/hdwallet-provider`。

補充：何謂 HD Wallet

> Hierarchical Deterministic wallet (簡稱 “HD Wallet”)，是一個系統可以從單一個 seed 產生一樹狀結構儲存多組 keypairs（私鑰和公鑰）。
> seed 用方便記憶和書寫的單字表示。一般由 12 個單字組成，稱為 mnemonic code(phrase)，中文稱為助記詞或助記碼。

#### MetaMask 錢包與私鑰

1. **首先我們要先安裝 [MetaMask](https://chrome.google.com/webstore/detail/metamask/nkbihfbeogaeaoehlefnkodbefgpgknn) 並使用 MetaMask 創建一個 Account 錢包。**

 <img src='https://i.imgur.com/bFIzOxt.png' width='300' height='500'/>

2. **接著我們要匯出該錢包的 Private Key。**
步驟為：帳戶詳情 -> 導出私鑰 -> 輸入密碼後顯示私鑰

<img src='https://i.imgur.com/JD0RoXw.png' width='300' height='500'/>

3. **將錢包私鑰複製進專案 `.env` 中**
   ```javascript
   /* .env */
   privateKey = YOUR - PRIVATW - KEY
   ```

#### 最後回來看一下程式碼

```javascript=
/* index.js */
const HDWalletProvider = require("@truffle/hdwallet-provider");
// 錢包私鑰
const privateKey = process.env.privateKey;
// 將 錢包私鑰 與  rpcURL (就是上面 infura 部分的 url) 整合
const provider = new HDWalletProvider(privateKey, rpcURL);
// HDWalletProvider is compatible with Web3. Use it at Web3 constructor, just like any other Web3 Provider
const web3 = new Web3(provider);
```

### 領取測試幣

我們現在有了錢包但裡面沒有錢，所以我們要來領一些測試幣！
我們可以透過 [ https://gitter.im/kovan-testnet/faucet ](https://gitter.im/kovan-testnet/faucet)這個網站來領取 Kovan 測試幣。

1. **到網站中點擊下方 『SIGN IN TO START TALKING』按鈕**
   可以使用 Gitlab、Github、Twitter 登入

   ![](https://i.imgur.com/DFpC5d2.png)

   <img src='https://i.imgur.com/2wI9p9T.png' width='400' height='400'/>

2. **接著將 Metamask 錢包地址輸入到聊天室裡面**

   ![](https://i.imgur.com/7TwuYgy.png)

3. **看到`sent`回應後，就可以打開 Metamask 錢包看看測試幣有沒有正確到帳。**

<img src='https://i.imgur.com/biy8G1a.png' width='300' height='500'/>

#### 補充 1：請確認您目前所選擇的網路，如果不是 『Kovan 測試網路』則可以參考一下步驟去開啟。

![](https://i.imgur.com/MIAWDEi.png)

1. 點選網路中的 『新增網路』
2. 進到『設定』頁後，點選進階設定
3. 往下滑開啟 『Show test networks』

![](https://i.imgur.com/BEF7AYb.png)

---

#### 補充 2：目前找到 [https://faucets.chain.link/](https://faucets.chain.link/) 這個網站能夠一次領取較多的測試幣，而且能夠選擇不少的測試網，目前一次能夠領取 0.1 顆 ETH，而且尚未觀察到有需等待多久才能夠再領取的提示，目前是直接連續領取了 3 次。 :satisfied:

如果要領取測試網上面的 **Link token** 則需要將代幣地址加入到 Metamask 中，可以參考以下步驟。

1. **點擊 MetaMask 下方的 import tokens**

   <img src='https://i.imgur.com/DtOEwY5.png' width='300' height='500'/>

2. **查詢 token 的 address**
   這邊以 Kovan 測試網為例，可以透過 [kovan.etherscan](https://kovan.etherscan.io/token/0xa36085f69e2889c224210f603d836748e7dc0088) 上找尋到 link token 的代幣地址。

   ![](https://i.imgur.com/0uoNGoX.png)

3. **貼回到 MetaMask import tokens 中**

<img src='https://i.imgur.com/G3uk4Qn.png' width='300' height='500'/>

4. **最後就可以從 MetaMask 看到 Link token**

<img src='https://i.imgur.com/BgMgsCL.png' width='300' height='500'/>

---

## 進入 Web3.js 世界 - 餘額查詢

在上面我們已經將 測試網節點、錢包、測試幣 都設定完成了，現在就是要進入到使用 Web3.js 的部分。

首先當然就是要先初始化 web3.js，主要傳入一個 Provider ([JSON-RPC Server](https://geth.ethereum.org/docs/rpc/server)）。
這邊我覺得可以想像成，我們需要傳入一個節點，而我們是透過 infura 來幫我們與節點溝通，所以這邊可以直接傳入 infura 的 Endpoint。

而我們使用 `HDWalletProvider` 產出的 priovider，主要是將錢包與 Endpoint 綁定，而不用在去創建帳號並綁定 wallet。

```javascript=
const web3 = new Web3(provider);
```

### 查看帳戶餘額

因為我們上面提供的 provider 已經將錢包給設定完成，所以這邊我們可以直接透過 `getAccounts` 這個函式去查看『節點』下的帳戶資料（錢包地址）。

接著透過 `getBalance` 去取得所擁有的 `Wei`，接著再透過 `fromWei` 將 Wei 轉換成 ETH。 1 以太币= 10^18 Wei

```javascript=
  const accounts = await web3.eth.getAccounts(); // 取得當前節點的地址資料。
  console.log(`Account: ${accounts[0]}\n`);// 將會顯示 錢包地址

  await keypress();
  // Task 2: Get ETH balance of an address
  const balanceInWei = await web3.eth.getBalance(accounts[0]); // getBalance 取得的單位為 Wei , 1Eth = 10 的 18 次方的 Wei
  const balanceInETH = web3.utils.fromWei(balanceInWei, "ether");
  console.log(`There's ${balanceInETH} ETH in the address ${accounts[0]}\n`);

  await keypress();
```

#### 另一種方法（Provider 未綁定 wallet）

我們可以自己去建立一個 Account，透過將 Metamask 的 privateKey 傳入 `web3.eth.accounts.privateKeyToAccount` 來建立 Account，接著可以透過 `account.address` 去取得錢包地址並查詢餘額。

```javascript=
let account =  await web3.eth.accounts.privateKeyToAccount(privateKey);
// 將 account 的 address 就是我們的 錢包地址
const balanceInWei2 = await web3.eth.getBalance(account.address);
const balanceInETH2 = web3.utils.fromWei(balanceInWei2, "ether");
console.log(`There's ${balanceInETH2} ETH in the address ${account.address}\n`);
```

兩者的差別在於，透過 `web3.eth.accounts` 創建的 account 無法透過 `web3.eth.getAccounts()` 去查詢到，因為 `getAccounts` 是
查看『節點』下的帳戶資料， [參考資料](https://ethereum.stackexchange.com/questions/68577/accounts-created-by-using-web3-eth-accounts-create-dont-appear-when-web3-eth)

## 進入 Web3.js 世界 - 智能合約（概念）

在設置智能合約之前，我們要先來簡單瞭解一下【何謂智能合約?】以及一些常見的相關名詞，例如:【以太坊虛擬機 EVM】、【Remix】、【Opcodes & Bytecode】...等。

**_以下將會介紹智能合約相關名詞，如果已了解的讀者可以直接跳過這部分，往下到程式碼講解的部分。_**

### 何謂智能合約(Smart Contract)

**智慧合約**其實就是一段區塊鏈程式碼且它可以在區塊鏈上被執行，我們可以透過這些程式碼進行交易、查看餘額或與其他區塊鏈程式碼(合約)進行互動。
我們都知道區塊鏈有【不可竄改性】與【不可逆性】的特性，所以當這些程式碼被部署到區塊鏈上後，它就會變的不能再更改，而這就跟我們生活中簽合約一樣，當雙方合約簽訂後就不能夠隨意更改，如果要更改就得再重新擬定一份新的合約，簡單來說 : 智能合約其實就是一種讓合約概念能在虛擬世界中能有保障的特殊協議。

### 以太坊虛擬機 EVM

相信大家在查關於智能合約的文章時一定常常看到 **EVM** 這個詞，而 EVM 就是以太坊上的執行環境，就像 Java 會有 JVM 一樣，簡單一點可以想像成一台在以太坊上用來幫我們執行指令的電腦，

在智能合約的開發上有許多語言都可以用來開發，像是 **Solidity**、**Vyper**、**Bamboo**，而目前最熱門的、最廣泛使用的語言是 **Solidity**，但這些高階語言都無法直接被 EVM 所執行，因此需要將它們編譯成低階機器語言(low-level machine instructions)後再轉成 Bytecode 才能讓 EVM 所執行，而這些低階機器語言也被稱為 **Opcodes**。

### Opcodes & Bytecode

以太坊定義了一系列的低階機器語言(Opcodes)與 16 進字(Bytecode)的[對照表](https://ethereum.org/en/developers/docs/evm/opcodes/)，透過組合這些低階機器語言所產生的一串 Bytecode 讓 EVM 能透過這些 16 進字碼去執行任務 (ex.執行智能合約)。

Opcodes 是由 1 byte (8 bit) 所組成，且每一個 Opcodes 都有它各自代表的意義與功能，例如: `0x60` 代表 `PUSH1` (可以對照下面圖表)，然而最多只能有 256 個 Opcodes，因為 `8 bit` 的最大值為 `11111111 (二進字) => 0~255`，因此 Opcodes 的最大值為 `0xFF`。

![](https://miro.medium.com/max/2000/1*I4v8ArsePBK_iFSxgljxTg.png)
(Reference: [The Ethereum Virtual Machine — How does it work? - Luit Hollander](https://medium.com/mycrypto/the-ethereum-virtual-machine-how-does-it-work-9abac2b7c9e))

我們可以透過下面這張圖片的例子來看 Bytecode 是如何轉換成 Opcodes 以及最後輸出值為何。裡面值得注意的是 `PUSH` 開頭的這些指令，因為它們是負責將後續的值推入到 Stack 中，所以 `PUSH` 後面的 Bytecode 將不會被轉成 Opcodes，而從上面圖表我們可以發現 `PUSH` 有分成 `PUSH1 ~ PUSH32` 這代表著要推入往後數多少個 Bytecode，例如: `PUSH2` 代表推入往後兩位的 Bytecode。

![](https://miro.medium.com/max/2000/1*BhaR7mREQOIj_CzbuWVb5g.png)
(Reference: [The Ethereum Virtual Machine — How does it work? - Luit Hollander](https://medium.com/mycrypto/the-ethereum-virtual-machine-how-does-it-work-9abac2b7c9e))

### ABI

先來看看[維基百科](https://en.wikipedia.org/wiki/Application_binary_interface)如何說

> In computer software, an application binary interface (ABI) is an interface between two binary program modules. Often, one of these modules is a library or operating system facility, and the other is a program that is being run by a user.

簡單來說 ABI(Application Binary Interface) 是電腦軟體中兩個 binary 程式模組之間互相溝通的方式，而在以太訪智能合約中的 ABI 又可以稱為 Contract ABI，它是讓智能合約與 EVM bytecode 互動的 interface，就像網頁開發中我們會透過 API(Application Programming Interface)與後端程式互動，如果我們想透過 JavaScript 呼叫一些智能合約裡的 function，則可以透過 ABI 幫助我們在 JavaScript 與 EVM 之間互動。

![](https://miro.medium.com/max/1400/1*y3MlDKVKoQcEv03UrCPEHA.png)
(Reference: [Explaining Ethereum Contract ABI & EVM Bytecode - eiki](https://medium.com/@eiki1212/explaining-ethereum-contract-abi-evm-bytecode-6afa6e917c3b))

### Remix

Remix 是一個由以太坊官方所開發的智能合約 IDE，它提供線上編輯的功能，讓我們可以直接透過 [Remix 網站](https://remix.ethereum.org/) 進行開發，不需額外安裝任何開發環境，就跟我們前端常會用 [CodePen](https://codepen.io/) 或 [CodeSandbox](https://codesandbox.io/) 去寫一些 Side Project 一樣。

### 瞭解智能合約開發與執行流程

在開始看關於智能合約的程式碼之前，我們要先來瞭解一下整個智能合約的開發與執行流程。

首先使用 Solidity 語法撰寫智能合約 (可以使用上面提到的 Remix 平台進行開發)，接著將 Solidity 語法 Compile 成 Bytecode 讓以太坊虛擬機 (EVM) 能夠執行智能合約，接著透過 IPC 或 RPC 將 Bytecode 發佈到 Ethereum 上，而前端可以透過發布後產生的合約地址，與 Compile 後產生的 ABI(Application Binary Interface) 對該智能合約進行互動。

![](https://blog.fukuball.com/images/ethereum/ethereum-2-1-01.png)
(Reference: [ethereum-開發筆記-21ethereum-開發整體脈絡 - Fukuball](https://blog.fukuball.com/ethereum-%E9%96%8B%E7%99%BC%E7%AD%86%E8%A8%98-21ethereum-%E9%96%8B%E7%99%BC%E6%95%B4%E9%AB%94%E8%84%88%E7%B5%A1/))

**補充 :**

1. 如果想瞭解如何操作 Remix 推薦觀看 [開發智能合約 - 線上版 IDE 之 Remix 基礎篇 (Day09) - alincode](https://ithelp.ithome.com.tw/articles/10201750)
2. 如果想瞭解基礎智能合約語法 推薦觀看 [Day29|現實中的區塊鏈(6)：基礎智能合約語法 - lkm543](https://ithelp.ithome.com.tw/articles/10216370)

## 進入 Web3.js 世界 - 智能合約（程式碼）

有了上面對智能合約的基本知識後，現在再回來看講者的 Demo 範例是如何透過 Remix 發布智能合約以及透過 Web3.js 與 ABI 來呼叫智能合約裡的函式，廢話不多說直接開始吧～

### Remix 發布智能合約

這邊直接使用 [Remix](https://remix.ethereum.org/) 官方的智能合約範例來作測試，以下範例的程式碼意思是『賦值到變數(number)』中與從『變數(number)取出值』的範例，這邊先簡單知道這個合約在做什麼就可以了，等等我們會用 Web3.js 去觸發合約裡的這兩個函式。

```solidity=
pragma solidity >=0.7.0 <0.9.0;

/**
 * @title Storage
 * @dev Store & retrieve value in a variable
 */
contract Storage {

    uint256 number;

    /**
     * @dev Store value in variable
     * @param num value to store
     * 將 輸入的 num 存到 number 中
     */
    function store(uint256 num) public {
        number = num;
    }

    /**
     * @dev Return value
     * @return value of 'number'
     * 將 number 回傳
     */
    function retrieve() public view returns (uint256){
        return number;
    }
}
```

再要給 Web3.js 使用之前，我們需要先將這個智能合約上到區塊鏈，並且拿到這個合約的 **ABI** 與 **合約地址(contract address)**，這樣我們才能告訴 Web3.js 我們要操作哪個合約，所以我們先將寫好的智能合約 Compile 成 Bytecode 與產生 ABI，應該還記得為什麼要 Compile 吧！如果不清楚的話可以往上看『概念』的部分，簡單來說主要是讓 EVM 執行 Bytecode 以及給 Web3.js 透過 ABI 操作使用。

![](https://i.imgur.com/FnTSu5X.png)

接著將智能合約部署到到 Kovan 測試網上面，記得要選擇 **Injected Web3** 來與 MetaMask 連接，它會自動偵測到 Kovan 測試網並自動幫我們選擇，可以看到 Account 欄位會顯示我們 MetaMask 的錢包地址，等等 Deploy 時後透過該錢包扣除上架的費用。

![](https://i.imgur.com/sSZdt8o.png)

現在我們來部署一下智能合約，點擊 Deploy 後會看到右邊的 MetaMask 跳出來並且顯示要扣除多少的 gas fee，所以在部署之前要記得先去領取測試幣，不知怎麼領取可以往上滑到『錢包設定』的部分。

<img src='https://i.imgur.com/jIv5f5E.png' width='300' height='500'/>

部署完成後可以透過畫面左下方的 **Deployed Contracts** 來取得部署完後的合約地址，將合約地址與剛剛上面的 ABI 複製回我們的 Demo 程式碼中。

![](https://i.imgur.com/2xADzrI.png)

```javascript=
// Remix 透過將智能合約 Compile 後產生的 ABI
const abi = [
    {
      inputs: [],
      name: 'retrieve',
      outputs: [
        {
          internalType: 'uint256',
          name: '',
          type: 'uint256',
        },
      ],
      stateMutability: 'view',
      type: 'function',
    },
    {
      inputs: [
        {
          internalType: 'uint256',
          name: 'num',
          type: 'uint256',
        },
      ],
      name: 'store',
      outputs: [],
      stateMutability: 'nonpayable',
      type: 'function',
    },
  ];

// 透過 Remix 發布(Deploy)到區塊鏈上後的 合約地址
const contractAddress = "0x7e09C5fBC8828635eFD3C03ded05C29bAc645B68";

```

### Web3.js 操作智能合約

有了『ABI』與『合約地址』後現在可以透過 Web3.js 來操作合約了，首先透過 `web3.eth.Contract` 來與以太鏈上的智能合約連結，它接收兩個參數，一個是合約的 ABI、一個是合約地址，因此我們可以把剛剛所產生的 ABI 與 **Deployed Contracts** 後的合約地址傳入 `web3.eth.Contract` 函式中讓它們綁定。

```javascript=
const contract = new web3.eth.Contract(abi, contractAddress);
console.log("Smart contract functions: ", contract.methods);

// log
Smart contract functions:  {
  retrieve: [Function: bound _createTxObject],
  '0x2e64cec1': [Function: bound _createTxObject],
  'retrieve()': [Function: bound _createTxObject],
  store: [Function: bound _createTxObject],
  '0x6057361d': [Function: bound _createTxObject],
  'store(uint256)': [Function: bound _createTxObject]
}
```

我們可以從 log 中看到它印出我們剛剛傳入的 ABI 裡面的 name，可以試著增減 ABI 結構或是更改裡面的 name 看看，會發現 log 也會更著改變，這代表我們呼叫的函式是與 ABI 結構互相依賴的，且 ABI 的 name 需要與智能合約的 function 名稱相對應，否則將會報錯，接下來就來呼叫一下智能合約裡的 function。

```javascript=
// 呼叫合約裡的方法 retrieve (因為還沒有設定值 所以這邊會是 0)
const initialValue = await contract.methods.retrieve().call();
console.log("\nInitial value:", initialValue);

// log
// Initial value: 0
```

上面我們呼叫了 `call()` 這個函式來告訴 EVM 執行我們智能合約裡的 `retrieve` 函式，因為不改變智能合約裡的狀態(變數)，所以這邊直接使用 `call()` 就可以了，這可以想像是我們在呼叫 API 會分 GET / POST 一樣的道理，單純取資料用 GET 要新增資料進資料庫用 POST ...等等。

最後我們來更改智能合約裡的狀態(改變合約裡的變數 number 的值)，因為要更改狀態所以要使用 `send()` 函式並且傳入錢包地址，因為當我們要更改區塊鏈上的狀態時(新增紀錄到區塊鏈)，需要花費一些 Gas Fee 作為手續費，所以要傳入錢包地址到參數中。

```javascript=
const newValue = 700;
console.log(`Setting the value to ${newValue}...`);
const txReceipt = await contract.methods
    .store(newValue) // 呼叫智能合約裡的 store 函式，並且將 700 賦值到智能合約裡的變數(number)
    .send({ from: accounts[0] }); // 執行合約

console.log('呼叫完 sand 後錢包餘額：',web3.utils.fromWei( await web3.eth.getBalance(accounts[0]), 'ether'))

const updatedValue = await contract.methods.retrieve().call();
// 再次查看 合約裡變數(number) 現在的值
console.log("Value afterwards:", updatedValue);

```

![](https://i.imgur.com/EWyEIZo.png)

## 結論

這次跟著講者的程式碼實際的一步一步實作一遍，過程中也順便補足一些基本知識，整體大概只能說是順過了整個流程，從智能合約的部署產生 ABI、Bytecode 到領取測試幣並透過 Web3.js 操作整個智能合約，雖然只是一個簡單的例子，但對智能合約的部分還是有了更深一點的理解。

本次的範例是一開始就將錢包地址寫死，並且直接綁定到 Account 中，如果想要透過點擊按鈕來讓使用者連接自己的 MetaMask 的話可以[參考此文章](https://medium.com/valist/how-to-connect-web3-js-to-metamask-in-2020-fee2b2edf58a)，接下來也會花一些時間去了解 Solidity 的部分，預計到時會再整理個 Solidity 語法的筆記，到時還請各位大大們指教了。

#### 以上就是這次實作的結論，如果有新的見解或新的發現會再繼續更新這篇文章，如果有任何講錯的地方或冒犯的部分也歡迎大家提出來告訴我一聲。

### 謝謝觀看。

#### Github : [https://github.com/librarylai/JSDC-web3-demo](https://github.com/librarylai/JSDC-web3-demo)

## Reference

1. [開發 Dapp - web3.js 初始化 (Day05) - alincode](https://ithelp.ithome.com.tw/articles/10203431)
2. [智能合约简介 - ethereum.org](https://ethereum.org/zh/developers/docs/smart-contracts/)
3. [不可不知 何謂「智能合約」？ - 加沛](https://blockcast.it/2018/03/11/what-is-a-smart-contract/)
4. [開發智能合約 - 線上版 IDE 之 Remix 基礎篇 (Day09) - alincode](https://ithelp.ithome.com.tw/articles/10201750)
5. [Day29|現實中的區塊鏈(6)：基礎智能合約語法 - lkm543](https://ithelp.ithome.com.tw/articles/10216370)
6. [ethereum-開發筆記-21ethereum-開發整體脈絡 - Fukuball](https://blog.fukuball.com/ethereum-%E9%96%8B%E7%99%BC%E7%AD%86%E8%A8%98-21ethereum-%E9%96%8B%E7%99%BC%E6%95%B4%E9%AB%94%E8%84%88%E7%B5%A1/)
7. [The Ethereum Virtual Machine — How does it work? - Luit Hollander](https://medium.com/mycrypto/the-ethereum-virtual-machine-how-does-it-work-9abac2b7c9e)
8. [How to Connect Web3.js to MetaMask in 2022 - Alec M. Wantoch](https://medium.com/valist/how-to-connect-web3-js-to-metamask-in-2020-fee2b2edf58a)
