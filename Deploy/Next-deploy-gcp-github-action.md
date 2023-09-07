# 【筆記】前端 CI/CD 部署到 GCP 系列(二) - Github Action 整合自動化部署到 Cloud Run

###### tags: `筆記文章`

![](https://hackmd.io/_uploads/ryeCEgMA3.jpg)

> 在[上一篇](https://hackmd.io/@Librarylai/HkwVS-Mp2)基本上我們已經完成了透過『手動下指令的方式』並且在 Google Cloud Platform 上一步一步完成部署。

> 那本篇將會接續上一篇在 GCP 上建制的內容來近一步完成在 Github Action 上的自動化部署，我們先從 Github Action 介紹起，最後再來一步一步講解程式碼，如有任何講錯再麻煩大家告知，感謝！

## 什麼是 Github Action

> GitHub Actions is a continuous integration and continuous delivery (CI/CD) platform that allows you to automate your build, test, and deployment pipeline. You can create workflows that build and test every pull request to your repository, or deploy merged pull requests to production.

#### 簡單來說：

Github Action 是 Github 提供的一個服務，讓我們能做到持續整合(CI)、持續部署(CD)，我們可以建立一些流程(workflow)，當 push、pull request...等 Event 觸發時就去執行我們所設定好的 workflow。

**一般實務上比較常見的使用方式像是：**

- **每當 push or pull request 時就自動跑測試 (ex. npm run test) 檢查本次的更動是否有造成任何測試未通過。**
- **每當 push 且帶有 tag 時，基本上該情況大多是要發版的時候，這時就會自動跑 Deploy 來完成部署。**

> 詳細可參考：https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions#overview

## 前置作業：專案建立 Github Action Workflow 與 Test 環境

### 1. 在專案中建立`.github/workflows` 資料夾，並且裡面建立一支 `<name>.yml` 檔

`.github/workflows` 是官方規範的建立格式，所有要在 Github Action 上跑的 workflow 都需建立在該路徑底下，而檔案名稱可以隨便取，以下為筆者的範例：

![](https://hackmd.io/_uploads/BkRHgGQA2.png)

### 2. 建立 Jest 等測試環境

因為等等需要模擬跑自動化測試的部分，並且會模擬當『測試不通過』時則不進行 Deploy，所以要先在專案中將 Jest 等測試環境建立起來。

> 提醒：
> 如果您並不想跑測試(npm run test) 則可以跳過此步驟。
> 這邊將會使用 Next.js 官方的 [Jest and React Testing Library](https://nextjs.org/docs/pages/building-your-application/optimizing/testing#jest-and-react-testing-library) 方式快速將測試相關套件與環境加入到專案中。

### 2-1. install jest 等套件

這邊直接將 jest 與 react-testing-library 等相關套件一併安裝，雖然等等只會用 jest 的語法，隨便寫一個簡單的測試而已，但這邊還是一勞永逸通通裝一裝。 :rolling_on_the_floor_laughing: :rolling_on_the_floor_laughing: :rolling_on_the_floor_laughing:

```
npm install --save-dev jest jest-environment-jsdom @testing-library/react @testing-library/jest-dom
```

### 2-2. Setting up Jest (with Babel)

這邊是直接使用 Next.js 範例的設定檔，簡單改了一下對應的路徑，基本上應該可以直接複製下方程式碼來使用，大家也可以在依照自己專案的需求下去調整。

```javascript=
/* jest.config.js */
module.exports = {
  collectCoverage: true,
  // on node 14.x coverage provider v8 offers good speed and more or less good report
  coverageProvider: 'v8',
  collectCoverageFrom: [
    '**/*.{js,jsx,ts,tsx}',
    '!**/*.d.ts',
    '!**/node_modules/**',
    '!./out/**',
    '!./.next/**',
    '!./*.config.js',
    '!./coverage/**',
  ],
  moduleNameMapper: {
    // Handle CSS imports (with CSS modules)
    // https://jestjs.io/docs/webpack#mocking-css-modules
    '^.+\\.module\\.(css|sass|scss)$': 'identity-obj-proxy',

    // Handle CSS imports (without CSS modules)
    '^.+\\.(css|sass|scss)$': './__mocks__/styleMock.js',

    // Handle image imports
    // https://jestjs.io/docs/webpack#handling-static-assets
    '^.+\\.(png|jpg|jpeg|gif|webp|avif|ico|bmp|svg)$/i': `./__mocks__/fileMock.js`,

    // Handle module aliases
    '^@/components/(.*)$': './src/components/$1',
  },
  // Add more setup options before each test is run
  // setupFilesAfterEnv: ['./jest.setup.js'],
  testPathIgnorePatterns: ['./node_modules/', './.next/'],
  testEnvironment: 'jsdom',
  transform: {
    // Use babel-jest to transpile tests with the next/babel preset
    // https://jestjs.io/docs/configuration#transform-objectstring-pathtotransformer--pathtotransformer-object
    '^.+\\.(js|jsx|ts|tsx)$': ['babel-jest', { presets: ['next/babel'] }],
  },
  transformIgnorePatterns: ['/node_modules/', '^.+\\.module\\.(css|sass|scss)$'],
}

```

#### package.json 增加 script 指令

```json=
 "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "test": "jest" // 加上這行～～～～
  },
```

### 2-3. 建立一個 test 測試檔

Jest 基本上預設會去跑專案中 `__tests__` 資料夾內所有`.test` 的檔案，因此我們在裡面建立一支 `sample.test.js` 測試檔並且寫上以下程式碼：

```javascript=
/* sample.test.js */
describe('Sample', () => {
  it('one plus one', () => {
    expect(1 + 1).toBe(2)
  })
})
```

![](https://hackmd.io/_uploads/H1bgHz7Rh.png)

### 2-4 最後在終端機下 `npm run test` 跑看看

這邊我們只需要有看到 `sample.test.js` 這隻顯示 **pass** 的話，大致上測試的環境就成功了。

![](https://hackmd.io/_uploads/S1jHHzmRn.png)

## 攥寫 workflow yml 檔

這邊我們直接從最終完整的程式碼來講起，在 Github Action 中比較用重要的幾個關鍵字分別為：**`name`、`on`、`jobs`、`steps`、`uses`、`run `** 這幾個關鍵字，那接下來就簡單介紹一下。

- **name(名稱)：** 代表整個 workflow 的名稱，或是每一個要執行的 jobs 任務的名稱，方便 debug 時能夠一目瞭然的知道現在跑到哪個步驟，或是在哪個地方掛掉。
- **on(觸發條件)：** 用來定義『什麼樣的操作事件觸發』時，才會開始執行 workflow。一般常見的事 `push`、`pull request` 的時候。

  > 參考文件：[Events that trigger workflows](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#push)

- **jobs(任務)：** 用來定義一個工作流程(workflow)中的每一個任務。 >參考文件：[Using jobs in workflow](https://docs.github.com/en/actions/using-jobs/using-jobs-in-a-workflow#overview) >**補充：** >A job is a set of steps in a workflow that is executed on the same runner. >每一個 jobs 都是在同一個 runner 環境中，例如以本篇範例來說：Testing job 是跑在 ubuntu-latest 環境中。

      * **needs**： 它是 jobs 中的一個參數，它代表等待某一個 jobs 成功後才觸發，因為『**預設每一個 jobs 是同時間並行的在啟動**』，因此今天我們需要等待某一個成功後才觸發，就會需要使用 **needs** 這個參數，**例如：Deploy 之前要先跑過測試(testing job) 並且成功沒有任何錯誤。**
          > 補充： 如果是只想要等待『完成』就好，『不管成功失敗』，則為以下：

  ![](https://hackmd.io/_uploads/BkLF6mQR3.png)

- **Steps：** 算是 GitHub Actions 工作流程的核心，它代表 jobs 中的一連串的操作步驟，它會依序執行當前面一個完成後才會繼續進行，比較常用的指令有：
  - **uses：** 透過使用 `uses` 指令來使用其他外部別人寫好的 Action。
  - **run：** 透過使用 `run` 指令來執行一些命令、腳本(ex. `docker build`、`npm run test`)...等。

### 完整 Github Action 程式碼

上面簡單介紹了 Github Action 核心的指令，現在直接看一下程式碼應該可以比較好理解了。關於程式碼中 `gcloud`、`docker build` 等指令都是[上一篇](https://hackmd.io/@Librarylai/HkwVS-Mp2)介紹並且使用過的指令，這邊只是原封不動地搬上來而已。

> 相關 `uses` 套件介紹與 Github Secrets 等建置操作將會在下面做補充。

```yaml=
name: deploy-google-cloud-run

on:
  # https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#push
  # 設定當 push 時帶有 tag 時才會觸發
  push:
    tags:
      - 'v*'
# https://docs.github.com/en/actions/using-jobs/using-jobs-in-a-workflow
jobs:
  # 第一個 job，用來執行測試
  testing: # testing 為 job id
    name: Testing # job 名稱
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm run test

  # 第二個 job，用來建立 Docker Image -> Google Artifact Registry -> Google Cloud Run
  build-and-deploy-gcr: # build-and-deploy-gcr 為 job id
    name: Build and Deploy to GCR
    # needs 主要為：設定需要 testing job 執行完後才會執行
    # 且如果 npm run test 有錯誤，則不會執行。
    needs: testing
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      # 抓所有 tags 並且排序，取最後一個 tag
      - name: Get tag
        id: get_tag
        # 將 tag 存到 output 變數中的 latest_tag 上面
        # latest_tag=$latest_tag" >> $GITHUB_OUTPUT 這段為設定 output 變數
        # 參考：https://github.blog/changelog/2022-10-11-github-actions-deprecating-save-state-and-set-output-commands/
        run: |
          git fetch --tags
          latest_tag=$(git describe --tags $(git rev-list --tags --max-count=1))
          echo "latest_tag=$latest_tag" >> $GITHUB_OUTPUT

      # 取得  Google Cloud 授權，這邊將之前建立的 IAM service account key 透過 base64 轉換後
      # 存到 Github secrets 中，這邊取名為 GCP_IAM_SERVICE_ACCOUNT_KEY
      - name: 'Authenticate with Google Cloud'
        id: 'auth'
        uses: 'google-github-actions/auth@v1'
        with:
          credentials_json: ${{ secrets.GCP_IAM_SERVICE_ACCOUNT_KEY }}}}
      # 安裝 Cloud SDK 相關套件
      - name: 'Set up Cloud SDK'
        uses: 'google-github-actions/setup-gcloud@v1'
      # 將 Docker 跟 gcloud 上的環境連接，並且增加 asia-east1-docker.pkg.dev 這個地區
      # 允許將 Docker Image 上傳到該表中的地區，或是允許從該地區下載 Image。
      - name: Configure Docker Client of Gcloud
        run: |-
          gcloud auth configure-docker --quiet
          gcloud auth configure-docker asia-east1-docker.pkg.dev --quiet
      # 使用 gcloud CLI 來確認現在的帳號...等資訊，方便 debug
      - name: 'Use gcloud CLI'
        run: gcloud info
      # 使用 Docker Buildx 來建立 Docker Image
      # 這邊因為 runner 是使用 ubuntu-latest，因此應該可以用 docker build 即可
      - name: 'Docker Buildx'
        id: buildx
        run: docker buildx build --platform linux/amd64 -t ${{secrets.GCP_ARTIFACT_REPOSITORY}}/${{secrets.GCP_DOCKER_IMAGE_NAME}}:${{steps.get_tag.outputs.latest_tag}} .
      # 將 Docker Image 上傳到 Google Artifact Registry
      - name: 'Docker push to Google Artifact Registry'
        run: |
          echo ${{secrets.GCP_ARTIFACT_REPOSITORY}}/${{secrets.GCP_DOCKER_IMAGE_NAME}}:${{steps.get_tag.outputs.latest_tag}}
          docker push ${{secrets.GCP_ARTIFACT_REPOSITORY}}/${{secrets.GCP_DOCKER_IMAGE_NAME}}:${{steps.get_tag.outputs.latest_tag}}
      # 將 Google Artifact Registry Image 發布到 Google Cloud Run
      # ${{secrets.GCP_CLOUD_RUN_SERVICE_NAME}} 為指定 deploy 到哪個 cloud run service 上面，
      # 因上一篇已經在 cloud run 上建立過一個 next-deploy-image service 因此這邊就直接指定該 service
      # --port 設定為 3000 , 因為 dockerfile 中的 EXPOSE 為 3000
      # --region 設定為 asia-east1 , 因為 cloud run service 是在 asia-east1 這個地區
      # --allow-unauthenticated 因為此為公開 website
      # --quiet 為不顯示互動式介面
      - name: Deploy
        run: |
          gcloud run deploy  ${{secrets.GCP_CLOUD_RUN_SERVICE_NAME}} \
          --image=${{secrets.GCP_ARTIFACT_REPOSITORY}}/${{secrets.GCP_DOCKER_IMAGE_NAME}}:${{steps.get_tag.outputs.latest_tag}} \
          --platform managed \
          --port 3000 \
          --region asia-east1 \
          --allow-unauthenticated \
          --quiet \

```

### 相關外部 Actions 補充

#### 1. [actions/checkout@v3](https://github.com/actions/checkout)

主要是用來檢查你的 Repository 並且讓 Workflow 能夠使用它，且允許針對 code 使用 script 等相關指令（ex. build, test）。
![](https://hackmd.io/_uploads/SkKk_P40n.png)

#### 2. [actions/setup-node@v3](https://github.com/actions/setup-node)

顧名思義就是幫我們在環境中安裝 node，並且我們可以指定想要的 node 版本，且也可以設定 `cache` 來將 dependency 做快取。
![](https://hackmd.io/_uploads/rJ5v9DVR2.png)

#### 3. [google-github-actions/auth@v1](https://github.com/google-github-actions/auth)

因為等等我們需要透過指令來對 Google Cloud 進行操作，因此我們需要用該 Action 來『**授權**』給它，那這邊採用的是 **Service Account Key JSON** 的方式。

**還記得上一篇我們建立了一個 IAM Service Account 並且還下載了一個 `<Key>.json` 檔嗎？** 現在我們需要將那個檔案存到 Github Secrets 上面，我們可以先將 `<Key>.json` 檔『**進行 base64 壓縮後再存到 Github Secrets 上面**』。

![](https://hackmd.io/_uploads/BkeZ0wER2.png)

#### 4. [google-github-actions/setup-gcloud@v1](https://github.com/google-github-actions/setup-gcloud)

setup-gcloud 主要讓我們能在 Github Action 環境中使用 Google Cloud SDK，讓我們能夠使用 **gcloud、gsutil** 指令。

當 `auth` 與 `setup-gcloud` 都使用後，我們就可以使用 `gcloud info` 來查看目前使用的帳號狀態，**如果有看到該 `Key.json` 的 IAM Service Account 就是有正確設定完成了**，基本上會是上一篇創建的那個 Service Account。

![](https://hackmd.io/_uploads/Syg1zuN02.png)

## Github Secrets 設定

當我們在攥寫 Github Action 的 yml 檔時，**有時會有一些敏感資訊(ex. **Private Key, Service Account, Project ID**...等)無法直接放在檔案中**，因為該檔案之後也會到 github 上面因此會被任何人看到，所以我們需要將這些資訊存在 Github Secret 中，**最後只需要在 yml 檔中使用 `${{secrets.<您所設定的 SECRETS_NAME>}}` 語法**，Github Action 在執行時就能正確找到這些敏感資訊嘍！

### 1. 前往 Repository 的 Settings 頁

![](https://hackmd.io/_uploads/SJLaV_N03.png)

### 2. 選擇 Secrets and variables 裡的 Actions 並點擊建立 secret

![](https://hackmd.io/_uploads/H1iyPuV02.png)

### 3. 貼上需要存入的敏感資訊

例如剛剛將 **`Key.json` 轉成 base64 的內容**，或是 **Google Cloud 的 project ID**、**Artifact Reposity 的路徑**、**IAM Service Account 帳號名稱**...等資訊。

![](https://hackmd.io/_uploads/rkmRwONRn.png)

### 4. 最後回到 yml 檔中使用

```yaml=
# 取得  Google Cloud 授權，這邊將之前建立的 IAM service account key 透過 base64 轉換後
# 存到 Github secrets 中，這邊取名為 GCP_IAM_SERVICE_ACCOUNT_KEY
- name: 'Authenticate with Google Cloud'
  id: 'auth'
  uses: 'google-github-actions/auth@v1'
  with:
    credentials_json: ${{ secrets.GCP_IAM_SERVICE_ACCOUNT_KEY }}}}
```

## 成果展示 - 成功篇

### 1. Github Action 上狀態

可以看到當 `push` 且帶有 `tag` 時會成功觸發 Github Action，且它會先跑 **Testing job** 如果『測試成功沒有任何 failed』則會進入到 **Build and Deploy job** 開始進行『打包與部署』到 Google Artifact Reposity 與 Google Cloud Run。

![](https://hackmd.io/_uploads/S1dWcdN0n.png)

### 2. 查看 Google Artifact Reposity

可以看到 `v0.0.4` 這個版本成功推到 Google Artifact Reposity 中。

![](https://hackmd.io/_uploads/SJQrjuV0n.png)

### 3. 查看 Google Cloud Run

可以看的目前 Cloud Run 正在使用的版本就是剛剛推到 Google Artifact Reposity 上的 `v0.0.4` 版本的 Docker Image，**這就代表成功部署完成摟！！！** :raised_hands: :raised_hands: :raised_hands:

![](https://hackmd.io/_uploads/HkKxhuECn.png)

## 成果展示 - 失敗篇(測試 Failed)

還記得我們的 Action Workflow 是有跑兩個 job，先『**進行測試 再 進行部署**』，這邊將模擬前面測試失敗時的情況。

### 1. 將 sample.test.js 改為以下

```javascript=
/* sample.test.js */
describe('Sample', () => {
  it('one plus one', () => {
    expect(1 + 1).toBe(3) // 讓測試失敗～～
  })
})

```

### 2. push 並附加 tag 來查看 Github Action 上情況

這邊重新推了一個 `v0.0.5` 版本的到 Github Action 上面跑，可以看到在 **Testing job** 的部分就因發生錯誤而停止了。

![](https://hackmd.io/_uploads/r1UkhoBCh.png)

### 3. 進入失敗的 Job 中查看原因

點進 **Testing job** 後可以看到發生錯誤的原因就在剛剛調整的那段測試上面。因此當 Action 失敗時也不需要緊張，只需要點進該失敗的 job 查看錯誤的原因即可。

![](https://hackmd.io/_uploads/ByB82or03.png)

#### 以上就是本篇『前端 CI/CD 部署到 GCP 系列(二) - Github Action 整合自動化部署到 Cloud Run』的全部內容，主要是延續上一篇所建置的內容，將用到的指令(docker、gcloud)整合進 Github Action 中，因此如果有一些地方跳太快可以再麻煩跟我說，謝謝。

#### 本篇是一步一步紀錄實作的過程，對於 Github Action 的部分也是一邊看文件一邊寫，過程中也碰到了一些 bug，感謝網路上各個大大的文章解救。

#### 之後也會再找一些 GCP 上的服務來完看看，再麻煩大家多多指教，如有任何錯誤的地方也再麻煩告知，謝謝您的觀看。

#### Github：[https://github.com/librarylai/next-deploy-gcp](https://github.com/librarylai/next-deploy-gcp)

## Reference

1.  [This Is How I Deploy Next.js into Google Cloud Run with Github Actions - JM Santos](https://medium.com/weekly-webtips/this-is-how-i-deploy-next-js-into-google-cloud-run-with-github-actions-1d7d2de9d203)
2.  [Understanding GitHub Actions - Github](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions#overview)
3.  [Jest and React Testing Library - Next.js](https://nextjs.org/docs/pages/building-your-application/optimizing/testing#jest-and-react-testing-library)
4.  [Events that trigger workflows - Github](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#push)
5.  [Using jobs in workflow - Github](https://docs.github.com/en/actions/using-jobs/using-jobs-in-a-workflow#overview)
6.  [GitHub Actions: Deprecating save-state and set-output commands - github.blog](https://github.blog/changelog/2022-10-11-github-actions-deprecating-save-state-and-set-output-commands/)
