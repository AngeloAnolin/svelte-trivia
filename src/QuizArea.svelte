<div id="main" in:fadeIn out:fadeOut>
  {#if representation.length > 0 && !resultsScreen}
    <span id="heading">
      Question No. { questionNo + 1}
      <i id="category">(Category: { representation[questionNo].category })</i>
    </span>

    <hr />

    <span>
      { representation[questionNo].question }
    </span>

    <div id="difficulty" 
      class="{ checkDifficulty(representation[questionNo].difficulty) }">
      { representation[questionNo].difficulty.toUpperCase() }
    </div>

    { #if representation[questionNo].answerChoices } 
      { #each representation[questionNo].answerChoices as choice }
        { #if representation[questionNo].answered && representation[questionNo].correct}
          { #if choice === representation[questionNo].answerChoice }
            <div id="choice"
              style="background: #7DDF64; color: white; border-color: white">
              <i>{ choice }</i>
            </div>
          { :else }
            <div id="choice">
              <i>{ choice }</i>
            </div>
          { /if }
        { :else if representation[questionNo].answered && !representation[questionNo].correct }
          {#if choice === representation[questionNo].answer}
            <div
              id="choice"
              style="background: #7DDF64; color: white; border-color: white">
              <i>{choice}</i>
            </div>
          {:else}
            <div
              id="choice"
              style="background: #DE3C4B; color: white; border-color: white">
              <i>{choice}</i>
            </div>
          {/if}
        { :else }
          <div id="choice" on:click={ e => handleAnswerChoice(e) }>
            <i>{choice}</i>
          </div>
        { /if }
      {/each} 
    { /if }

    {#if navigationButtonsVisibility}
      <div id="button-bar">
        {#if (questionNo < (numberOfQuestions - 1))}
          <button value="Next" on:click="{() => handleClick('f')}">
            Next &nbsp; &#xbb;
          </button>
        {/if}

        {#if (questionNo > 0)}
          <button value="Back" on:click="{() => handleClick('b')}">
            &#xab; &nbsp; Previous
          </button>
        {/if}

        {#if (questionNo === (numberOfQuestions - 1))}
          <button class="button-show-score" value="Show Score" on:click="{() => showScore()}">
            Show Score
          </button>
        {/if}
      </div>
    {/if}

    {#if snackbarVisibility}
      <div id="snackbar" in:fadeIn out:fadeOut>
        <Snackbar message="{snackbarMessage}"></Snackbar>
      </div>
    {/if}
  
  {:else if resultsScreen}
    <div id="results">
      <p id="score">
        Score: <i>{ score } / { numberOfQuestions }</i>
      </p>
      <p style="font-size: 24px">
        {@html finalMessage}
      </p>
      <p>
        <button class="button-play-again" value="Play Again" on:click="{() => refreshToPlay()}">
          Play Again
        </button>
      </p>
    </div>
  {:else}
    <span
      style="position: absolute; left: 50%; top: 50%; transform:
      translateX(-50%) translateY(-50%); font-weight: bolder; font-size: 36px;
      margin: 0">
      Getting Trivia Questions...
    </span>
  {/if}
</div>

<script>
  import { onMount, createEventDispatcher } from 'svelte';
  import Snackbar from './Snackbar.svelte';

  import { fadeIn, fadeOut } from './transition.js';
  import { htmlDecode, shuffle } from './utils.js';

  const dispatch = createEventDispatcher();
  let data = [];
  
  let numberOfQuestions = 5;
  let questionNo = 0;
  let navigationButtonsVisibility = true;
  let snackbarVisibility = false;
  let snackbarMessage = false;
  let resultsScreen = false;

  $: representation = [];
  $: score = 0;
  $: finalMessage = '';
  $: isQuestionNoEqualToNumberOfQuestions = (questionNo === numberOfQuestions);

  let correct = false;
  
  $: score = 0;

  // Fetch Data
  function fetchData () {
    const url = `https://opentdb.com/api.php?amount=${numberOfQuestions}`
    fetch(url)
      .then(resp => resp.json())
      .then(res => {
        data = res.results

        representation = data.reduce((acc, curr) => {
          acc.push({
            question: htmlDecode(curr.question),
            answerChoices: shuffle(
              [...curr.incorrect_answers, curr.correct_answer]
              .map(ans => htmlDecode(ans))
            ),
            answer: htmlDecode(curr.correct_answer),
            category: htmlDecode(curr.category),
            difficulty: curr.difficulty,
            answerChoice: '',
            correct: false,
            answered: false
          });

          return acc;
        }, []);
      })
      .catch(e => console.error(e));
  }

  onMount( fetchData )

  function handleClick (change) {
    if (snackbarVisibility) snackbarVisibility = !snackbarVisibility;

    if (change === 'f') questionNo += 1;
    else questionNo -= 1;
  }

  function showScore () {
    if (questionNo === (numberOfQuestions - 1)) {
      navigationButtonsVisibility = false;
      resultsScreen = true;

      dispatch('resultsScreen', { showScore: false });

      if (score < 5) {
        finalMessage = 'You need to brush up your skills!';
      } else if (score === 5) {
        finalMessage = "Good try but you can do better!";
      } else {
        finalMessage = "Great job!";
      }
    }
  }

  function handleAnswerChoice (e = {}) {
    if (!representation[questionNo].answered) {
      const representationCopy = {...representation[questionNo]};
      representationCopy.answered = true;

      if (e.target.innerText === representation[questionNo].answer) {
        representationCopy.correct = true;
        representationCopy.answerChoice = representation[questionNo].answer;
        representation[questionNo] = representationCopy;
        score += 1;
        snackbarMessage = true;
        dispatch('score', { score: score });
      } else {
        representationCopy.answerChoice = e.target.innerText;
        representation[questionNo] = representationCopy;
        snackbarMessage = false;
      }

      // TODO: Show Button To View The Score.
      

      if (!snackbarVisibility) snackbarVisibility = true;
    }
  }

  function checkDifficulty (item) {
    // class="{ ( representation[questionNo].difficulty === 'medium' ? 'category-medium' : 'category-hard') }">
    let className = '';
    switch (item) {
      case 'easy':
        className = 'category-easy';
        break;
      case 'medium':
        className = 'category-medium';
        break;
      case 'hard':
        className = 'category-hard';
        break;
      case 'difficult':
        className = 'category-difficult';
        break;
    }

    return className;
  }

  function refreshToPlay () {
    representation = [];
    fetchData();
    questionNo = 0;
    resultsScreen = false;
    navigationButtonsVisibility = true;
  }

</script>

<style>
  #main {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translateX(-50%) translateY(-50%);
    height: calc(100vh - 40%);
    width: calc(100vw - 40%);
    padding: 15px;
    margin: 15px;

    background-color: white;
    border-radius: 6px;
    box-shadow: 0 0 5px white;

    text-align: left;
  }

  span {
    display: block;
    margin-top: 20px;
  }

  button {
    margin-top: 15px;
    margin-right: 10px;
    padding: 10px;
    float: right;
    width: 105px;

    color: white;
    background-color: darkblue;
    border: none;
    border-radius: 10px;
    cursor: pointer;
    font-size: 14px;
  }

  button:hover {
    box-shadow: 0 0 5px darkslateblue;
  }

  #heading {
    font-size: 24px;
    font-weight: bolder;
    font-family: 'Kulim Park', sans-serif;
  }

  #difficulty {
    position: absolute;
    right: 16px;
    top: 16px;
    height: 24px;
    width: 80px;
    padding: 5px;

    background: rgb(97, 225, 230);
    color: white;
    text-align: center;
    border-radius: 16px;
    font-weight: bold;
  }

  #category {
    font-size: 14px;
    font-weight: normal;
    font-family: 'Karla', sans-serif;
  }

  #button-bar {
    position: absolute;
    bottom: 24px;
    right: 0;
  }

  #choice {
    margin-top: 16px;
    padding: 8px;

    border: 1px solid #4e5656;
    border-radius: 8px;
  }

  .choice-text {
    margin-left: 8px;
    margin-right: 8px;
    font-family: 'Lato', sans-serif;
    font-style: normal !important;
    font-size: 16px;
  }

  #choice:hover {
    cursor: pointer;
    background: green;
    border: 1px solid green;
    color: white;
  }

  #snackbar {
    position: absolute;
    left: 16px;
    bottom: 24px;
  }

  #results {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translateX(-50%) translateY(-50%);
    text-align: center;
  }

  #score {
    font-size: 44px;
    font-family: 'Kulim Park', sans-serif;
  }

  @media screen and (max-width: 960px) {
    #main {
      width: calc(100vw - 15%);
      height: calc(100vh - 30%);
    }
    #difficulty {
      top: -16px;
    }
    #score {
      font-size: 32px;
    }

    button {
      margin-top: 8px;
      margin-right: 8px;
      padding: 8px;
      float: right;
      width: 90px;

      color: white;
      background-color: darkblue;
      border: none;
      border-radius: 10px;
      cursor: pointer;
      font-size: 14px;
    }
  }

  .category-difficult {
    background: darkred !important;
  }

  .category-hard {
    background: red !important;
  }

  .category-medium {
    background: orange !important;
  }

  .category-easy {
    background: green !important;
  }

  .button-show-score {
    color: whitesmoke;
    background-color: darkolivegreen;
  }

  .button-play-again {
    margin-top: 15px;
    margin-right: 15px;
    padding: 10px;
    width: 160px;
    float: none;

    color: white;
    background-color: darkblue;
    border: none;
    border-radius: 10px;
    cursor: pointer;
  }
</style>