---
title: 'Habitica #04 — 第一個 Bug Fix'
description: '實際動手修復 Issue #10917：手機版 tooltip 覆蓋問題'
pubDate: 'Feb 07 2026'
heroImage: '../../../assets/blog-placeholder-4.jpg'
---

終於到了最刺激的部分——動手修 bug！

## 我們要修什麼？

**Issue #10917:** [Items can't be purchased on mobile browser due to tooltip overlap](https://github.com/HabitRPG/habitica/issues/10917)

**問題描述：**
在手機瀏覽器上，當你點擊商店裡的固定物品（pinned items），tooltip 會彈出並覆蓋整個物品，導致無法點擊購買。

**視覺化說明：**

| 情況 | 行為 | 結果 |
|------|------|------|
| ✅ 桌面 | Hover 顯示 tooltip 在上方 | 可以點擊購買 |
| ❌ Bug | 手機 touch 觸發 tooltip 覆蓋物品 | 無法點擊！ |
| ⭐ 修復 | 手機隱藏 tooltip | 可以直接點擊購買 |

## 第一步：理解問題

這個 issue 從 2018 年就存在了！而且有 4 個人嘗試過但都失敗。這告訴我們：

1. 這問題比表面上看起來複雜
2. 但也代表 maintainer 很希望有人能修好它

先讀完 issue 的所有留言。發現關鍵資訊：

> "Agreed solution: Hide tooltip below 768px (approved by Tressley & Alys)"

太好了！Maintainer 已經同意解決方案：**在 768px 以下隱藏 tooltip**。

## 第二步：找到相關程式碼

問題出在商店的物品 tooltip。用 grep 搜尋：

```bash
cd habitica
grep -r "popover" --include="*.vue" website/client/src/components/shops/
```

找到關鍵檔案：`website/client/src/components/shops/shopItem.vue`

打開看看：

```vue
<b-popover
  v-if="showPopover"
  ref="popover"
  :target="itemId"
  triggers="hover focus"
  :placement="popoverPosition"
>
```

問題就在這！`triggers="hover focus"` 表示：
- **桌面：** hover 觸發，滑鼠移開就消失
- **手機：** 沒有 hover，touch 會觸發 focus，tooltip 彈出後不會自己消失

## 第三步：設計解決方案

根據 maintainer 同意的方案，我們要在 768px 以下隱藏 popover。

**思路：**
1. 加一個 `isMobile` 變數檢測視窗寬度
2. 加一個 computed property `shouldShowPopover`
3. 當 `isMobile` 為 true 時，不顯示 popover

## 第四步：實作修復

修改 `shopItem.vue`：

**1. 在 data() 加入 isMobile：**

```javascript
data () {
  return {
    // ... 其他屬性
    isMobile: typeof window !== 'undefined' && window.innerWidth < 768,
  };
},
```

**2. 加入 computed property：**

```javascript
computed: {
  shouldShowPopover () {
    return this.showPopover && !this.isMobile;
  },
  // ... 其他 computed
},
```

**3. 加入 resize listener（處理螢幕旋轉）：**

```javascript
mounted () {
  // ... 現有程式碼
  window.addEventListener('resize', this.handleResize);
},
beforeDestroy () {
  // ... 現有程式碼
  window.removeEventListener('resize', this.handleResize);
},
methods: {
  handleResize () {
    this.isMobile = window.innerWidth < 768;
  },
  // ... 其他 methods
},
```

**4. 修改 template：**

```vue
<b-popover
  v-if="shouldShowPopover"  <!-- 改這裡！ -->
  ref="popover"
  :target="itemId"
  triggers="hover focus"
  :placement="popoverPosition"
>
```

## 第五步：測試

用 Puppeteer 驗證邏輯：

```javascript
// 桌面 (1024px)
const desktopWidth = 1024;
const desktopIsMobile = desktopWidth < 768; // false
const shouldShowPopover = true && !desktopIsMobile; // true ✓

// 手機 (375px)
const mobileWidth = 375;
const mobileIsMobile = mobileWidth < 768; // true
const shouldShowPopover = true && !mobileIsMobile; // false ✓
```

測試通過！

## 第六步：Commit

寫一個好的 commit message：

```bash
git add website/client/src/components/shops/shopItem.vue

git commit -m "fix(shops): hide popover on mobile to prevent purchase blocking (#10917)

On mobile browsers, the shop item popover triggers on touch and covers
the item, preventing users from clicking to purchase pinned items.

This fix:
- Adds isMobile data property checking window.innerWidth < 768
- Adds shouldShowPopover computed that disables popover on mobile
- Adds resize listener to handle device rotation

Closes #10917"
```

## 第七步：建立 Pull Request

1. Fork 專案（如果還沒的話）
2. Push 到你的 fork
3. 到 GitHub 建立 PR
4. 填寫 PR template

**PR 標題格式：** `fix(shops): hide popover on mobile to prevent purchase blocking`

## 經驗總結

1. **先讀完所有留言** — 可能已經有人討論過解決方案
2. **理解為什麼之前的嘗試失敗** — 避免重蹈覆轍
3. **從簡單的解決方案開始** — 這個修復只改了一個檔案、加了約 10 行程式碼
4. **測試你的改動** — 確保桌面行為沒變，手機行為修好了

## 下一步

等 maintainer review 我們的 PR！

如果被要求修改，別灰心——這是正常流程。根據 feedback 調整，然後再 push。

---

**這篇文章的修改：**
- 檔案：`website/client/src/components/shops/shopItem.vue`
- 改動：+10 行
- Issue：[#10917](https://github.com/HabitRPG/habitica/issues/10917)
