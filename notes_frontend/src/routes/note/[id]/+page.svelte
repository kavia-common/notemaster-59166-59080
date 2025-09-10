<script lang="ts">
  import { page } from '$app/stores';
  import { notesStore, type Note } from '$lib/stores/notes';
  import NoteEditor from '$lib/components/NoteEditor.svelte';
  import TagChip from '$lib/components/TagChip.svelte';
  import { goto } from '$app/navigation';

  let note: Note | undefined;
  let edit = false;

  $effect(() => {
    const id = $page.params.id;
    note = notesStore.getById(id);
  });

  function save(draft) {
    const saved = notesStore.upsert({ ...draft, id: note?.id });
    note = saved;
    edit = false;
  }

  function del() {
    if (!note) return;
    if (confirm('Delete this note?')) {
      notesStore.remove(note.id);
      goto('/');
    }
  }

  function togglePin() {
    if (!note) return;
    notesStore.togglePinned(note.id);
    note = notesStore.getById(note.id);
  }

  function toggleArchive() {
    if (!note) return;
    notesStore.toggleArchived(note.id);
    note = notesStore.getById(note.id);
  }
</script>

<svelte:head>
  <title>{note ? `${note.title || '(Untitled)'} • Notemaster` : 'Note • Notemaster'}</title>
</svelte:head>

{#if !note}
  <section class="wrap">
    <p>Note not found.</p>
    <button onclick={() => goto('/')}>Back</button>
  </section>
{:else}
  <section class="wrap">
    <div class="topbar">
      <div class="left">
        <button class="icon" onclick={() => goto('/')} title="Back">← Back</button>
      </div>
      <div class="right">
        <button class="icon" onclick={togglePin} title={note.pinned ? 'Unpin' : 'Pin'}>{note.pinned ? '📌' : '📍'}</button>
        <button class="icon" onclick={toggleArchive} title={note.archived ? 'Unarchive' : 'Archive'}>{note.archived ? '🗂️' : '🗄️'}</button>
        <button class="icon danger" onclick={del} title="Delete">🗑️</button>
        <button class="primary" onclick={() => (edit = !edit)}>{edit ? 'Close' : 'Edit'}</button>
      </div>
    </div>

    {#if edit}
      <NoteEditor initial={note} onSubmit={save} onCancel={() => (edit = false)} />
    {:else}
      <article class="note" data-archived={note.archived}>
        <h2>{note.title || '(Untitled)'}</h2>
        {#if note.tags?.length}
          <div class="tags">
            {#each note.tags as t (t)}
              <TagChip label={t} />
            {/each}
          </div>
        {/if}
        <div class="meta">
          <span>Created: {new Date(note.createdAt).toLocaleString()}</span>
          <span>Updated: {new Date(note.updatedAt).toLocaleString()}</span>
        </div>
        <div class="content" innerHTML={note.content ? note.content.replace(/\n/g, '<br/>') : ''}></div>
      </article>
    {/if}
  </section>
{/if}

<style>
  .wrap {
    padding: 1rem;
    max-width: 900px;
    width: 100%;
    margin: 0 auto;
  }
  .topbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: .5rem;
    margin-bottom: .75rem;
  }
  .right, .left {
    display: flex;
    gap: .5rem;
    align-items: center;
  }
  .icon {
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: .5rem;
    cursor: pointer;
    padding: .4rem .6rem;
  }
  .icon.danger {
    background: #fff1f2;
    border-color: #fecdd3;
  }
  .primary {
    background: var(--color-accent);
    color: #fff;
    border: none;
    padding: .5rem .9rem;
    border-radius: .6rem;
    cursor: pointer;
  }
  article.note h2 {
    margin: .25rem 0 .5rem 0;
  }
  .tags {
    display: flex;
    gap: .35rem;
    flex-wrap: wrap;
    margin-bottom: .25rem;
  }
  .meta {
    display: flex;
    gap: 1rem;
    color: #64748b;
    font-size: .85rem;
    margin-bottom: .75rem;
  }
  .content {
    white-space: pre-wrap;
    color: var(--color-text-primary);
    line-height: 1.6;
  }
  [data-archived="true"] {
    opacity: .75;
  }
</style>
