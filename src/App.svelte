<script>
  import { onMount } from "svelte";

  let dictionary = {};
  let baseWords = [];

  let word = "Loading...";
  let foundWords = [];
  let missedWords = [];

  let entry = "";

  let maxTime = 15;
  let timeRemaining = 10;

  let gameState = "waiting";

  onMount(async () => {
    const res = await fetch("/words_dictionary.json");
    const json = await res.json();
    dictionary = json;

    baseWords = Object.keys(dictionary).filter(
      x => x.length >= 5 && x.length <= 7
    );
  });

  const startGame = time => {
    word = baseWords[Math.floor(Math.random() * baseWords.length)];
    maxTime = time;
    timeRemaining = maxTime;
    gameState = "play";
    foundWords = [];
    entry = "";
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
    findMissedWords();
  };

  const findMissedWords = () => {
    let matchWords = Object.keys(dictionary).filter(
      x => x.length < word.length && x.length > 2 && checkLetters(x)
    );

    missedWords = matchWords.filter(x => foundWords.indexOf(x) === -1);
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
      !dictionary[check] ||
      !checkLetters(check)
    ) {
      return false;
    }

    return true;
  };

  const checkLetters = ent => {
    let checkLetters = ent;
    word.split("").forEach(x => (checkLetters = checkLetters.replace(x, "")));
    return checkLetters.length == 0;
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
  .time {
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
  a {
    display: inline-block;
    padding: .2em;
    margin: .3em;
    background-color: #333;
  }
  .missed {
    color: red;
  }
</style>

<main>
  {#if gameState === 'play'}
    <h1>{word}</h1>

    <p>{foundWords.join(', ')}</p>

    <form on:submit|preventDefault={submit}>
      <input use:focus bind:value={entry} />
      <span class="time">{timeRemaining}</span>
    </form>
  {:else if gameState === 'end'}
    <h1>Game Over</h1>
    <h3>{word}</h3>
    <p>
      You found {foundWords.length}/{foundWords.length + missedWords.length}
      words:
      <br />
      <br />
      {#each foundWords as word, ix}
        <a href="https://www.dictionary.com/browse/{word}?s=t" target="_blank">
          {word}
        </a>
      {/each}
      {#each missedWords as word, ix}
        <a
          href="https://www.dictionary.com/browse/{word}?s=t"
          target="_blank"
          class="missed">
          {word}
        </a>
      {/each}
    </p>
  {:else}
    <h1>Anagram Game</h1>
    <p>Click the button to start playing!</p>
  {/if}
  {#if gameState !== 'play'}
    <button on:click={() => startGame(30)}>Easy (30 seconds)</button>
    <button on:click={() => startGame(20)}>Normal (20 seconds)</button>
    <button on:click={() => startGame(10)}>Hard (10 seconds)</button>
  {/if}
</main>
