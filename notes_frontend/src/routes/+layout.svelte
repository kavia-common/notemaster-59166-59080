<script lang="ts">
  import '../app.css';
  import Header from '$lib/components/Header.svelte';
  let { children } = $props();
</script>

<Header />
<main class="container">
  {@render children()}
</main>

<style>
  .container {
    max-width: 1100px;
    margin: 0 auto;
    min-height: calc(100vh - 56px);
    display: flex;
    flex-direction: column;
    gap: 0;
  }
</style>
