{#if message}
  <div class="box has-background-success has-text-white">
    <span class="is-size-5">Correct!</span>
  </div>
{:else}
  <div class="box has-background-danger has-text-white">
    <span class="is-size-5">Wrong!</span>
  </div>
{/if}

<script>
  export let message = '';
</script>

<style>
  
</style>