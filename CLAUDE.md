# CLAUDE.md — 給 Claude 的項目說明

## 項目概覽

單檔案中國象棋遊戲。所有邏輯、樣式、繪圖均內嵌於 `index.html`（`chess.html` 為同一份的備份）。

## 架構

```
index.html
├── <style>          視覺樣式（CSS）
├── <body>           HTML 結構（棋盤容器、控制列）
└── <script>
    ├── 常數與狀態     CELL, ROWS, COLS, board, turn, history ...
    ├── 棋盤初始化     initBoard()
    ├── 走法規則       getMoves(), getLegalMoves(), isMoveSafe()
    ├── 將軍／將死     isInCheck(), isCheckmate()
    ├── AI 引擎        evaluate(), minimax(), getBestMove(), triggerAI()
    ├── 繪圖           draw(), drawBoard(), drawPiece(), drawHighlights()
    └── 輸入處理       handleClick(), doMove(), undoMove()
```

## 重要設計決策

### 棋盤座標
- `board[row][col]`，row 0 = 頂部（黑方主場），row 9 = 底部（紅方主場）
- `RED = 'r'`，`BLACK = 'b'`
- 棋子物件：`{ type: 'K'|'A'|'B'|'N'|'R'|'C'|'P', side: 'r'|'b' }`

### 棋子類型對照
| type | 紅方 | 黑方 |
|------|------|------|
| K | 帥 | 將 |
| A | 仕 | 士 |
| B | 相 | 象 |
| N | 傌 | 馬 |
| R | 俥 | 車 |
| C | 炮 | 砲 |
| P | 兵 | 卒 |

### AI 引擎
- **演算法**：Minimax + Alpha-Beta Pruning
- **難度對應深度**：簡單=2、普通=3、困難=4、地獄=5
- **評估函數**：`evaluate(brd)` — 正值利黑，負值利紅
  - 子力值：`PIECE_BASE`（K=9999, R=600, C=285, N=270, B=120, A=110, P=30）
  - 位置獎勵：`POS_TABLE` 各 10×9 矩陣，黑方視角，紅方翻轉 row
- **走法排序**：吃子走法優先排列，提升 alpha-beta 剪枝效率
- **AI 觸發**：`doMove()` 後若輪到黑方且 `aiMode=true`，用 `setTimeout(30ms)` 非同步觸發，避免 UI 凍結

### Canvas 繪圖
- 畫布尺寸：`W = CELL*(COLS-1) + CELL*2`，`H = CELL*(ROWS-1) + CELL*2`
- 棋盤原點偏移：`OX = OY = CELL`（留邊距放框線）
- 棋子用 `drawPiece(x, y, side, char)` 繪製：原木色 + 正弦木紋線 + 兩條刻環 + 啞光反光
- 棋盤：米白底 + 紅線格 + 宮格斜線 + 楚河漢界（左側鏡像）

### 視覺配色（仿傳統實體棋）
- 棋盤：`#f8f4ec` 底，`#c41208` 紅線
- 棋子底色：`#dcc890`（淺原木色），正弦木紋
- 紅方字色：`#b81008`，黑方字色：`#185020`（深綠）

### 悔棋邏輯
- `history` 陣列存 `{ board, turn, aiLastMove }` 快照
- 人機模式下 `undoMove()` 一次彈出 **2 個**快照（撤回玩家步 + AI 回應步）

## 修改時注意

- **新增難度**：在 HTML 加按鈕、`setDiff()` 的 id map 加入新 depth 值即可
- **調整 AI 強度**：修改 `POS_TABLE` 各棋子的位置分數，或調整 `PIECE_BASE` 子力值
- **修改走法規則**：`getMoves()` 內各 case，注意馬腿、象眼、炮跳吃的特殊處理
- **繪圖效能**：木紋用確定性正弦函數，不用 `Math.random()`，避免每幀閃爍

## 部署

GitHub Pages，從 `main` branch 根目錄的 `index.html` 自動部署。

```bash
# 修改後推上去即自動更新
git add -A && git commit -m "描述" && git push
```
