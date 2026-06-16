<script lang="ts">
  import { createEventDispatcher } from 'svelte';

  export let gridSize: number;
  export let playerCount: number;
  export let playerColors: string[];
  export let isDarkMode: boolean;

  const dispatch = createEventDispatcher();

  // ---- Types ----
  type Cell = { pc: number; hasBlocker: boolean; blockerOwner: number | null };
  type Phase = 'setup' | 'game' | 'over';
  type SubPhase = 'choose' | 'placing_blocker';
  type PushMove = { type: 'push'; axis: 'row' | 'col'; index: number; direction: string };
  type LastMove = PushMove | { type: 'blocker' } | null;

  // ---- State ----
  let gamePhase: Phase = 'setup';
  let subPhase: SubPhase = 'choose';
  let currentPlayer = 0;
  let winner: number | null = null;
  let lastMove: LastMove = null;
  let grid: (Cell | null)[][] = [];
  let players: { eliminated: boolean; blockerPos: [number, number] | null }[] = [];
  let blocksLeft: number[] = [];
  let log: string[] = [];

  // ---- Init ----
  (function init() {
    const total = gridSize * gridSize;
    const blocksEach = Math.floor(total / playerCount);
    const blackCount = total - blocksEach * playerCount;

    grid = Array.from({ length: gridSize }, () =>
      Array.from({ length: gridSize }, () => null as unknown as Cell)
    );

    const positions = shuffleArr(
      Array.from({ length: total }, (_, i) => [Math.floor(i / gridSize), i % gridSize] as [number, number])
    );
    for (let i = 0; i < blackCount; i++) {
      const [r, c] = positions[i];
      grid[r][c] = { pc: -1, hasBlocker: false, blockerOwner: null };
    }

    players = Array.from({ length: playerCount }, () => ({ eliminated: false, blockerPos: null }));
    blocksLeft = Array(playerCount).fill(blocksEach);

    addLog(`Game started: ${playerCount} players on ${gridSize}×${gridSize}`);
    if (blackCount > 0) addLog(`${blackCount} neutral block${blackCount !== 1 ? 's' : ''} pre-placed`);
    addLog(`Each player places ${blocksEach} blocks`);
  })();

  // ---- Helpers ----
  function shuffleArr<T>(arr: T[]): T[] {
    const a = [...arr];
    for (let i = a.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [a[i], a[j]] = [a[j], a[i]];
    }
    return a;
  }

  function addLog(msg: string) {
    log = [msg, ...log].slice(0, 30);
  }

  function countBlocks(p: number): number {
    let n = 0;
    for (const row of grid) for (const cell of row) if (cell?.pc === p) n++;
    return n;
  }

  function hexDarken(color: string): string {
    let hex = color;
    if (hex.startsWith('#') && hex.length === 4)
      hex = '#' + hex[1] + hex[1] + hex[2] + hex[2] + hex[3] + hex[3];
    if (!hex.startsWith('#') || hex.length !== 7) return color;
    const r = parseInt(hex.slice(1, 3), 16);
    const g = parseInt(hex.slice(3, 5), 16);
    const b = parseInt(hex.slice(5, 7), 16);
    return `rgb(${Math.round(r * 0.68)},${Math.round(g * 0.68)},${Math.round(b * 0.68)})`;
  }

  // ---- Reactive derived ----
  $: cellSize = gridSize <= 4 ? 68 : gridSize <= 6 ? 62 : gridSize <= 8 ? 54 : gridSize <= 10 ? 46 : 38;
  $: btnSize = Math.round(cellSize * 0.58);
  $: hasAnyBlackCell = grid.some(row => row.some(cell => cell !== null && cell.pc === -1));
  $: canUseBlocker = gamePhase === 'game' && hasAnyBlackCell;
  $: currentColor = playerColors[currentPlayer] ?? '#888';

  // ---- Setup phase ----
  function onSetupCellClick(r: number, c: number) {
    if (gamePhase !== 'setup' || grid[r][c] !== null) return;
    if (blocksLeft[currentPlayer] <= 0) return;

    grid[r][c] = { pc: currentPlayer, hasBlocker: false, blockerOwner: null };
    blocksLeft[currentPlayer]--;
    blocksLeft = [...blocksLeft];

    const anyEmpty = grid.some(row => row.some(cell => cell === null));
    if (!anyEmpty) {
      gamePhase = 'game';
      currentPlayer = 0;
      addLog('— Setup complete, game phase begins —');
    } else {
      advanceSetupPlayer();
    }
    grid = grid;
  }

  function advanceSetupPlayer() {
    let next = (currentPlayer + 1) % playerCount;
    for (let i = 0; i < playerCount; i++) {
      if (blocksLeft[next] > 0) break;
      next = (next + 1) % playerCount;
    }
    currentPlayer = next;
  }

  // ---- Game phase ----
  function advanceGamePlayer() {
    let next = (currentPlayer + 1) % playerCount;
    for (let i = 0; i < playerCount; i++) {
      if (!players[next].eliminated) break;
      next = (next + 1) % playerCount;
    }
    currentPlayer = next;
    subPhase = 'choose';
  }

  function canPush(axis: 'row' | 'col', index: number, direction: string): boolean {
    if (gamePhase !== 'game' || subPhase !== 'choose') return false;
    if (axis === 'row') {
      for (let c = 0; c < gridSize; c++) if (grid[index]?.[c]?.hasBlocker) return false;
    } else {
      for (let r = 0; r < gridSize; r++) if (grid[r]?.[index]?.hasBlocker) return false;
    }
    if (lastMove?.type === 'push') {
      const lm = lastMove as PushMove;
      if (lm.axis === axis && lm.index === index) {
        const rev: Record<string, string> = { right: 'left', left: 'right', up: 'down', down: 'up' };
        if (rev[lm.direction] === direction) return false;
      }
    }
    return true;
  }

  function doPush(axis: 'row' | 'col', index: number, direction: string) {
    if (!canPush(axis, index, direction)) return;
    const g = gridSize;
    const newBlack = (): Cell => ({ pc: -1, hasBlocker: false, blockerOwner: null });

    if (axis === 'row') {
      const row = [...grid[index]] as Cell[];
      grid[index] = direction === 'right'
        ? [newBlack(), ...row.slice(0, g - 1)]
        : [...row.slice(1, g), newBlack()];
    } else {
      const col = grid.map(r => ({ ...r[index] })) as Cell[];
      const newCol = direction === 'down'
        ? [newBlack(), ...col.slice(0, g - 1)]
        : [...col.slice(1, g), newBlack()];
      for (let r = 0; r < g; r++) grid[r] = [...grid[r]];
      for (let r = 0; r < g; r++) grid[r][index] = newCol[r];
    }

    const arrow = ({ right: '→', left: '←', up: '↑', down: '↓' } as Record<string, string>)[direction] ?? direction;
    addLog(`P${currentPlayer + 1} pushed ${axis === 'row' ? 'row' : 'col'} ${index + 1} ${arrow}`);

    lastMove = { type: 'push', axis, index, direction };
    syncBlockers();
    checkEliminations();
    if (gamePhase !== 'over') advanceGamePlayer();
    grid = grid;
  }

  function syncBlockers() {
    for (let p = 0; p < playerCount; p++) players[p].blockerPos = null;
    for (let r = 0; r < gridSize; r++)
      for (let c = 0; c < gridSize; c++) {
        const cell = grid[r][c];
        if (cell?.hasBlocker && cell.blockerOwner !== null)
          players[cell.blockerOwner].blockerPos = [r, c];
      }
    players = players;
  }

  function toggleBlockerMode() {
    if (gamePhase !== 'game') return;
    subPhase = subPhase === 'placing_blocker' ? 'choose' : 'placing_blocker';
  }

  function onGameCellClick(r: number, c: number) {
    if (gamePhase !== 'game' || subPhase !== 'placing_blocker') return;
    const cell = grid[r][c];
    if (!cell || cell.pc !== -1) return;
    if (cell.hasBlocker && cell.blockerOwner !== currentPlayer) return;

    const p = currentPlayer;
    const bp = players[p].blockerPos;

    if (bp) {
      grid[bp[0]][bp[1]] = { ...grid[bp[0]][bp[1]]!, hasBlocker: false, blockerOwner: null };
    }

    grid[r][c] = { ...cell, hasBlocker: true, blockerOwner: p };
    players[p].blockerPos = [r, c];

    addLog(`P${p + 1} ${bp ? 'moved' : 'placed'} blocker → (${r + 1},${c + 1}) shields row ${r + 1} & col ${c + 1}`);

    lastMove = { type: 'blocker' };
    subPhase = 'choose';
    checkEliminations();
    if (gamePhase !== 'over') advanceGamePlayer();
    grid = grid;
    players = players;
  }

  function checkEliminations() {
    for (let p = 0; p < playerCount; p++) {
      if (players[p].eliminated) continue;
      if (countBlocks(p) === 0) {
        players[p].eliminated = true;
        const bp = players[p].blockerPos;
        if (bp) {
          grid[bp[0]][bp[1]] = { ...grid[bp[0]][bp[1]]!, hasBlocker: false, blockerOwner: null };
          players[p].blockerPos = null;
        }
        addLog(`Player ${p + 1} eliminated!`);
      }
    }
    const alive = players.filter(pl => !pl.eliminated);
    if (alive.length <= 1) {
      winner = alive.length === 1 ? players.indexOf(alive[0]) : null;
      gamePhase = 'over';
      if (winner !== null && winner >= 0) addLog(`Player ${winner + 1} wins! 🎉`);
    }
    players = players;
  }

  function pushTooltip(axis: 'row' | 'col', index: number, direction: string): string {
    if (gamePhase !== 'game' || subPhase !== 'choose') return '';
    if (axis === 'row') {
      for (let c = 0; c < gridSize; c++)
        if (grid[index]?.[c]?.hasBlocker) return 'Blocked by a blocker token';
    } else {
      for (let r = 0; r < gridSize; r++)
        if (grid[r]?.[index]?.hasBlocker) return 'Blocked by a blocker token';
    }
    if (lastMove?.type === 'push') {
      const lm = lastMove as PushMove;
      if (lm.axis === axis && lm.index === index) {
        const rev: Record<string, string> = { right: 'left', left: 'right', up: 'down', down: 'up' };
        if (rev[lm.direction] === direction) return "No backsies — can't undo last push";
      }
    }
    const arrow = ({ right: '→', left: '←', up: '↑', down: '↓' } as Record<string, string>)[direction] ?? '';
    return `Push ${axis === 'row' ? 'row' : 'col'} ${index + 1} ${arrow}`;
  }

  function goHome() {
    dispatch('goHome');
  }
