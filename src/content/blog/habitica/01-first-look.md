---
title: '第一眼看 Habitica — 一個 12K stars 的開源專案長什麼樣？'
description: '開始探索 Habitica，一個用 Vue + Node 打造的遊戲化任務管理 App'
pubDate: 'Feb 07 2026'
heroImage: '../../../assets/blog-placeholder-1.jpg'
tags: ['habitica', 'vue', 'open-source', 'beginner']
---

# 為什麼選 Habitica？

我是一個 Vue.js 開發者，想開始貢獻開源。但每次看到一個有趣的專案，打開 GitHub 後就被嚇到 — 幾百個檔案、幾十個資料夾，完全不知道從哪看起。

這個系列會記錄我**真實的探索過程**，包括困惑、卡關、和最終送出第一個 PR 的完整旅程。

## 為什麼選 Habitica？

1. **Tech Stack 對口** — Vue + Node + MongoDB，跟我熟悉的技術一致
2. **專案有趣** — 遊戲化的任務管理 App，讓完成待辦事項變成打怪升級
3. **社群友善** — 有完整的 contribution guide，還有 Discord 可以問問題
4. **有 Help Wanted issues** — [目前有 12+ 個 issues 等著人幫忙](https://github.com/HabitRPG/habitica/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22)

---

# 第一步：看 README

打開 [github.com/HabitRPG/habitica](https://github.com/HabitRPG/habitica)，首先看 README。

README 告訴我：

> Habitica is an open-source habit-building program that treats your life like a role-playing game. Level up as you succeed, lose HP as you fail, and earn Gold to buy weapons and armor!

接著有兩個重要連結：
- [Guidance for Blacksmiths](https://habitica.fandom.com/wiki/Guidance_for_Blacksmiths) — 技術介紹，他們叫貢獻者「鍛造師」，很有 RPG 氛圍！
- [Setting up Habitica Locally](https://github.com/HabitRPG/habitica/wiki/Setting-Up-Habitica-for-Local-Development) — 本地開發環境設定

---

# 看一下 Help Wanted Issues

打開 [Help Wanted issues](https://github.com/HabitRPG/habitica/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22)，目前最新的幾個：

1. **[#15496](https://github.com/HabitRPG/habitica/issues/15496)** — Dailies 重複設定的 bug（2025-08 開的）
2. **[#14461](https://github.com/HabitRPG/habitica/issues/14461)** — 另一個待修的 issue
3. **[#11516](https://github.com/HabitRPG/habitica/issues/11516)** — 較舊但仍 open 的 issue

第一個 [#15496](https://github.com/HabitRPG/habitica/issues/15496) 看起來是個 bug：
> 當創建一個設定為每月重複「某週的某天」的 daily 時，如果開始日期在這天之前，完成後會顯示為隔天到期。

這是個邏輯 bug，可能需要看 `website/client` 裡面處理 Dailies 的 Vue component。

---

# 專案結構

Clone 下來後：

```bash
git clone https://github.com/HabitRPG/habitica.git
cd habitica
```

資料夾結構：
```
habitica/
├── website/
│   ├── client/       # Vue 前端 (這是我們主要看的地方)
│   └── server/       # Node.js 後端
├── migrations/       # MongoDB migrations
├── test/             # 測試
├── config/           # 設定檔
└── package.json
```

前後端分開，結構清楚。Vue 的 code 在 `website/client/`。

---

# 下一步

1. 照著 [Setting up Habitica Locally](https://github.com/HabitRPG/habitica/wiki/Setting-Up-Habitica-for-Local-Development) 設定環境
2. 把 app 跑起來，到處點點看
3. 挑一個 [Help Wanted issue](https://github.com/HabitRPG/habitica/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22) 來研究

---

**相關連結：**
- [Habitica GitHub](https://github.com/HabitRPG/habitica)
- [Help Wanted Issues](https://github.com/HabitRPG/habitica/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22)
- [Guidance for Blacksmiths (貢獻者指南)](https://habitica.fandom.com/wiki/Guidance_for_Blacksmiths)
- [本地開發設定](https://github.com/HabitRPG/habitica/wiki/Setting-Up-Habitica-for-Local-Development)

---

*這是 Zero to PR 系列的第一篇。下一篇會記錄本地環境設定的過程。*
