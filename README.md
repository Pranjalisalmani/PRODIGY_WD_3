# PRODIGY_WD_3
I have build a tic-tac-toe web application using HTML,CSS and Java.
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Tic‑Tac‑Toe — Vanilla HTML/CSS/JS</title>
  <style>
    :root{
      --bg:#0f1724; --card:#0b1220; --accent:#7c3aed; --muted:#94a3b8; --win:#16a34a;
      --cell-size:100px;
      color: #e6eef8; font-family: Inter, system-ui, -apple-system, Roboto, 'Segoe UI', sans-serif;
    }
    *{box-sizing:border-box}
    body{margin:0; min-height:100vh; display:flex; align-items:center; justify-content:center; background:linear-gradient(180deg,#071029,#071827);}
    .app{width:min(720px,96vw); padding:20px; background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); border-radius:16px; box-shadow:0 8px 30px rgba(2,6,23,.6);}
    header{display:flex; gap:16px; align-items:center; justify-content:space-between; margin-bottom:14px}
    h1{font-size:20px; margin:0}
    .controls{display:flex; gap:8px; align-items:center}
    select, button{background:transparent; border:1px solid rgba(255,255,255,0.08); color:var(--muted); padding:6px 10px; border-radius:8px}
    button.primary{background:var(--accent); color:white; border:0}
    .board-wrap{display:grid; grid-template-columns:1fr 260px; gap:18px}
    .board{display:grid; grid-template-columns:repeat(3, var(--cell-size)); grid-template-rows:repeat(3, var(--cell-size)); gap:8px; justify-content:center;}
    .cell{width:var(--cell-size); height:var(--cell-size); background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); display:flex; align-items:center; justify-content:center; font-size:48px; font-weight:700; color:#e6eef8; border-radius:10px; cursor:pointer; user-select:none; transition:transform .08s ease, background .12s;}
    .cell:hover{transform:translateY(-4px)}
    .muted{color:var(--muted); font-size:14px}
    .side{background:linear-gradient(180deg, rgba(255,255,255,0.015), rgba(255,255,255,0.01)); padding:12px; border-radius:12px}
    .status{margin-bottom:10px}
    .score{display:flex; gap:10px; margin-top:8px}
    .score .item{background:rgba(0,0,0,0.25); padding:8px 10px; border-radius:8px; text-align:center}
    .highlight{background:linear-gradient(90deg, rgba(124,58,237,0.12), rgba(16,185,129,0.06)); border:1px solid rgba(124,58,237,0.12)}
    .winner{border:2px solid var(--win)}
    footer{margin-top:14px; display:flex; justify-content:space-between; align-items:center; color:var(--muted); font-size:13px}
    @media (max-width:700px){
      .board-wrap{grid-template-columns:1fr;}
      .side{order:2}
    }
  </style>
</head>
<body>
  <main class="app">
    <header>
      <h1>Tic‑Tac‑Toe</h1>
      <div class="controls">
        <label class="muted">Mode</label>
        <select id="modeSelect" aria-label="Game mode">
          <option value="pvp">2 Players (Local)</option>
          <option value="pvc">Play vs Computer</option>
        </select>
        <label class="muted">Difficulty</label>
        <select id="diffSelect">
          <option value="easy">Easy</option>
          <option value="hard" selected>Hard (Perfect)</option>
        </select>
        <button id="restartBtn" class="primary">Restart</button>
      </div>
    </header>

    <section class="board-wrap">
      <div>
        <div class="status">
          <div id="statusText">Turn: <strong id="turnIndicator">X</strong></div>
          <div class="muted" id="subStatus">Make a move</div>
        </div>
        <div class="board" id="board" role="grid" aria-label="tic tac toe board">
          <!-- 9 cells inserted by JS -->
        </div>
      </div>

      <aside class="side">
        <div style="display:flex; justify-content:space-between; align-items:center;">
          <div>
            <div class="muted">You are</div>
            <div><strong id="playerSymbol">X</strong></div>
          </div>
          <div>
            <div class="muted">First player</div>
            <div><select id="firstSelect"><option value="X">X</option><option value="O">O</option></select></div>
          </div>
        </div>

        <div class="score">
          <div class="item">X: <span id="scoreX">0</span></div>
          <div class="item">O: <span id="scoreO">0</span></div>
          <div class="item">Draws: <span id="scoreD">0</span></div>
        </div>

        <div style="margin-top:12px">
          <div class="muted">Tips</div>
          <ul style="margin:8px 0 0 16px; padding:0;">
            <li>Center is strongest opening.</li>
            <li>Block opponent's two-in-a-row.</li>
            <li>Use Hard difficulty for perfect play.</li>
          </ul>
        </div>
      </aside>
    </section>

    <footer>
      <div class="muted">Built with HTML • CSS • JS</div>
      <div class="muted">Click a cell to play</div>
    </footer>
  </main>

  <script>
    // Game state
    const boardEl = document.getElementById('board');
    const statusText = document.getElementById('statusText');
    const subStatus = document.getElementById('subStatus');
    const turnIndicator = document.getElementById('turnIndicator');
    const modeSelect = document.getElementById('modeSelect');
    const diffSelect = document.getElementById('diffSelect');
    const restartBtn = document.getElementById('restartBtn');
    const playerSymbolEl = document.getElementById('playerSymbol');
    const firstSelect = document.getElementById('firstSelect');

    let board = Array(9).fill(null); // null, 'X', 'O'
    let current = 'X';
    let gameActive = true;
    let scores = {X:0, O:0, D:0};

    // Win combinations
    const wins = [
      [0,1,2],[3,4,5],[6,7,8],
      [0,3,6],[1,4,7],[2,5,8],
      [0,4,8],[2,4,6]
    ];

    function createBoard(){
      boardEl.innerHTML = '';
      for(let i=0;i<9;i++){
        const cell = document.createElement('div');
        cell.className = 'cell';
        cell.dataset.index = i;
        cell.setAttribute('role','button');
        cell.setAttribute('aria-label', 'cell ' + (i+1));
        cell.addEventListener('click', onCellClick);
        boardEl.appendChild(cell);
      }
    }

    function onCellClick(e){
      const idx = Number(e.currentTarget.dataset.index);
      if(!gameActive || board[idx]) return;

      makeMove(idx, current);

      if(modeSelect.value === 'pvc' && gameActive){
        // If player symbol is not current, AI plays as soon as it's its turn
        if(diffSelect.value === 'easy'){
          setTimeout(aiEasyMove, 250);
        } else {
          setTimeout(aiMove, 200);
        }
      }
    }

    function makeMove(idx, player){
      if(board[idx] || !gameActive) return;
      board[idx] = player;
      render();
      const result = checkGame();
      if(result){
        endGame(result);
        return;
      }
      current = current === 'X' ? 'O' : 'X';
      updateStatus();

      // If PvC and it's AI's turn and game still active
      if(modeSelect.value === 'pvc' && gameActive && current !== playerSymbolEl.textContent){
        // AI will act from onCellClick or here depending on flow; handled above
      }
    }

    function render(){
      for(let i=0;i<9;i++){
        const cell = boardEl.children[i];
        cell.textContent = board[i] || '';
        cell.classList.remove('winner');
      }
    }

    function updateStatus(){
      turnIndicator.textContent = current;
      subStatus.textContent = (modeSelect.value === 'pvc' ? (current === playerSymbolEl.textContent ? 'Your turn' : 'Computer thinking...') : `Player ${current} to move`);
    }

    function checkGame(){
      // returns {winner:'X'/'O'} or {draw:true} or null
      for(const combo of wins){
        const [a,b,c] = combo;
        if(board[a] && board[a] === board[b] && board[a] === board[c]){
          return {winner: board[a], combo};
        }
      }
      if(board.every(Boolean)) return {draw:true};
      return null;
    }

    function endGame(result){
      gameActive = false;
      if(result.draw){
        subStatus.textContent = 'Draw!';
        scores.D++;
        document.getElementById('scoreD').textContent = scores.D;
      } else {
        subStatus.textContent = `Winner: ${result.winner}`;
        scores[result.winner]++;
        document.getElementById('score' + result.winner).textContent = scores[result.winner];
        // highlight winning cells
        for(const i of result.combo){
          boardEl.children[i].classList.add('winner');
        }
      }
    }

    // AI easy: random available cell
    function aiEasyMove(){
      const avail = board.map((v,i)=>v?null:i).filter(v=>v!==null);
      if(avail.length===0) return;
      const idx = avail[Math.floor(Math.random()*avail.length)];
      makeMove(idx, current);
    }

    // AI hard: minimax
    function aiMove(){
      const ai = current; // AI plays as 'current'
      const human = ai === 'X' ? 'O' : 'X';
      const best = minimax(board.slice(), ai, ai, human);
      if(best.index !== undefined){
        makeMove(best.index, current);
      }
    }

    function minimax(bd, player, aiPlayer, human){
      const game = checkStatic(bd);
      if(game){
        if(game.winner === aiPlayer) return {score: 10};
        if(game.winner === human) return {score: -10};
        if(game.draw) return {score: 0};
      }

      const moves = [];
      for(let i=0;i<9;i++){
        if(!bd[i]){
          const move = {};
          move.index = i;
          bd[i] = player;

          const result = minimax(bd, player === 'X' ? 'O' : 'X', aiPlayer, human);
          move.score = result.score;
          bd[i] = null;
          moves.push(move);
        }
      }

      // choose best
      let bestMove;
      if(player === aiPlayer){
        let bestScore = -Infinity; for(const m of moves){ if(m.score > bestScore){ bestScore = m.score; bestMove = m; } }
      } else {
        let bestScore = Infinity; for(const m of moves){ if(m.score < bestScore){ bestScore = m.score; bestMove = m; } }
      }
      return bestMove || {score:0};
    }

    function checkStatic(bd){
      for(const combo of wins){
        const [a,b,c] = combo;
        if(bd[a] && bd[a] === bd[b] && bd[a] === bd[c]){
          return {winner: bd[a]};
        }
      }
      if(bd.every(Boolean)) return {draw:true};
      return null;
    }

    // UI hooks
    restartBtn.addEventListener('click', ()=>{
      resetGame();
    });

    modeSelect.addEventListener('change', ()=>{
      // Show/hide difficulty when PvC
      diffSelect.style.display = modeSelect.value === 'pvc' ? 'inline-block' : 'none';
      // If switching to PvP, set player symbol to X
      if(modeSelect.value === 'pvp') playerSymbolEl.textContent = 'X';
      resetGame();
    });

    diffSelect.addEventListener('change', ()=>{
      resetGame();
    });

    firstSelect.addEventListener('change', ()=>{
      resetGame();
    });

    function resetGame(){
      board = Array(9).fill(null);
      gameActive = true;
      // starting player depends on firstSelect
      current = firstSelect.value || 'X';
      // which symbol the human plays when PvC
      if(modeSelect.value === 'pvc'){
        playerSymbolEl.textContent = 'X'; // let human be X by default for clarity
      }
      render();
      updateStatus();
      subStatus.textContent = 'Make a move';

      // If PvC and computer goes first
      if(modeSelect.value === 'pvc' && current !== playerSymbolEl.textContent){
        if(diffSelect.value === 'easy') setTimeout(aiEasyMove, 250); else setTimeout(aiMove, 200);
      }
    }

    // initialize
    createBoard();
    resetGame();

    // Keyboard accessibility (1-9 keys)
    window.addEventListener('keydown', (e)=>{
      if(!/^[1-9]$/.test(e.key)) return;
      const idx = Number(e.key)-1;
      if(boardEl.children[idx]) boardEl.children[idx].click();
    });

  </script>
</body>
</html>
