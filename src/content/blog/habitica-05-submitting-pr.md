---
title: 'Habitica #05 — 提交 PR：從翻車到成功的真實記錄'
description: '重現 Bug、截圖驗證、提交 PR 的完整過程——包括所有翻車的部分'
pubDate: 'Feb 08 2026'
heroImage: '../../assets/blog-placeholder-5.jpg'
---

上一篇我們寫好了修復程式碼。但「寫好 code」和「提交 PR」之間，還有一段意想不到的艱辛旅程。

這篇文章記錄了**真實的過程**，包括所有失敗和翻車——因為這才是開源貢獻的真相。

## 目標：精準重現 Bug 並截圖

PR 不是貼個 code diff 就好。好的 PR 需要：
- **Before/After 截圖**證明 bug 存在，且修復有效
- **清楚的描述**讓 reviewer 快速理解

聽起來簡單？我花了**超過兩小時**才搞定截圖。

## 🚧 翻車 #1：帳號卡在新手教學

我用 Puppeteer（無頭瀏覽器自動化工具）來截圖。第一次嘗試：

```javascript
// 登入 → 進商店 → 截圖
await page.goto('http://localhost:5173/shops/market');
```

截出來的圖：

![卡在角色創建頁面](/images/habitica-05/wrong-avatar-page.png)

**這是角色創建頁面，不是商店！** 😅

原因：測試帳號是在 habitica.com 創的，但我們跑的是本地 Docker 環境——本地資料庫根本沒有這個帳號。登入後被導到新手教學。

**教訓：本地開發環境和線上環境的資料庫是獨立的。**

### 解法：用 API 直接建本地帳號

```bash
curl -X POST http://localhost:3000/api/v3/user/auth/local/register \
  -H 'Content-Type: application/json' \
  -d '{
    "username": "bugtester2",
    "email": "bugtester2@test.com",
    "password": "Test1234!",
    "confirmPassword": "Test1234!"
  }'

# 順便加金幣，讓商店有東西可以買
curl -X PUT http://localhost:3000/api/v3/user \
  -H "x-api-user: <USER_ID>" \
  -H "x-api-key: <API_TOKEN>" \
  -d '{"stats.gp": 1000}'
```

## 🚧 翻車 #2：Puppeteer 登入選擇器壞掉

```javascript
await page.$('input[type="email"]');  // null!
```

Habitica 的帳號輸入框 type 是 `text` 不是 `email`，placeholder 是 "Username or Email (case-sensitive)"。選擇器配對失敗。

**教訓：永遠別假設 HTML 結構，先用 DevTools 或 screenshot 確認。**

## 🚧 翻車 #3：Mobile 截圖看不到 Bug

解決登入問題後，我切換到手機視窗大小 (375px)，hover 商品... 沒有 popover 出現。

![Mobile 沒有 popover](/images/habitica-05/mobile-no-popover.png)

**因為我們的修復已經生效了！** 😂 我忘了自己已經在修復後的程式碼上測試。

然後我試了 `touchscreen.tap()`，結果直接打開了購買對話框：

![購買對話框，不是 bug](/images/habitica-05/modal-not-bug.png)

這是正常的購買流程，不是 bug 的樣子。

**教訓：要截「bug 狀態」的圖，得先還原到 bug 存在的程式碼。**

## 🚧 翻車 #4：嘗試動態修改 Vue 組件

我想偷懶，不重建 client，直接在瀏覽器 console 裡改 Vue component 的 computed property：

```javascript
el.__vue__.$options.computed.shouldShowPopover = function() {
  return this.showPopover; // 模擬 bug：移除 isMobile 檢查
};
el.__vue__.$forceUpdate();
```

結果 popover 還是 0。**Vue 2 的 computed property 是在初始化時建立的 getter，不能這樣動態替換。**

**教訓：不要試圖 hack 框架的內部機制，走正路比較快。**

## ✅ 最終解法：Vite HMR 大法

既然是 Vite dev server，直接改原始碼，HMR（Hot Module Replacement）會自動更新！

