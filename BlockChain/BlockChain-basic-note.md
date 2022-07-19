# 【筆記】區塊鏈相關 - 基礎知識

###### tags: `筆記文章`

###### 作者：Library Lai、Zack Chu

## 相關名詞與平台介紹

本篇主要是筆者們對區塊鏈各名詞不懂而找資料所做的筆記，資料來源主要是以各大文章、平台、官方文件等等...，所有的連結皆會放在最下面，如有任何不妥再麻煩告知一聲。thx~~~

## 何謂區塊鏈

區塊鏈常聽到的專有名詞是『去中心化分散式資料庫』，或是說『去中心化分散式帳本』，最近從區塊勢 Podcast 上聽到另一個更貼近生活上的解釋，是說每一個區塊鏈就像是一個『城市』，以太鏈就像是紐約、BSC 鏈就像是舊金山，每一個區塊鏈都有自己的一套規則、治理機制，而彼此之間可以透過一些方式來達成跨鏈，最簡單的像是透過交易所，或其他去中心化橋的服務來達成。

之前也從某一集 Podcast 上聽到說，區塊鏈為什麼會被大家發現，其實是因為比特幣先被大家發現，當年中本聰發布了一篇比特幣的白皮書，裡面寫到『比特幣是點對點的電子現金系統』，大家開始被大家發現，而大家也開始好奇比特幣背後的機制、原理，然好才發現了區塊鏈這項技術。

區塊鏈的發展從 『1.0 的加密貨幣』大家開始瘋挖礦（挖比特幣）或是打照自己的鏈發展其他的加密貨幣，或是 Fork 比特幣改寫的山寨幣，這個時期主要是很多加密貨幣開始陸陸續續地出現，但在區塊鏈 1.0 時能做到的實質交易應用很少，比較像是單純擁有這個虛擬貨幣，交易方式比較像『我把幣傳給你、你把幣傳給我』。

到了區塊鏈 2.0 時『智慧合約』的發展，能將程式編寫進區塊鏈中並且自動被執行，開啟了各種區塊鏈上的應用，像是：交易所、DiFi、NFT、GameFi 等應用，這讓整個區塊鏈開始蓬勃發展，開始被大家廣為知道。NFT 的出現更讓許多數位世界上的創作，像是：圖片、音樂、影片，被賦予了價值，變成了數位資產。

### 區塊鏈的特色：

1. 去中心化：區塊鏈是由眾多節點共同維護資料庫，而不是由某一個公司或機構來管理。
2. 不可篡改性：區塊鏈上的資料不能被隨意更改，主要原因在於區塊鏈獨有的技術，能將經驗證後的資料放進區塊鏈當中。
3. 匿名性：區塊鏈上的節點得以不具名參與其中，，只要不跟別人透露，就沒有人能夠知道節點背後的人是誰。

### 區塊鏈發展

1. 區塊鏈 1.0： 主要聚焦在「加密貨幣」的發展以及分散式帳本的應用，礦工共同維護資料庫，不須再經過第三方機構的審查，即可完成所有的交易過程，顯現出『去中心化』的系統。
2. 區塊鏈 2.0： 主要聚焦在「智慧合約」的發展，透過以太坊底層的技術，能將程式編寫進區塊鏈中並且自動被執行，造就了更多更廣泛的應用，例如：交易所、NFT、GameFi...等應用。
3. 區塊鏈 3.0：主要聚焦在「IOAT」的發展，可以想像成區塊鏈技術與物聯網技術的結合。

## NFT

NFT(Non-Fungible Token) 簡單來說，主要是讓我們能夠在數位世界裡知道該東西的所有權人是誰，例如：網路上面的一張圖片，透過 NFT 技術讓我們能夠知道這張圖的版權是誰的。
每個 NFT 上都有一個編碼且被記錄在區塊鏈上，它是獨一無二且不可被竄改的。NFT 的出現讓許多藝術創作（像是：畫、音樂、圖片、影片...等）在數位的世界裡被賦予了價值。

## Web3.js

Web3.js 是一套 librars，讓我們可以透過 Http, IPC, webSocker 來與 Ethereum 的節點互動，它有使用者社群，並有與 Ethereum Foundation 相關的維護人員，它透過 JSON-RPC 通訊協定與 Ethereum 節點通訊。

Web3.js 這套 library 是包含 Ethereum 生態系模組的集合，主要有一下幾個。

