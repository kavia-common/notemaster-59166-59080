<script lang="ts">
  import NoteEditor from '$lib/components/NoteEditor.svelte';
  import { notesStore } from '$lib/stores/notes';
  import { goto } from '$app/navigation';

  function handleSubmit(draft) {
    const note = notesStore.upsert(draft);
    goto(`/note/${note.id}`);
  }
</script>

<svelte:head>
  <title>New note • Notemaster</title>
</svelte:head>

<section class="wrap">
  <h2>Create note</h2>
  <NoteEditor initial={null} onSubmit={handleSubmit} onCancel={() => goto('/')} />
</section>

<style>
  .wrap {
    padding: 1rem;
    max-width: 900px;
    width: 100%;
    margin: 0 auto;
  }
  h2 {
    margin: .5rem 0 1rem 0;
  }
</style>
