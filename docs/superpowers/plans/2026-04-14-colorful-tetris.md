# Colorful Tetris Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** ブラウザで直接開けるシングルHTMLファイルのカラフルなテトリスゲームを作る。

**Architecture:** `tetris.html` 1ファイルに HTML・CSS・JavaScript をすべて含める。HTML5 Canvas でゲームボードとピースを描画し、Vanilla JS でゲームロジックを管理する。描画・ロジック・入力処理を関数単位で明確に分離する。

**Tech Stack:** HTML5 Canvas API, Vanilla JavaScript (ES6), CSS3

---

## File Structure

```
tetris.html   ← 唯一のファイル。以下を含む:
  <style>      CSS（背景・スコア表示・フォント）
  <canvas>     ゲームボード描画領域
  <script>
    CONSTANTS   定数（COLS, ROWS, CELL_SIZE, COLORS, SCORES）
    TETROMINOES テトリミノの形状定義（7種 × 4回転）
    Board       ボードの状態管理（2次元配列・ライン消去）
    Renderer    Canvas描画（グリッド・ピース・グラデーション）
    Input       キー入力処理
    Game        ゲームループ・スコア・状態管理
```

---

### Task 1: HTML骨格とCanvas設定

**Files:**
- Create: `tetris.html`

- [ ] **Step 1: HTMLファイルを作成し、基本骨格を書く**

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Colorful Tetris</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      background: #0a0a1a;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      font-family: 'Courier New', monospace;
      color: #fff;
    }
    h1 {
      font-size: 2rem;
      letter-spacing: 0.3em;
      margin-bottom: 12px;
      background: linear-gradient(90deg, #ff6b6b, #ffd93d, #6bcb77, #4d96ff, #c77dff);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      background-clip: text;
    }
    #score-display {
      font-size: 1.1rem;
      margin-bottom: 12px;
      color: #aaa;
      letter-spacing: 0.1em;
    }
    #score-display span {
      color: #ffd93d;
      font-weight: bold;
    }
    canvas {
      border: 2px solid #333;
      background: #111;
      display: block;
    }
    #message {
      margin-top: 14px;
      font-size: 0.9rem;
      color: #666;
      letter-spacing: 0.05em;
    }
  </style>
</head>
<body>
  <h1>TETRIS</h1>
  <div id="score-display">SCORE: <span id="score">0</span></div>
  <canvas id="board"></canvas>
  <div id="message">Press Enter to Start</div>
  <script>
    // ここにゲームコードを追加
  </script>
</body>
</html>
```

- [ ] **Step 2: ブラウザで開いて確認**

`tetris.html` をブラウザで開く。
期待値: タイトル「TETRIS」（レインボーグラデーション）・スコア0・黒いCanvas・「Press Enter to Start」が表示される。

---

### Task 2: 定数・ボードデータ構造

**Files:**
- Modify: `tetris.html` の `<script>` ブロック

- [ ] **Step 1: 定数とボード初期化コードを追加**

`<script>` 内に追加:

```js
// ── 定数 ──────────────────────────────────────────────
const COLS = 10;
const ROWS = 20;
const CELL = 30;

const canvas = document.getElementById('board');
const ctx = canvas.getContext('2d');
canvas.width  = COLS * CELL;  // 300
canvas.height = ROWS * CELL;  // 600

// スコアテーブル（消したライン数 → 得点）
const LINE_SCORES = [0, 100, 300, 500, 800];

