# Dram 共同投資帳 — 靜態發布版

給股東看的唯讀報表。資料與規則的**主檔在 Claude Artifact**，這裡只是定期產出的快照。

## 檔案

| 檔案 | 說明 |
|---|---|
| `index.html` | 頁面本身。整份自給自足，不需要後端 |
| `data.json` | 全部資料（參數、戶別、交易、出資）。**更新報表 = 換掉這一個檔** |

## 兩種檢視模式

| 網址 | 模式 |
|---|---|
| `.../index.html` | **唯讀**。隱藏所有新增／編輯／刪除功能與參數列 |
| `.../index.html?edit=1` | **試算**。顯示編輯功能，但改動只存在當下這個瀏覽器分頁，重新整理就回到 `data.json` 的內容 |

`?edit=1` 不會寫回 `data.json`，所以不用擔心有人改壞了對外的數字。

## 怎麼更新

1. 在 Claude 的 Artifact 版更新交易／出資／市價
2. 請 Claude 重新匯出 `data.json`
3. 把新的 `data.json` commit 上來

`index.html` 只有在頁面功能要改時才需要重新產生（`python3 build_gh.py`）。

## 資料結構（data.json）

```jsonc
{
  "generatedAt": "產出時間（UTC）",
  "params":  { "price": 市價, "fx": 匯率, "updated": "資料更新日期",
               "policyBuyFrom": "...", "policySellFrom": "...",
               "policyBuyMode": "proRataCash", "policySellMode": "prorata" },
  "buckets": [ { "id": "pool|bo_solo|solo_...", "label": "顯示名稱", "owner": "擁有人或 null" } ],
  "trades":  [ { "id": "...", "date": "YYYY-MM-DD", "side": "買|賣",
                 "units": 單位數, "price": 單價, "fee": 手續費, "amt": 成交金額（原幣）,
                 "splits": { "戶id": 該戶出資NT$ },   // 買進，手動指定時才有
                 "splitsManual": true,                // 有此旗標代表不套用自動規則
                 "sellAlloc": { "戶id": 單位數 },      // 賣出，手動指定時才有
                 "sellMode": "prorata|soloFirst",
                 "note": "備註" } ],
  "contribs":[ { "id": "...", "date": "YYYY-MM-DD", "bucket": "戶id", "name": "姓名",
                 "dir": "存入|提領|補償收|補償付", "amt": NT$,
                 "role": "fixed|residual", "note": "備註" } ]
}
```

### 注意事項

- **`amt` 優先於 `units × price`。** 對帳單上的成交金額直接填進 `amt`，價格只作顯示用。
- **`splitsManual: true` 的交易不可套用自動分配規則**（目前是 7/01、7/13、7/15 三筆）。
- **`dir` 為 `補償收` / `補償付`** 的出資紀錄是帳戶外的私下結算：不動現金、不動份額、不動持有單位，只調整該人的最終損益，在總表上獨立列為「補償／調整」。三人合計應為 0。

## 驗算錨點

每次更新後，這三個數字必須與 DBS 對帳單吻合：

| 項目 | 應為 |
|---|---|
| 持有單位數 | 391 |
| FIFO 成本基礎 | US$22,138.25 |
| 共同池現金最低餘額 | ≥ 0（頁面下方「資金流水」會自動警示） |
