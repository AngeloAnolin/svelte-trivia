{#if message}
  <div class="correct">
    <span>Correct!</span>
  </div>
{:else}
  <div class="incorrect">
    <span>Wrong</span>
  </div>
{/if}

<script>
  export let message = '';
</script>

<style>
  div {
    height: 36px;
    width: 100px;
    padding: 0 16px;
    line-height: 32px;
    background-color: #4e5656;
    border-radius: 5px;
    box-shadow: 0 0 5px #4e5656;
  }

  span {
    font-size: 12px;
    color: white;
  }

  .correct {
    background-color: green; 
    box-shadow: 0 0 5px green;
  }

  .incorrect {
    background-color: red; 
    box-shadow: 0 0 5px red;
  }

  @media (min-width: 640px) {
    div {
      height: 48px;
      width: 344px;
      padding: 0 16px;
      line-height: 45px;
      background-color: #4e5656;
      border-radius: 5px;
      box-shadow: 0 0 5px #4e5656;
    }

    span {
      font-size: 16px;
      color: white;
    }
  }
</style>