// ── ボード状態 ────────────────────────────────────────
// board[row][col] = 0（空）または グラデーション色文字列
function createBoard() {
  return Array.from({ length: ROWS }, () => Array(COLS).fill(0));
}
```

- [ ] **Step 2: ブラウザで開いて確認**

開発者コンソール（F12）で以下を実行:
```js
console.log(createBoard().length, createBoard()[0].length)
```
期待値: `20 10`

---

### Task 3: テトリミノ定義（形状とグラデーション）

**Files:**
- Modify: `tetris.html` の `<script>` ブロック

- [ ] **Step 1: テトリミノの形状と色を定義**

```js
// ── テトリミノ定義 ─────────────────────────────────────
// 各ピースは4×4の行列で4回転分を持つ
// 1=ブロックあり, 0=空
const PIECES = [
  // I ピース
  {
    color: ['#ff6b6b', '#ff8e53'],  // 赤 → オレンジ
    shapes: [
      [[0,0,0,0],[1,1,1,1],[0,0,0,0],[0,0,0,0]],
      [[0,0,1,0],[0,0,1,0],[0,0,1,0],[0,0,1,0]],
      [[0,0,0,0],[0,0,0,0],[1,1,1,1],[0,0,0,0]],
      [[0,1,0,0],[0,1,0,0],[0,1,0,0],[0,1,0,0]],
    ]
  },
  // O ピース
  {
    color: ['#ffd93d', '#6bcb77'],  // 黄 → 黄緑
    shapes: [
      [[0,1,1,0],[0,1,1,0],[0,0,0,0],[0,0,0,0]],
      [[0,1,1,0],[0,1,1,0],[0,0,0,0],[0,0,0,0]],
      [[0,1,1,0],[0,1,1,0],[0,0,0,0],[0,0,0,0]],
      [[0,1,1,0],[0,1,1,0],[0,0,0,0],[0,0,0,0]],
    ]
  },
  // T ピース
  {
    color: ['#6bcb77', '#4dddbf'],  // 緑 → シアン
    shapes: [
      [[0,1,0,0],[1,1,1,0],[0,0,0,0],[0,0,0,0]],
      [[0,1,0,0],[0,1,1,0],[0,1,0,0],[0,0,0,0]],
      [[0,0,0,0],[1,1,1,0],[0,1,0,0],[0,0,0,0]],
      [[0,1,0,0],[1,1,0,0],[0,1,0,0],[0,0,0,0]],
    ]
  },
  // S ピース
  {
    color: ['#4d96ff', '#74b9ff'],  // 水色 → 青
    shapes: [
      [[0,1,1,0],[1,1,0,0],[0,0,0,0],[0,0,0,0]],
      [[0,1,0,0],[0,1,1,0],[0,0,1,0],[0,0,0,0]],
      [[0,0,0,0],[0,1,1,0],[1,1,0,0],[0,0,0,0]],
      [[1,0,0,0],[1,1,0,0],[0,1,0,0],[0,0,0,0]],
    ]
  },
  // Z ピース
  {
    color: ['#7b2fff', '#c77dff'],  // 青紫 → 紫
    shapes: [
      [[1,1,0,0],[0,1,1,0],[0,0,0,0],[0,0,0,0]],
      [[0,0,1,0],[0,1,1,0],[0,1,0,0],[0,0,0,0]],
      [[0,0,0,0],[1,1,0,0],[0,1,1,0],[0,0,0,0]],
      [[0,1,0,0],[1,1,0,0],[1,0,0,0],[0,0,0,0]],
    ]
  },
  // J ピース
  {
    color: ['#c77dff', '#ff6fcf'],  // 紫 → ピンク
    shapes: [
      [[1,0,0,0],[1,1,1,0],[0,0,0,0],[0,0,0,0]],
      [[0,1,1,0],[0,1,0,0],[0,1,0,0],[0,0,0,0]],
      [[0,0,0,0],[1,1,1,0],[0,0,1,0],[0,0,0,0]],
      [[0,1,0,0],[0,1,0,0],[1,1,0,0],[0,0,0,0]],
    ]
  },
  // L ピース
  {
    color: ['#ff6fcf', '#ff6b6b'],  // ピンク → 赤
    shapes: [
      [[0,0,1,0],[1,1,1,0],[0,0,0,0],[0,0,0,0]],
      [[0,1,0,0],[0,1,0,0],[0,1,1,0],[0,0,0,0]],
      [[0,0,0,0],[1,1,1,0],[1,0,0,0],[0,0,0,0]],
      [[1,1,0,0],[0,1,0,0],[0,1,0,0],[0,0,0,0]],
    ]
  },
];
```

- [ ] **Step 2: コンソールで確認**

```js
console.log(PIECES.length)          // → 7
console.log(PIECES[0].shapes.length) // → 4
console.log(PIECES[0].color)        // → ['#ff6b6b', '#ff8e53']
```

---

### Task 4: 描画関数（Renderer）

**Files:**
- Modify: `tetris.html` の `<script>` ブロック

- [ ] **Step 1: セル描画・グリッド・ボード描画関数を追加**

```js
// ── 描画 ──────────────────────────────────────────────
function drawCell(x, y, colors) {
  const grad = ctx.createLinearGradient(
    x * CELL, y * CELL,
    (x + 1) * CELL, (y + 1) * CELL
  );
  grad.addColorStop(0, colors[0]);
  grad.addColorStop(1, colors[1]);
  ctx.fillStyle = grad;
  ctx.fillRect(x * CELL + 1, y * CELL + 1, CELL - 2, CELL - 2);
  // ハイライト（上・左エッジ）
  ctx.fillStyle = 'rgba(255,255,255,0.2)';
  ctx.fillRect(x * CELL + 1, y * CELL + 1, CELL - 2, 3);
  ctx.fillRect(x * CELL + 1, y * CELL + 1, 3, CELL - 2);
}

