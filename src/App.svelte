<script>
  import { onMount } from "svelte";

  let dictionary = {};
  let baseWords = [];

  let word = "Loading...";
  let foundWords = [];

  let entry = "";

  let maxTime = 15;
  let timeRemaining = 10;

  let gameState = "waiting";

  onMount(async () => {
    const res = await fetch("/words_dictionary.json");
    const json = await res.json();
    dictionary = json;

    baseWords = Object.keys(dictionary).filter(
      x => x.length > 5 && x.length < 7
    );
  });

  const startGame = (time) => {
    word = baseWords[Math.floor(Math.random() * baseWords.length)];
    maxTime = time;
    timeRemaining = maxTime;
    gameState = "play";
    foundWords = [];
    entry = '';
    startTimer();
  };

  const startTimer = () => {
    let int = window.setInterval(() => {
      timeRemaining -= 1;
      if (timeRemaining <= 0) {
        window.clearInterval(int);
        endGame();
      }
    }, 1000);
  };

  const endGame = () => {
    gameState = "end";
  };

  const submit = () => {
    if (checkWord(entry)) {
      foundWords = [...foundWords, entry];
    }

    entry = "";
  };

  const checkWord = check => {
    if (
      check.length < 3 ||
      check === word ||
      foundWords.find(x => x === check) != null ||
      !dictionary[check]
    ) {
      return false;
    }

    return true;
  };

  const focus = el => el.focus();
</script>

<style>
  p {
    min-height: 100px;
  }
  input {
    font-size: 1.25em;
    padding: 0.25em;
  }
  span {
    background-image: url("/stopwatch.png");
    background-repeat: no-repeat;
    background-size: contain;
    background-position: center;
    width: 48px;
    height: 48px;
    display: inline-block;
    text-align: center;
    line-height: 54px;
  }
  main {
    width: 600px;
    margin: auto;
    padding-left: 2em;
    text-align: center;
  }
  button {
    font-size: 1.25em;
    display: block;
    margin: 1em auto;
  }
</style>

<main>
  {#if gameState === 'play'}
    <h1>{word}</h1>

    <p>{foundWords.join(', ')}</p>

    <form on:submit|preventDefault={submit}>
      <input use:focus bind:value={entry} />
      <span>{timeRemaining}</span>
    </form>
  {:else if gameState === 'end'}
    <h1>Game Over</h1>
    <p>You found {foundWords.length} words:<br /><br />{foundWords.join(', ')}</p>
  {:else}
    <h1>Anagram Game</h1>
    <p>Click the button to start playing!</p>
  {/if}
  {#if gameState !== 'play'}
    <button on:click={() => startGame(20)}>Easy (20 seconds)</button>
    <button on:click={() => startGame(15)}>Normal (15 seconds)</button>
    <button on:click={() => startGame(10)}>Hard (10 seconds)</button>
  {/if}
</main>