- `web3-eth`is for the ethereum blockchain and smart contracts.
- `web3-shh` is for the whisper protocol, to communicate p2p and broadcast.
- `web3-bzz` is for the swarm protocol, the decentralized file storage.
- `web3-utils` contains useful helper functions for Dapp developers.

#### 安裝

我們可以透過以下幾種方式將 web3.js 安裝進我們的專案

- npm: `npm install web3`
- yarn: `yarn add web3`
- pure js: link the `dist/web3.min.js`

#### 建立一個 Web3.js instance

在創建 Web3.js 的 instance 之前，我們要先設置一個可以被以太鏈支援的 Provider，有些支援以太坊的瀏覽器像是 Ｍ etaMask 可以使用 `window.ethereum` 來取得來查看目前瀏覽器是否有支援。

#### Provider 取得

1. 我們可以透過 `web3.givenProvider` 來取得當前瀏覽器設置的原生 provider
2. 使用 infura 建立專案取得 Endpoint
3. 進階 - 透過 HDWalletProvider 將 metamask 與 infura 綁在一起 （推薦）

_注：對 Ｍ etaMask 不懂的話可以先往下去看關於 Ｍ etaMask 的介紹_

```javascript=
// In Node.js use: const Web3 = require('web3');
/*
 * givenProvider 會返回當前瀏覽器設置的原生 provider，
 * 如果沒有則返回 null，所以可以當作是一個判斷，
 * 如果沒有則可以手動去設定一個 provider
 */
const web3 = new Web3(Web3.givenProvider || "ws://localhost:8545");
```

## Kovan

Kovan 是以太坊自行發行的一個測試網絡（test net）且採用 POA 共識機制，由於區塊鏈有著不可篡改特性，因此智能合約必須要保證其完整性才能上鏈，否則會導致不可逆的狀況發生，對此以太坊發佈了四種測試網絡（其中正在運行的有三種），分別為：Ropsten, Rinkeby, Kovan。

### Kovan 測試幣取得

