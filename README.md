# 實驗室取樣備料系統

實驗室人員建立取樣需求，備料人員在備料時一併取樣並確認完成。  
前端靜態部署於 GitHub Pages，資料庫使用 Supabase。

---

## 檔案結構

```
專案資料夾/
├── lab.html       # 實驗室人員使用
├── pick.html      # 備料人員使用
└── README.md      # 本說明文件
```

## 已部署網址

- [實驗室頁面](https://btohri.github.io/lab-sampling-system/lab.html)
- [備料頁面](https://btohri.github.io/lab-sampling-system/pick.html)
- [GitHub 專案](https://github.com/btohri/lab-sampling-system)

---

## 系統架構

```
GitHub Pages（靜態前端）
    ↕ Supabase JS Client（REST API）
Supabase PostgreSQL（資料庫）
```

- 無需後端伺服器，純靜態網頁直連 Supabase
- 兩個頁面共用同一張資料表 `lab_samples`

---

## 部署步驟

### Step 1：建立 Supabase Table

登入 [Supabase](https://supabase.com) → 選擇專案 → SQL Editor，執行以下語法：

```sql
create table lab_samples (
  id uuid primary key default gen_random_uuid(),
  item_code text not null,
  item_name text not null,
  work_order text,
  status text not null default 'pending',
  created_at timestamptz default now(),
  done_at timestamptz
);

-- 啟用 Row Level Security（開放讀寫）
alter table lab_samples enable row level security;
create policy "allow all" on lab_samples for all using (true) with check (true);
```

### Step 2：取得 Supabase 連線資訊

Supabase 專案頁面 → Settings → API，複製：
- **Project URL**（格式：`https://xxxxxxxx.supabase.co`）
- **Publishable key**

### Step 3：填入兩個 HTML 檔案

`lab.html` 與 `pick.html` 頂部各有一段設定，替換成你的資料：

```js
const SUPABASE_URL = 'https://YOUR_PROJECT.supabase.co';  // ← 替換
const SUPABASE_ANON_KEY = 'YOUR_PUBLISHABLE_KEY';         // ← 替換
```

### Step 4：推上 GitHub Pages

```bash
git init
git add .
git commit -m "init"
git remote add origin https://github.com/btohri/lab-sampling-system.git
git push -u origin main
```

[GitHub repo](https://github.com/btohri/lab-sampling-system) → Settings → Pages → Branch: `main` → Save

部署完成後取得兩個網址分發給對應人員：
- [https://btohri.github.io/lab-sampling-system/lab.html](https://btohri.github.io/lab-sampling-system/lab.html)（實驗室）
- [https://btohri.github.io/lab-sampling-system/pick.html](https://btohri.github.io/lab-sampling-system/pick.html)（備料）

---

## 頁面功能說明

### `lab.html`　實驗室人員

| 功能 | 說明 |
|------|------|
| 新增取樣項目 | 填入料號、名稱，工單號可選填 |
| 編輯 | 料號、名稱、工單號均可修改 |
| 刪除 | 刪除前會跳出確認視窗 |
| 查看清單 | 顯示所有項目，含狀態、建立時間、完成時間 |
| 手動重新整理 | 點擊「↻ 重新整理」更新資料 |

**操作流程：**
1. 左側填入料號、名稱（工單號選填）
2. 按「＋ 新增取樣項目」
3. 右側清單即時更新
4. 需修改時按該列「編輯」，修改後按「💾 儲存修改」
5. 需刪除時按「刪除」並確認

---

### `pick.html`　備料人員

| 功能 | 說明 |
|------|------|
| 待備料清單 | 顯示所有狀態為「待備料」的項目 |
| 工單備註 | 若有對應工單號則顯示，無則顯示「無對應工單」 |
| 完成備料 | 按「完成備料」後狀態改為已完成，並記錄完成時間 |
| 已完成清單 | 切換分頁查看已完成項目及完成時間 |
| 自動更新 | 每 30 秒自動重新整理一次 |

**操作流程：**
1. 開啟頁面，預設顯示「待備料」清單
2. 備料時找到對應料號，確認工單備註
3. 取樣完成後按「完成備料」
4. 該項目移至「已完成」分頁，並記錄完成時間

---

## 資料欄位說明

| 欄位 | 類型 | 說明 |
|------|------|------|
| `id` | uuid | 自動產生，唯一識別碼 |
| `item_code` | text | 料號，例如 `11001026` |
| `item_name` | text | 名稱，例如 `植物固醇` |
| `work_order` | text | 對應工單號，例如 `10091560`，可為空 |
| `status` | text | `pending`（待備料）或 `done`（已完成）|
| `created_at` | timestamptz | 建立時間，自動填入 |
| `done_at` | timestamptz | 完成時間，備料人員按完成時填入 |

---

## 狀態流程

```
實驗室新增
    ↓
pending（待備料）  ──→  備料人員看到，取樣後按完成
    ↓
done（已完成）         記錄完成時間
```

---

## 注意事項

- 本系統無登入機制，請自行管理網址的分發範圍
- Publishable key 為前端可見，建議 Supabase RLS policy 僅開放必要操作，正式使用可依需求收緊權限
- 刪除操作無法復原，請謹慎使用
- 修改 `lab.html` 與 `pick.html` 後需重新推送至 GitHub 才會生效
