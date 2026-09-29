# 出國會議注意事項｜接接樂

以《寶寶過中秋》接物遊戲改作的靜態網頁小遊戲。玩家拖動黑貓咪，在限時內接住出國會議準備文件，避開護照、身分證與期限警示，並留意返國後的資料登錄、心得報告和逾期提醒。

## 開始遊戲

直接用瀏覽器開啟 `index.html`，不需安裝套件、登入或連線資料庫。部署到 GitHub Pages 時，將本資料夾內容放入儲存庫根目錄即可。

## 操作

- 手機：按住黑貓咪左右拖動，也可按住畫面下方的左／右移動鍵。
- 電腦：按住黑貓咪拖動，也可用 `←`／`→` 或 `A`／`D`。
- 按右上角喇叭切換音效。iOS 音訊會在使用者觸控操作中嘗試解鎖。

## 遊戲設定

- 起始 30 秒，倒數 3、2、1 後才開始計時與掉落。
- 遊玩累積滿 30 秒後升速，之後每 10 秒升一階。
- 十二種道具的實際效果依 [`接接樂_遊戲道具清單.md`](接接樂_遊戲道具清單.md) 和使用者提供的素材圖製作。
- 道具機率未由清單指定，因此採用可調的初版平衡值，文件中已標明。

## 專案結構

```text
conference-game/
├── index.html
├── README.md
├── LESSONS_LEARNED.md
├── 接接樂_遊戲道具清單.md
├── favicon-transparent.ico
├── favicon-transparent.png
├── og-image.png
└── assets/
    ├── cat-mascot.webp
    ├── cat-catch-good.webp
    ├── cat-catch-bad.webp
    ├── cat-stunned.webp
    ├── cat-paw-fast.webp
    ├── conference-badge-small-fast.webp
    ├── conference-badge-large-fast.webp
    ├── airport-wide.webp
    ├── airport-tall.webp
    ├── items-atlas.webp
    └── scorecard-bg.webp
```

角色、直橫式出國場景、十二種道具圖集與計分卡皆使用本機 WebP 素材；遊戲中的背景合成、角色移動與掉落效果以 Canvas 繪製。圖集約 193 KB，不依賴第三方 JavaScript 套件。不依賴第三方 JavaScript 套件。

