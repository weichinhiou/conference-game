# 跨裝置與素材載入檢查紀錄

## 本專案採取的設計

- 黑貓咪使用透明角色素材 `assets/cat-mascot.webp`；寬螢幕和直式手機分別載入 `assets/airport-wide.webp`、`assets/airport-tall.webp`，兩張背景沿用柔和水彩出國視覺。其他動態元素仍由 Canvas 繪製。
- 遊戲結算時將標題、分數與重玩按鈕疊在使用者提供的直式計分卡插畫上；桌機依遊戲區高度縮放，手機依寬度縮放，保持完整構圖不裁切。
- 道具沿用九格原始素材並加入三張返國道具圖；裁切、去除連通的棋盤格背景，整理成 4 欄 × 3 列的單一 WebP 圖集 `assets/items-atlas.webp`（約 193 KB）。以一個相對路徑素材請求取代多張未優化原圖，降低下載量與請求數。開場圖例的小圖以 Canvas 從同一圖集直接裁切繪製；這樣本機 `file://` 開啟和網站部署使用同一繪圖流程，不依賴 CSS 背景定位。所有角色、場景及圖集素材都隨專案保存，不依賴外部圖床。
- 圓角卡片繪圖使用 `roundRect()` 同時保留 `arcTo()` 備援，供較舊 Canvas 實作使用。
- 音效使用 Web Audio API，按效果類別播放不同音型：+1／+2 分、+5 分、加秒、扣分、扣秒與卡住各有明確差異；設計沿用《寶寶過中秋》的音效對應。程式在開始按鈕、音效按鈕及觸控結束事件嘗試恢復 AudioContext，以涵蓋 iOS 對使用者手勢的限制。
- 採用 Pointer Events 拖曳，保留 `touch-action: none`、指標取消與失焦清理；支援直式手機、橫式及桌面。

- 2026-09-27：參考 baby-mid-autumn-game 的 *-fast.webp 做法，將開場會直接載入的 cat-paw.png（約 1.50 MB）與 conference-badge-large.png（約 789 KB）轉為 assets/cat-paw-fast.webp（約 113 KB）及 assets/conference-badge-large-fast.webp（約 94 KB），並更新 index.html 引用。兩張圖片合計從約 2.28 MB 降至約 207 KB；原始 PNG 保留作為素材來源。另產生 conference-badge-small-fast.webp 與 og-image-fast.webp 備用，但目前頁面未直接載入後兩者。

## 驗證狀態

- 已執行 JavaScript 語法檢查與啟動流程 smoke test。
- 尚未在 iPhone、iPad 或舊版 iOS Safari 實機驗證音效和觸控。桌面瀏覽器手機尺寸模擬不能代表 iOS 實測。
- 已檢查本機檔案大小與路徑引用；尚未在實際慢速行動網路、冷快取及不同手機瀏覽器量測首屏秒數，故不宣稱特定裝置的實際載入時間。

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
