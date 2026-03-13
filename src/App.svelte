<script lang="ts">
  import Setup from './lib/Setup.svelte';
  import Game from './lib/Game.svelte';

  let screen: 'home' | 'setup' | 'game' = 'home';
  let gridSize: number;
  let playerCount: number;
  let playerColors: string[];
  let isDarkMode = true;

  function toggleMode() {
    isDarkMode = !isDarkMode;
  }

  function enterArena() {
    screen = 'setup';
  }

  function goBack() {
    screen = 'home';
  }

  function startGame(event: CustomEvent<{ gridSize: number; playerCount: number; playerColors: string[] }>) {
    gridSize = event.detail.gridSize;
    playerCount = event.detail.playerCount;
    playerColors = event.detail.playerColors;
    screen = 'game';
  }

  function goHome() {
    screen = 'home';
  }
</script>

<div class="app-container" class:light-mode={!isDarkMode}>
  <button class="mode-toggle" on:click={toggleMode}>
    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
      {#if isDarkMode}
        <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>
      {:else}
        <circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>
      {/if}
    </svg>
  </button>

  <div class="content-wrapper">
    {#if screen === 'home'}
      <div class="main-column">
        <main>
          <div class="main-content">
            <h1>{isDarkMode ? 'Overflow' : 'Underflow'}</h1>
            <p>
              A turn-based game where the goal depends on the mode.
            </p>
            <div class="buttons">
              <button class="primary" on:click={enterArena}>
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-shield"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"></path></svg>
                Enter Arena
              </button>
              <button class="secondary">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-help-circle"><path d="M9.09 9a3 3 0 0 1 5.83 1c0 2-3 3-3 3"></path><line x1="12" y1="17" x2="12.01" y2="17"></line></svg>
                How to Play
              </button>
            </div>
          </div>

          <div class="cards">
            <div class="card">
              <h2>2 Game Modes</h2>
              <p>Overflow - Last One Standing</p>
              <p>Underflow - First One Out</p>
            </div>
            <div class="card">
              <h2>Credits</h2>
              <p>
                Rules made by Gerg - <a href="#">YouTube<span>&rarr;</span></a>
              </p>
              <p>
                Code made by CAJ654 - <a href="#">Github<span>&rarr;</span></a>
              </p>
            </div>
          </div>
        </main>
      </div>
    {:else if screen === 'setup'}
      <Setup on:back={goBack} on:startGame={startGame} {isDarkMode} />
    {:else if screen === 'game'}
      <Game {gridSize} {playerCount} {playerColors} on:goHome={goHome} {isDarkMode} />
    {/if}
  </div>
</div>

<style>
  :global(:root) {
    --background-color: #1a1a1a;
    --text-color: white;
    --card-background-color: #2a2a2a;
    --primary-button-color: #6d28d9;
    --primary-button-text-color: white;
    --secondary-button-border-color: white;
    --anchor-color: #6d28d9;
    --game-cell-color: #333;
    --game-cell-border-color: #444;
    --black-border-cell-color: #000;
    --grid-background-color: #1a1a1a;
  }

  :global(body) {
    margin: 0;
    font-family: sans-serif;
  }

  :global(.light-mode) {
    --background-color: #f0f0f0;
    --text-color: #1a1a1a;
    --card-background-color: #d5d5d5;
    --primary-button-color: #92d726;
    --primary-button-text-color: #1a1a1a;
    --secondary-button-border-color: #1a1a1a;
    --anchor-color: #92d726;
    --game-cell-color: #ccc;
    --game-cell-border-color: #bbb;
    --black-border-cell-color: #fff;
    --grid-background-color: #f0f0f0;
  }

  .app-container {
    width: 100%;
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
    background-color: var(--background-color);
    color: var(--text-color);
    transition: background-color 0.3s, color 0.3s;
  }

  .content-wrapper {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .mode-toggle {
    position: absolute;
    top: 20px;
    right: 20px;
    background: none;
    border: none;
    color: inherit;
    cursor: pointer;
    padding: 10px;
    border-radius: 50%;
    z-index: 100;
  }

  .mode-toggle:hover {
    background-color: rgba(255, 255, 255, 0.1);
  }

  .light-mode .mode-toggle:hover {
    background-color: rgba(0, 0, 0, 0.1);
  }

  .main-column {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
    max-width: 800px;
    padding: 20px;
    box-sizing: border-box;
  }
  main {
    text-align: center;
  }
  .main-content {
    margin-bottom: 40px;
  }
  h1 {
    font-size: 4em;
    margin: 0;
  }
  p {
    font-size: 1.2em;
    margin: 20px auto;
  }
  .buttons {
    display: flex;
    justify-content: center;
    gap: 20px;
  }
  button {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 15px 30px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 1.2em;
  }
  .primary {
    background-color: var(--primary-button-color);
    color: var(--primary-button-text-color);
  }

  .secondary {
    background: none;
    border: 1px solid var(--secondary-button-border-color);
    color: var(--text-color);
  }

  .cards {
    display: flex;
    gap: 20px;
    justify-content: center;
  }
  .card {
    background-color: var(--card-background-color);
    padding: 20px;
    border-radius: 5px;
    width: 300px;
    text-align: left;
  }

  .card h2 {
    margin-top: 0;
  }
  .card a {
    color: var(--anchor-color);
    text-decoration: none;
  }

  .card a span {
    margin-left: 5px;
  }
</style>