---
title: 'Habitica #05 — 提交 PR：驗證修復並送出'
description: '用瀏覽器 DevTools 重現 Bug、驗證修復、提交第一個 Pull Request'
pubDate: 'Feb 08 2026'
heroImage: '../../assets/blog-placeholder-5.jpg'
---

上一篇我們寫好了修復程式碼。這篇來做最後也是最重要的一步——**驗證修復確實有效，然後提交 PR**。

## 驗證策略

提交 PR 不是貼 code 就好。好的 PR 需要：

1. **Before 截圖** — 證明 bug 確實存在
2. **After 截圖** — 證明修復有效
3. **Desktop 截圖** — 證明沒有破壞原有功能

所以我們的計畫是：

1. 先還原到修復前的程式碼，截 bug 的圖
2. 套用修復，截修復後的圖
3. 兩張對比，一目了然

## Step 1：還原到修復前

因為我們用的是 Vite dev server，改檔案會自動 hot-reload，非常方便：

```bash
# 保存修復後的版本
cp website/client/src/components/shops/shopItem.vue shopItem-fixed.vue

# 還原到修復前（有 bug 的版本）
git checkout HEAD~1 -- website/client/src/components/shops/shopItem.vue
```

幾秒後 Vite HMR 自動更新頁面，現在跑的是原始（有 bug 的）程式碼。

## Step 2：重現 Bug

打開瀏覽器，進入本地 Habitica 商店 `http://localhost:5173/shops/market`。

### Desktop 測試

先確認桌面版是正常的。打開 Chrome DevTools（F12），確保視窗寬度大於 768px。

把滑鼠 hover 到裝備上：

![Desktop popover 正常](/images/habitica-05/desktop-popover.png)

✅ Popover 正常顯示裝備屬性（Rime Reaper Suit, CON +13.5）。滑鼠移開就消失。這是預期行為。

### Mobile 測試

現在按 **Ctrl+Shift+M** 開啟 DevTools 的 Responsive Mode，切換到手機尺寸（375 x 812）。

重新整理頁面，然後 hover（在 responsive mode 裡模擬 touch）到一個裝備上：

![Mobile bug — popover 覆蓋 UI](/images/habitica-05/mobile-bug.png)

🐛 **Bug 重現了！** Popover 浮窗（Leather Armor 屬性表）直接蓋住了 "Market" 標題、"Equipment" 區域，而且在觸控模擬下**這個 popover 不會自動消失**。

這就是 issue #10917 描述的問題：`b-popover` 的 `triggers="hover focus"` 在觸控設備上被 touch 事件觸發，但沒有對應的消失機制。使用者被 popover 擋住，無法正常操作商店。

## Step 3：套用修復並驗證

```bash
# 換回修復後的程式碼
cp shopItem-fixed.vue website/client/src/components/shops/shopItem.vue
```

Vite HMR 再次自動更新。同樣在 375px 手機模式下，hover 裝備：

![Mobile fixed — 沒有 popover](/images/habitica-05/mobile-fixed.png)

✅ **沒有 popover 了！** UI 乾淨，使用者可以直接點擊裝備進入購買流程。

### 三種狀態對比

| | Desktop (>768px) | Mobile BUG (<768px) | Mobile FIX (<768px) |
|---|---|---|---|
| Hover 裝備 | ✅ 顯示 popover | 🐛 Popover 蓋住 UI | ✅ 不顯示 popover |
| 點擊購買 | ✅ 正常 | ❌ 被擋住 | ✅ 直接開啟購買 |

![三種狀態對比圖](/images/habitica-05/comparison.png)

## Step 4：Fork & Push

```bash
# 加入你的 fork 作為 remote
git remote add fork git@github.com:YOUR_USERNAME/habitica.git

# Push 修復分支
git push fork fix/10917-mobile-shop-popover
```

## Step 5：簽署 Commit

GitHub commit 旁邊會顯示 "Unverified" 灰標。要變成綠色的 **Verified** ✅，需要簽署 commit。

用 SSH key 最簡單：

```bash
# 1. 設定 SSH 簽名
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true

# 2. 去 GitHub Settings → SSH and GPG keys → New SSH key
#    Key type 選 "Signing Key"，貼上你的公鑰

# 3. 重新簽署 commit
git commit --amend --no-edit -S

# 4. Force push
git push fork fix/10917-mobile-shop-popover --force
```

## Step 6：建立 Pull Request

到 GitHub 建立 PR。Habitica 有自己的 [PR template](https://github.com/HabitRPG/habitica/blob/develop/.github/PULL_REQUEST_TEMPLATE.md)，照著填：

```markdown
Fixes #10917

### Changes

On mobile browsers (<768px), the shop item popover (triggered by
`hover focus` in Bootstrap-Vue `b-popover`) activates on touch and
covers the item card, preventing users from purchasing.

This fix adds a mobile check to conditionally disable the popover:

- `isMobile` data property — checks window.innerWidth < 768
- `shouldShowPopover` computed — returns showPopover && !isMobile
- `handleResize` method + resize listener for device rotation

Desktop popover behavior is unchanged. On mobile, users tap items
to open the purchase modal directly.

File changed: `website/client/src/components/shops/shopItem.vue` (+10 lines)

----
UUID: <your-habitica-user-id>
```

**重點：**
- `Fixes #10917` 會在 PR merge 時自動關閉 issue
- 附上 Before/After 截圖，讓 reviewer 一目了然
- UUID 是你的 Habitica 帳號 ID（Settings → API）

## 🎉 PR 已提交

**[PR #15606](https://github.com/HabitRPG/habitica/pull/15606)** — 我們的第一個開源 PR！

回顧一下整個 Zero to PR 旅程：

| 篇章 | 做了什麼 |
|------|---------|
| #01 First Look | 選定專案，了解 Habitica |
| #02 Local Setup | Docker 建起本地開發環境 |
| #03 Reading Codebase | 學會閱讀大型 Vue + Node 專案 |
| #04 First Fix | 定位 bug、設計方案、寫修復程式碼 |
| **#05 Submitting PR** | **驗證修復、截圖、提交 PR** |

從「我想貢獻開源但不知道怎麼開始」到「PR 已提交等 review」，這就是完整的流程。

---

**PR 連結：** [habitica/pull/15606](https://github.com/HabitRPG/habitica/pull/15606)
**Issue 連結：** [habitica/issues/10917](https://github.com/HabitRPG/habitica/issues/10917)
**修改檔案：** `website/client/src/components/shops/shopItem.vue` (+10 lines)
