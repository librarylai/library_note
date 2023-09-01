# 【筆記】前端 CI/CD 部署到 GCP 系列(一) - 建構篇(Next.js、Docker、Artifact Registry、Cloud Run)

###### tags: `筆記文章`

![](https://hackmd.io/_uploads/r19Eefr6h.png)

> **本文開始前提醒：** >**本篇主要以『手動』的方式來部署到 Cloud Run 上面，預計下一篇才會用 Github Action 的方式來自動化部署。**

## 前置作業 - 建立 Next.js 專案

### 方法一：使用指令 Next 建立專案

請輸入以下指令，基本上全部選 `Yes` 即可，因為這邊只是要建立一個 Next.js 專案而已，並不會對裡面內容進行任何操作。

```
npx create-next-app@latest
```

### 方法二：Clone Next 官方範例

[with-docker](https://github.com/vercel/next.js/tree/canary/examples/with-docker) 這個是 Next 官方使用 Docker Image 來部署到 Google Cloud Run 的範例專案，因此可以直接 Clone 該專案下來實作即可。

> 如果您是 Clone 該專案的話，等等下方的 『**Dockerfile 可以直接沿用官方範例**』的即可，可以『**直接往下跳到 Google Artifact Registry 等相關 GCP 內容**』

## 使用 Docker 來將 Next.js 專案容器化

首先我們先使用 Docker 來將剛剛建立出來的 Next.js 專案包成一個 Docker Image，方便等等將這個 Image 上傳到 Google Artifact Registry（等等會需要再調整成符合 google 規範的 image 名稱），那這邊先簡單介紹一下如何包成一個 Image 吧。

### 建立 Dockerfile

```dockerfile=
# 可以從 Dockerhub Node 上選擇阪本：https://hub.docker.com/_/node/
FROM node:latest

WORKDIR /app

# Copy application dependency manifests to the container image.
# A wildcard is used to ensure copying both package.json AND package-lock.json (when available).
# Copying this first prevents re-running npm install on every code change.
COPY . .

# Install production dependencies.
# npm ci 與 npm install 差別在於，ci 是使用 package-lock.json 上的版本，而 install 則是使用 package.json 上的版本來安裝，因此常導致安裝時其他套件的版本被提升上去了。
RUN npm ci --only=production

# Copy local code to the container image.

RUN npm run build
EXPOSE 3000
ENV PORT 3000
# Run the web service on container startup.
CMD [ "npm", "start" ]
```

### 建立 .dockerignore

`.dockerignore` 主要是用來讓我們 **『篩選掉不需要進入建置階段的檔案』**，也就是等等 `docker build` 時不會把這些檔案都傳送到 Docker Daemon 中，這樣可以加快建置速度。

```dockerfile=
README.md
Dockerfile
.dockerignore
node_modules
```

### 建立 Docker image

上面寫好 `Dockerfile` 與 `.dockerignore` 後，就可以開始建置 Docker image 了，指令如下：

> docker build .(Dockerfile 所在路徑) -t (image 名稱):(tag 名稱) --no-cache(不要 cache 避免內容更新時沒有被重新 build 到)

```
 docker build . -t next-deploy:latest --no-cache
```

### 執行 Docker image

我們現在可以透過在 command line 上下達 `docker images` 指令，來查看剛剛建立的 image 是否存在，如下：

![](https://hackmd.io/_uploads/HJodr-Mpn.png)

如果有正常看到剛剛 build 出來的 image 的話，那現在就可以來把這個 image 啟起來看一下了，這邊介紹兩種方式來啟動 docker image，一種是透過指令、一種是透過 Docker Desktop。

#### 指令啟動 Docker image

> docker run -d(背景執行) -p(對外 port) 3000:3000(等等透過 Web 上網址連進的 port:容器 container port,剛剛 `Dockerfile` 有設定 `EXPOSE 3000`) (image 名稱):(tag 名稱)

```=
docker run -d -p 3000:3000 next-deploy:latest
```

啟動完成後可以透過以下指令查看目前全部容器(Containers) 的狀況，可以看到 `STATUS` 的地方代表該容器已經被運行多長時間。

```=
docker ps -a
```

![](https://hackmd.io/_uploads/BkOE8m7an.png)

如果要停止容器的話則可以用以下指令：

> docker stop 0e45364abeeb(Container ID)

```
docker stop 0e45364abeeb
```

#### GUI 啟動 Docker image

> 因為圖片太大導致顯示後會 lag，因此再麻煩想看如何操作的可以點擊下面這個連結～～
> [https://i.imgur.com/zPvEGKP.gif](https://i.imgur.com/zPvEGKP.gif)

## 建立 Google Artifact Registry

Google Artifact Registry 是新一代的 Container Registry。 主要用來儲存、管理及保護 Docker Image，並且可以與 Google Cloud CI/CD 服務結合使用，像是 [Google Kubernetes Engine (GKE)](https://cloud.google.com/run?hl=zh-tw) 、[Cloud Run](https://cloud.google.com/run?hl=zh-tw)，這邊預計等等會使用 Cloud Run 來做部署，那這邊我們先來『建立一個 Artifact Registry』來存放 Docker Image 吧。

### 安装 gcloud CLI

我們可以透過官網上的[安装 gcloud CLI](https://cloud.google.com/sdk/docs/install-sdk?hl=zh-cn) 來在電腦中安裝，方便等等在 command line 上使用 `gcloud` 指令。

#### 1. 下載 gcloud 壓縮檔

> 因為筆者是 Macbook Air M2 晶片，所以選擇的是第二個。

![](https://hackmd.io/_uploads/rJ3d5H762.png)

#### 2. 執行 google-cloud-sdk

下載完並解壓縮後會出現一個 **google-cloud-sdk** 資料夾，之後開啟終端機直接輸入下面指令，接著它就會問你一些問題，基本上可以都選 `Y` 或是按下 `Enter` 就可以了。

**詳細推薦看這篇文章：[M2 Mac 安裝 gcloud CLI - 磐凌科技](https://penueling.com/%E7%B7%9A%E4%B8%8A%E5%AD%B8%E7%BF%92/m2-mac-%E5%AE%89%E8%A3%9D-gcloud-cli/)**

```
./google-cloud-sdk/install.sh
```

![](https://hackmd.io/_uploads/HyeKsrX6h.png)

#### 3.初始化 gcloud

```bash=
gcloud init
```

**重開終端機輸入 `gcloud init` 後它會選項的方是來讓我們選擇要用『哪個帳號』、『哪個專案』...等，那一開始這邊筆者是選擇 `1` 重新初始化**。

![](https://hackmd.io/_uploads/HJqyRHmT2.png)

**接下來選擇帳號的部分，筆者這邊因為要用另一個帳號，所以這邊選擇 `2`，那它會將你跳到 Browser 上叫你登入並驗證權限...等操作。**

![](https://hackmd.io/_uploads/BJx1yIXT3.png)

**最後會叫你選擇要使用哪一個專案 Project，那這邊筆者是直接選擇辦完 Google Cloud Account 後官方預設建立的那個 `My Project`，如果不知道的話可以在 Google Cloud 上找一下，大概會像下圖這樣。**

![](https://hackmd.io/_uploads/r1Lix8Q6n.png)

![](https://hackmd.io/_uploads/ByUJg87Tn.png)

**這個選完後基本上初始化就大功告成了～～～** :fireworks:

### 建立 Artifact Repositories

#### 1. 點擊上面『＋建立存放區』

![](https://hackmd.io/_uploads/S1ph-oN6n.png)

#### 2. 填寫 Repository Name、存放類型、地區後點擊儲存

> 官方文件：[Create standard repositories](https://cloud.google.com/artifact-registry/docs/repositories/create-repos)

![](https://i.imgur.com/QiIdFsN.gif)

## 推送 Docker Image 到 Actifact Repository 上

在上面的步驟我們已經**建立好一個 Actifact Reposity** 了，現在我們需要『重新 `docker build`』 一下剛剛的 Next.js 專案，因為 **『Actifact Repository 這邊有規範上傳的 Image 格式』**，所以這邊我們先來重新 build 一個 Image 吧！

### 1. 首先，先來看一下 Actifact Repository 規範的 Image 格式

![](https://hackmd.io/_uploads/BJ8o2o4an.png)

- `LOCATION`：代表剛剛建立的 Actifact Repository 所在地區
- `PROJECT_ID`：代表 GCP 上的 Project ID。
- `IMAGE`：代表 Docker Image 的名稱。
- `REPOSITORY`：代表剛剛建立的 Actifact Repository 的名字

> **補充：**
> 如果您不知道上述的東西要到哪裡找的話，可以到剛剛建立好的 Actifact Repository 項目上並且『**點擊上方的複製按鈕**』，它就會『**幫你組好這些資訊**』摟！！！
> 您只需要再自己『**上想要的 Docker Image 的名稱**』就可以了。
>
> ![](https://hackmd.io/_uploads/Skqv1hVTn.png)

### 2. 回到 Next 專案上重新 build Docker Image

請輸入下面指令，這邊主要是將 Docker Image 的名稱調整成 Actifact Repository 規範的名稱，並且 **『後面直接加上 Tag 名稱』**。

```bash=
docker build . --no-cache -t asia-east1-docker.pkg.dev/tribal-catfish-131123/next-deploy-demo/next-deploy-image:v1
```

當然這邊您也可以拆成兩個指令來做：

```bash=
# 1. build docker image
docker build . --no-cache -t <LOCATION>/<PROJECT_ID>/<REPOSITORY>/<Docker Image>

# 2. set image tag
docker tag <LOCATION>/<PROJECT_ID>/<REPOSITORY>/<Docker Image> <LOCATION>/<PROJECT_ID>/<REPOSITORY>/<Docker Image>:<Tag>
```

> **補充：**
> 這邊如果 **『不加上 Tag 名稱』** 也可以，預設會自動幫加上是`latest`，但如果您一開始在建立 Actifact Repository（建立儲存區）時有『**勾選 Immutable Tags**』的話則會跳出『錯誤』，可以參考下圖。
>
> - **勾選 Immutable Tags：**
>   如果是勾選情況，則會跳出錯誤。
>   ![](https://hackmd.io/_uploads/HkOHhyrah.png)
>
> - **未勾選 Immutable Tags：**
>   如果是未勾選情況，如果 Tag 重複則將會『轉移到新的 Image 上面』。
>   ![](https://hackmd.io/_uploads/HyB9F1BT2.png)

### 2-1. 補充：部署時碰到問題：

部署時碰到 **Failed to start and then listen on the port defined by the PORT environment variable** 可改用此指令打包 Image。

```
docker buildx build --platform linux/amd64 -t asia-east1-docker.pkg.dev/tribal-catfish-131123/next-deploy-demo/next-deploy-image:v22 .
```

> **_筆者最後是用這個方式 build Image，詳細可往下滑倒部署問題區塊_** :arrow_down: :arrow_down: :arrow_down:

### 3. 登入 Google Cloud

還記得上面我們安裝了 gcloud CLI 嗎？現在我們就要來用 `gcloud` 這些指令來操作『**推送 Image 到 Actifact Repository**』，首先當然就是要先『**選擇要用哪個登入帳號來操作 gcloud**』，指令如下：

```bash=
gcloud auth login
```

### 4. 設置 Docker 到 Artifact Registry hosts 環境

簡單來說可以想像是：我們可以透過下面這個指令來讓 Docker 跟 gcloud 上的環境連接

```bash=
gcloud auth configure-docker <你的 Artifact Repository 所在地區>

ex. gcloud auth configure-docker asia-east1-docker.pkg.dev
```

![](https://hackmd.io/_uploads/ByemdbHp3.png)

### 5. 推送 Docker Image 到 Google Artifact Registry 上

上面都設定好後，現在就差最後一個步驟『**推送剛剛前面 build 好的 Docker Image 到剛剛建立的 Artifact Repository 上**』，指令如下：

```bash=
docker push asia-east1-docker.pkg.dev/tribal-catfish-131123/next-deploy-demo/next-deploy-image:v1
```

![](https://hackmd.io/_uploads/SkiZc-B63.png)

### 6. 回到 Google Artifact Registry 上查看結果

上傳成功後回到 Google Artifact Registry 上並且到剛剛建立的 Repository 儲存庫裡面應該可以看到剛上傳的 Docker Image，如果再點到 Image 裡面則可以看到剛剛附上的 Tag 標籤。（ps.可以看剛剛上面 Immutable Tags 那張圖）

![](https://hackmd.io/_uploads/r1fw9ZHah.png)

> **補充：如果碰到權限問題！！！**
> 如果您在 `docker push` 時碰到權限問題導致上傳不上去的話，這時就需要去建立一個 IAM Service Account 並且賦予 Artifact Registry 相關的操作權限，詳細等等會教你如何建立 IAM 的 Service Account。
>
> **一般來說，我們自己在操作時不太會碰到這個問題，因為我們的登入帳號就是 Google Cloud Console Account 一般應該擁有最高權限的情況。**
> 比較會發生的情況是在於『**團隊開發時要讓其他人來操作 Google Cloud 時**』這時就需要建立一個 IAM 服務帳號並設定好權限讓他來使用。

## 建立 IAM Service Account

當我們加入別人的團隊或是使用 Github Action 來操作 GCP 時，可能就會需要建立一個 IAM Service Account 來設定對應所需的操作權限，**那這邊先以 `docker push` 如果權限不足時該如何建立一個 IAM Service Account 來做介紹。**

### 1. 前往 IAM Service Account（服務帳號）頁

![](https://hackmd.io/_uploads/B1zPKCB62.png)

### 2. 點擊上方『Create Service Account』

![](https://hackmd.io/_uploads/SyQcYRBah.png)

### 3. 填寫 Service Account 名稱、描述

![](https://hackmd.io/_uploads/BkipY0San.png)

### 4. 選擇要賦予該 Account 的權限

這邊是直接給`Artifact Registry Administrator` 的權限，您也可以依照公司或團隊的管理規則來分配權限，詳細權限選項會附在下面。那因為第三步這邊沒有要設定，因此這邊做完後就可以**點選 DONE 按鈕**

![](https://hackmd.io/_uploads/SyuGc0HT2.png)

> **補充：Artifact Registry 權限項目** >![](https://hackmd.io/_uploads/HJohiRr6n.png)

### 5. 回到 Service Account 列表並點擊『Manage keys』

![](https://hackmd.io/_uploads/Bk1rT0ST2.png)

### 6. 建立一個 key.json

![](https://hackmd.io/_uploads/BJecXkUp3.png)

### 7. 使用這個 Service Account 來進行身份驗證

```
gcloud auth activate-service-account <ACCOUNT> --key-file=<KEY-FILE>

ex.
gcloud auth activate-service-account docker-artifact-demo@tribal-catfish-131123.iam.gserviceaccount.com --key-file=$HOME/Desktop/tribal-catfish-131123-3c9e36ef9689.json
```

> **補充：**
> 這邊我是將下載下來的 `key.json` 檔放在桌面，因此 `--key-file` 這邊是抓取桌面上的 Json 檔，大家可以在依照自己擺放位置來抓取。
>
> **當『指令成功後』透過 `gcloud info` 來觀察目前的帳號狀態，可以發現 `Account` 這邊從原本的登入 email 變成了剛剛建立的 IAM Service Account 帳號了，因此之後『如果有哪些操作權限不足的話，只需要去擴充該帳號底下的權限』即可。**

![](https://hackmd.io/_uploads/B1xXjJUT2.png)

## Cloud Run 部署 - 透過平台建置

到目前為止我們已經能夠將 Docker Image 放到 Google Artifact Registry 了，現在就差最後一步『**使用 Cloud Run 來部署 Artifact Registry 裡的 Image**』，那[什麼是 Cloud Run](https://cloud.google.com/run/docs/overview/what-is-cloud-run) 呢？相信大家應該會有這個疑問，因此在開始之前先來簡單介紹一下 Cloud Run。

> **補充：如果想更詳細了解 Google Cloud Run，非常推薦看 [Cloud Run 是什麼？6 大特色介紹與實作教學 - Cloud Ace](https://blog.cloud-ace.tw/application-modernization/serverless/cloud-run-overview-and-tutorial/) 講得非常清楚！！！** :+1: :+1: :100:

### Cloud Run 介紹

Google Cloud Run 是 Google Cloud Platform（GCP）上提供的一個 Serverless 服務，它能夠讓我們輕鬆將『容器化的應用程式』部署到完全托管的環境中，而無需管理底層的基礎架構(ex.架設機房、購買硬體設備...等)，且 Cloud Run 還有幾個特點：

#### 1. 整合其他 GCP 服務：

Cloud Run 可以輕鬆與其他 Google Cloud 服務整合，例如等等搭配 Google Artifact Registry 就也算是一個與其他服務的整合。

#### 2. 自動擴展：

Cloud Run 可以在應用程式高流量時自動增加容器實例，而在低流量時自動減少實例數，從而節省資源和成本。

#### 3. 依照用量付費：

Cloud Run 只會針對您實際使用的資源向您收費，計費單位為 100 毫秒。詳細可以看官方文件 [Cloud Run pricing](https://cloud.google.com/run/pricing)

> ![](https://hackmd.io/_uploads/ryT_nRYT2.png)

#### 4. 流量分配：

每當部署新版本上去時，我們可以指定將流量的百分比發送到每個版本。例如，將 20% 的流量發送到 A 版本，80% 的流量發送到 B 版本，**甚至當發現部署上去的新版本出問題時，也可以快速 roll back 將流量倒回舊版本(previous revision)。**

### 1. 前往 Google Cloud Run 並點擊 Cloud Run Service

![](https://hackmd.io/_uploads/ryYAWJ9ah.png)

### 2. 選擇 Artifact Registry 中 Docker Image

這邊點擊 `SELECT` 選擇剛剛上傳到 Artifact Registry 中的 Docker Image 並且選擇你要的版本(標籤 Tag)，**因此這邊建議每次上傳到 Artifact Registry 時都要給一個唯一的標籤方便選取**。

![](https://hackmd.io/_uploads/H1zaG1qTn.png)

### 3. 選擇 Region

這邊是選擇 `asia-east1(台灣)`，大家可以依照自己的需求去設定，這邊要注意確認一下『**你所選的區域是什麼收費標準（ex.台灣：`Tier 1 pricing`）**』呦！

![](https://hackmd.io/_uploads/ryQEmycph.png)

### 4. 選擇 Authentication 並且點擊 Create 建立

Authentication 這邊因為是『**公開網站**』所以這邊選擇 `Allow unAuthenticated invocations`，而其他選項這邊都是直接使用『預設值』即可，最後點擊 `CREATE` 來完成建立 Cloud Run 服務。

![](https://hackmd.io/_uploads/rJ2V_ycT3.png)

## 部署過程錯誤處理：

### Q1. 如果您在部署時碰到下面這個錯誤訊息：[Container failed to start](https://cloud.google.com/run/docs/troubleshooting?&_ga=2.62467884.-186107341.1690270126&_gac=1.112726774.1693206806.Cj0KCQjwi7GnBhDXARIsAFLvH4kdB7szDjpIhQXIPNiSVyKA3rcHamD6d-tXF3fsL4jNtMEEe9Y_Xq8aAuDEEALw_wcB#container-failed-to-start)

`Container failed to start. Failed to start and then listen on the port defined by the PORT environment variable.`

**這是代表『您的 Cloud Run 設置的 PORT 與 Dockerfile 中 Expose 設定的 PORT 不一樣』，因此沒辦法正確連接上。**

> Cloud Run 預設 PORT 為 8080。\*\*

![](https://hackmd.io/_uploads/H13cuVq62.png)

#### 解決方式：點擊 `EDIT & DEPLOY NEW REVISION` 去重新設定 `Container Port`

![](https://hackmd.io/_uploads/SysKFVc6h.png)

### Q2：如果更改 PORT 依舊出現問題，請確認目前 Next 版本，或是 build 指令需調成，下面提供相關 issue 與解法

#### 1. Next.JS 13.4.12 以上版本可能會碰到 PORT 問題：

可能需要您將 Next 版本調整成 `13.4.12` (含)以下版本 `ex. 13.4.6`。

> 筆者自己專案版本為 `13.4.19` 是沒有碰到這個問題，並且測試過 `13.4.6` 可以正常運作。

- [Docker Image Build Failed on Google Cloud Run next:13.4.16 #54155](https://github.com/vercel/next.js/discussions/54155)
- [nextjs 13.4.13+ broke self-hosted docker setup #54133](https://github.com/vercel/next.js/issues/54133)

#### 2. M1/M2 晶片 ARM 架構碰到 PORT 問題：

將 Docker build 指令調整成以下：

```
docker buildx build --platform linux/amd64 -t <LOCATION>/<PROJECT_ID>/<REPOSITORY>/<Docker Image>:<Tag> .
```

> **筆者最後是透過這邊的方式解決～～**
>
> 這是因為 Macbook M1/M2 晶片在 Docker build 時預設是使用 ARM 架構（ARM64），但在部署到 Cloud Run 上時會需要包含 `amd64` 的架構，因此這邊會需要透過 `docker buildx` 指令去做 multi-architecture。
>
> ![](https://hackmd.io/_uploads/BJKJAqsp3.png)

- [Cloud Run: "Failed to start and then listen on the port defined by the PORT environment variable." When I use 8080](https://stackoverflow.com/questions/66127933/cloud-run-failed-to-start-and-then-listen-on-the-port-defined-by-the-port-envi)
- [[教學] 用 Docker 的 buildx 輕鬆多架構編譯 (multi-architecture build) - JOHNNY](https://blog.jks.coffee/docker-multi-architecture-build/)

## Cloud Run 部署 － 透過 gcloud 指令部署

上面介紹的方式是透過 Cloud Run 平台來做部署，那還有另外一個方法就是『**透過 `gclud deploy` 指令的方式來部署**』，這邊將會也將會帶大家一步一步實作，**建議可以先看過上面透過平台部署的內容，這樣等等比較好對照 Cammand Line 上的畫面呦！**

### 1. 擴充 IAM 操作權限

還記得我們在前面時有建立一個 IAM 的 Service Account 嗎？那時我們只有設定了『**Artifact Registry Administrator**』這個權限而已，現在因為我們要操作 Cloud Run 因此需要再增加 『**Cloud Run Admin**』

![](https://hackmd.io/_uploads/HyFAWvh6n.png)

![](https://hackmd.io/_uploads/Hkw6GD2p2.png)

### 2. 回到 Command Line 上，透過指令部署

設定完權限後，現在就可以透過 `gcloud deploy` 的指令來部署了，指令如下：

```
gcloud run deploy  --image=<LOCATION>/<PROJECT_ID>/<REPOSITORY>/<Image>:<Tag> --platform managed

ex.

gcloud run deploy  --image=asia-east1-docker.pkg.dev/tribal-catfish-131123/next-deploy-demo/next-deploy-image:v24 --platform managed

```

### 3. 設定服務名稱(Service Name)、地區(Region)、是否公開網站(Allow unauthenticated)

> 這邊建議可以比對上面透過 Cloud Run 平台上建置的圖片一起對照。

![](https://hackmd.io/_uploads/rkgoHvnp2.png)

![](https://hackmd.io/_uploads/BJBsHv26h.png)

## 部署成果

**上面兩種部署的方式都做完後，現在我們回到 Cloud Run 平台上來看一下成果吧！！！**

![](https://hackmd.io/_uploads/HJTiUD362.png)

可以看到 Cloud Run 平台上有我們剛剛『透過兩種不同方式部署』的專案，那現在我們可以來測試看看這些網站是否都有正常的運行起來，我們『點進專案』後點擊上方的 `URL` 就可以連到網頁嘍，看看是否跟在本地端跑起來的一樣吧！

![](https://hackmd.io/_uploads/rJA-Twha2.gif)

#### 以上就是本篇『前端 CI/CD 部署到 GCP 系列(一) - 建構篇』的全部內容，從透過 Docker 建立 Image 到推送上 Artifact Registry 並且設定 IAM Service Account，最後再透過 Cloud Run 進行部署。

#### 本篇是一步一步紀錄實作的過程，中間也碰到不少權限或是版本上的問題，感謝網路上各個大大的文章解救。

#### 下一篇將會『透過 Github Action 來完成整個 CI/CD 自動化部署的功能，再麻煩大家多多指教，如有任何錯誤的地方也再麻煩告知，謝謝您的觀看。

#### Github：[https://github.com/librarylai/next-deploy-gcp](https://github.com/librarylai/next-deploy-gcp)

## Reference

1. [This Is How I Deploy Next.js into Google Cloud Run with Github Actions - JM Santos](https://medium.com/weekly-webtips/this-is-how-i-deploy-next-js-into-google-cloud-run-with-github-actions-1d7d2de9d203)
2. [with-docker - Next.js 官方](https://github.com/vercel/next.js/tree/canary/examples/with-docker)
3. [安装 gcloud CLI - Google Cloud Doc](https://cloud.google.com/sdk/docs/install-sdk?hl=zh-cn)
4. [What is Cloud Run - Google Cloud Doc](https://cloud.google.com/run/docs/overview/what-is-cloud-run)
5. [Cloud Run 是什麼？6 大特色介紹與實作教學 - Cloud Ace](https://blog.cloud-ace.tw/application-modernization/serverless/cloud-run-overview-and-tutorial/)
6. [Cloud Run pricing - Google Cloud Doc](https://cloud.google.com/run/pricing)
7. [Docker Image Build Failed on Google Cloud Run next:13.4.16 #54155 - Github](https://github.com/vercel/next.js/discussions/54155)
8. [nextjs 13.4.13+ broke self-hosted docker setup #54133 - Github](https://github.com/vercel/next.js/issues/54133)
9. [Cloud Run: "Failed to start and then listen on the port defined by the PORT environment variable." When I use 8080 - Stack Overflow](https://stackoverflow.com/questions/66127933/cloud-run-failed-to-start-and-then-listen-on-the-port-defined-by-the-port-envi)
10. [[教學] 用 Docker 的 buildx 輕鬆多架構編譯 (multi-architecture build) - JOHNNY](https://blog.jks.coffee/docker-multi-architecture-build/)
