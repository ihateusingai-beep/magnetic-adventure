# 磁力探險實驗 — SPEC.md
> v8 · 2026-05-28 · 關卡改為全手動（無自動跳轉）

---

## 1. Project Overview

| Field | Value |
|-------|-------|
| **Name** | 磁力探險實驗（磁力探險） |
| **Type** | 互動科學學習遊戲 / 單一 HTML 檔 |
| **Summary** | 學生拖曳磁鐵、摆放羅盤針，觀察磁場線分佈，學習磁鐵特性（同極相斥、異極相吸、指南針原理） |
| **Target Users** | 匡智張玉瓊晨輝學校（輕度智障小學生），特殊教育情境 |
| **Key Student** | 張鈞保（語音提示對他特別有效） |
| **Output** | `index.html`（完全獨立，無需 server，所有資源內聯） |
| **Deploy** | GitHub Pages：`https://ihateusingai-beep.github.io/magnetic-adventure/` |

---

## 2. Learning Objectives（5 個關卡）

| Level | 課題 | 核心概念 |
|-------|------|---------|
| 1 | 磁鐵吸邊個？ | 磁鐵可以吸鐵製品（鐵釘），但唔吸塑膠 |
| 2 | 認識兩極 | 磁鐵有兩極：紅色 N 極、藍色 S 極 |
| 3 | 同極相斥 | N+N 互相推開、S+S 互相推開 |
| 4 | 異極相吸 | N+S 互相吸引 |
| 5 | 指南針原理 | 地球係大磁鐵，羅盤針永遠指向北方 |

---

## 3. Visual & Rendering Specification

### 3.1 Scene Setup
- **Canvas 2D**，滿屏自適應（devicePixelRatio 處理 Retina）
- **背景**：深色漸變（`#1a1a2e` → `#16213e`），40px 格線
- **實驗枱**：木紋質感下半部（`#8B7355` → `#a08060` → `#6b5030`），30px 木紋線

### 3.2 Color Palette
```
--bg:         #1a1a2e   深色背景
--grid:       #2a2a4a   格線
--table:      #8B7355   木枱
--n-pole:     #e63946   N 極（紅）
--n-pole-dark:#b0202f   N 極陰影
--s-pole:     #457b9d   S 極（藍）
--s-pole-dark:#2d5a7b   S 極陰影
--iron-obj:   #a8dadc   鐵製品（灰藍）
--field-line: #ffd166   磁場線（金色）
--accent:     #06d6a0   UI 高亮（按鈕/標示）
--text:       #f1faee   文字
--panel:      rgba(255,255,255,0.08)   半透明白
```

### 3.3 Typography
- **Font**：Noto Sans TC（Google Fonts CDN），純文字 fallback
- **字體大小**：可調，預設 0 → 最大 +16px（header/bottom-bar 實時響應）
- **SEN 按鈕**：min 48×48px touch target

### 3.4 Visual Elements
| 元素 | 形狀 | 顏色/標示 |
|------|------|---------|
| 磁鐵 | 80×30 圓角長條，兩截不同色 | N 半紅/S 半藍，大字母 N/S |
| 羅盤針 | 針形（三角形頭尾），中心圓黑底 | 紅色 = N 極，藍色 = S 極，圍繞中心旋轉 |
| 鐵釘 | 36×8 圓角條 + 圓頭（nail type） | 灰藍 `#a8dadc`，陰影 `#7bb8ba` |
| 塑膠珠 | 半徑 12 圓 | 彩色（紅/黃/綠/藍/紫） |
| 磁場線 | 貝塞爾曲線 + 箭頭 | 金色半透明 `rgba(255,209,102,0.55)` |

---

## 4. Simulation Specification

### 4.1 磁場計算（磁偶極子模型）
```
B = k × m / r³      （徑向衰減）
k = 5000（調整系數，固定值）
方向：遠離 N 極，進入 S 極
```
- 多磁鐵：向量相加
- Level 5 附加地球磁場：`bx += 800`（恆定向右）

### 4.2 羅盤針旋轉
- 目標角度：`θ = atan2(By, Bx)`
- 平滑過渡：lerp factor `0.12` per frame（`n.angle += diff * 0.12`）
- 針長：固定 22px

### 4.3 鐵製品吸引
- 距離 < 35px：被磁鐵捕獲 `collected = true`（消失）
- 距離 < 120px：受吸引力 `force = (120-d)/120 * 4`
- 動畫：lerp factor `0.08` per frame

