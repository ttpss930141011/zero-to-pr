---
title: 'Immich PR 後續: Review、修正、合併'
description: 'PR 提交後被 review，maintainer 直接 push commit 補完不足的部分，最後合併的過程'
pubDate: 'Feb 10 2026'
heroImage: '../../../assets/blog-placeholder-4.jpg'
tags: ['immich', 'svelte', 'code-review', 'open-source']
---

PR 提交了，然後呢？等。

## Review 來了

PR [#26041](https://github.com/immich-app/immich/pull/26041) 提交大約一天後，收到兩個 maintainer 的回應。

第一個是 **jrasm91 (Jason Rasmussen)**，他先推了一個 `fix: linting` 的 commit 到我的 branch 上，修掉 lint 問題，然後直接 approve。

第二個是 **michelheusschen**，他留了一個 comment:

> The overflow is fixed, but you still can't scroll to get to those last items. I think we should add scroll to the context menu once it overflows.

意思是: 你修了溢出，但被截掉的選項還是看不到，應該要能捲動。

## 我漏掉了什麼

回頭看，這個問題其實很明顯。我們用 `max-height` 限制了選單高度，但外層 div 是 `overflow-hidden`，裡面的 `<ul>` 的 overflow 又是靠 `isTransitioned` 這個 state 控制的，動畫結束後才會變成 `overflow-auto`。

問題是: 就算動畫結束後開了 `overflow-auto`，外層的 `overflow-hidden` 還是會擋住捲動。所以選單超出 `max-height` 的部分就直接消失了，使用者根本點不到。

我當時只想到「選單不要跑出畫面」，沒有接著想「超出的內容怎麼辦」。修了 A 問題，製造了 B 問題。

## Maintainer 的修法

Jason 直接在我的 branch 上推了第三個 commit `fix: overflow`，改動很乾淨:

```svelte
<!-- 外層 div: 加上 immich-scrollbar -->
<div class="fixed min-w-50 w-max max-w-75 overflow-hidden rounded-lg shadow-lg z-1 immich-scrollbar">

<!-- ul: 直接 overflow-auto，不再用 isTransitioned 切換 -->
<ul class="flex flex-col ... overflow-auto immich-scrollbar">
```

他做了三件事:

1. 外層 div 加了 `immich-scrollbar`（Immich 自訂的 scrollbar 樣式）
2. `<ul>` 直接寫死 `overflow-auto`，不再透過 `isTransitioned` 動態切換
3. 移除了 `isTransitioned` state 和相關的 `onintroend` / `onoutrostart` 事件

整個邏輯簡化了。不需要追蹤動畫狀態，選單永遠可以捲動。

michelheusschen 看完這個修正後也 approve 了，PR 合併。

## 學到的事

**修 bug 要想完整流程，不是只修眼前的症狀。** 我只驗證了「選單不超出 viewport」，沒有驗證「所有選項都還能操作」。如果當時多花一分鐘，用一個選項很多的情境測試，馬上就會發現底部的選項點不到。

**Maintainer 可以直接 push 到你的 branch。** 這在開源專案蠻常見的，特別是改動小的時候。不需要來回 review comment，他們覺得改幾行就能解決，就直接幫你補上。這不是在否定你的貢獻，是在推進進度。

**讀懂你要改的每一行。** 原本的 `isTransitioned` 邏輯我沒有完全理解為什麼存在，只是避開它，結果留下了問題。如果當時花時間搞懂它在幹嘛，可能就會意識到 overflow 的控制需要一起調整。

PR 最終是三個 commit: 我的修復 + Jason 的兩個補丁。結果是好的，但過程可以更好。