1. [ https://gitter.im/kovan-testnet/faucet ](https://gitter.im/kovan-testnet/faucet)
2. [https://faucet.kovan.network/](https://faucet.kovan.network/)

其他測試網資訊可參考 [ethereum.org - networks](https://ethereum.org/en/developers/docs/networks/)

## 區塊鏈共識機制

共識機制說白了就是一種信任協議，就是解決分布式帳本有誰來記以及確保所有的帳本都一致的技術，因為在區塊鏈上如果每個人都隨意寫上紀錄的話，肯定會造成帳本的混亂，所以就得依靠『**共識機制**』這一項機制，讓所有的參與者對每筆資料達到共識、獲取一致性，這也讓所有在區塊鏈上的紀錄更有安全性。

**PoW、PoS 與 DPoS 是為區塊鏈三大共識機制**
區塊鏈作為去中心化體系，必須要有一款參與者都能接受的底層機制以維持整個生態系運作，類似憲法的概念，規定什麼樣的作為會受到獎勵或懲罰。

- **PoW：工作量證明**
  Pow 是最早出現的共識機制，想要記帳的人彼此競爭，透過自身電腦運算能力去解題，也就造成了計算能力越高的電腦答對率約高，因此導致大家拼命再增加電腦的運算效能等，例如：國外就有公司在冰島設立比特幣挖礦廠房，每一台礦機都是一次組合 10 多張顯卡進行平行運算，增加答對率(挖礦)。

  PoW 機制是對於先解題運算出來的人給予獎勵，最著名的就是比特幣(BitCoin)挖礦機制，所以導致大家都拼命提升效能，這也導致了消耗大量能源的問提，例如：電力消耗。
  例如：BTC

- **PoS：權益證明**
  因為 PoW 算法造成大量的能源消耗，而且也漸漸的變成無法個人或小團體就能玩的，財力無法與大公司或專門挖礦的公司較量，講白點就是打不贏、算不贏，所以有了 PoS 的新共識機制。

  PoS 是透過持幣數量與時間作為獎勵的依據，當你持有幣的數量越多，就擁有越高的概率來取得記帳的權利，避免 PoW 的過度耗能競爭，不過此類共識機制可能會造成富人更富的情況發生。
  例如：ETH

- **PoA：權威證明**
  PoA 一詞是由以太坊聯合創始人、以太坊前技術專家 Gavin Wood 於 2017 年提出。
  PoA 算法使用的是身份的價值，區塊和交易都是由已被認可過的參與者來驗證，節點需要先通過身份核驗，用戶需要公開自己的身份，通過放置這個身份來獲得擔保網絡的權利，從而換取區塊獎勵，目前用於 Kovan 以太坊的測試網絡（testnet networks）之一。

  **成爲 PoA 驗證者的三個基本條件：**

  1. Formal identification on-chain for validators;( 正是驗證過身份在鏈上 )
  2. Eligibility based on certain criteria, including but not limited to, association with the organization or good reputation, etc.; ( 符合某些標準，例如：好的名聲 )
  3. Complete conformance to defined procedures required for producing blocks and validating on the network. (建立權威的檢查和程序應該保持一致)

  PoA 的缺點在於有悖於去中心化理念，因為參與區塊鏈的驗證者是有限的 ; 優點在於可以防止作惡，因為能成為驗證者的人都是經過審查的。

- **DpoS：代理權益證明（又稱股份授權證明**）
  DpoS 作為第三代的共識機制，保留了 PoS 節能的優點，並引入了代議民主的概念，透過減少節點的數量，除了能夠達成比 PoS 更低的能耗，更是解決了交易確認的時間過長的問題。可以把 DpoS 想像成一間公司，當中的超級節點就是公司的董事與大股東，他們可以參與大公司決策。
  例如：FIL

- 結論：有共識機制才能讓該區塊鏈生態適性發展，參與者也才能在有一定的規準之下進行相關項目開發及獲取利益。

## Geth（以太坊節點）

- 節點介紹
  目前最受歡迎的以太坊節點為 Geth 和 Parity，不同在於選擇的編程語言：Geth 使用 Golang，而 Parity 使用 Rust。用戶可通過安裝 Geth 來接入以太坊網絡併成為一個完整節點。Geth 也可作為一個 HTTP-RPC 服務器，對外暴露 JSON-RPC 接口，供用戶與以太坊網絡進行互動連結，Geth 的使用需要基本的命令行基礎，其功能相對完整。
- Geth（go-ethereum）基本功能
  Geth 是為最常用的以太坊客戶端（節點）之一。為了與區塊鏈進行通信，必須使用區塊鏈客戶端，與其他客戶建立 p2p 通信信道，簽署和廣播交易，挖礦，部署和與智能合約交互等。客戶端通常被稱為節點。基於以太坊黃皮書，任何人都能夠以他們認為合適的語言創建自己的以太坊節點實現。

- geth 使用方針 1. 安裝 Geth (https://geth.ethereum.org/docs/install-and-build/installing-geth) 2. 利用終端機啟動 Geth 3. 選擇 Geth-light 節點，並訪問 Geth 控制檯 4. 現在你已經創建了一個節點，你可以通過在終端中打開一個新選項卡並運行以下命令來訪問它：geth attach 5. 這將把 Geth 控制檯（一個用於與區塊鏈通信的 Javascript 環境）連接到你的運行節點。這可以在完全客戶端模式和輕模式下完成。 6. 打開控制台，輸入：web3.eth.blockNumber 7. 你應該輸出一個數字（例如 5631487），表示以太坊網絡的當前塊號。 8. 你可以通過在終端中運行以下命令來實現：geth account new，用以創建一個新帳戶 9. 完成後，它會詢問你輸入密碼，以保護你的帳戶。確保使用安全密碼並安全存儲。運行 geth account new 時 Geth 所做的是更新 Geth 數據目錄中的文件（Geth 存儲所有必要數據的目錄，包括塊和塊頭信息）。目錄在每個平臺的位置：
  macOS：~/Library/Ethereum
  Linux：~/.ethereum
  Windows：%APPDATA%\Ethereum 10. 當你啟動 Geth 時，客戶端會自動在端口 8545 啟動 RPC 服務器。你可以通過使用 web3js 或 web3j 等庫連接到 localhost:8545 或使用 curl 或 wget 手動調用它來訪問此端口上的 RPC 服務器及其方法。
  要了解如何與正在運行的 Geth 實例（在啟動你自己的區塊鏈時是私有的，或在上面的說明中公開）的外部工具的連接，請參閱此文章。

## 山寨幣（AltCoin）、加密貨幣（Cryptocurrency）與代幣（Token）

當我們在看區塊鏈文章時或是聽一些 podcase、youtube 等等節目時，一定會聽過 『xx 幣, xx token, xx 山寨幣』 等等的名詞，然而究竟『幣(coin)』、『代幣(token)』、『山寨幣』到底怎麼區分，而各別又代表什麼意思呢？相信一定多人會有這個疑問，而這一部份將會簡單介紹這三個專有名詞。

### 加密貨幣（Cryptocurrency）

加密貨幣(Cryptocurrency)就是透過密碼學原理，以確保其交易的安全性的一種數位資產，也可以簡稱 Coin，它們就如同貨幣一樣可以用來交易、轉賬、儲值和充當記帳單位，可以想像成我們的新台幣，它們的特點在於每種貨幣都擁有自己的區塊鏈，且在自己的鏈上運行。

### 代幣（Token）

代幣(Token)基本上都是當作某種功能性(Utility Tokens)或代表某種資產(Asset Tokens)而被創造出來的，像是當作區塊鏈應用程式(Dapp)的交易媒介，而大部分通過首次代幣發行(ICO)發行的都是 Token，它們基本上沒有自己的區塊鏈，都是依賴在別人的原生區塊鏈上面(ex. Ethereum 區塊鏈)

代幣(Token)與貨幣(Coin)最簡單的區分方式就是看它是否發行在某一個公鏈上面，舉例來說，透過官網或是白皮書等等文件一定會看過這句話 `xxx Shards are an ERC 20 governance token`，這代表該幣是符合【ERC-20 規範】且可以在乙太鏈上流通的代幣(Token)，例如: AXS。

根據 [Coinmarketcap](https://coinmarketcap.com/tokens/views/all/) 數據顯示，在加密貨幣市場上有將近 9 成的代幣都來自於乙太坊區塊鏈上，由此可知乙太坊區塊鏈可以說是所有代幣的母體。

### 山寨幣（AltCoin）

山寨幣主要是指比特幣的代替貨幣，大多數的山寨幣都是 fork 比特幣的專案，因為比特幣的專案是開源且免費的(Bitcoin’s open-sourced)，所以才被其他社群或是團隊拿來修改底層代碼（ex.改發行數量、規則等），創造出構出具有不同特色的、不同用處的新幣。
所有山寨幣的共通點是，它們都擁有自己獨立的區塊鏈並且在自己的鏈上運行(ex.交易)。

## ICO、IPO 與 IEO

- ICO（Initial Coin Offerings），又稱為首次代幣發行
- IPO（Initial Public Offerings），首次公開募股（股票）
- IEO（Initial Exchange Offerings），首次交易發行（由交易所主導的 ICO）
- 三種模式不同之處：
  IPO 根據投資人購買股份的數量給予公司所有權，而 ICO 可能只給投資人特定項目的權利，而不是公司所有權。IPO 的財務資料根據交易所規則來發佈，而 ICO 是將這些資料透過區塊鏈的方式公開，或者按照 ICO 白皮書與投資人的協議進行公開。

  透過 IPO 方式推出的公司必須納稅，投資人也必須繳納資本利得稅；而 ICO 公司可能不需要繳稅，只有投資人需要繳納資本利得稅。IPO 是一次性銷售，在確定條件、定價等過程中會有多家機構參與，但 ICO 可以多輪募資。

  ICO 的開放時間往往從幾周到一個月不等，有些 ICO 的開放時間更長，而且 ICO 籌資可能會在多個場合進行，不像 IPO 是一次性事件。證券交易所和透過 IPO 上市的公司受到嚴格監管，而 ICO 發起的交易所則不受監管。

- 整體來說，ICO 是一種籌措資金的方式，一家公司希望籌集資金來創建一個新的虛擬貨幣或服務，就會推出 ICO 來募集資金。投資人透過 ICO 購買代幣就可以先獲得這個虛擬貨幣的所有權，一旦未來這個虛擬貨幣的價值上漲，之後就有很大的獲利空間。

  ICO 不受美國證券交易委員會(SEC)等金融當局的監管，因此若是被詐騙，這些資金可能永遠都無法追回。ICO 與 IPO 完全不同，ICO 發行虛擬貨幣、IPO 則是發行股份。參與 ICO 前建議先完整了解加密貨幣市場，並控制投入的部位規模不宜過大。可以用大間的虛擬貨幣交易所參與 ICO。

## Infura

Infura 是一種 IaaS（Infrastructure as a Service）產品，目的是為了降低訪問 Ethereum 的門檻，它提供一些 API 例如：Ethereum API、WEBSOCKETS API、IPFS API 幫助我們更方便的去跟 Ethereum 主網或測試網節點對接，而不用自己去建立一個節點( Geth )來跟 Ethereum 溝通。

當我們在使用 Infura 的 Ethereum APIs 時，需要在 request URL 上提供一個有效的 Project ID，且可以加上 `--user :YOUR-PROJECT-SECRET` 做基本的身份驗證，對 request 增加有額外的保護 。

```javascript=
curl --user :YOUR-PROJECT-SECRET \
  https://<network>.infura.io/v3/YOUR-PROJECT-ID
```

- **_對 `--user` 不太理解的的可以參考 [這個](https://stackoverflow.com/questions/36292406/what-does-user-mean-with-curl)_**
- **_對 `curl` 不太理解的的可以參考 [這個](https://blog.techbridge.cc/2019/02/01/linux-curl-command-tutorial/)_**

選擇 `network` 可透過以下列表中的 Ethereum client provider 或 IPFS endpoint 來做選擇，請記得要將 `YOUR-PROJECT-ID` 改成你 Dashboard 的 Project ID

![](https://i.imgur.com/DkSWc30.png)
( reference: infura.io )

現在來嘗試呼叫一個 JSON-RPC Methods，這邊直接節錄官方的例子來說明。**（發送一個 POST 的 API 格式為 JSON，且使用 ethblockNumber 這個 JSON-RPC Methods，network 位置為乙太鏈主網。）**

```
$ curl -X POST \
-H "Content-Type: application/json" \
--data '{"jsonrpc": "2.0", "id": 1, "method": "eth_blockNumber", "params": []}' \
"https://mainnet.infura.io/v3/YOUR-PROJECT-ID"

// Result
$ {"jsonrpc": "2.0","result": "0x657abc", "id":1}
```

## MetaMask

### Buy, store, send and swap tokens

MetaMask （小狐狸）是用於與以太坊區塊鏈進行互動的軟體加密貨幣錢包，可用作瀏覽器擴展程序和移動應用程序，提供`key vault`、`secure login`、`token wallet`、`token exchange`等互動功能，且主要用來管理加密貨幣資產。

### Explore blockchain apps

MetaMask 提供簡單且安全的方式讓我們可以方便的連接區塊鏈應用程式，讓我們可以在去中心化網站中互動。

### Own your data

MetaMask 會在你的裝置上生成密碼與金鑰，所以只有你能查看你的帳號與資料，你可以自己選擇哪些要分享哪些是要保密的內容

### What is EIP-1559

EIP-1559 改變了 Ethereum 支付交易費用的市場機制，因為在過去 Gas(交易費用)浮動很大，且是以一個拍賣機制來確定 Gas 價格，所以當交易處於顛峰時期的狀態下，用戶就必須增加更高的 Gas 費以用來激勵礦工來接受自己的交易，Gas 的浮動大也造成費用難以預測。

EIP-1559 廢除了拍賣制度改用固定交易費，交易費會隨著鏈上的流量而變化
(>50% 則增加 ; <50%則減小)，且 EIP-1559 將手續費結構改成，每筆交易的 GAS 費會被分為基礎費和礦工收益小費，且會銷毀部分的基礎費，如果想要交易優先被執行，則可以提供額外的『tips』支付礦工當費用。

## Remix

Remix 是一個由以太坊官方所開發的智能合約 IDE，它提供線上編輯的功能，讓我們可以直接透過 Remix 網站 進行開發，不需額外安裝任何開發環境，就跟我們前端常會用 CodePen 或 CodeSandbox 去寫一些 Side Project 一樣。

## 區塊鏈板塊

在講到區塊鏈之前，我們必須先知道區塊鏈各種不同的「鏈」，每種類型的鏈又有不同的意義與用途，在一開始，我們可以先理解比較大宗的「公鏈」、「私鍊」以及「聯盟鏈」（如下圖）：
![](https://i.imgur.com/evfXBkJ.png)

- 各大板塊
  加密貨幣與股市非常像，會有許多不同類型的幣，就像是股票市場常聽到食品、塑膠玻璃等等類型股票。因此每種幣的特性本質都會有不同功能，例如底層公鏈、基礎鏈、側鏈概念、基礎數字貨幣、匿名貨幣、IFO 概念、智能合約、跨鏈等類型。

  也就是因為這些類型，在交易市場上就會有不同類型的供需，會因為各種幣的利多、利空造成幣價會有上漲、下跌情形，所以我們要利用這些板塊賺錢，這些板塊都會出現領頭羊。

  - Uni 是 Eth 上的 Dex 領頭羊，接著後面是 Sushi、1inch、dodo
  - Cake 是幣安鏈上的頭部去中心化交易所
  - Dydx 是 Eth 鏈上的 L2 合約交易所
  - AXS 是遊戲鏈上的領頭羊(Gamefi)

  抓出該板塊領頭羊、優先買他，再去評估其他類似項目、發現他的優點、知道領頭羊的缺點。這隻就有機會爆漲

  - 加密貨板塊查詢：https://www.feixiaohao.co/concept/

- 加密貨幣新聞面
  資訊獲取的速度，能大大決定你的金錢獲利、損失幅度，因為資訊傳遞有利不力消息都會左右價格變動。

  會在幣圈炒幣的投資人，資訊獲取能力都不差，因為動動手就可以交易加密貨幣，再來沒有漲幅限制所以價格崩的會非常快跟誇張，相對來說，如果利好消息，也會一直漲的很誇張。許多有心人士都會新聞炒作利用這點去獲利，也有可能放不好的消息去價格暴跌。

  例如加密貨幣 在本月 29 號會跟某某銀行簽訂合約，利用其區塊鏈技術進行合作，這就是利好，短期會漲。例如大陸要管制加密貨幣，這就是利空消息。幣圈市場投資人能接觸加密貨幣市場，基礎資訊能力都不差所以你必須比多數人資訊獲取更快

  不管利多利空新出來 都會有時間醞釀傳播期，也就是訊息到達投資人上的快慢、約 1-7 天，醞釀期，評估手上資訊的廣度 去推敲別人知不知道。

  - 如何及時獲取幣的動態
    加密貨幣新聞網

    巴比特(新聞)
    https://www.8btc.com/

    Aicoin(新聞、分析)
    https://www.aicoin.com/news/all

- 加密貨幣籌碼面
  籌碼面，就是每種幣的發行數量，發行數量攸關於幣的價格，發行數量\*單顆現價=幣總市值

  通常幣有兩種：「總幣量」以及「流通幣量」，因為初始幣的設定模式不同、有人用挖的、有人打遊戲可以得、鎖倉，所以加密貨幣市值。

  流通幣數量\*單顆現價=幣市值
  幣的流通數量 "越多" 幣的單顆現價 "越低"
  幣的流通數量 “越少” 幣的單顆現價 “越高”

  ex
  ETH 流通量 1.17 億
  單顆現價 3000 USD
  總市值$352,799,906,561

  XRP 流通量為 467.5 億
  單顆現價 0.94 USD
  總市值$$44,429,870,225

  由上得知
  代幣量越少單顆價值越高相對交易手續少
  代幣量越多單顆價值越小相對交易手續高

  代幣量越少交易所"掛單價格"間隙大 幣量少 掛單價格間隙大 拉盤容易(例如 ETH)
  代幣量越多交易所掛單價格間隙小小至一聰拉盤困難(例如 XRP)

  比特幣的市值永遠市最大的、因為一開始誕生就是創世神
  接著才有其他加密貨幣、所以任何一支幣的市值絕對不會超過比特幣

- 幣種分級與市場分級
- 幣種分級

  - 主流幣種(一級市場)

    - 通常是排名前 100 大的幣種
    - 是公鏈性質
      例如 eth、bnb、sol 這些是公鏈幣、
      下面會帶側鏈
      eth 下面有 uni、axs、dydx。bnb 下面有 cake、bake
    - 通常可以開槓桿合約
    - 已上線交易所 10 家以上
    - 已上線幣安火幣歐易 COINBASE

  - 分流幣種(二級市場)

    - 二級市場通常都是功能型代幣
    - uni 是基於 eth 網路上的去中心化交易所
    - bake 是基於 bnb 網路上 NFT 去中心化交易所
    - 幣種分類有 /BTC /ETH/ /USDT
    - 三大交易兌 也屬於此市場
    - 幣種排名在 200 名以內 常常上去百名之內又跌出百名
    - 此段幣種會處於二級交易所內 還沒上主流大型交易所

  - 支流幣種(三級市場)
    - 剛完成募資 上交易所
    - 出生半年內的幣種
    - 只有單一交易兌 /USDT
    - 上線交易所不超過三家

- 市場分級，加密貨幣交易所區分兩種

  - 中心化交易所 CEX（通常會拿比特幣交易量排名去做分級）

    - 一級交易所
      幣安、火幣、歐易、bitfinex、coinbase、ftx
    - 二級交易所:
      GATE 芝麻、MXC 抹茶、LBANK
    - 三級交易所:
      不知名交易所、此類因詐騙太多、交易已經直接轉移到"去中心化交 易所 DEX"原因是需要註冊，審核 KYC，還不一定可以出金所以全都 轉移去 DEX 免審核

  - 去中心化交易所(DEX)
    - 基於 ETH 網路上的 DEX:
      UNI、1INCH、SUHSHI、DODO
    - 基於 ETH 網路上的 NFT-DEX:
      OPENSEA
    - 基於 ETH 網路上的合約 DEX:
      DYDX
    - 基於 BSC 網路上 DEX:
      CAKE、MDX
    - 基於 BSC 網路上的 NFT-DEX:
      BAKE、BUNNY

- 接下來 SOL、FTX、DOTFIL、AXS，這些都會在獨立發展出自己的 DEX 交易所，因為他們都是屬於公鏈性質、公鏈才有辦法建交易所。過去的老幣：NEO QTUM EOS ADA 也有能力自己做交易所，但是老了走不動
- 去中心化交易所，意味無人管理只能只用 tokenpocket、狐狸這類錢包。登入 dex 交易所進行交易

## Reference

1. [區塊鏈 Blockchain – 以太坊測試網絡 Ethereum Testing network
   WeChat](https://www.samsonhoi.com/677/blockchain-ethereum-testing-network)
2. [PoW、PoS 與 DPoS 區塊鏈三大共識機制 - ACE Exchange
   Follow](https://medium.com/ace-exchange/pow-pos-dpos-blockchain101-45d18e3108e4)
3. [區塊鏈共識機制 PoS、PoW、PoA 的區別 - 每下頭條 風口邊的豬](https://kknews.cc/zh-tw/tech/562rbn8.html)
4. [PoS 讓富人更富？ 詳細對比 PoW 和 PoS 機制下規模經濟盈利率](https://news.cnyes.com/news/id/4708274)
5. [Gavin Wood 提出的 PoA 有哪些區塊鏈在用？優劣勢何在？ - 鏈聞](https://www.chainnews.com/zh-hant/articles/824457289655.htm)
6. [開發智能合約 - 用戶端 Geth, Parity (Day08)](https://ithelp.ithome.com.tw/articles/10201462)
7. [Geth 介紹及如何運行以太坊節點](https://codertw.com/程式語言/687432/)
8. [ICO 是什麼意思？如何參加 ICO？買虛擬貨幣 ICO 有風險嗎？](https://rich01.com/what-is-crypto-ico-risky/)
9. [Infura：一键接入以太坊 - TurkeyCock](https://blog.csdn.net/TurkeyCock/article/details/85103434)
10. [三分鐘讀懂 EIP-1559 對以太坊經濟模型的改變 - 鏈聞](https://news.cnyes.com/news/id/4697936)
11. [加密貨幣 101：貨幣 (Coin) ？代幣 (Token) - GENESIS BLOCK](https://genesisblockhk.com/zh/whats-a-token/)
12. [【幣圈淺談】Coin vs Token-幣與代幣 - antonsteemit](https://steemit.com/cn/@antonsteemit/coin-vs-token)
13. [ALTCOINS VS. TOKENS: WHAT’S THE DIFFERENCE? - Aziz, Master the Crypto Founder](https://masterthecrypto.com/differences-between-cryptocurrency-coins-and-tokens/)
14. [私有鏈、公有鏈和聯盟鏈有何區別？](https://academy.binance.com/zt/articles/private-public-and-consortium-blockchains-whats-the-difference)
15. [【區塊鏈白話文】 區塊鏈 是什麼？區塊鏈產業版圖介紹，熱門加密貨幣平台總整理！ - chihyuan](https://www.stockfeel.com.tw/%E5%8D%80%E5%A1%8A%E9%8F%88-%E5%8A%A0%E5%AF%86%E8%B2%A8%E5%B9%A3%E5%B9%B3%E5%8F%B0-%E6%AF%94%E8%BC%83/)
