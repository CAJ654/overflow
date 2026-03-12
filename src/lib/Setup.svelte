
<script lang="ts">
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  let playerCount = 2;
  let gridSize = 4;

  function goBack() {
    dispatch('back');
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
      <div class="slider-container">
        <label for="grid-size">Grid Size: {gridSize}x{gridSize}</label>
        <input type="range" id="grid-size" min="4" max="12" bind:value={gridSize}>
      </div>
      <button class="start-game">Start Game</button>
    </main>
  </div>
</div>

<style>
  .setup-container {
    display: flex;
    flex-direction: column;
    height: 100vh;
    width: 100%;
    color: white;
  }

  header {
    padding: 20px;
    flex-shrink: 0; /* Prevent header from shrinking */
  }

  .home-button {
    background: none;
    border: 1px solid white;
    color: white;
    padding: 10px 20px;
    cursor: pointer;
  }

  .content-column {
    flex-grow: 1; /* Allow content to fill remaining space */
    display: flex;
    flex-direction: column;
    justify-content: center; /* Center content vertically */
    align-items: center; /* Center content horizontally */
    padding-bottom: 80px; /* Add padding to push content up from the bottom */
  }

  h1 {
    margin: 0 0 40px 0;
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

  .start-game {
    background-color: #6d28d9;
    color: white;
    padding: 15px 30px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 1.2em;
    margin-top: 20px;
  }
</style>