function drawGrid() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.strokeStyle = '#1e1e2e';
  ctx.lineWidth = 1;
  for (let r = 0; r < ROWS; r++) {
    for (let c = 0; c < COLS; c++) {
      ctx.strokeRect(c * CELL, r * CELL, CELL, CELL);
    }
  }
}

function drawBoard(board) {
  for (let r = 0; r < ROWS; r++) {
    for (let c = 0; c < COLS; c++) {
      if (board[r][c]) {
        drawCell(c, r, board[r][c]);
      }
    }
  }
}

function drawPiece(piece) {
  const shape = PIECES[piece.type].shapes[piece.rot];
  const colors = PIECES[piece.type].color;
  for (let r = 0; r < 4; r++) {
    for (let c = 0; c < 4; c++) {
      if (shape[r][c]) {
        drawCell(piece.x + c, piece.y + r, colors);
      }
    }
  }
}

function render(board, piece) {
  drawGrid();
  drawBoard(board);
  if (piece) drawPiece(piece);
}
```

- [ ] **Step 2: 動作確認用コードをコンソールで実行**

ブラウザコンソールで:
```js
const testBoard = createBoard();
drawGrid();
drawCell(0, 0, ['#ff6b6b', '#ff8e53']);
drawCell(1, 0, ['#ffd93d', '#6bcb77']);
```
期待値: Canvasの左上2マスにグラデーションブロックが表示される。

---

### Task 5: 衝突判定とピース操作

**Files:**
- Modify: `tetris.html` の `<script>` ブロック

- [ ] **Step 1: 衝突判定・移動・回転関数を追加**

```js
// ── ピース操作・衝突判定 ──────────────────────────────
function isValid(board, piece, dx, dy, drot) {
  const rot   = (piece.rot + drot + 4) % 4;
  const shape = PIECES[piece.type].shapes[rot];
  const nx    = piece.x + dx;
  const ny    = piece.y + dy;
  for (let r = 0; r < 4; r++) {
    for (let c = 0; c < 4; c++) {
      if (!shape[r][c]) continue;
      const cx = nx + c;
      const cy = ny + r;
      if (cx < 0 || cx >= COLS) return false;   // 左右壁
      if (cy >= ROWS)           return false;   // 床
      if (cy >= 0 && board[cy][cx]) return false; // 他ブロック
    }
  }
  return true;
}

function spawnPiece() {
  return {
    type: Math.floor(Math.random() * PIECES.length),
    rot:  0,
    x:    3,   // ボード中央付近にスポーン
    y:    0,
  };
}

function lockPiece(board, piece) {
  const shape  = PIECES[piece.type].shapes[piece.rot];
  const colors = PIECES[piece.type].color;
  for (let r = 0; r < 4; r++) {
    for (let c = 0; c < 4; c++) {
      if (shape[r][c]) {
        board[piece.y + r][piece.x + c] = colors;
      }
    }
  }
}
```

- [ ] **Step 2: コンソールで衝突判定テスト**

```js
const board = createBoard();
const piece = { type: 0, rot: 0, x: 3, y: 0 };
// 左端まで動かせるか
console.log(isValid(board, piece, -3, 0, 0)); // → true  (x=0)
console.log(isValid(board, piece, -4, 0, 0)); // → false (x=-1: 壁外)
// 底まで落ちるか（Iピースはrow1に存在するのでROWS-1=19行目に届く）
console.log(isValid(board, piece,  0, 18, 0)); // → true
console.log(isValid(board, piece,  0, 19, 0)); // → false (床抜け)
```

---

### Task 6: ライン消去とスコア

**Files:**
- Modify: `tetris.html` の `<script>` ブロック

- [ ] **Step 1: ライン消去関数を追加**

```js
// ── ライン消去 ────────────────────────────────────────
function clearLines(board) {
  let cleared = 0;
  for (let r = ROWS - 1; r >= 0; r--) {
    if (board[r].every(cell => cell !== 0)) {
      board.splice(r, 1);
      board.unshift(Array(COLS).fill(0));
      cleared++;
      r++; // 同じ行を再チェック（上からの行が落ちてくるため）
    }
  }
  return cleared;
}
```

- [ ] **Step 2: コンソールでテスト**

```js
const board = createBoard();
// 最下行を全部埋める
board[19] = Array(COLS).fill(['#ff6b6b', '#ff8e53']);
const lines = clearLines(board);
console.log(lines);         // → 1
console.log(board[19]);     // → [0, 0, 0, ...] （空行）
console.log(board.length);  // → 20 （行数が保たれている）
```

---

### Task 7: ゲームループと状態管理

**Files:**
- Modify: `tetris.html` の `<script>` ブロック

- [ ] **Step 1: ゲーム状態とループを追加**

```js
// ── ゲーム状態 ────────────────────────────────────────
const state = {
  board:     null,
  piece:     null,
  score:     0,
  running:   false,
  over:      false,
  lastDrop:  0,
  dropInterval: 1000,  // ms
};

