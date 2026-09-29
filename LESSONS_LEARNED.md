# 跨裝置與素材載入檢查紀錄

## 本專案採取的設計

- 黑貓咪使用透明角色素材 `assets/cat-mascot.webp`；寬螢幕和直式手機分別載入 `assets/airport-wide.webp`、`assets/airport-tall.webp`，兩張背景沿用柔和水彩出國視覺。其他動態元素仍由 Canvas 繪製。
- 遊戲結算時將標題、分數與重玩按鈕疊在使用者提供的直式計分卡插畫上；桌機依遊戲區高度縮放，手機依寬度縮放，保持完整構圖不裁切。
- 道具沿用九格原始素材並加入三張返國道具圖；裁切、去除連通的棋盤格背景，整理成 4 欄 × 3 列的單一 WebP 圖集 `assets/items-atlas.webp`（約 193 KB）。以一個相對路徑素材請求取代多張未優化原圖，降低下載量與請求數。開場圖例的小圖以 Canvas 從同一圖集直接裁切繪製；這樣本機 `file://` 開啟和網站部署使用同一繪圖流程，不依賴 CSS 背景定位。所有角色、場景及圖集素材都隨專案保存，不依賴外部圖床。
- 圓角卡片繪圖使用 `roundRect()` 同時保留 `arcTo()` 備援，供較舊 Canvas 實作使用。
- 音效使用 Web Audio API，按效果類別播放不同音型：+1／+2 分、+5 分、加秒、扣分、扣秒與卡住各有明確差異；設計沿用《寶寶過中秋》的音效對應。程式在開始按鈕、音效按鈕及觸控結束事件嘗試恢復 AudioContext，以涵蓋 iOS 對使用者手勢的限制。
- 採用 Pointer Events 拖曳，保留 `touch-action: none`、指標取消與失焦清理；支援直式手機、橫式及桌面。

- 2026-09-27：參考 baby-mid-autumn-game 的 *-fast.webp 做法，將開場會直接載入的 cat-paw.png（約 1.50 MB）與 conference-badge-large.png（約 789 KB）轉為 assets/cat-paw-fast.webp（約 113 KB）及 assets/conference-badge-large-fast.webp（約 94 KB），並更新 index.html 引用。兩張圖片合計從約 2.28 MB 降至約 207 KB；原始 PNG 保留作為素材來源。另產生 conference-badge-small-fast.webp 與 og-image-fast.webp 備用，但目前頁面未直接載入後兩者。

- 2026-09-29：新增三張透明 WebP 動作圖：`cat-catch-good.webp`（正向接到）、`cat-catch-bad.webp`（負向接到）、`cat-stunned.webp`（被卡住）。`applyCatch()` 依道具 `kind` 切換正／負動作圖，`stun` 期間固定顯示暈眩圖，動作結束後回到待機貓咪。

## 驗證狀態

- 已執行 JavaScript 語法檢查與啟動流程 smoke test。
- 尚未在 iPhone、iPad 或舊版 iOS Safari 實機驗證音效和觸控。桌面瀏覽器手機尺寸模擬不能代表 iOS 實測。
- 已檢查本機檔案大小與路徑引用；尚未在實際慢速行動網路、冷快取及不同手機瀏覽器量測首屏秒數，故不宣稱特定裝置的實際載入時間。

## 待處理

- iPad／iOS Web Audio：目前 YouTube 有聲，但本遊戲在 iPad 的 Safari 與 Chrome 仍無聲；需取得 Safari Web Inspector 的 `AudioContext.state`、系統靜音／音量與輸出路由資訊，再確認是否需要改用 HTMLAudio fallback。

## 每次修改後的檢查清單

1. 檢查 `index.html` 的 JavaScript 語法；確認首次載入的角色、對應方向背景和單一輕量道具圖集均為本機相對路徑素材。
2. 在桌面測開始／倒數／結算、鍵盤控制、十二種道具效果、音效切換及重新開始。
3. 測手機直式與橫式：Overlay 可捲動、圖例沒有被裁切、拖曳不會捲動整頁、遊戲區沒有超出視窗。
4. 以 iOS Safari 真機測試：先確認實體音量，再按開始並接到物件；分別測音效預設開啟、關閉後重新開啟。
5. 若替換或加入圖片，確認檔案大小、透明背景、圖集裁切、相對路徑與快取更新；GitHub Pages 專案站避免使用指向網站根目錄的錯誤絕對路徑。
6. 若變更 Canvas API，確認目標舊版瀏覽器支援情況並提供備援；不可只用桌面 F12 裝置模擬宣稱完成 iPhone 實測。


## 踩坑紀錄：圖片載入過慢與 fast.webp

### 症狀

首次開啟遊戲時，開場徽章與標題圖示載入較慢；原始 PNG 合計約 2.28 MB，會讓慢速網路或冷快取的首屏等待變明顯。

### 解決方案

參考 `baby-mid-autumn-game`，保留原始 PNG 作為素材來源，另外產生適合網頁傳輸的 `*-fast.webp`：

- `assets/cat-paw.png` 約 1.50 MB → `assets/cat-paw-fast.webp` 約 113 KB。
- `assets/conference-badge-large.png` 約 789 KB → `assets/conference-badge-large-fast.webp` 約 94 KB。
- `index.html` 改用兩個 `fast.webp`；原始 PNG 不再由遊戲頁面直接載入。
- 優化後主要啟動圖片約 207 KB，減少約 91%。

