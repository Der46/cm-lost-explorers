# CLAUDE.md

本檔提供 Claude Code（及其他協作者）在此 repo 工作時所需的背景與規範。

## 專案概要

- **內容**：Coin Master「失落探險隊（Lost Explorers）」活動攻略頁，說明活動玩法、主活動獎勵、恐龍血量、全隊估算、下注順序與每週活動時間表。
- **署名**：大佬群CSL製作。頁首印章與頁尾 `© {年份} 大佬群CSL製作` 必須保留；瀏覽器分頁標題只顯示各語言的活動名稱，不加署名。
- **線上網址**：https://der46.github.io/cm-lost-explorers/
- **部署**：GitHub Pages，來源為 `main` 分支根目錄。push 到 `main` 後約 30 秒自動重新部署，不需建置步驟。

## 檔案結構

| 檔案 | 用途 |
|---|---|
| `index.html` | 整個網站：CSS、SVG sprite、HTML、資料、翻譯與 JS 全部內嵌於單一檔案 |
| `README.md` | 對外說明與網址 |
| `.nojekyll` | 讓 GitHub Pages 略過 Jekyll 處理 |
| `CLAUDE.md` | 本檔 |

刻意維持**單檔、零依賴、無建置工具**。外部資源只有 Google Fonts（Baloo 2、Nunito）；CJK 與泰文依賴系統字型 fallback（見 `--font-display`、`--font-body`）。除非使用者明確要求，不要引入框架、打包工具或 npm 套件。

## `index.html` 內部結構

依序為：

1. **`<style>`**：以 `/* ---------- 區塊名 ---------- */` 分段（header、hero、sections、prize trail、dinos、plan、bets、schedule、footer）。
2. **SVG sprite**（`<svg class="sprite">`）：
   - `<defs>` 內是共用漸層（`gGold`、`gGlass`、`gCoin`…）與 `chestClip`。
   - 獎勵圖示以 `r-` 開頭（`r-chest-green`、`r-token`、`r-spins`、`r-chest-mystery`、`r-chest-purple`、`r-tokenbox`、`r-chest-red`、`r-spins-grand`），依使用者提供的附圖繪製；寶箱共用 `chest-frame`。
   - 介面小圖示以 `i-` 開頭（`i-flag`、`i-elixir`、`i-dino`、`i-bolt`…）。
   - 不使用 emoji 當圖示，一律用 sprite 的 `<use href="#id">`；JS 端用 `ico(id, viewBox)` 產生。
3. **HTML 區塊**：`.hero`、`#how`、`#prizes`、`#dinos`、`#plan`、`#bets`、`#schedule`、`footer`。靜態文字以 `data-i18n="key"` 標記，動態內容由 JS 渲染進對應容器。
4. **`<script>`**（單一 IIFE，ES5 寫法：`var`、`function`，請維持一致）：
   - **資料**：`DINO_HP`（20 隻恐龍血量）、`PRIZES`（8 關獎勵：旗數 `cost`、圖示、名稱 key、`detail`）、`SEQS`（兩套下注順序）、`SCHEDULE`（每週活動）。
   - **翻譯**：`I18N`（主要字串）與 `EXTRA`（時區相關字串，啟動時合併進 `I18N`）。
   - **時區**：`SRC_TZ` / `SRC_OFF_H`（原始資料為台灣時間 UTC+8）、`ZONES`（各語言對應時區）。
   - **工具函式**：`T(key, vars)` 翻譯並代入 `{var}`，缺字時退回英文；`N()` 依語系格式化數字；`esc()` HTML 跳脫；`ico()` 產生圖示。
   - **渲染**：`renderAll()` 依序呼叫 `renderStatic`、`renderPrizes`、`renderDinos`、`renderPlan`、`renderSeqs`、`renderSchedule`、`syncHeader`；切換語言時整頁重繪。

## 多語系

支援 9 種語言，順序與下拉選單一致：

| 代碼 | 選單顯示 | 時區（`ZONES`） |
|---|---|---|
| `zh-Hant` | 中文 | Asia/Taipei |
| `en` | English | America/New_York |
| `vi` | Tiếng Việt | Asia/Ho_Chi_Minh |
| `ja` | 日本語 | Asia/Tokyo |
| `ko` | 한국어 | Asia/Seoul |
| `th` | ไทย | Asia/Bangkok |
| `id` | Bahasa Indonesia | Asia/Jakarta |
| `es` | Español | America/Mexico_City |
| `pt` | Português | America/Sao_Paulo |

