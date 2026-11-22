# 115 StudentHub 實作：Docker、MongoDB 與 Postman

本實作會帶你從 GitHub 複製專案，使用 Docker 啟動 MongoDB，建立資料庫管理員、匯入學生 CSV，設定後端 `.env`，最後啟動 API 並用 Postman 驗證。

> 本文件以 macOS、Windows 10/11 及 Docker Desktop 為操作環境。MongoDB Compass 是圖形化管理工具；MongoDB Server 會在 Docker 容器中執行，不需要另外安裝在電腦上。

## 你會完成什麼

- Docker Desktop 正在執行，且有一個 MongoDB 容器。
- MongoDB 有 `studenthub` 資料庫、`students` collection、管理員帳號及範例學生資料。
- 後端成功連線資料庫並在 `http://localhost:8877` 提供 API。
- Postman 的 `GET /api/v1/user/findAll` 回傳學生清單。

## 1. 安裝需要的軟體

### Docker Desktop（執行 MongoDB）

- [Docker Desktop 官方下載與安裝說明](https://docs.docker.com/desktop/)
- [macOS 安裝步驟](https://docs.docker.com/desktop/setup/install/mac-install/)
- [Windows 安裝步驟](https://docs.docker.com/desktop/setup/install/windows-install/)

**macOS：** 先確認 Mac 使用 Apple 晶片或 Intel 晶片，再從官方頁面下載相符版本的 Docker Desktop。打開下載的 `.dmg`，將 Docker 圖示拖到 `Applications`，再從「應用程式」開啟 Docker。首次啟動時閱讀並接受使用條款，等選單列的 Docker 圖示顯示正在執行。

**Windows：** 依官方頁面確認電腦符合需求（通常使用 WSL 2 backend），下載 Docker Desktop Installer，執行安裝程式並依畫面指示完成。若安裝程式要求登出或重新啟動，請照指示操作。從開始功能表開啟 Docker Desktop，等畫面顯示 Docker 正在執行。若 WSL 2 未啟用，依 Docker 安裝頁與 Windows 提示完成設定後再重開 Docker Desktop。

開啟終端機確認 Docker 可用：

- **macOS：** 開啟「終端機」（Terminal）。
- **Windows：** 開啟 PowerShell（開始功能表搜尋 PowerShell）。

輸入：

```bash
docker --version
docker compose version
```

兩行都應顯示版本號。若顯示找不到命令，先確認 Docker Desktop 已開啟，重新開啟終端機再試。

### MongoDB Compass（管理及匯入資料）

- [MongoDB Compass 官方下載頁](https://www.mongodb.com/try/download/compass)
- [Compass 安裝說明](https://www.mongodb.com/docs/compass/install/)

**macOS：** 下載 macOS 版本的 `.dmg`，開啟後依畫面將 MongoDB Compass 移至「應用程式」，再啟動它。

**Windows：** 下載 Windows 安裝檔（通常是 `.msi`），執行並依畫面完成安裝，從開始功能表啟動 MongoDB Compass。

### Postman（測試 API）

- [Postman 官方下載頁](https://www.postman.com/downloads/)
- [Postman 安裝說明](https://learning.postman.com/docs/getting-started/installation/install-app/)

**macOS：** 下載 macOS 版本並開啟安裝檔，依畫面將 Postman 移至「應用程式」，再啟動。**Windows：** 下載 Windows 版本，執行安裝檔並依畫面完成安裝，再從開始功能表啟動。可登入帳號或選擇可用的輕量使用方式；本實作不需要建立雲端工作區。

## 2. 從 GitHub 複製專案

先安裝 Git。若電腦尚未安裝，請使用 [Git 官方下載頁](https://git-scm.com/downloads) 選擇 macOS 或 Windows 版本安裝。macOS 也可在終端機執行 `git --version`，依提示安裝 Xcode Command Line Tools。

1. 到課程提供的 `115_StudentHub` GitHub 專案頁，按 **Code**，複製 HTTPS clone 網址。
2. 開啟 Terminal（macOS）或 PowerShell（Windows），移至想放置課程專案的資料夾。例如：

   ```bash
   cd C:/
   ```

3. 將下方網址換成剛複製的 GitHub 網址：

   ```bash
   git clone https://github.com/yyyxi18/115_StudentHub.git
   cd 115_StudentHub
   ```

4. 確認資料夾內有 `studentslist.csv`、`BackEndExample`、`FrontEndExample`。

若 `cd Desktop` 找不到資料夾，改用檔案總管/Finder 找到要放專案的位置，在該位置開啟終端機，或輸入該位置的完整路徑。

## 3. 下載並啟動 MongoDB 容器

以下命令會下載 MongoDB 8 映像檔，建立名為 `studenthub-mongo` 的容器，並將電腦的 `27017` 埠對應到容器內的 `27017` 埠。帳號密碼是本機練習用，請勿用於正式或公開服務。

在 Docker Desktop 已啟動的 Terminal / PowerShell 執行：

```bash
docker pull mongo:8
docker run -d --name studenthub-mongo -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=studentAdmin -e MONGO_INITDB_ROOT_PASSWORD=StudentHub_2026 -v studenthub-mongo-data:/data/db --restart unless-stopped mongo:8
```

確認容器狀態：

```bash
docker ps
```

應看到 `studenthub-mongo`，狀態為 `Up`。也可在 Docker Desktop 的 **Containers** 頁籤看到正在執行的容器。

> MongoDB 官方映像檔只會在第一次初始化空白資料目錄時建立 `MONGO_INITDB_ROOT_USERNAME` / `MONGO_INITDB_ROOT_PASSWORD`。本練習使用 Docker volume 保存資料；若你之前已建立同名 volume 並換過密碼，舊密碼會繼續有效。若是本練習初次操作，以上命令即可。

## 4. 用 MongoDB Compass 連線並建立資料庫帳號

1. 開啟 MongoDB Compass，在連線欄輸入：

   ```text
   mongodb://studentAdmin:StudentHub_2026@localhost:27017/?authSource=admin
   ```
   ![alt text](image.png)

2. 按 **Connect**。左側應能看到資料庫清單。這組 `studentAdmin` 是 MongoDB 的 root 管理員，建立於 `admin` 認證資料庫；後端將使用下一步建立的 `studenthub_app` 一般帳號。
3. 在 Compass 開啟 **MONGOSH**（通常在視窗底部），貼上並執行以下指令：

   ```javascript
   use studenthub
   db.createUser({
     user: "studenthub_app",
     pwd: "StudentHub_App_2026",
     roles: [{ role: "readWrite", db: "studenthub" }]
   })
   ```
   ![alt text](image-1.png)
   
   若訊息表示已切換到 `studenthub` 且建立使用者成功，即完成。`studenthub` 資料庫會在第一次建立 collection 或寫入文件時正式出現在清單中。

## 5. 匯入 CSV 學生範例資料

CSV 位於專案根目錄：`studentslist.csv`。匯入後會建立 `students` collection；集合名稱需與後端使用的名稱一致。

1. 在 Compass 左側點 **Create Database**。
2. Database Name 填 `studenthub`；Collection Name 填 `students`，按 **Create Database**。
3. 點開 `studenthub` 資料庫中的 `students` collection，選 **Add Data** → **Import File**。
4. 選擇 clone 專案中的 `studentslist.csv`。檔案類型選 CSV；確認欄位名稱列是第一列（header），再按 **Import**。
5. 匯入後應看到文件清單。確認文件至少有 `userName`、`sid`、`name`、`department`、`grade`、`class`、`Email` 欄位。後端 schema 以這些欄位大小寫為準。

Compass 通常會依 CSV 欄位判斷資料型態；後端的 `sid` 欄位預期為字串。匯入後在 **Documents** 頁面確認 `sid` 是字串（顯示為引號包住的文字）。如果 Compass 將 `sid` 判斷成數字，請先在匯入對話框的欄位型態設定中將 `sid` 設為 String 再匯入；或用 `mongoimport` 的 `--columnsHaveTypes` 方案需先將 CSV 欄名改成 `sid.string()`，不建議直接改動課程原始 CSV。

預期：collection 有多筆學生文件，欄位和值可在 Compass 的 Documents 頁籤瀏覽。

## 6. 設定後端 `.env`

1. 在檔案總管/Finder 開啟 `115_StudentHub/BackEndExample` 資料夾。
2. 將 `.env.example` 複製一份，命名為 `.env`。若資料夾已經有 `.env`，直接編輯它。請不要把 `.env` 上傳到 GitHub。
3. 讓 `.env` 內容設定如下（值依本手冊的本機容器帳號）：
> 小提醒💡：請依照你實際設定的內容為主，範例.env有註解可以對照，使用範例的.env很容易執行錯誤喔
   ```dotenv
   DBUSER=studenthub_app
   DBPASSWORD=StudentHub_App_2026
   DBHOST=127.0.0.1
   DBPORT=27017
   DBNAME=studenthub
   PORT=8877
   LogPath=logs
   assetsPath=../FrontEndExample/dist/assets
   HomePagePath=../FrontEndExample/dist/index.html
   privateKey=replace_this_with_a_long_random_local_secret
   ```

4. 儲存檔案。請勿在等號兩側加空格或在值後面加註解。`PORT=8877` 要與前端程式目前使用的 API 埠相同；本次驗收用 Postman 直接測 API。

## 7. 安裝 Node.js 並啟動後端

此專案後端使用 Node.js/TypeScript。安裝 [Node.js LTS 官方版本](https://nodejs.org/en/download)（macOS 可選 `.pkg`；Windows 可選 `.msi`，依安裝程式提示完成）。安裝後關閉並重新開啟終端機，確認：

```bash
node --version
npm --version
```

接著從專案根目錄執行：

```bash
cd BackEndExample
npm install
npm run dev
```

第一次安裝相依套件會需要網路。啟動命令持續執行是正常的；不要關閉這個終端機。成功時後端會開始監聽 `8877`，MongoDB 連線成功時會在 `BackEndExample/logs` 產生紀錄。若要停止後端，在該終端機按 `Ctrl+C`。

若看見連線失敗，依序確認：Docker Desktop 正在執行；`docker ps` 顯示 `studenthub-mongo` 為 `Up`；Compass 能使用同一組資料庫帳號連線；`.env` 五個 `DB...` 值正確；`.env` 位於 `BackEndExample` 內且檔名不是 `.env.txt`。

## 8. 使用 Postman 測試 API

保持後端終端機執行中。開啟 Postman，新增 **HTTP Request**：

1. 方法選 **GET**。
2. URL 輸入：

   ```text
   http://localhost:8877/api/v1/user/findAll
   ```

3. 按 **Send**。

成功時應收到 HTTP `200 OK`，回應 JSON 物件包含 `code: 200`、`message: "find sucess"` 與 `body` 學生文件陣列（含 `_id`、`userName`、`name` 等欄位）。`body` 內容應與 Compass 匯入的學生資料相符。

若 Postman 顯示無法連線，確認後端仍在執行且 URL 埠號是 `8877`。若回應 HTTP `500` 或 `body` 不符合預期，先看 `BackEndExample/logs` 的最新日誌、檢查 MongoDB 容器與 `.env`，並確認匯入的是 `studenthub.students`。

## 常見問題

| 狀況 | 檢查方式 |
|---|---|
| `docker` 找不到 | 開啟 Docker Desktop，重新開啟終端機；確認安裝完成。 |
| Docker Desktop 無法啟動 | Windows 依畫面啟用 WSL 2 並重開；Mac 確認下載符合 Intel/Apple 晶片的版本。 |
| 容器名稱已存在 | 執行 `docker start studenthub-mongo`，不要重複 `docker run`。 |
| `port is already allocated` | 電腦的 `27017` 埠已被占用。停止另一個 MongoDB 服務或容器，再重新啟動本容器。 |
| Compass 認證失敗 | 使用 root 管理員連線 URI，確認 `studenthub-mongo` 正在執行；已有舊 volume 時可能仍是先前設定的 root 密碼。 |
| API 回傳空陣列 | Compass 確認資料匯入到 `studenthub` 資料庫的 `students` collection。 |
| 想保留已匯入資料 | 停止/移除容器不會自動刪除 `studenthub-mongo-data` volume。下次可用 `docker start studenthub-mongo` 重新啟動。 |

## 結束練習

停止後端，在另一個終端機視窗執行：

```bash
docker stop studenthub-mongo
```

下次練習時先開啟 Docker Desktop，再執行 `docker start studenthub-mongo`。資料保存在 Docker volume 中。若要完全刪除練習資料（此動作無法復原），依序執行：

```bash
docker rm -f studenthub-mongo
docker volume rm studenthub-mongo-data
```

## 延伸：在 Postman 測試新增 API

本專案也提供新增學生 API：方法選 **POST**，URL 為 `http://localhost:8877/api/v1/user/insertOne`，在 **Body** 選 **raw** → **JSON**，送出：

```json
{
  "userName": "tkuxx9999",
  "name": "測試學生",
  "department": "測試系",
  "grade": "一年級",
  "class": "A",
  "Email": "tkuxx9999@example.com"
}
```

`userName` 格式為 `tku` + 科系代碼 + 四位座號，座號不可和既有學生重複；請先依學生資料自行挑一個未出現的座號。成功回應 HTTP `200` 後，再送一次 GET 確認新文件。此專案目前要求資料庫至少已有一筆學生資料，否則新增 API 會回傳錯誤。

## 本次練習的專案設定備註

- 專案後端路由的資料集合名稱是 `students`；資料庫名稱由 `.env` 的 `DBNAME` 決定。
- 前端 `FrontEndExample/src/enum/api.ts` 目前寫的是 `http://localhost:8877/api/v1/user/findAll`。若你更改後端 `PORT`，需要同步修改該檔案。
- `.env.example` 是範本，不等同本機可直接使用的設定；請按本文件內容建立自己的 `.env`。
- 此 README 的 API 測試流程以專案原始碼與上述 MongoDB 操作流程為依據；請以自己執行時 Postman 回應及日誌作為完成證據。
