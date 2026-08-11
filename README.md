# 台南旅遊巡禮｜AI輔助旅遊景點推薦平台

## 專題簡介

本專題是職訓局結業專題「AI輔助旅遊景點推薦平台」，聚焦台南單一縣市的景點推薦。前端以 jQuery + Ajax 串接自製 Flask API，並用 Vue.js 呈現管理頁的統計圖表；後端使用 Flask 完成景點資料的完整 CRUD，資料庫採 SQLite。

系統可以瀏覽景點列表（分類篩選、關鍵字搜尋、分頁）、查看景點詳細內容（含開放資訊、相關活動）、在管理頁新增／編輯／刪除景點資料，並顯示各分類景點數量統計圖表。管理頁需登入後才能進入。

視覺風格定案為「府城印章風」：米色紙感背景 + 廟宇藍（indigo）+ 朱紅印章元素（vermilion），呼應台南古都主題。

## 使用技術

| 類別 | 技術 |
| --- | --- |
| 前端 | HTML、CSS（自訂府城印章風配色）、jQuery、Vue.js（統計圖表） |
| 後端 | Python、Flask |
| 資料庫 | SQLite |
| 設計 | Figma（設計稿）、Google Places API（景點真實照片）、AI 生成圖像（補充素材） |
| 版本管理 | Git、GitHub |

> 前端圖表用的圖表套件（例如 Chart.js）、部署方式如果有實際採用，請在這裡補上一列。

## 系統功能說明

| 頁面 | 說明 | 截圖位置 |
| --- | --- | --- |
| 首頁（index.html） | 專題介紹與導覽入口 | `docs/screenshots/home.png` |
| 景點列表（attractions1.html） | 卡片呈現景點資料，可用分類（古蹟／美食／熱門景點／自然景觀）篩選、關鍵字搜尋、切換每頁筆數與分頁瀏覽 | `docs/screenshots/attractions.png` |
| 景點詳細（detail1.html） | 左圖右文雙欄式版型，顯示景點圖片、分類、地址、開放時間、票價、官網連結、相關活動，並附圖片輪播 | `docs/screenshots/detail.png` |
| 管理頁（admin.html） | 新增／編輯／刪除景點的表單與資料表，需登入後才能進入 | `docs/screenshots/admin.png` |
| 統計圖表 | 用 Vue.js 串接真實資料庫資料，呈現各分類景點數量長條圖 | `docs/screenshots/charts.png` |

> 截圖請放在 `docs/screenshots/` 資料夾，檔名可比照上表。

## 資料庫設計說明

資料庫使用 SQLite，主要資料表如下：

### attractions 景點資料表

| 欄位 | 型別 | 說明 |
| --- | --- | --- |
| id | INTEGER | 主鍵，自動編號 |
| name | TEXT | 景點名稱 |
| district | TEXT | 所在行政區 |
| category | TEXT | 分類（古蹟／美食／熱門景點／自然景觀） |
| image_url | TEXT | 圖片網址 |
| description | TEXT | 景點介紹文字 |
| created_at | TEXT | 建立時間 |

### attraction_details 景點詳細資訊表（關聯表）

| 欄位 | 型別 | 說明 |
| --- | --- | --- |
| id | INTEGER | 主鍵 |
| attraction_id | INTEGER | 對應 `attractions.id` |
| opening_hours | TEXT | 開放時間 |
| address | TEXT | 詳細地址 |
| ticket_info | TEXT | 票價資訊 |
| official_url | TEXT | 官方網站連結 |
| tips | TEXT | 小提醒 |

### attraction_events 景點活動表（關聯表）

| 欄位 | 型別 | 說明 |
| --- | --- | --- |
| id | INTEGER | 主鍵 |
| attraction_id | INTEGER | 對應 `attractions.id` |
| event_name | TEXT | 活動名稱 |
| event_date | TEXT | 活動日期 |
| event_description | TEXT | 活動說明 |

### 資料表關聯

`attraction_details` 與 `attraction_events` 皆以 `attraction_id` 對應 `attractions.id`，為一對一／一對多關聯；景點詳細頁透過後端 JOIN 查詢，一次取回景點基本資料、詳細資訊與相關活動。

> 請依實際欄位型別與是否有 NOT NULL／UNIQUE 等限制修正上表。

## API 說明

| 方法 | 路徑 | 功能 |
| --- | --- | --- |
| GET | `/attractions` | 查詢景點列表 |
| GET | `/attractions/<id>` | 查詢單一景點詳細資料（含 details、events） |
| POST | `/attractions` | 新增景點 |
| PUT | `/attractions/<id>` | 修改景點（完整更新） |
| PATCH | `/attractions/<id>` | 修改景點（部分欄位更新） |
| DELETE | `/attractions/<id>` | 刪除景點 |

> 如果有另外做統計圖表用的 API（例如 `/attractions/stats`），或分類／搜尋是後端支援而非前端處理，請補上對應路徑與參數說明。

## 管理頁登入驗證

管理頁採前端登入狀態檢查：使用者點擊導覽列或列表頁的「管理頁面」連結時，會先確認 `sessionStorage` 是否有 `isAdmin` 標記，沒有登入就攔截並導向 `login.html`。

```javascript
document.querySelectorAll(".admin-link").forEach(function (link) {
    link.addEventListener("click", function (e) {
        if (sessionStorage.getItem("isAdmin") !== "true") {
            e.preventDefault();
            window.location.href = "login.html";
        }
    });
});
```

> 這裡只做前端攔截，如果 admin.html 的操作（新增／編輯／刪除）本身沒有後端權限驗證，建議補充說明，或註記此為專題展示用的簡化版本。

## AI 功能說明

專題名稱包含「AI輔助」，AI 輔助的部分包括：

- 景點圖片：40 筆景點資料優先使用 Google Places API 取得現成真實照片，缺圖或需要補充素材的景點再用 AI 生成圖像。
- （請依實際使用情況補充：AI 是否也用於生成景點介紹文字、推薦文案，或其他生成內容？）

## 測試紀錄

| 日期 | 測試項目 | 測試方法 | 結果 |
| --- | --- | --- | --- |
| | Flask API 語法檢查 | | |
| | 景點列表 CRUD 測試 | | |
| | 管理頁登入攔截測試 | | |

> 請填入實際測試日期與方法（例如用 Postman／瀏覽器手動測試 CRUD 各項操作）。

## 安裝與執行方式

1. 建立虛擬環境

```
python -m venv .venv
```

2. 安裝套件

```
.venv\Scripts\python -m pip install -r requirements.txt
```

3. 啟動 Flask

```
.venv\Scripts\python app.py
```

4. 開啟網站

```
http://127.0.0.1:5000
```

> 目前前端程式碼裡的 API_BASE 是寫死的區網 IP（`http://192.168.60.11:5000`），正式上傳 GitHub／要讓別人在自己電腦跑起來之前，建議改成 `http://127.0.0.1:5000` 或用相對路徑，否則別人本機執行會連不到後端。

## 開發者資訊

| 項目 | 內容 |
| --- | --- |
| 開發者 | Beem |
| 專題名稱 | AI輔助旅遊景點推薦平台（台南旅遊巡禮） |
| GitHub Repository |https://github.com/flag78631-bot/userpage|