- 語言決定順序：網址 `?lang=` → `localStorage` 的 `cm-lost-lang` → 瀏覽器語言（任何 `zh*` 皆對應 `zh-Hant`）→ 預設中文。
- **新增或修改文字時，9 種語言都要同步更新**，不可只改中文。字串內的變數用 `{name}` 佔位，數字一律經 `N()` 格式化後再代入。
- 活動名稱（各語言的 `ev` 物件）的外語版本為字面翻譯，非遊戲官方譯名；若使用者提供官方名稱，以官方為準。
- 泰文、越南文有上下附加符號，行高需較寬鬆（已有 `html[lang="th"]` 規則）。任何使用 `background-clip:text` 的文字（例如 hero 標題）都要留足行高與上下 padding，否則字形會被切掉。

## 時間表與時區規則

- `SCHEDULE` 的 `d` 為 0–6（星期一至星期日），`d: 7` 代表下週一；`time` 一律填**台灣時間**。
- 星期六 `00:00` 的正常競賽是**星期六一開始**的 00:00。
- `renderSchedule()` 以「本週台灣時間星期一 00:00」為基準，把每個活動換成絕對時間，再依目前語言的時區重新分組到當地日期；同一個星期名稱第二次出現時以 `nextDay` 字串標示（例如「下星期一」）。
- 「今天」標記以所選時區的當地日期判斷。
- 時區標籤（例如 `UTC+8`）由 `offsetLabel()` 動態計算，會自動反映夏令時間。

## 資料規則

- 數值以使用者提供的社群攻略圖為準（2026 年 9 月資料）。`DINO_HP` 總和應為 1,541,000，4 人平均 385,250。
- 魔藥換算：1,000 魔藥 ≈ 30,000–40,000 點（常數 `PTS_LO`、`PTS_HI`、`PTS_PER_ELX`）。
- 下注倍數 ×1 = 10 魔藥。
- 主活動獎勵條為**靜態展示**，重點是讓使用者看到每關獎勵與所需旗數；不要加入「擊倒幾隻恐龍」的滑桿或進度互動。
- 「集中攻擊建議」與「隊伍固定 4 人」字樣已依使用者要求移除；除非使用者再次要求，否則不要加回。

## 設計規範

- 主題：古代叢林與恐龍。色彩以 `:root` 的 CSS 變數為準（`--canopy` 深叢林綠底、`--parchment` 羊皮紙卡片、`--amber` 琥珀強調、`--lava` 熔岩紅、`--moss` 苔綠）；新增元素時使用既有變數，不要另寫色碼。
- 字型：標題 `--font-display`（Baloo 2），內文 `--font-body`（Nunito）。
- 視覺重點只有一個（hero 標題與場景），其餘區塊保持克制，不要加入大量進場動畫；既有動畫需遵守 `prefers-reduced-motion`。
- 無障礙：可點擊元素最小 44×44px，保留 `:focus-visible` 外框，裝飾性 SVG 加 `aria-hidden="true"`。
- 頁首為 sticky；`syncHeader()` 會把實際高度寫入 `--hdr`，各 section 以 `scroll-margin-top` 避免標題被遮擋。調整頁首時要確認錨點跳轉不會蓋住標題。

## RWD

- 主要斷點：1180px（導覽列換到第二行並可橫向捲動）、980px、900px、860px、700px、560px（隱藏頁首品牌文字）、520px、440px、420px、400px。
- 驗收寬度至少：375px、768px、1024px、1440px。任何語言在 375px 下都不得出現整頁橫向捲動。

## 修改後的驗證

本專案沒有自動化測試，修改後請在瀏覽器實際檢查：

1. 開啟 `index.html`（或線上網址），切換全部 9 種語言，確認沒有 console 錯誤，也沒有顯示成原始 key 的字串。
2. 在 375px 寬度下確認 `document.documentElement.scrollWidth <= innerWidth`，且 hero 標題字形完整（上下緣沒有被切到）。
3. 確認恐龍血量合計為 1,541,000，全隊估算為每人 385,250 點、約 9,600–12,800 魔藥。
4. 時間表：中文版星期六依序為 00:00、22:05；切換日文時星期六應為 01:00、23:05。

## Git 與部署

- 直接在 `main` 上提交並 push（使用者要求修改後直接 push）。
- commit 訊息用英文祈使句標題，內文條列重點。
- push 後可用以下指令確認 Pages 已部署到最新 commit：

```bash
gh api repos/Der46/cm-lost-explorers/pages/builds/latest --jq '.status+" "+.commit[0:7]'
```
