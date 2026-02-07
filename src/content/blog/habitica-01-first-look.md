---
title: '第一眼看 Habitica — 一個 12K stars 的開源專案長什麼樣？'
description: '開始探索 Habitica，一個用 Vue + Node 打造的遊戲化任務管理 App'
pubDate: 'Feb 07 2026'
heroImage: '/blog-placeholder-1.jpg'
tags: ['habitica', 'vue', 'open-source', 'beginner']
---

# 為什麼選 Habitica？

我是一個 Vue.js 開發者，想開始貢獻開源。但每次看到一個有趣的專案，打開 GitHub 後就被嚇到 — 幾百個檔案、幾十個資料夾，完全不知道從哪看起。

這個系列會記錄我**真實的探索過程**，包括困惑、卡關、和最終送出第一個 PR 的完整旅程。

**為什麼選 Habitica？**

1. **Tech Stack 對口** — Vue + Node + MongoDB，跟我熟悉的技術一致
2. **專案有趣** — 遊戲化的任務管理 App，讓完成待辦事項變成打怪升級
3. **社群友善** — 有完整的 contribution guide，還有 Discord 可以問問題
4. **有 good first issues** — 標記了適合新手的 issues

---

# 第一步：看 README

打開 [github.com/HabitRPG/habitica](https://github.com/HabitRPG/habitica)，首先看 README。

README 告訴我：

> Habitica is a habit tracker app which treats your goals like a Role Playing Game.

接著有一段 **Contributing** 的連結，指向他們的 Wiki。這是好兆頭！代表他們重視新貢獻者的體驗。

---

# 專案結構初探

Clone 下來後，第一眼看到的資料夾結構：

```
habitica/
├── website/          # 主要的 webapp
│   ├── client/       # Vue frontend
│   └── server/       # Node.js backend
├── migrations/       # 資料庫 migrations
├── test/             # 測試
└── ...
```

好消息：前後端分離，結構清楚。

我的第一個目標：先讓它在本地跑起來。

---

# 下一步

1. 照著 Wiki 的 [Setting up Habitica Locally](https://habitica.fandom.com/wiki/Setting_up_Habitica_Locally) 設定環境
2. 把 app 跑起來，到處點點看
3. 找一個 `good first issue` 來嘗試

---

*這是 Zero to PR 系列的第一篇。我會持續更新這個旅程的每一步。*
