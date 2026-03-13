
<script lang="ts">
  import { createEventDispatcher } from 'svelte';

  export let isDarkMode: boolean;
  const dispatch = createEventDispatcher();

  let playerCount = 2;
  let gridSize = 4;
  let playerColors = ['#ff0000', '#0000ff', '#00ff00', '#ffff00'];

  function goBack() {
    dispatch('back');
  }

  function startGame() {
    dispatch('startGame', {
      gridSize,
      playerCount,
      playerColors
    });
  }
</script>

<div class="setup-container">
  <header>
    <button class="home-button" on:click={goBack}>Home</button>
  </header>

  <div class="content-column">
    <h1>Setup</h1>
    <main class="setup-content">
      <div class="slider-container">
        <label for="player-count">Players: {playerCount}</label>
        <input type="range" id="player-count" min="2" max="4" bind:value={playerCount}>
      </div>

      {#each Array(playerCount) as _, i}
        <div class="player-color-container">
          <label for="player-color-{i}">Player {i + 1}</label>
          <input type="color" id="player-color-{i}" bind:value={playerColors[i]}>
        </div>
      {/each}

      <div class="slider-container">
        <label for="grid-size">Grid Size: {gridSize}x{gridSize}</label>
        <input type="range" id="grid-size" min="4" max="12" bind:value={gridSize}>
      </div>
      <button class="start-game" on:click={startGame}>Start Game</button>
    </main>
  </div>
</div>

<style>
  .setup-container {
    display: flex;
    flex-direction: column;
    color: var(--text-color);
  }

  header {
    padding: 20px;
    flex-shrink: 0;
  }

  .home-button {
    background: none;
    border: 1px solid var(--secondary-button-border-color);
    color: var(--text-color);
    padding: 10px 20px;
    cursor: pointer;
  }

  .content-column {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  h1 {
    margin: 40px 0;
    text-align: center;
    font-size: 3em;
  }

  .setup-content {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
  }

  .slider-container {
    margin: 20px 0;
    width: 80%;
    max-width: 400px;
  }

  label {
    display: block;
    margin-bottom: 10px;
  }

  input[type="range"] {
    width: 100%;
  }

  .player-color-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 80%;
    max-width: 400px;
    margin: 10px 0;
  }

  input[type="color"] {
    width: 40px;
    height: 40px;
    border: none;
    background: none;
    cursor: pointer;
  }

  .start-game {
    background-color: var(--primary-button-color);
    color: var(--primary-button-text-color);
    padding: 15px 30px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 1.2em;
    margin-top: 20px;
  }
</style>
