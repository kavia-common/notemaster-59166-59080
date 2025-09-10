<script lang="ts">
  import SearchBar from '$lib/components/SearchBar.svelte';
  import NoteCard from '$lib/components/NoteCard.svelte';
  import EmptyState from '$lib/components/EmptyState.svelte';
  import { notesStore, type Note } from '$lib/stores/notes';
  import { goto } from '$app/navigation';
  let query = $state('');
  let selectedTag: string | null = $state(null);
  let showArchived = $state(false);

  function filteredNotes(): Note[] {
    return notesStore
      .list()
      .filter((n) => (showArchived ? true : !n.archived))
      .filter((n) => (selectedTag ? (n.tags ?? []).includes(selectedTag) : true))
      .filter((n) => {
        if (!query.trim()) return true;
        const q = query.toLowerCase();
        return (n.title || '').toLowerCase().includes(q) || (n.content || '').toLowerCase().includes(q);
      });
  }

  function handleDelete(id: string) {
    if (confirm('Delete this note? This cannot be undone.')) {
      notesStore.remove(id);
    }
  }

  function createNoteQuick() {
    const n = notesStore.upsert({ title: '', content: '' });
    goto(`/note/${n.id}`);
  }
</script>

<svelte:head>
  <title>Notemaster</title>
</svelte:head>

<SearchBar
  {query}
  {selectedTag}
  {showArchived}
  tags={notesStore.allTags()}
  onChange={(q)=> (query = q)}
  onTagChange={(t)=> (selectedTag = t)}
  onToggleArchived={(v)=> (showArchived = v)}
/>

<section class="toolbar">
  <button class="primary" onclick={createNoteQuick}>New Note</button>
</section>

{#if filteredNotes().length === 0}
  <EmptyState title="No notes found" description="Try creating a new note or adjusting your filters." actionLabel="Create note" onAction={createNoteQuick} />
{:else}
  <section class="grid">
    {#each filteredNotes() as n (n.id)}
      <NoteCard
        note={n}
        onTogglePinned={(id)=> notesStore.togglePinned(id)}
        onToggleArchived={(id)=> notesStore.toggleArchived(id)}
        onDelete={handleDelete}
      />
    {/each}
  </section>
{/if}

<style>
  .toolbar {
    display: flex;
    justify-content: flex-end;
    padding: .75rem 1rem;
    border-bottom: 1px solid #eee;
    background: #fff;
  }
  .toolbar .primary {
    background: var(--color-accent);
    color: #fff;
    border: none;
    padding: .5rem .9rem;
    border-radius: .6rem;
    cursor: pointer;
  }
  .grid {
    padding: 1rem;
    display: grid;
    grid-template-columns: repeat(12, 1fr);
    gap: .9rem;
  }
  @media (max-width: 1200px) {
    .grid { grid-template-columns: repeat(8, 1fr); }
  }
  @media (max-width: 800px) {
    .grid { grid-template-columns: repeat(6, 1fr); }
  }
  @media (max-width: 640px) {
    .grid { grid-template-columns: repeat(1, 1fr); }
  }
  .grid :global(article.card) { grid-column: span 3; }
  @media (max-width: 1200px) { .grid :global(article.card) { grid-column: span 4; } }
  @media (max-width: 800px) { .grid :global(article.card) { grid-column: span 3; } }
  @media (max-width: 640px) { .grid :global(article.card) { grid-column: span 1; } }
</style>