function updateScore(lines) {
  state.score += LINE_SCORES[lines] ?? 0;
  document.getElementById('score').textContent = state.score;
}

function setMessage(text) {
  document.getElementById('message').textContent = text;
}

function startGame() {
  state.board        = createBoard();
  state.piece        = spawnPiece();
  state.score        = 0;
  state.running      = true;
  state.over         = false;
  state.lastDrop     = performance.now();
  document.getElementById('score').textContent = '0';
  setMessage('← → ↑ ↓ Space');
  loop(performance.now());
}

function gameOver() {
  state.running = false;
  state.over    = true;
  setMessage('GAME OVER — Press Enter to Restart');
}

function loop(now) {
  if (!state.running) return;

  // 自動落下
  if (now - state.lastDrop >= state.dropInterval) {
    if (isValid(state.board, state.piece, 0, 1, 0)) {
      state.piece.y++;
    } else {
      lockPiece(state.board, state.piece);
      const lines = clearLines(state.board);
      if (lines > 0) updateScore(lines);
      state.piece = spawnPiece();
      // ゲームオーバー判定: 新しいピースが即座に衝突
      if (!isValid(state.board, state.piece, 0, 0, 0)) {
        lockPiece(state.board, state.piece);
        render(state.board, null);
        gameOver();
        return;
      }
    }
    state.lastDrop = now;
  }

  render(state.board, state.piece);
  requestAnimationFrame(loop);
}
```

- [ ] **Step 2: ブラウザで確認（コンソール）**

```js
startGame();
```
期待値: ランダムなテトリミノがゆっくり落下し始める。スコアが 0 のまま表示される。

---

### Task 8: キー入力処理

**Files:**
- Modify: `tetris.html` の `<script>` ブロック

- [ ] **Step 1: キーボードイベントを追加**

```js
// ── 入力処理 ──────────────────────────────────────────
document.addEventListener('keydown', e => {
  if (e.key === 'Enter') {
    if (!state.running) startGame();
    return;
  }
  if (!state.running) return;

  const p = state.piece;
  switch (e.key) {
    case 'ArrowLeft':
      if (isValid(state.board, p, -1, 0, 0)) p.x--;
      break;
    case 'ArrowRight':
      if (isValid(state.board, p,  1, 0, 0)) p.x++;
      break;
    case 'ArrowDown':
      if (isValid(state.board, p,  0, 1, 0)) p.y++;
      else {
        // ソフトドロップで即着地
        lockPiece(state.board, p);
        const lines = clearLines(state.board);
        if (lines > 0) updateScore(lines);
        state.piece = spawnPiece();
        if (!isValid(state.board, state.piece, 0, 0, 0)) {
          lockPiece(state.board, state.piece);
          render(state.board, null);
          gameOver();
          return;
        }
      }
      break;
    case 'ArrowUp':
      if (isValid(state.board, p, 0, 0, 1)) p.rot = (p.rot + 1) % 4;
      break;
    case ' ':
      e.preventDefault();
      // ハードドロップ: 着地できる最下行まで一気に落とす
      while (isValid(state.board, p, 0, 1, 0)) p.y++;
      lockPiece(state.board, p);
      {
        const lines = clearLines(state.board);
        if (lines > 0) updateScore(lines);
        state.piece = spawnPiece();
        if (!isValid(state.board, state.piece, 0, 0, 0)) {
          lockPiece(state.board, state.piece);
          render(state.board, null);
          gameOver();
          return;
        }
      }
      break;
  }
  render(state.board, state.piece);
});
```

- [ ] **Step 2: ゲームをプレイして動作確認**

ブラウザで `tetris.html` を開き Enter で開始。
確認項目:
- [ ] ← → でピースが左右に動く
- [ ] ↑ で時計回りに回転する（壁に当たって回転できない場合は動かない）
- [ ] ↓ で1マス落下する
- [ ] Space でハードドロップ（即座に着地）する
- [ ] ライン消去時にスコアが加算される
- [ ] 積み上がるとゲームオーバーになり「GAME OVER」と表示される
- [ ] Enter でリスタートできる

---

### Task 9: バグチェックと修正

**Files:**
- Modify: `tetris.html`

- [ ] **Step 1: 境界外判定の確認**

以下のシナリオを手動でテスト:
- [ ] Iピースを左端・右端まで移動して壁を抜けないことを確認
- [ ] Iピースを底に落として床を抜けないことを確認
- [ ] ピースを壁際で回転させて壁を抜けないことを確認

- [ ] **Step 2: ライン消去の確認**

以下をコンソールで実行してスコアを確認:
```js
// テスト: 4ライン同時消去 → 800点
startGame();
// ボードの下4行を手動で埋める
for (let r = 16; r <= 19; r++) {
  state.board[r] = Array(COLS).fill(['#ff6b6b', '#ff8e53']);
}
// 次のハードドロップでライン消去が発生するはず
```

- [ ] **Step 3: ゲームオーバー条件の確認**

ピースをすべて上に積み上げてゲームオーバーになることを確認。ゲームオーバー後 Enter でリスタートできることを確認。

---

### Task 10: リファクタリング

**Files:**
- Modify: `tetris.html`

- [ ] **Step 1: マジックナンバーをすべて定数に移動**

`<script>` の先頭の定数ブロックに追加:
```js
const DROP_INTERVAL = 1000;  // ms（既存の state.dropInterval を置き換え）
const SPAWN_X = 3;
const BOARD_BG = '#111';
const GRID_COLOR = '#1e1e2e';
const HIGHLIGHT_ALPHA = 0.2;
```

`state.dropInterval: 1000` を `state.dropInterval: DROP_INTERVAL` に変更。
`x: 3` を `x: SPAWN_X` に変更。
描画関数内の `'#1e1e2e'` を `GRID_COLOR` に変更。

- [ ] **Step 2: ゲームオーバー後の重複ロジックを関数化**

`loop`・`ArrowDown`・`Space` の各キー処理で繰り返されている「lockPiece → clearLines → updateScore → spawnPiece → ゲームオーバー判定」を関数にまとめる:

```js
function placePiece() {
  lockPiece(state.board, state.piece);
  const lines = clearLines(state.board);
  if (lines > 0) updateScore(lines);
  state.piece = spawnPiece();
  if (!isValid(state.board, state.piece, 0, 0, 0)) {
    lockPiece(state.board, state.piece);
    render(state.board, null);
    gameOver();
    return false;  // ゲームオーバー
  }
  return true;
}
```

各箇所の重複コードを `placePiece()` 呼び出しに置き換える。

- [ ] **Step 3: リファクタリング後に全動作を再確認**

- [ ] Enter でゲーム開始
- [ ] ← → ↑ ↓ Space が正常動作
- [ ] ライン消去・スコア計算が正常
- [ ] ゲームオーバー・リスタートが正常

---

## Self-Review

### Spec Coverage Check

| Spec要件 | カバーするタスク |
|---|---|
| シングルHTMLファイル | Task 1 |
| Canvas描画（10×20） | Task 1, 2 |
| 7種テトリミノ | Task 3 |
| レインボーグラデーション | Task 3, 4 |
| ← → ↑ ↓ Space 操作 | Task 8 |
| ライン消去 | Task 6 |
| スコア計算（1=100,2=300,3=500,4=800） | Task 6, 7 |
| ゲームオーバー（上端到達） | Task 7, 8 |
| バグチェック | Task 9 |
| リファクタリング | Task 10 |

全要件カバー済み。

### Placeholder Scan

TBD・TODO・「後で実装」なし。すべてのステップに具体的なコードあり。

### Type Consistency

- `createBoard()` → `board[r][c] = 0 | colors[]` — Task 2, 4, 5, 6 で一貫
- `spawnPiece()` → `{ type, rot, x, y }` — Task 5, 7, 8 で一貫
- `isValid(board, piece, dx, dy, drot)` — Task 5, 7, 8 で同一シグネチャ
- `lockPiece(board, piece)` — Task 5, 7, 8 で一貫
- `clearLines(board)` → `number` — Task 6, 7, 8, 10 で一貫
- `placePiece()` → `boolean` — Task 10 で定義、Task 10 で利用

問題なし。