```bash
# 1. 還原到原始（有 bug 的）程式碼
git show HEAD~1:website/client/src/components/shops/shopItem.vue > shopItem.vue

# 2. Vite 自動 hot-reload，幾秒後頁面更新

# 3. 截圖：Mobile hover → popover 出現 = Bug confirmed!

# 4. 換回修復後的程式碼
cp shopItem-fixed.vue shopItem.vue

# 5. 再截圖：Mobile hover → 沒有 popover = Fix works!
```

**終於成功了！**

## 📸 最終成果

### Desktop — Popover 正常顯示（這是預期行為）

![Desktop popover](/images/habitica-05/desktop-popover.png)

### 🐛 Mobile BUG — Popover 覆蓋 UI

![Mobile bug](/images/habitica-05/mobile-bug.png)

在手機寬度下，`b-popover` 的 `triggers="hover focus"` 在 touch 時觸發，浮窗直接蓋住 "Market"、"Equipment" 標題和操作區域。在觸控設備上，這個 popover **無法被關閉**。

### ✅ Mobile FIX — 乾淨的 UI

![Mobile fixed](/images/habitica-05/mobile-fixed.png)

修復後，768px 以下不渲染 popover，使用者可以直接點擊物品進入購買流程。

### 對比圖

![三種狀態對比](/images/habitica-05/comparison.png)

## 提交 PR

### Fork & Push

```bash
# 添加 fork remote
git remote add fork git@github.com:YOUR_USERNAME/habitica.git

# Push 修復分支
git push fork fix/10917-mobile-shop-popover
```

### 簽署 Commit（Verified 綠勾）

GitHub 上有個小細節——commit 旁邊會顯示 "Unverified" 灰標。要變成綠色的 "Verified"，需要簽署 commit。

最簡單的方式是用 SSH key：

```bash
# 設定 SSH 簽名
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global gpg.format ssh
git config --global commit.gpgsign true

# 重新簽署 commit
git commit --amend --no-edit -S

# Force push
git push fork fix/10917-mobile-shop-popover --force
```

別忘了去 **GitHub → Settings → SSH and GPG keys** 把你的 SSH key 加為 **Signing Key**。

### PR Template

Habitica 有自己的 PR template，照著填就好：

```markdown
Fixes #10917

### Changes

On mobile browsers (<768px), the shop item popover activates on
touch and covers the item card, preventing users from purchasing.

This fix:
- Adds `isMobile` data property (window.innerWidth < 768)
- Adds `shouldShowPopover` computed (showPopover && !isMobile)
- Adds resize listener for device rotation

File changed: `website/client/src/components/shops/shopItem.vue` (+10 lines)

----
UUID: <your-habitica-user-id>
```

### 最終 PR

🎉 **[PR #15606](https://github.com/HabitRPG/habitica/pull/15606)** 已提交！

## 時間線回顧

| 時間 | 事件 |
|------|------|
| 開始 | 「截個圖應該很快」|
| +20min | 帳號卡在新手教學 🤦 |
| +40min | 登入選擇器壞掉 |
| +60min | 截到的圖是修復後的狀態 |
| +80min | 嘗試 hack Vue computed，失敗 |
| +100min | 用 Vite HMR 終於截到正確的 bug 圖 |
| +110min | PR 提交成功 🎉 |

**「截個圖而已」花了將近兩小時。** 這就是軟體開發的現實。

## 這趟旅程學到的

1. **本地環境 ≠ 線上環境** — 資料庫、帳號、狀態都是獨立的
2. **不要假設 HTML 結構** — 永遠先檢查再寫選擇器
3. **測試要在正確的狀態下進行** — 截 bug 圖要用有 bug 的程式碼
4. **不要試圖走捷徑 hack 框架** — 正路通常更快
5. **Vite HMR 是你的好朋友** — 改檔案自動更新，debug 神器
6. **完美主義有代價** — 但精準的截圖讓 PR 更有說服力

## 下一步

等待 maintainer review。

如果被要求修改，記住：**這是正常流程，不是否定你的能力。** 根據 feedback 調整，push 新 commit，然後繼續等。

開源貢獻不是寫完 code 就結束，而是一段持續溝通的過程。

---

**PR 連結：** [habitica/pull/15606](https://github.com/HabitRPG/habitica/pull/15606)
**Issue 連結：** [habitica/issues/10917](https://github.com/HabitRPG/habitica/issues/10917)
**修改檔案：** `website/client/src/components/shops/shopItem.vue` (+10 lines)
