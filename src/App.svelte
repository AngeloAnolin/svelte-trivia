<script>
  import './styles.scss'
  import QuizArea from './QuizArea.svelte'
  let playerScore = 0
  let visible = true
</script>

<main>
  <div class="container is-fluid">
    <div class="container-expand">
      <h1 class="title is-1 has-text-info header">Ultimate Trivia!</h1>
      
      <QuizArea></QuizArea>
    </div>
  </div>

  <footer class="has-background-light footerCss">
    <div class="container is-fluid" style="padding-top: 20px;">
      Trivia Quiz App built with <a href="https://svelte.dev" target="_blank">Svelte</a> and <a href="https://bulma.io" target="_blank">Bulma</a> by <strong>Angelo Anolin</strong>.
    </div>
  </footer>
</main>

<style>
  .header {
    padding-top: 20px;
    text-align: center;
  }

	@media (min-width: 640px) {
		main {
			max-width: none;
    }
	}

  .container-expand {
    min-height: calc(100vh - 70px);
  }

  .footerCss {
    padding-top: 12px;
    height: 70px;
    margin-top: 12px;
  }
</style>