### 驗證與限制

已確認檔案存在、路徑引用正確，並通過 inline JavaScript 語法檢查；尚未以冷快取、慢速行動網路及多種實機量測首屏秒數，因此不宣稱特定裝置的實際載入時間。

## 踩坑紀錄：iPad／iOS Web Audio 沒有聲音

### 症狀與線索

寶寶月餅遊戲曾遇到 iPad 或舊版 iOS Safari 沒有音效。桌面瀏覽器的 iPad 尺寸模擬不能代表 iPad 實機；iOS 對 Web Audio 的使用者手勢、AudioContext 狀態與音訊路由有額外限制。單純呼叫 `resume()` 不等於音效已解鎖。

### 解決方案

本專案在音效按鈕與開始按鈕的 `touchstart`、`touchend`、`pointerdown` 使用者手勢中呼叫 `ensureAudio(true)`；建立或恢復 AudioContext 後，以極低音量的短 oscillator 做一次解鎖 priming，並在分頁回到前景時再次嘗試恢復。保留 `resume()` Promise 的失敗處理，避免未處理 rejection。

### 驗證與限制

已通過 JavaScript 語法檢查並確認事件綁定；尚未取得本次 iPad 實機的 Safari Web Inspector console 或回歸錄影，因此仍需 Jean 在同一台 iPad 上重新開啟最新版網址、按開始並實際接到道具確認。若仍無聲，下一步記錄 `AudioContext.state`、系統靜音／音量與音訊輸出路由。
## 2026-09-29：底圖冷快取載入優化

### 診斷結果

- 原本 `airport-wide.webp` 約 162 KB、`airport-tall.webp` 約 191 KB；程式初始化時兩張都設定 `src`，手機也會下載未使用的橫式底圖。
- GitHub Pages 強制下載量測各約 2.4～2.8 秒，回應 `Cache-Control: max-age=600`；一般快取不能涵蓋硬重新整理、Safari 記憶體回收或超過快取時間的情境。
- 原本首屏還會同時請求貓咪正／負／暈眩動作圖與其他素材，可能與底圖競爭頻寬。

### 採用方案

- 新增 `assets/airport-wide-fast.webp`（720 × 405，約 23 KB）與 `assets/airport-tall-fast.webp`（540 × 960，約 34 KB）。
- 依目前畫面方向只啟動一組底圖；fast 圖完成後再以低優先權載入高畫質版本。
- 貓咪接物動作圖改在按下開始後延後載入；原圖不覆蓋，原本高畫質底圖仍保留。
- 此次程式與素材 commit：`ebd9a8b`；回復基準：`checkpoint-before-background-optimization`／`d966977`。

### 驗證與限制

- 已通過 inline JavaScript `node --check`、fast.webp 尺寸與檔案存在檢查、方向載入條件靜態檢查及 `git diff --check`。
- 尚未在真實 iPhone／iPad、慢速行動網路與冷快取實測；部署後需比較首次開啟的底圖出現時間與清晰度。
## 2026-09-29：物件圖集 fast.webp 載入優化

- `items-atlas.webp` 保留 800 × 600 的高畫質圖集與既有 4 × 3 裁切座標，另外產生同尺寸的 `items-atlas-fast.webp`。
- fast 圖集約 87 KB，相較原圖約 193 KB 減少約 55%；圖例與掉落物先用 fast 圖集，原圖在 fast 圖集完成後以低優先權載入並替換。
- 不拆成 12 個獨立圖片請求，避免增加手機網路連線與 HTTP 請求數。
- 已通過 inline JavaScript `node --check`、兩種圖集尺寸／模式檢查、fast／full 繪製路徑靜態檢查與 `git diff --check`。初次靜態 assertion 將兩種 Canvas API 誤當成同一個呼叫，修正檢查條件後通過。
- 尚未以真實手機冷快取量測圖例首次出現時間；部署後需確認 fast 圖清晰度與高畫質替換是否正常。
## 2026-09-29：回合時間與結算卡版面調整

- 回合時間由 30 秒延長為 60 秒，開場畫面、初始化計時、重新開始與 README 已同步更新。
- `返國後資料逾期30天` 的 5 秒卡住物件權重由 8.5% 降至 5.5%；差額補給 `會議資料登錄(返國)`，維持總權重 100%，3 秒卡住的 `公假上簽不足45天前` 不變。
- 結算卡桌機與手機版的上內距由 10% 調為 15%，讓標題與分數往下移，避免貼住結算圖上緣。
- 已通過 inline JavaScript `node --check`、設定值靜態檢查與 `git diff --check`；尚未以真實手機完成 60 秒完整回合與結算卡實機目測。
## 2026-09-30：降低卡住物件機率與結算卡位置

- 3 秒卡住物件 `deadline` 權重由 `8.5` 降為 `5.5`。
- 5 秒卡住物件 `overdue` 權重由 `5.5` 降為 `3.5`。
- `rndItem()` 改以所有道具權重總和作為抽取上限，降低個別權重後仍維持正確相對比例，不會因固定使用 100 造成尾端 fallback 偏差。
- 結算卡桌機與手機版的上方 padding 由 `15%` 調整為 `calc(15% + 2.4em)`，讓結算文字下移約兩行。