### 4.4 磁場線渲染
- 每極 20 條線（20 次 traceLine）
- 積分步長：4px，Max 600 步
- 終止條件：到達 opposite pole / 超出邊界 / 到磁鐵半徑 22px 內
- 渲染：貝塞爾曲線（quadraticCurveTo），末端繪製箭頭

---

## 5. Interaction Specification

### 5.1 拖曳
- **Mouse**：`mousedown` → `mousemove` → `mouseup`
- **Touch**：`touchstart`（`preventDefault`）→ `touchmove` → `touchend`
- 目標：磁鐵（半徑 40×20 hit box）

### 5.2 關卡完成條件（v8 — 全手動）
- **關卡 1**：收集晒所有鐵釘 → 顯示橫額 `showComplete()`，**停 autoNextLevel**，需按「下一關 ▶」
- **關卡 2-4**：studentInteracted + 羅盤針偏轉閾值 → 顯示橫額，停 autoNextLevel，需按「下一關 ▶」
- **關卡 5**：干擾地球磁場 → 顯示橫額，停 autoNextLevel，需按「下一關 ▶」

### 5.3 語音提示（張鈞保專用）
- API：`window.speechSynthesis`
- 語言：`zh-HK`，rate `0.9`
- 時機：
  - 關卡載入後 800ms 朗讀任務
  - Level 5 載入後 500ms 朗讀「地球磁場已經設定好，羅盤針永遠指向北方」
  - 完成關卡時朗讀祝賀語
- 可關閉（UI 右上角 🔊/🔇 按鈕）

### 5.4 字體大小 Slider
- Range input，`min=0, max=16, value=0`
- `--fs` CSS 變數實時更新，影響所有 `calc(XXpx + var(--fs,0px))`

---

## 6. UI Layout

```
┌────────────────────────────────────────────────────────┐
│  🔬 磁力探險       [字體 A───A]  [🔊] [📊] [🔄]        │  ← #header
├────────┬───────────────────────────────────────────────┤
│        │  #level-info (top-left overlay)              │
│ 關卡1  │                                    [木紋枱]  │
│ 關卡2  │         Canvas (磁鐵 + 羅盤針 + 磁場線)      │  ← #canvas-wrap
│ 關卡3  │                                               │
│ 關卡4  │                                    [木紋枱]  │
│ 關卡5  ├───────────────────────────────────────────────┤
│        │           ◀ 上一關    下一關 ▶                │  ← #toolbar
├────────┴───────────────────────────────────────────────┤
│               提示：拖動磁鐵去吸鐵釘！                   │  ← #hint-bar
└────────────────────────────────────────────────────────┘

橫額（完成時）：fixed center，border 3px solid accent，20px padding
版權：absolute bottom-right，font-size 11px，opacity 0.4
```

---

## 7. Level Completion Flow（v8 — Manual Navigation）

```
checkLevelComplete()
    ↓ 滿足條件
showComplete(text)             ← 顯示橫額，朗讀，標記 completed
    ↓ 用戶按「下一關 ▶」
nextLevelBanner()              ← 關閉橫額，studentInteracted=false，initLevel(next)
```

**v8 關鍵改動**：`showComplete()` 不再自動調用 `autoNextLevel()`。舊版（v7）完成後 1500ms 自動跳轉；v8 必須用戶按橫額或 toolbar 按鈕。

---

## 8. File Structure

```
magnetic-adventure/
├── index.html     ← 唯一檔案（SFC），所有 CSS/JS 內聯
├── SPEC.md        ← 本檔
└── .git/          ← git repo
```

---

## 9. Acceptance Criteria

- [ ] 關卡 1 完成條件觸發後，橫額出現，**不再自動跳轉**
- [ ] 按橫額「下一關 ▶」→ 進下一關，studentInteracted reset
- [ ] 按 toolbar「下一關 ▶」→ 直接進下一關（獨立於橫額）
- [ ] 「上一關」按鈕正常運作
- [ ] 羅盤針實時響應磁鐵位置旋轉
- [ ] 磁場線從 N→S 曲線，金色箭頭
- [ ] Level 5 地球磁場（羅盤默認→右）
- [ ] 語音開/關正常
- [ ] 字體大小 slider 實時生效
- [ ] 無 console.error
- [ ] 完全 offline
- [ ] iPad/iPhone touch 拖曳流暢
