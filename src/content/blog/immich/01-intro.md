---
title: '為什麼選 Immich 當開源貢獻的目標'
description: '想找一個專案來貢獻開源，挑了 Immich'
pubDate: 'Feb 08 2026'
heroImage: '../../../assets/blog-placeholder-1.jpg'
tags: ['immich', 'svelte', 'nestjs', 'open-source', 'beginner']
---

想找一個專案來貢獻開源，挑了 Immich。

## Immich 是什麼

[Immich](https://github.com/immich-app/immich) 是一個自架的照片管理系統，可以想成自己架的 Google Photos。完全開源，跑在自己的伺服器上。

幾個數據:
- GitHub 91.9k stars
- 9,000+ commits，非常活躍
- 每天都有新的 PR 被合併
- 有 Discord 社群，#contributing 頻道可以問問題

功能面，支援照片影片備份、AI 人臉辨識、智慧搜尋、地圖檢視。

## 技術棧

- 前端: Svelte
- 後端: NestJS (Node.js)
- 資料庫: PostgreSQL
- Mobile: Flutter

Svelte 跟 Vue 概念類似但語法不同，NestJS 有明確的 module/controller/service 分層。對想學新框架的人來說蠻適合的。

## 新手友善嗎

這是我在挑專案時最在意的，看了幾個指標:

- good first issue: 12 個開放中，184 個已解決。已解決數量很重要，代表 maintainer 真的有在 review 新手的 PR
- PR 回應速度: 從提交到合併大概 1-3 天，有些當天就合了
- 有完整的 CONTRIBUTING.md 跟開發環境 setup 文件

## 怎麼挑 Issue

一開始只看 good first issue 標籤，但發現蠻多坑的:
- 有些已經有人在做了，只是沒被 assign
- 有些留言裡有人說要做，但幾個月都沒提 PR
- 有些問題描述不夠清楚，maintainer 也沒定調

所以不能只看標題跟標籤，要把所有留言讀完，還要檢查有沒有 linked PR。做法就是把 issue 底下的留言滑完，再看右邊 sidebar 的 "Development" 區塊有沒有連結的 PR。

最後跳出 good first issue 範圍，在 bug 標籤裡找到 #25414: 照片詳細頁的三點選單溢出視窗。0 留言、0 PR、問題清楚有截圖、scope 明確。

下一篇會講怎麼把開發環境跑起來，然後重現這個 bug。

## 重點整理

- 挑 issue 不能只看標題，要讀完所有留言和檢查 linked PR
- good first issue 以外的 bug 標籤也值得看
- 確認沒人在做再認領
