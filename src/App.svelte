
<script lang="ts">
  import Setup from './lib/Setup.svelte';

  let showSetup = false;

  function enterArena() {
    showSetup = true;
  }

  function goBack() {
    showSetup = false;
  }
</script>

{#if showSetup}
  <Setup on:back={goBack} />
{:else}
  <div class="main-column">
    <main>
      <div class="main-content">
        <h1>Overflow</h1>
        <p>
          Overflow is a turn-based game focused on making sure you are the last remaining!
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
          <h2>Unique Minions</h2>
          <p>18 specialized units with distinct move and attack patterns.</p>
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
{/if}

<style>
  :global(body) {
    background-color: #1a1a1a;
    color: white;
    font-family: sans-serif;
    margin: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
  }
  .main-column {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
    max-width: 800px; /* Adjust as needed */
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
    margin: 20px auto; /* This will center the paragraph */
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
    background-color: #6d28d9;
    color: white;
  }
  .secondary {
    background: none;
    border: 1px solid white;
    color: white;
  }
  .cards {
    display: flex;
    gap: 20px;
    justify-content: center;
  }
  .card {
    background-color: #2a2a2a;
    padding: 20px;
    border-radius: 5px;
    width: 300px;
    text-align: left;
  }
  .card h2 {
    margin-top: 0;
  }
  .card a {
    color: #6d28d9;
    text-decoration: none;
  }
  .card a span {
    margin-left: 5px;
  }
</style>
