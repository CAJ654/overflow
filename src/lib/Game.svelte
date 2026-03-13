<script lang="ts">
  import { createEventDispatcher } from "svelte";

  export let gridSize: number;
  export let playerCount: number;
  export let playerColors: string[];
  export let isDarkMode: boolean;

  const dispatch = createEventDispatcher();

  let currentPlayer = 0;

  const goHome = () => {
    dispatch("goHome");
  };

  const endTurn = () => {
    currentPlayer = (currentPlayer + 1) % playerCount;
  };

  const fullGridSize = gridSize + 4;
</script>

<div class="game-page-container">
  <div class="player-indicators-container">
    {#if playerCount > 0}
      <div class="player-indicator player-1" class:active={currentPlayer === 0} style="background-color: {playerColors[0]};"></div>
    {/if}
    {#if playerCount > 1}
      <div class="player-indicator player-2" class:active={currentPlayer === 1} style="background-color: {playerColors[1]};"></div>
    {/if}
    {#if playerCount > 2}
      <div class="player-indicator player-3" class:active={currentPlayer === 2} style="background-color: {playerColors[2]};"></div>
    {/if}
    {#if playerCount > 3}
      <div class="player-indicator player-4" class:active={currentPlayer === 3} style="background-color: {playerColors[3]};"></div>
    {/if}
  </div>

  <div class="game-and-controls">
    <div class="game-area">
      <div class="grid" style="grid-template-columns: repeat({fullGridSize}, 1fr);">
        {#each Array(fullGridSize * fullGridSize) as _, i}
          {@const row = Math.floor(i / fullGridSize)}
          {@const col = i % fullGridSize}
          {@const isGameCell = row >= 2 && row < fullGridSize - 2 && col >= 2 && col < fullGridSize - 2}
          {@const isBlackBorder = !isGameCell && (row >= 1 && row < fullGridSize - 1 && col >= 1 && col < fullGridSize - 1)}
          <div class="cell {isGameCell ? 'game-cell' : isBlackBorder ? 'black-border-cell' : 'outer-border-cell'}" />
        {/each}
      </div>
    </div>
    <button class="end-turn-button" on:click={endTurn}>End Turn</button>
  </div>

  <button class="home-button" on:click={goHome}>Home</button>
</div>

<style>
  .game-page-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    position: relative;
    color: var(--text-color);
  }

  .player-indicators-container {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
  }

  .player-indicator {
    position: absolute;
    width: 100px;
    height: 100px;
    border-radius: 50%;
    transition: transform 0.3s ease-in-out;
  }

  .player-indicator.active {
    transform: scale(1.2);
    z-index: 1;
  }

  .player-1 {
    top: -50px;
    left: -50px;
  }

  .player-2 {
    top: -50px;
    right: -50px;
  }

  .player-3 {
    bottom: -50px;
    left: -50px;
  }

  .player-4 {
    bottom: -50px;
    right: -50px;
  }

  .game-and-controls {
    display: flex;
    align-items: center;
    gap: 20px;
  }

  .game-area {
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 20px 0;
    overflow: auto;
  }

  .grid {
    display: grid;
    gap: 5px;
    background-color: var(--grid-background-color);
  }

  .cell {
    width: 40px;
    height: 40px;
  }

  .game-cell {
    background-color: var(--game-cell-color);
    border: 1px solid var(--game-cell-border-color);
  }

  .black-border-cell {
    background-color: var(--black-border-cell-color);
  }

  .outer-border-cell {
    background-color: transparent;
  }

  .home-button, .end-turn-button {
    background-color: var(--primary-button-color);
    color: var(--primary-button-text-color);
    padding: 10px 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 1em;
    z-index: 1;
    margin-top: 20px;
  }

  .end-turn-button {
    margin-top: 0;
  }
</style>
