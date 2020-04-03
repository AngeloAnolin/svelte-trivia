<div class="container" in:fadeIn out:fadeOut>
  {#if representation.length > 0 && !resultsScreen}
    
    <div class="level is-mobile">
      <div class="level-left">
        <div class="level-item">
          <div class="columns is-multiline">
            <div class="column is-12">
              <span class="title is-3">
                Question No. { questionNo + 1}
              </span>
            </div>
            <div class="column is-12">
              <span class="subtitle is-5">
                Category: { representation[questionNo].category }
              </span>
            </div>
          </div>
        </div>
      </div>

      <div class="level-right">
        <div class="level-item">
          <div class="{ checkDifficulty(representation[questionNo].difficulty) }">
            { representation[questionNo].difficulty.toUpperCase() }
          </div>
        </div>
      </div>
    </div>

    <hr />
    
    <span class="is-size-4">
      { representation[questionNo].question }
    </span>

    <div class="section">

      { #if representation[questionNo].answerChoices } 
        
        { #each representation[questionNo].answerChoices as choice }

          { #if representation[questionNo].answered && representation[questionNo].correct}
            { #if choice === representation[questionNo].answerChoice }
              <button class="button is-success is-medium is-fullwidth is-rounded">
                { choice }
              </button>
            { :else }
              <button class="button is-warning is-medium is-fullwidth is-rounded">
               { choice }
              </button>
            { /if }
          { :else if representation[questionNo].answered && !representation[questionNo].correct }
            {#if choice === representation[questionNo].answer}
              <button class="button is-link is-success is-medium is-rounded is-fullwidth">
                { choice }
              </button>
            {:else}
              <button class="button is-link is-danger is-medium is-rounded is-fullwidth">
                { choice }
              </button>
            {/if}
          { :else }
            <button class="button is-link is-outlined is-medium is-rounded is-fullwidth" on:click={ e => handleAnswerChoice(e) }>
              { choice }
            </button>
          { /if }
        {/each}

      { /if }

    </div>

    <div class="section">
      <div class="level">
        <div class="level-left">
          <div class="level-item is-pulled-left">
            {#if snackbarVisibility}
              <div in:fadeIn out:fadeOut>
                <Snackbar message="{snackbarMessage}"></Snackbar>
              </div>
            {/if}
          </div>
        </div>
        
        <div class="level-right">
          <div class="level-item is-pulled-right">
            {#if navigationButtonsVisibility}
              <div class="field is-grouped">
                {#if (questionNo > 0)}
                  <button class="button is-normal is-info is-medium is-outlined is-fullwidth button-navigation" value="Back" on:click="{() => handleClick('b')}">
                    Previous
                  </button>
                {/if}

                {#if (questionNo < (numberOfQuestions - 1))}
                  <button class="button is-normal is-link is-medium is-outlined is-fullwidth button-navigation" value="Next" on:click="{() => handleClick('f')}">
                    Next
                  </button>
                {/if}

                {#if (questionNo === (numberOfQuestions - 1))}
                  <button class="button is-normal is-primary is-medium is-outlined" value="Show Score" on:click="{() => showScore()}">
                    Show Score
                  </button>
                {/if}
              </div>
            {/if}
          </div>
        </div>
      </div>
    </div>

  {:else if resultsScreen}
    <div class="section has-text-centered">
      <p id="score">
        Score: <i>{ score } / { numberOfQuestions }</i>
      </p>
      <p style="font-size: 24px">
        {@html finalMessage}
      </p>
      <p>
        <button class="button is-link is-light is-medium" value="Play Again" on:click="{() => refreshToPlay()}">
          Play Again
        </button>
      </p>
    </div>
  {:else}
    <div class="columns is-mobile">
      <div class="column is-offset-4-fullhd is-offset-3-widescreen is-offset-3-desktop is-offset-2-tablet is-4-fullhd is-6-widescreen is-6-desktop is-8-tablet is-12-mobile">
      <div class="box">
        <div class="content">
          <span class="title is-4 has-text-centered has-text-info">Retrieving Trivia Questions</span>
          <progress class="progress is-small is-primary" max="100"></progress>
        </div>
      </div>
      </div>
    </div>
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
    // if (snackbarVisibility) snackbarVisibility = !snackbarVisibility;
    snackbarVisibility = false;

    if (change === 'f') questionNo += 1;
    else questionNo -= 1;
  }

  function showScore () {
    if (questionNo === (numberOfQuestions - 1)) {
      navigationButtonsVisibility = false;
      resultsScreen = true;
      snackbarVisibility = false;

      dispatch('resultsScreen', { showScore: false });

      if (score < 1) {
        finalMessage = 'You need to brush up your skills!';
      } else if (score === 3) {
        finalMessage = "Good try but you can do better!";
      } else {
        finalMessage = "Great Job!";
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
    let className = '';
    switch (item) {
      case 'easy':
        className = 'box has-background-success has-text-centered';
        break;
      case 'medium':
        className = 'box has-background-warning has-text-centered';
        break;
      case 'hard':
        className = 'box has-background-danger has-text-centered';
        break;
      case 'difficult':
        className = 'box has-background-dark has-text-centered';
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
    score = 0;
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
    margin:  20px 20px 20px 20px;
    padding: 10px;
  }

  button:hover {
    box-shadow: 0 0 5px darkslateblue;
  }

  .button-navigation {
    width: 120px !important;
    margin-bottom: 20px;
  }

  .choice-text {
    margin-left: 8px;
    margin-right: 8px;
    font-family: 'Lato', sans-serif;
    font-style: normal !important;
    font-size: 16px;
  }

  #score {
    font-size: 44px;
    font-family: 'Kulim Park', sans-serif;
  }

  @media screen and (max-width: 960px) {
    #score {
      font-size: 32px;
    }

    button {
      margin:  8px 8px 8px 8px;
    }
  }
</style>