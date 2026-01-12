# 換行問題深入分析報告

## 問題描述

編輯器輸入：
```
這是我的故事
哈哈
```

Canvas 渲染結果：
```
這是我的故事哈哈
```

換行被忽略了。

---

## 根本原因分析

### ContentEditable 的 HTML 結構

當用戶在 contenteditable 區域按 Enter 時，瀏覽器生成的 HTML 結構是：

```html
這是我的故事<div>哈哈</div>
```

**注意**：第一行是裸露的文字節點，第二行才被包在 `<div>` 裡。

### 現有程式碼的邏輯問題

`parseEditorContent()` 函數的 traverse 邏輯：

```javascript
// 塊級元素結束後加換行（除非已經是換行）
if (isBlock && segments.length > 0 && segments[segments.length - 1].text !== '\n') {
  segments.push({ text: '\n', size, color, bg, bold });
}
```

這只在 `<div>` **結束後**加換行，但沒有在 `<div>` **開始前**加換行。

### 解析流程

| 步驟 | 遇到 | 動作 | segments |
|------|------|------|----------|
| 1 | 文字 "這是我的故事" | 加入 | `["這是我的故事"]` |
| 2 | `<div>` 開始 | 無動作 | `["這是我的故事"]` |
| 3 | 文字 "哈哈" | 加入 | `["這是我的故事", "哈哈"]` |
| 4 | `<div>` 結束 | 加換行 | `["這是我的故事", "哈哈", "\n"]` |

**問題**：步驟 2 沒有加換行，導致兩段文字連在一起。

---

## 修復方案

在進入塊級元素**之前**，如果前面有內容且不是換行，先加一個換行：

```javascript
// 判斷是否為塊級元素
const isBlock = node.tagName === 'DIV' || node.tagName === 'P';

// 塊級元素開始前加換行（如果前面有內容）
if (isBlock && segments.length > 0 && segments[segments.length - 1].text !== '\n') {
  segments.push({ text: '\n', size, color, bg, bold });
}

// Traverse children
for (const child of node.childNodes) {
  traverse(child, { size, color, bg, bold });
}

// 不需要在結束後再加換行了（已在開始前加過）
```

### 修正後的解析流程

| 步驟 | 遇到 | 動作 | segments |
|------|------|------|----------|
| 1 | 文字 "這是我的故事" | 加入 | `["這是我的故事"]` |
| 2 | `<div>` 開始 | **加換行** | `["這是我的故事", "\n"]` |
| 3 | 文字 "哈哈" | 加入 | `["這是我的故事", "\n", "哈哈"]` |

現在換行正確了！

---

## 其他考慮

### 多個 `<div>` 嵌套
如果用戶輸入多行：
```html
這是我的故事<div>哈哈</div><div>第三行</div>
```

需要確保不會產生重複的換行符。

### 空行處理
空的 `<div><br></div>` 也要正確處理為空行。