</script>

<!-- ============================================================
     TEMPLATE
     ============================================================ -->

<div class="game-container" style="--current-color: {currentColor}">

  {#if gamePhase === 'over'}
    <!-- Game over screen -->
    <div class="over-screen">
      {#if winner !== null && winner >= 0}
        <div class="trophy">🏆</div>
        <div class="winner-banner" style="background: {playerColors[winner]}; box-shadow: 0 8px 40px {playerColors[winner]}66;">
          Player {winner + 1} Wins!
        </div>
        <div class="winner-sub" style="color: {playerColors[winner]}">
          {countBlocks(winner)} blocks remaining on the board
        </div>
      {:else}
        <div class="trophy">🤝</div>
        <div class="winner-banner" style="background:#555;">It's a draw!</div>
      {/if}
      <div class="over-log">
        {#each log as entry}
          <div class="over-log-entry">{entry}</div>
        {/each}
      </div>
      <div class="over-buttons">
        <button class="btn-home" on:click={goHome}>← Home</button>
      </div>
    </div>

  {:else}
    <!-- Status bar -->
    <div class="status-bar">
      <div class="turn-info">
        {#if gamePhase === 'setup'}
          <span class="phase-badge setup-badge">SETUP</span>
          <span class="player-label" style="color: {currentColor}">Player {currentPlayer + 1}</span>
          <span class="muted">— place a block ({blocksLeft[currentPlayer]} left)</span>
        {:else}
          <span class="phase-badge game-badge">GAME</span>
          <span class="player-label" style="color: {currentColor}">Player {currentPlayer + 1}</span>
          {#if subPhase === 'placing_blocker'}
            <span class="mode-badge">BLOCKER MODE</span>
          {:else}
            <span class="muted">— choose action</span>
          {/if}
        {/if}
      </div>
      <button class="btn-home-sm" on:click={goHome}>← Home</button>
    </div>

    <!-- Main content area -->
    <div class="main-content">

      <!-- Board with push buttons -->
      <div class="board-scroll">
        <div
          class="board"
          style="
            grid-template-columns: {btnSize}px repeat({gridSize}, {cellSize}px) {btnSize}px;
            grid-template-rows: {btnSize}px repeat({gridSize}, {cellSize}px) {btnSize}px;
          "
        >
          <!-- Top row: push column DOWN -->
          <div class="corner"></div>
          {#each Array(gridSize) as _, c}
            {@const ok = canPush('col', c, 'down')}
            <button
              class="push-btn"
              class:push-disabled={!ok}
              style="width:{cellSize}px;height:{btnSize}px;"
              disabled={!ok}
              title={pushTooltip('col', c, 'down')}
              on:click={() => doPush('col', c, 'down')}
            >▼</button>
          {/each}
          <div class="corner"></div>

          <!-- Grid rows -->
          {#each Array(gridSize) as _, r}
            {@const okRight = canPush('row', r, 'right')}
            <button
              class="push-btn"
              class:push-disabled={!okRight}
              style="width:{btnSize}px;height:{cellSize}px;"
              disabled={!okRight}
              title={pushTooltip('row', r, 'right')}
              on:click={() => doPush('row', r, 'right')}
            >▶</button>

            {#each Array(gridSize) as _, c}
              {@const cell = grid[r][c]}
              {@const isEmpty = cell === null}
              {@const isBlack = cell !== null && cell.pc === -1}
              {@const isPlayer = cell !== null && cell.pc >= 0}
              {@const isBTarget = gamePhase === 'game' && subPhase === 'placing_blocker' && isBlack && (!cell?.hasBlocker || cell?.blockerOwner === currentPlayer)}
              <div
                class="cell"
                class:cell-empty={isEmpty && gamePhase === 'setup'}
                class:cell-black={isBlack}
                class:cell-blocker-target={isBTarget}
                style="
                  width:{cellSize}px;height:{cellSize}px;
                  {isPlayer ? `background:${playerColors[cell!.pc]};border-color:${hexDarken(playerColors[cell!.pc])};` : ''}
                "
                role="button"
                tabindex="0"
                on:click={() => gamePhase === 'setup' ? onSetupCellClick(r, c) : onGameCellClick(r, c)}
                on:keydown={e => { if (e.key === 'Enter' || e.key === ' ') gamePhase === 'setup' ? onSetupCellClick(r, c) : onGameCellClick(r, c); }}
              >
                {#if cell?.hasBlocker}
                  <span class="blocker-icon" title="Blocker token — shields row {r+1} & col {c+1}">🛡</span>
                {/if}
              </div>
            {/each}

            {@const okLeft = canPush('row', r, 'left')}
            <button
              class="push-btn"
              class:push-disabled={!okLeft}
              style="width:{btnSize}px;height:{cellSize}px;"
              disabled={!okLeft}
              title={pushTooltip('row', r, 'left')}
              on:click={() => doPush('row', r, 'left')}
            >◀</button>
          {/each}

          <!-- Bottom row: push column UP -->
          <div class="corner"></div>
          {#each Array(gridSize) as _, c}
            {@const okUp = canPush('col', c, 'up')}
            <button
              class="push-btn"
              class:push-disabled={!okUp}
              style="width:{cellSize}px;height:{btnSize}px;"
              disabled={!okUp}
              title={pushTooltip('col', c, 'up')}
              on:click={() => doPush('col', c, 'up')}
            >▲</button>
          {/each}
          <div class="corner"></div>
        </div>
      </div>

      <!-- Sidebar -->
      <div class="sidebar">

        <!-- Player cards -->
        {#each Array(playerCount) as _, p}
          <div
            class="player-card"
            class:player-active={currentPlayer === p && gamePhase !== 'over'}
            class:player-eliminated={players[p]?.eliminated}
            style="--pc: {playerColors[p]};"
          >
            <div class="player-card-header">
              <div class="player-swatch" style="background:{playerColors[p]};"></div>
              <span class="player-card-name">Player {p + 1}</span>
              {#if players[p]?.eliminated}
                <span class="elim-tag">OUT</span>
              {:else if currentPlayer === p && gamePhase === 'game'}
                <span class="active-tag">▶</span>
              {/if}
            </div>
            {#if !players[p]?.eliminated}
              <div class="player-card-stats">
                {#if gamePhase === 'setup'}
                  <span>{blocksLeft[p]} block{blocksLeft[p] !== 1 ? 's' : ''} to place</span>
                {:else}
                  <span>{countBlocks(p)} block{countBlocks(p) !== 1 ? 's' : ''} on board</span>
                  <span class="blocker-stat">
                    🛡 {players[p].blockerPos
                      ? `at (${players[p].blockerPos![0]+1}, ${players[p].blockerPos![1]+1})`
                      : 'not placed'}
                  </span>
                {/if}
              </div>
            {/if}
          </div>
        {/each}

        <!-- Action area (game phase only) -->
        {#if gamePhase === 'game'}
          <div class="action-area">
            <button
              class="btn-blocker"
              class:btn-blocker-active={subPhase === 'placing_blocker'}
              disabled={!canUseBlocker}
              on:click={toggleBlockerMode}
            >
              {#if subPhase === 'placing_blocker'}
                ✕ Cancel Blocker
              {:else}
                🛡 Place / Move Blocker
              {/if}
            </button>
            {#if subPhase === 'placing_blocker'}
              <div class="help-box">
                Click a <strong>gray (neutral)</strong> cell to place your blocker there.
                It will protect that row &amp; column from being pushed.
              </div>
            {:else}
              <div class="help-box">
                Click the <strong>arrows</strong> around the grid to push a row or column.
                The block on the far edge falls off. Last color standing wins!
              </div>
            {/if}
          </div>
        {:else if gamePhase === 'setup'}
          <div class="help-box">
            <strong style="color: {currentColor}">Player {currentPlayer + 1}</strong>
            — click any empty cell to place your block.
            Gray cells are pre-placed neutral blocks.
          </div>
        {/if}

        <!-- Game log -->
        <div class="log-area">
          {#each log as entry}
            <div class="log-entry">{entry}</div>
          {/each}
        </div>

      </div>
    </div>
  {/if}
</div>

<style>
  /* ---- Layout ---- */
  .game-container {
    width: 100%;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 16px;
    box-sizing: border-box;
    color: var(--text-color);
  }

  .status-bar {
    width: 100%;
    max-width: 1000px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px 16px;
    background: var(--card-background-color);
    border-radius: 10px;
    margin-bottom: 14px;
    flex-wrap: wrap;
    gap: 8px;
  }

  .turn-info {
    display: flex;
    align-items: center;
    gap: 8px;
    flex-wrap: wrap;
    font-size: 1rem;
  }

  .player-label { font-weight: 700; }

  .muted { color: #888; font-size: 0.9rem; }

  .phase-badge {
    font-size: 0.68rem;
    font-weight: 800;
    padding: 3px 8px;
    border-radius: 4px;
    letter-spacing: 0.06em;
  }

  .setup-badge { background: #b45309; color: #fff; }
  .game-badge  { background: #15803d; color: #fff; }

  .mode-badge {
    font-size: 0.68rem;
    font-weight: 800;
    padding: 3px 8px;
    border-radius: 4px;
    background: #6d28d9;
    color: #fff;
    letter-spacing: 0.06em;
  }

  .main-content {
    display: flex;
    gap: 18px;
    align-items: flex-start;
    width: 100%;
    max-width: 1000px;
  }

  /* ---- Board ---- */
  .board-scroll {
    overflow: auto;
    flex: 0 0 auto;
  }

  .board {
    display: grid;
    gap: 3px;
  }

  .corner { /* spacer */ }

  /* ---- Push buttons ---- */
  .push-btn {
    background: transparent;
    border: 1px solid #3a3a3a;
    border-radius: 5px;
    color: #666;
    cursor: pointer;
    font-size: 0.8rem;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: background 0.12s, border-color 0.12s, color 0.12s;
    padding: 0;
  }

  .push-btn:hover:not(:disabled) {
    border-color: var(--primary-button-color);
    color: var(--primary-button-color);
    background: rgba(109, 40, 217, 0.12);
  }

  .push-disabled {
    opacity: 0.25;
    cursor: not-allowed;
  }

  .push-btn:disabled { cursor: not-allowed; }

  /* ---- Cells ---- */
  .cell {
    border-radius: 6px;
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
    border: 2px solid transparent;
    cursor: default;
    box-sizing: border-box;
    transition: box-shadow 0.1s;
  }

  .cell-empty {
    background: #1e1e1e;
    border: 2px dashed #3a3a3a;
    cursor: pointer;
  }

  .cell-empty:hover {
    border-color: var(--current-color);
    background: color-mix(in srgb, var(--current-color) 18%, transparent);
  }

  .cell-black {
    background: #374151;
    border-color: #4b5563;
  }

  .cell-blocker-target {
    cursor: pointer;
  }

  .cell-blocker-target:hover {
    border-color: #9f7aea !important;
    box-shadow: 0 0 10px rgba(159, 122, 234, 0.45);
  }

  .blocker-icon {
    font-size: 1.1em;
    line-height: 1;
    pointer-events: none;
    user-select: none;
    filter: drop-shadow(0 1px 2px rgba(0,0,0,0.6));
  }

  /* ---- Sidebar ---- */
  .sidebar {
    flex: 1;
    min-width: 190px;
    max-width: 250px;
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  /* ---- Player cards ---- */
  .player-card {
    background: var(--card-background-color);
    border: 2px solid #2a2a2a;
    border-radius: 10px;
    padding: 11px 13px;
    transition: border-color 0.2s, box-shadow 0.2s;
  }

  .player-active {
    border-color: var(--pc);
    box-shadow: 0 0 12px color-mix(in srgb, var(--pc) 40%, transparent);
  }

  .player-eliminated { opacity: 0.35; }

  .player-card-header {
    display: flex;
    align-items: center;
    gap: 7px;
    margin-bottom: 5px;
  }

  .player-swatch {
    width: 18px;
    height: 18px;
    border-radius: 4px;
    flex-shrink: 0;
  }

  .player-card-name { font-weight: 700; font-size: 0.88rem; }

  .elim-tag {
    margin-left: auto;
    font-size: 0.65rem;
    font-weight: 800;
    padding: 2px 6px;
    background: #dc2626;
    border-radius: 4px;
    color: #fff;
  }

  .active-tag {
    margin-left: auto;
    font-size: 0.75rem;
    color: var(--pc);
  }

  .player-card-stats {
    font-size: 0.78rem;
    color: #888;
    display: flex;
    flex-direction: column;
    gap: 2px;
  }

  .blocker-stat { font-size: 0.75rem; }

  /* ---- Action area ---- */
  .action-area {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }

  .btn-blocker {
    width: 100%;
    padding: 9px 12px;
    border-radius: 8px;
    border: 2px solid #6d28d9;
    color: #9f7aea;
    background: transparent;
    font-size: 0.85rem;
    font-weight: 700;
    cursor: pointer;
    transition: background 0.15s, border-color 0.15s;
  }

  .btn-blocker:hover:not(:disabled) { background: rgba(109, 40, 217, 0.15); }
  .btn-blocker-active { background: rgba(109, 40, 217, 0.22); border-color: #9f7aea; }
  .btn-blocker:disabled { opacity: 0.35; cursor: not-allowed; }

  .help-box {
    font-size: 0.76rem;
    color: #888;
    line-height: 1.45;
    padding: 9px 10px;
    background: var(--card-background-color);
    border-radius: 7px;
    border: 1px solid #2a2a2a;
  }

  /* ---- Log ---- */
  .log-area {
    background: #111;
    border: 1px solid #222;
    border-radius: 7px;
    padding: 9px 10px;
    max-height: 180px;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    gap: 3px;
  }

  .log-entry {
    font-size: 0.72rem;
    color: #777;
    line-height: 1.35;
  }

  /* ---- Buttons ---- */
  .btn-home, .btn-home-sm {
    background: transparent;
    border: 1px solid #444;
    color: var(--text-color);
    padding: 8px 16px;
    border-radius: 7px;
    cursor: pointer;
    font-size: 0.88rem;
    transition: background 0.15s;
  }

  .btn-home:hover, .btn-home-sm:hover { background: rgba(255,255,255,0.08); }

  /* ---- Game over ---- */
  .over-screen {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 70vh;
    gap: 20px;
    padding: 24px;
    width: 100%;
    max-width: 480px;
  }

  .trophy { font-size: 4rem; line-height: 1; }

  .winner-banner {
    font-size: 2rem;
    font-weight: 900;
    padding: 18px 36px;
    border-radius: 14px;
    color: #fff;
    text-align: center;
  }

  .winner-sub {
    font-size: 1rem;
    font-weight: 600;
    opacity: 0.85;
  }

  .over-log {
    background: var(--card-background-color);
    border-radius: 10px;
    padding: 14px 16px;
    max-height: 200px;
    overflow-y: auto;
    width: 100%;
    display: flex;
    flex-direction: column;
    gap: 4px;
  }

  .over-log-entry { font-size: 0.82rem; color: #aaa; }

  .over-buttons { display: flex; gap: 12px; }

  /* ---- Light mode overrides ---- */
  :global(.light-mode) .log-area       { background: #f0f0f0; border-color: #ddd; }
  :global(.light-mode) .log-entry      { color: #555; }
  :global(.light-mode) .help-box       { color: #555; border-color: #ddd; }
  :global(.light-mode) .player-card-stats { color: #666; }
  :global(.light-mode) .push-btn       { border-color: #ccc; color: #666; }
  :global(.light-mode) .cell-black     { background: #9ca3af; border-color: #6b7280; }
  :global(.light-mode) .cell-empty     { background: #e5e7eb; border-color: #d1d5db; }
  :global(.light-mode) .over-log-entry { color: #444; }
  :global(.light-mode) .player-card    { border-color: #ddd; }
  :global(.light-mode) .muted          { color: #666; }
</style>
