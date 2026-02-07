---
title: 'Habitica #03 — 在 12K Stars 的專案裡找 Bug'
description: '如何閱讀大型 codebase、追蹤 issue、定位問題程式碼'
pubDate: 'Feb 07 2026'
heroImage: '../../assets/blog-placeholder-3.jpg'
---

環境跑起來了，現在要找一個真正的 bug 來修。

## 選擇 Issue 的策略

打開 [Habitica Issues](https://github.com/HabitRPG/habitica/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22)，有 `help wanted` 標籤的 issue 目前有 12+ 個。

我的篩選條件：
1. **有 `help wanted` 標籤** — 官方確認需要幫忙
2. **沒有人正在做** — 看最新留言確認
3. **跟前端相關** — Vue.js 是我的專長
4. **描述清楚** — 能重現問題

最後選了 [#14461 - Avatar stats don't update after another user uses a buffing ability](https://github.com/HabitRPG/habitica/issues/14461)。

## 理解問題

Issue 描述：
> The web's party interface doesn't show people's updated health/exp/mana **on mouseover**, say after you've used a MP sharing ability (and hit sync), or been away for a while.

翻譯：當隊友對你施放 buff 技能後，你 hover 在頭像上看到的數值不會更新，要刷新頁面才會正確。

這是個典型的 **Vue.js 響應式問題** — 資料更新了，但 UI 沒跟著更新。

## 看留言找線索

往下滑，有人已經做了初步分析：

> I believe the problem was introduced in #14544, because the state is no longer being updated in the `cast` method of bulk spells.
>
> My proposed change is by updating the state after the action `user:castSpell` is called in the `castEnd()` method in `website/client/src/mixins/spells.js`.

太棒了！有人已經指出：
1. 問題可能在 PR #14544 引入
2. 相關檔案：`website/client/src/mixins/spells.js`
3. 問題點：`castEnd()` 方法沒有更新 state

## 專案結構

先搞懂整個前端的結構：

```
website/client/src/
├── app.vue           # 根組件
├── main.js           # 入口
├── components/       # Vue 組件
├── pages/            # 頁面
├── mixins/           # 可重用邏輯 ← 問題在這
├── store/            # Vuex state
├── libs/             # 工具函數
├── assets/           # 靜態資源
└── directives/       # Vue 指令
```

Habitica 用的是傳統 Vue 2 架構：
- **Vuex** 管理全域 state
- **Mixins** 共享元件邏輯
- **Vue Router** 處理路由

## 閱讀問題程式碼

打開 [`spells.js`](https://github.com/HabitRPG/habitica/blob/develop/website/client/src/mixins/spells.js)：

```javascript
// website/client/src/mixins/spells.js
async castEnd (target, type) {
  if (!this.$store.state.spellOptions.castingSpell) return null;
  
  // ... 驗證邏輯 ...
  
  const apiResult = await this.$store.dispatch('user:castSpell', {
    key: spell.key,
    targetId,
    pinType: spell.pinType,
  });

  // 這裡只更新了當前用戶的 state
  if (apiResult.data.data.user) {
    Object.assign(this.$store.state.user.data, apiResult.data.data.user);
  }
  
  // 但是 partyMembers 的 state 沒有更新！
  // 所以 hover 顯示的還是舊資料
  
  // ... 其他邏輯 ...
}
```

問題找到了！

當施放技能後：
1. ✅ API 回傳新資料
2. ✅ 更新了 `user.data`（自己的資料）
3. ❌ 沒有更新 `partyMembers`（隊友的資料）

## 可能的修復方向

看 Vuex store 怎麼管理 party members：

```javascript
// 假設 store 結構是這樣
this.$store.state.partyMembers.data
```

修復思路：

```javascript
// castEnd 裡，API 回傳後加上
if (apiResult.data.data.partyMembers) {
  this.$store.commit('SET_PARTY_MEMBERS', apiResult.data.data.partyMembers);
}
```

或者更簡單：強制重新 fetch party members：

```javascript
await this.$store.dispatch('party:getMembers');
```

## 下一步

現在我知道問題在哪了，但還不能直接開始改。根據 [Guidance for Blacksmiths](https://habitica.fandom.com/wiki/Guidance_for_Blacksmiths)：

> Post a comment on the issue to state that you would like to work on it... **Wait for an admin to say that you can proceed.**

所以下一步是：
1. 在 issue 留言說我想修這個
2. 簡述我的分析和修復方向
3. 等 admin 回覆確認
4. 開始寫 code

## 重點整理

找 bug 的流程：
1. **選對 issue** — 有 help wanted、沒人做、描述清楚
2. **讀完所有留言** — 很可能有人已經分析過
3. **理解專案結構** — 知道程式碼在哪裡
4. **定位問題程式碼** — 跟著線索找到具體的檔案和函數
5. **先問再做** — 在 issue 留言，等確認才開始

下一篇會記錄實際寫 code 和測試的過程。

---

**資料來源：**
- [Issue #14461](https://github.com/HabitRPG/habitica/issues/14461)
- [spells.js 原始碼](https://github.com/HabitRPG/habitica/blob/develop/website/client/src/mixins/spells.js)
- [Guidance for Blacksmiths](https://habitica.fandom.com/wiki/Guidance_for_Blacksmiths)
