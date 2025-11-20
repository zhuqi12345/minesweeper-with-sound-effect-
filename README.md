# minesweeper-with-sound-effect-
// Just put the code in index.html and open it with browser.
//只需复制代码到 index.html 然后使用浏览器打开即可

<!doctype html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>扫雷</title>
  <style>
    :root {
      --cell-size: 30px;
      --bg: #c0c0c0;
      --border-light: #fff;
      --border-dark: #7b7b7b;
      --border-mid: #b0b0b0;
    }

    * { box-sizing: border-box; user-select: none; }

    body {
      margin: 0;
      background: #808080;
      color: #000;
      font-family: "Segoe UI", Helvetica, Arial, sans-serif;
      display: flex;
      justify-content: center;
      align-items: flex-start;
      min-height: 100vh;
      padding: 12px;
    }

    .frame {
      background: var(--bg);
      border: 2px solid var(--border-dark);
      box-shadow: inset 1px 1px 0 var(--border-light), inset -1px -1px 0 var(--border-mid);
      padding: 10px;
      width: 100%;
      max-width: 95%;
      min-width: 320px;
    }

    h1 {
      margin: 0 0 10px;
      font-size: 20px;
      text-align: center;
    }

    .top-bar {
      display: flex;
      gap: 10px;
      align-items: center;
      flex-wrap: wrap;
      margin-bottom: 8px;
    }

    .top-bar label {
      font-size: 13px;
      display: flex;
      align-items: center;
      gap: 4px;
    }

    input[type="number"] {
      width: 70px;
      padding: 4px;
      border: 1px solid var(--border-dark);
      background: #f4f4f4;
    }

    button {
      padding: 6px 12px;
      background: #e0e0e0;
      border: 2px solid var(--border-dark);
      box-shadow: inset 1px 1px 0 var(--border-light), inset -1px -1px 0 var(--border-mid);
      cursor: pointer;
      font-weight: 600;
    }

    button:active {
      box-shadow: inset -1px -1px 0 var(--border-light), inset 1px 1px 0 var(--border-mid);
    }

    .status-bar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 6px 10px;
      margin: 0 0 8px;
      background: #e6e6e6;
      border: 2px solid var(--border-dark);
      box-shadow: inset 1px 1px 0 var(--border-light), inset -1px -1px 0 var(--border-mid);
      font-weight: 700;
      font-size: 16px;
    }

    .status-bar span {
      min-width: 110px;
      display: inline-block;
    }

    .board-wrapper {
      width: 100%;
      overflow: auto;
      display: flex;
      justify-content: center;
    }

    #board {
      display: grid;
      gap: 1px;
      background: var(--border-dark);
      border: 2px solid var(--border-dark);
      width: max-content;
      max-width: 100%;
      margin: 0 auto;
      touch-action: manipulation;
    }

    .cell {
      width: var(--cell-size);
      height: var(--cell-size);
      background: var(--bg);
      border: 2px solid var(--border-dark);
      box-shadow: inset 1px 1px 0 var(--border-light), inset -1px -1px 0 var(--border-mid);
      display: flex;
      justify-content: center;
      align-items: center;
      font-weight: 700;
      font-size: calc(var(--cell-size) * 0.55);
      cursor: pointer;
    }

    .cell.revealed {
      background: #dcdcdc;
      box-shadow: inset 1px 1px 0 #dcdcdc, inset -1px -1px 0 #dcdcdc;
      border: 1px solid #a0a0a0;
      cursor: default;
    }

    .cell.mine {
      background: #f44;
    }

    .cell.flag::after {
      content: "🚩";
      font-size: calc(var(--cell-size) * 0.7);
      line-height: 1;
    }

    .cell.number-1 { color: #0000ff; }
    .cell.number-2 { color: #008200; }
    .cell.number-3 { color: #ff0000; }
    .cell.number-4 { color: #000084; }
    .cell.number-5 { color: #840000; }
    .cell.number-6 { color: #008284; }
    .cell.number-7 { color: #840084; }
    .cell.number-8 { color: #707070; }

    .footer {
      margin-top: 10px;
      font-size: 13px;
      text-align: center;
      color: #222;
    }
  </style>
</head>
<body>
  <div class="frame">
    <h1>扫雷 (Minesweeper)</h1>

    <div class="top-bar">
      <label>行数: <input id="rows" type="number" min="5" max="40" value="9"></label>
      <label>列数: <input id="cols" type="number" min="5" max="40" value="9"></label>
      <label>地雷数: <input id="mines" type="number" min="1" value="10"></label>
      <button id="newGame">开始新游戏</button>
    </div>

    <div class="status-bar">
      <span id="mineCounter">雷数: 0</span>
      <span id="message">准备开始</span>
      <span id="timer">00:00</span>
    </div>

    <div class="board-wrapper">
      <div id="board"></div>
    </div>

    <div class="footer">左键翻开方格 · 右键插旗 · 第一击必中</div>
  </div>

  <script>
    const boardEl = document.getElementById("board");
    const rowsInput = document.getElementById("rows");
    const colsInput = document.getElementById("cols");
    const minesInput = document.getElementById("mines");
    const mineCounterEl = document.getElementById("mineCounter");
    const timerEl = document.getElementById("timer");
    const messageEl = document.getElementById("message");
    const newGameBtn = document.getElementById("newGame");
    const audioCtx = window.AudioContext ? new AudioContext() : null;

    const directions = [
      [-1, -1], [-1, 0], [-1, 1],
      [0, -1],           [0, 1],
      [1, -1],  [1, 0],  [1, 1],
    ];

    let grid = [];
    let rows = 9;
    let cols = 9;
    let mineCount = 10;
    let flags = 0;
    let revealed = 0;
    let started = false;
    let gameOver = false;
    let timer = null;
    let seconds = 0;
    let minesPlaced = false;

    function clampMines(r, c, requested) {
      const maxMines = r * c - 1;
      return Math.min(Math.max(1, requested), maxMines);
    }

    function setStatus(text) {
      messageEl.textContent = text;
    }

    function updateMineCounter() {
      mineCounterEl.textContent = `雷数: ${mineCount - flags}`;
    }

    function setTimerDisplay(sec) {
      const m = String(Math.floor(sec / 60)).padStart(2, "0");
      const s = String(sec % 60).padStart(2, "0");
      timerEl.textContent = `${m}:${s}`;
    }

    function startTimer() {
      if (timer) return;
      timer = setInterval(() => {
        seconds += 1;
        setTimerDisplay(seconds);
      }, 1000);
    }

    function stopTimer() {
      clearInterval(timer);
      timer = null;
    }

    function resetState() {
      rows = Number(rowsInput.value) || 9;
      cols = Number(colsInput.value) || 9;
      mineCount = clampMines(rows, cols, Number(minesInput.value) || 10);
      minesInput.value = mineCount;
      flags = 0;
      revealed = 0;
      started = false;
      gameOver = false;
      seconds = 0;
      minesPlaced = false;
      stopTimer();
      setTimerDisplay(0);
      updateMineCounter();
      setStatus("准备开始");
    }

    function buildGrid() {
      grid = Array.from({ length: rows }, () =>
        Array.from({ length: cols }, () => ({
          mine: false,
          adj: 0,
          revealed: false,
          flagged: false,
          el: null,
        }))
      );

      boardEl.innerHTML = "";
      boardEl.style.gridTemplateColumns = `repeat(${cols}, var(--cell-size))`;
      boardEl.style.gridTemplateRows = `repeat(${rows}, var(--cell-size))`;

      for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
          const cell = document.createElement("div");
          cell.className = "cell";
          cell.dataset.row = r;
          cell.dataset.col = c;
          cell.addEventListener("click", onLeftClick);
          cell.addEventListener("contextmenu", onRightClick);
          boardEl.appendChild(cell);
          grid[r][c].el = cell;
        }
      }

      adjustCellSize();
    }

    function adjustCellSize() {
      const wrapperRect = document.querySelector(".board-wrapper").getBoundingClientRect();
      const topBarRect = document.querySelector(".top-bar").getBoundingClientRect();
      const statusRect = document.querySelector(".status-bar").getBoundingClientRect();
      const footerRect = document.querySelector(".footer").getBoundingClientRect();

      const paddingFactor = 0.95; // leave small margins to avoid scrollbars
      const availableWidth = wrapperRect.width * paddingFactor;
      const availableHeight = Math.max(
        100,
        (window.innerHeight - wrapperRect.top - statusRect.height - footerRect.height - 20) * paddingFactor
      );

      const size = Math.max(
        16,
        Math.floor(Math.min(availableWidth / cols, availableHeight / rows))
      );

      document.documentElement.style.setProperty("--cell-size", `${size}px`);
    }

    function placeMines(safeRow, safeCol) {
      let placed = 0;
      while (placed < mineCount) {
        const r = Math.floor(Math.random() * rows);
        const c = Math.floor(Math.random() * cols);
        if ((r === safeRow && c === safeCol) || grid[r][c].mine) continue;
        grid[r][c].mine = true;
        placed++;
      }
      computeAdjacents();
      minesPlaced = true;
    }

    function playSound(kind) {
      if (!audioCtx) return;
      if (audioCtx.state === "suspended") audioCtx.resume();

      const osc = audioCtx.createOscillator();
      const gain = audioCtx.createGain();

      let freq = 560;
      let duration = 0.08;

      switch (kind) {
        case "flag":
          freq = 430;
          break;
        case "win":
          freq = 720;
          duration = 0.3;
          break;
        case "lose":
          freq = 200;
          duration = 0.25;
          break;
        default:
          freq = 560;
      }

      const now = audioCtx.currentTime;
      osc.frequency.setValueAtTime(freq, now);
      gain.gain.setValueAtTime(0.12, now);
      gain.gain.exponentialRampToValueAtTime(0.0001, now + duration);

      osc.connect(gain);
      gain.connect(audioCtx.destination);
      osc.start(now);
      osc.stop(now + duration);
    }

    function computeAdjacents() {
      for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
          if (grid[r][c].mine) {
            grid[r][c].adj = -1;
            continue;
          }
          let count = 0;
          for (const [dr, dc] of directions) {
            const nr = r + dr;
            const nc = c + dc;
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc].mine) {
              count++;
            }
          }
          grid[r][c].adj = count;
        }
      }
    }

    function onLeftClick(e) {
      e.preventDefault();
      if (gameOver) return;
      const { row, col } = getCoords(e.currentTarget);
      const cell = grid[row][col];
      if (cell.flagged || cell.revealed) return;

      if (!started) {
        if (!minesPlaced) placeMines(row, col);
        started = true;
        startTimer();
      }

      revealCell(row, col);
    }

    function onRightClick(e) {
      e.preventDefault();
      if (gameOver) return;
      const { row, col } = getCoords(e.currentTarget);
      toggleFlag(row, col);
    }

    function getCoords(target) {
      return {
        row: Number(target.dataset.row),
        col: Number(target.dataset.col),
      };
    }

    function toggleFlag(r, c) {
      const cell = grid[r][c];
      if (cell.revealed) return;
      cell.flagged = !cell.flagged;
      cell.el.classList.toggle("flag", cell.flagged);
      flags += cell.flagged ? 1 : -1;
      updateMineCounter();
      playSound("flag");
    }

    function revealCell(r, c) {
      const cell = grid[r][c];
      if (cell.revealed || cell.flagged) return;
      cell.revealed = true;
      revealed++;

      const el = cell.el;
      el.classList.add("revealed");

      if (cell.mine) {
        el.classList.add("mine");
        endGame(false);
        return;
      }

      playSound("reveal");

      if (cell.adj > 0) {
        el.textContent = cell.adj;
        el.classList.add(`number-${cell.adj}`);
      } else {
        floodReveal(r, c);
      }

      checkWin();
    }

    function floodReveal(sr, sc) {
      const stack = [[sr, sc]];
      while (stack.length) {
        const [r, c] = stack.pop();
        const cell = grid[r][c];
        if (cell.adj > 0) continue;
        for (const [dr, dc] of directions) {
          const nr = r + dr;
          const nc = c + dc;
          if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;
          const neighbor = grid[nr][nc];
          if (neighbor.revealed || neighbor.flagged) continue;
          neighbor.revealed = true;
          revealed++;
          const el = neighbor.el;
          el.classList.add("revealed");
          if (neighbor.mine) continue;
          if (neighbor.adj > 0) {
            el.textContent = neighbor.adj;
            el.classList.add(`number-${neighbor.adj}`);
          } else {
            stack.push([nr, nc]);
          }
        }
      }
    }

    function checkWin() {
      if (gameOver) return;
      if (revealed === rows * cols - mineCount) {
        endGame(true);
      }
    }

    function revealAllMines() {
      for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
          const cell = grid[r][c];
          if (cell.mine) {
            cell.el.classList.add("revealed", "mine");
            cell.el.textContent = "*";
          }
        }
      }
    }

    function endGame(won) {
      gameOver = true;
      stopTimer();
      if (won) {
        setStatus("胜利！");
        playSound("win");
      } else {
        setStatus("踩雷了！再来一局吧");
        playSound("lose");
        revealAllMines();
      }
    }

    function newGame() {
      resetState();
      buildGrid();
    }

    function handleResize() {
      adjustCellSize();
    }

    newGameBtn.addEventListener("click", newGame);
    window.addEventListener("resize", handleResize);

    mineCounterEl.textContent = `雷数: ${mineCount}`;
    setTimerDisplay(0);
    buildGrid();
  </script>
</body>
</html>
