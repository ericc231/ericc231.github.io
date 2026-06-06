---
title: "用了一輪 Lovable 的真實心得：它的邊界，以及能不能搭 Flutter"
date: 2026-06-06 21:00:00 +0800
categories: ["AI工具", "Lovable"]
tags: ["Lovable", "AI", "Flutter", "Supabase", "FlutterFlow"]
---

這是 Lovable 小系列的最後一篇。[做完那個耳機收藏 App](/posts/lovable-build-headphone-tracker/) 之後，我想誠實聊聊用一輪下來的感受——它好在哪、邊界在哪，以及一個我自己最想搞清楚的問題：**如果我要做的是原生行動 App（例如用 Flutter），Lovable 接得上嗎？**

## 優點：摩擦力真的很低

最直接的感受就是「方便」。從一句話到一個能動、有質感的 App，中間幾乎沒有卡點：

- **生成速度快、結果有水準**：我只說「簡潔深色風格」，它端出來的深色玻璃擬態版面相當耐看，不是廉價範本感。
- **對話式迭代很順**：想改什麼講一句就好，省掉「改碼→重整→看結果」的來回。
- **中文友善**：整段中文需求它完全聽懂，連 UI 文字都直接是中文。
- **不用碰環境**：不裝套件、不開終端機，前端瑣事它全包。

對「想法多、但最怕處理前端」的人來說，這個體驗確實會讓你想多用幾次。

## 缺點與邊界

但用得越深，越要清楚它的邊界：

- **它做的是「網頁 App」**：沒有原生 iOS/Android 輸出，不能直接上架 App Store / Play Store。
- **預設資料存在本機**：像我那個耳機 App，第一版資料是存在瀏覽器（localStorage），換裝置就不見。要真正可用，得再接後端。
- **複雜邏輯仍需要人**：簡單到中等的 App 很順，但邏輯一複雜，你還是得看得懂它寫了什麼、能介入調整，否則容易卡在「它改不到我要的點」。
- **生成有隨機性**：同樣的需求，不同次結果可能略有差異，講得越精確越穩。

## 重點問題：能不能搭 Flutter / 做成原生 App？

這是我自己最在意的，特地查證了一下，結論很明確：

**Lovable 不會輸出 Flutter（Dart）程式碼，也不直接產生原生 App。** 它生成的是 React 網頁。所以沒有「Lovable 一鍵轉 Flutter」這回事。但「用 Lovable 的成果 + Flutter 做原生 App」是做得到的，關鍵在**共用後端**：

### 最乾淨的路線：一個後端，兩個前端

1. 讓 Lovable 負責**網頁版**，並用它幫你建好的 **Supabase 後端**（PostgreSQL 資料庫、登入、儲存）。
2. 到 Supabase 後台 → Project Settings → API，取得 **Project URL** 與 **anon public key**。
3. 在 Flutter 專案裝官方的 **`supabase_flutter`** 套件，用那組 URL + key 初始化。
4. 這樣 Flutter App 就能讀寫**同一個資料庫、同一套登入與儲存**——後端共用，Lovable 顧網頁、Flutter 顧原生，資料一致。
5. 務必在 Supabase 設好 **RLS（Row Level Security）** 權限規則，這是安全關鍵。

至於畫面，**React 的 UI 沒辦法直接搬到 Flutter**（兩套技術不同），但你可以把 Lovable 設計出來的那些有質感的畫面當成**設計稿 / UX 參考**，在 Flutter 裡重刻。對「不知道 App 該長怎樣」的人，這其實是很實用的起手式。

### 另外兩個選項

- **只想快速上架、不堅持原生**：用 **Capacitor** 把 Lovable 的網頁包成 App 殼上架（本質是 WebView，不是真原生）。
- **主要目標就是原生 mobile**：那 **FlutterFlow** 更對口——它直接產出 Flutter 程式碼，也能接 Supabase。Lovable 官方甚至有一篇 FlutterFlow vs Lovable 的比較。

簡單記法：**Lovable 擅長 web，FlutterFlow 擅長原生 mobile，而 Supabase 是能把兩邊接起來的共用後端。**

## 跟其他工具怎麼選

同類工具不少，快速給個我的分法：

- **Lovable / v0 / bolt.new**：自然語言生成網頁 App，比的是生成品質與迭代體驗。Lovable 在「完整度 + 後端整合」上很均衡。
- **FlutterFlow**：要原生行動 App 就看它。
- **Cursor**：偏向「給會寫程式的人」的 AI 編輯器，掌控力最高、但門檻也最高。

## 給「訂了卻沒在用」的人的建議

繞了一圈，回到我自己——一個訂了 Lovable 卻一直冷落它的人。用完這一輪，我的結論是：

**它最大的價值，是把「從想法到能動的原型」這段路縮到十幾分鐘。** 與其把訂閱費當沉沒成本，不如把它當成「點子的快速試做機」：腦中冒出一個小工具、一個一頁式網站、一個想驗證的流程，就丟給它做出來看看。做不下去也沒損失，做得起來就賺到。

如果你最終要的是上架的原生 App，那就把 Lovable 當成**前期原型 + 共用後端（Supabase）**的一環，原生那層交給 Flutter / FlutterFlow。各司其職，反而能把這筆訂閱用得更值得。

至少對我來說，這次之後，它不會再只是帳單上一個被遺忘的名字了。

---

*本系列：[① 認識 Lovable](/posts/lovable-rediscover-overview/) ｜ [② 從零做一個耳機收藏 App](/posts/lovable-build-headphone-tracker/) ｜ ③ 心得與 Flutter 整合（本篇）*
