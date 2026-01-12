# Text2Img 功能規格

## 問題修復

### 1. 換行間距問題
**現狀**: 編輯器中的空行在 canvas 渲染時間距不正確
**原因**: 空行 (只有 `\n`) 沒有被正確計算高度
**修復**: 空行應該使用基本大小的行高

## 新功能

### 2. 淺色/深色主題切換

**位置**: 右上角 nav 區域

**UI**:
- 一個切換按鈕，顯示當前主題圖示 (太陽/月亮)
- 點擊切換主題

**CSS 變數**:
```css
/* 深色主題 (預設) */
:root {
  --bg: #0a0a0a;
  --surface: #141414;
  --border: #262626;
  --text: #fafafa;
  --text-muted: #737373;
}

/* 淺色主題 */
:root.light {
  --bg: #ffffff;
  --surface: #f5f5f5;
  --border: #e5e5e5;
  --text: #171717;
  --text-muted: #737373;
}
```

**LocalStorage**: 儲存 `theme` 鍵值 (`light` | `dark`)

---

### 3. 收藏字體功能

**UI**:
- 每個字體卡片標題旁增加星星圖示 (空心/實心)
- 點擊星星切換收藏狀態
- 收藏的字體自動排序到列表最上方

**收藏區顯示**:
- 如果有收藏字體，顯示「我的最愛」分隔標題
- 其他字體顯示「全部字體」分隔標題

**LocalStorage**: 儲存 `favoriteFonts` 為 JSON 陣列 (字體名稱)

**排序邏輯**:
```javascript
const sortedFonts = [
  ...fonts.filter(f => favorites.includes(f.name)),
  ...fonts.filter(f => !favorites.includes(f.name))
];
```

---

## 技術實作

### Agent 1: 修復換行問題
- 修改 `parseEditorContent()` 和 `renderCanvas()` 函數
- 確保空行有正確的行高

### Agent 2: 實作淺色主題
- 增加 CSS 變數切換
- 增加 toggle 按鈕到 nav
- 實作 localStorage 持久化

### Agent 3: 實作收藏字體
- 修改 `initCards()` 函數
- 增加星星按鈕和事件處理
- 實作 localStorage 持久化
- 實作字體排序邏輯
