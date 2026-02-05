---
title: "工具介紹：GhostURL - 看不見的連結產生器"
description: "了解 GhostURL 如何透過在 URL 中插入零寬空格，使其在大多數平台上變得無法點擊，以及其背後的技術原理與應用場景。"
pubDate: 2026-02-06T00:00:00Z
categories: ["Projects", "Tools"]
tags: ["JavaScript", "URL", "Privacy"]
---

### LOG_ENTRY: 20260206

### SUBJECT: 新工具發布 - GhostURL

在 Mountos codeLab 的專案庫中，我們新增了一個實用的小工具：**GhostURL**。

GhostURL 的核心功能非常簡單：它能將您提供的任何 URL，轉換成一個看起來完全一樣，但在大多數社群媒體平台或訊息應用程式中都「無法點擊」的版本。

您可以在這裡親自體驗：[GhostURL 工具頁面](/projects/ghosturl)

### // 它是如何運作的？

這個工具的秘密武器是「零寬空格」（Zero-Width Space, `U+200B`）。這是一個在 Unicode 編碼中定義的特殊字元，它在視覺上完全不可見，不佔據任何寬度，但它確實存在於字串中。

當您輸入一個 URL，例如 `https://example.com`，GhostURL 會在原始 URL 的每個字元之間插入一個零寬空格，產生如下結果（當然，您用肉眼是看不出來的）：

`h​t​t​p​s​:​/​/​e​x​a​m​p​l​e​.​c​o​m`

這個小小的改動，足以騙過許多平台的自動連結辨識演算法。這些平台在掃描文字時，因為字元之間被零寬空格打斷，所以不會將其辨識為一個完整的、可點擊的 URL。

![GhostURL 工具介面](/images/blog/ghosturl-tool.png)
*上圖：GhostURL 的使用者介面。請將您的工具截圖命名為 `ghosturl-tool.png` 並放置在 `public/images/blog/` 資料夾中以取代此處的佔位符。*

### // 主要應用場景

您可能會問，為什麼我們需要這樣的工具？以下是一些實用的場景：

1.  **避免自動預覽**：在某些平台（如 Facebook, Line, Telegram）貼上連結時，系統會自動抓取連結的預覽圖文，有時這並不是我們想要的。使用 GhostURL 可以避免觸發這個功能。
2.  **分享純文字格式的連結**：當您希望接收者手動複製貼上連結，而不是直接點擊時，這個工具就非常有用。
3.  **繞過簡單的關鍵字過濾**：在某些情況下，自動化系統可能會過濾特定的網域名稱。雖然這不是一個可靠的安全措施，但零寬空格有時能繞過非常基礎的文字匹配規則。

### // 技術實現

整個工具完全在您的瀏覽器前端運行，使用 JavaScript 實現。核心的轉換邏輯非常直接：

```javascript
// 取得使用者輸入的 URL
const url = inputUrl.value.trim();

// 使用 split 和 join 在每個字元間插入零寬空格
const ghostUrl = url.split('').join('\u200B');

// 將結果顯示在輸出區域
outputUrl.textContent = ghostUrl;
```

因為所有操作都在本地完成，您的資料無須上傳到任何伺服器，完全確保了隱私與安全。這也是我們開發 codeLab 系列工具時秉持的重要原則之一。

### END_OF_LOG
