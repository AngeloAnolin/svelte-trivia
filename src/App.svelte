<script>
  import './styles.scss'
  import QuizArea from './QuizArea.svelte'
  let playerScore = 0
  let visible = true
</script>

<main>
	<h1 class="title is-1 has-text-primary">Ultimate Trivia!</h1>
	<QuizArea></QuizArea>
  <div class="footer">
    Trivia Quiz App built with <a href="https://svelte.dev" target="_blank">Svelte</a> by <strong>Angelo Anolin</strong>.
  </div>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	/* h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 2em;
    font-weight: 100;
    padding-bottom: 12px;
    margin-bottom: 12px;
	} */

	@media (min-width: 640px) {
		main {
			max-width: none;
    }
    
    /* h1 {
      font-size: 4em;
    } */
	}

  .footer {
    position: absolute;
    right: 0;
    bottom: 0;
    left: 0;
    padding: 1rem;
    background-color: #efefef;
    text-align: center;
  }
</style>