---
title: 'Habitica #02 — 在本地跑起來'
description: '用 Docker 架設 Habitica 開發環境，為貢獻程式碼做準備'
pubDate: 'Feb 07 2026'
heroImage: '../../assets/blog-placeholder-2.jpg'
---

上一篇我們決定了要對 [Habitica](https://github.com/HabitRPG/habitica) 貢獻。今天來把它跑在本機上。

## 為什麼要本地開發環境？

在送 PR 之前，你需要：
1. 能跑起整個應用程式
2. 確認你的改動真的有用
3. 跑測試確保沒有弄壞其他東西

Habitica 官方有完整的[本地開發指南](https://habitica.fandom.com/wiki/Setting_up_Habitica_Locally)，支援 Linux、MacOS、Windows、和 Docker。

我選 Docker，因為最乾淨、最不容易搞壞系統。

## 技術棧（來自官方文件）

根據 [Guidance for Blacksmiths](https://habitica.fandom.com/wiki/Guidance_for_Blacksmiths)，Habitica 用的技術：

**後端：**
- Node.js + Express.js
- MongoDB + Mongoose
- Gulp（建置工具）

**前端：**
- Vue.js（這是我熟的！）
- Bootstrap + SCSS

**測試：**
- Mocha

看到 Vue.js 就放心了。至少前端的部分我應該看得懂。

## Step 1: Fork 專案

首先，去 [HabitRPG/habitica](https://github.com/HabitRPG/habitica) 點右上角的 Fork。

這會在你的帳號下建立一份副本。所有改動都會先 push 到這裡，再透過 PR 送回主專案。

## Step 2: Clone 到本機

```bash
# 把 YourUsername 換成你的 GitHub 帳號
git clone https://github.com/YourUsername/habitica.git
cd habitica
```

## Step 3: 設定 upstream

```bash
git remote add upstream https://github.com/HabitRPG/habitica.git
```

驗證一下：

```bash
git remote -v
```

應該看到：
```
origin    https://github.com/YourUsername/habitica.git (fetch)
origin    https://github.com/YourUsername/habitica.git (push)
upstream  https://github.com/HabitRPG/habitica.git (fetch)
upstream  https://github.com/HabitRPG/habitica.git (push)
```

## Step 4: 複製設定檔

```bash
cp config.json.example config.json
```

預設設定就夠用了，除非你本機 3000 port 已經被佔用。

## Step 5: 用 Docker 啟動

確保你有裝 [Docker](https://docs.docker.com/get-docker/) 和 docker-compose。

```bash
docker-compose -f docker-compose.dev.yml up -d
```

這個指令會：
1. 建立 Habitica server container
2. 建立 client container  
3. 建立 MongoDB container
4. 安裝所有依賴
5. 啟動開發伺服器

第一次跑會比較久（要下載 image 和 npm install），之後就快了。

## Step 6: 確認跑起來了

```bash
docker ps
```

應該看到三個 container 在跑。

打開 `http://localhost:3000`，應該看到 Habitica 首頁。

## 遇到問題怎麼辦？

官方文件說得很清楚：

> 記錄每一個你輸入的指令和完整輸出。如果遇到錯誤，先停下來解決，不要繼續往下做。

如果真的卡住了，可以到 [GitHub Issues](https://github.com/HabitRPG/habitica/issues) 發問，記得附上：
- Docker 版本
- 作業系統
- 完整錯誤訊息
- 你做了什麼不同的步驟

## 找到值得修的 Issue

現在環境跑起來了，下一步是找一個「help wanted」的 issue 來修。

根據 [Guidance for Blacksmiths](https://habitica.fandom.com/wiki/Guidance_for_Blacksmiths)：

1. 去 [Issues](https://github.com/HabitRPG/habitica/issues) 找有 `help wanted` label 的
2. 讀完 issue 描述和所有留言
3. 確認沒有其他人已經在做
4. **在 issue 留言說你想做，等 admin 回覆才開始寫 code**

這一點很重要：就算是 `help wanted` 的 issue，也要先問過。有時候 issue 已經被修了但沒關，或者有其他考量。

## 下一步

下一篇，我會：
1. 從 `help wanted` issues 中挑一個適合新手的
2. 理解問題和 codebase
3. 開始動手修

---

**資料來源：**
- [Setting up Habitica Locally on Docker](https://habitica.fandom.com/wiki/Setting_up_Habitica_Locally_on_Docker)
- [Guidance for Blacksmiths](https://habitica.fandom.com/wiki/Guidance_for_Blacksmiths)
- [GitHub Labels](https://habitica.fandom.com/wiki/GitHub_Labels)
