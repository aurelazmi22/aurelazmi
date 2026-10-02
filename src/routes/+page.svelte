<script>
  import { onMount } from 'svelte';
  import { supabase, tanggal } from '$lib/supabase';
  let cats = [], items = [], cat = '', q = '', loading = true, err = '';

  async function load() {
    loading = true; err = '';
    let r = supabase.from('questions')
      .select('id,title,description,created_at,author,categories(nama),answers(count)')
      .order('created_at', { ascending: false }).limit(50);
    if (cat) r = r.eq('category_id', cat);
    if (q.trim()) r = r.ilike('title', `%${q.trim()}%`);
    const { data, error } = await r;
    if (error) err = error.message; else items = data;
    loading = false;
  }
  function pilih(id) { cat = id; load(); }
  onMount(async () => {
    const { data } = await supabase.from('categories').select('*').order('nama');
    cats = data || [];
    load();
  });
</script>

<h1>Tanya apa saja soal pertanian</h1>
<form class="row" on:submit|preventDefault={load}>
  <input type="search" placeholder="Cari pertanyaan..." bind:value={q} aria-label="Cari pertanyaan" />
  <button class="btn" style="margin-bottom:.5rem">Cari</button>
</form>

<div class="chips">
  <button class:on={cat === ''} on:click={() => pilih('')}>Semua</button>
  {#each cats as c}
    <button class:on={cat === c.id} on:click={() => pilih(c.id)}>{c.nama}</button>
  {/each}
</div>

{#if err}<p class="err">{err}</p>{/if}
{#if loading}<p>Memuat pertanyaan...</p>
{:else if !items.length}<p>Belum ada pertanyaan di sini. <a href="/ask">Jadilah yang pertama bertanya.</a></p>{/if}

{#each items as it}
  <article class="card">
    <h3><a href="/question/{it.id}">{it.title}</a></h3>
    <p style="margin:.2rem 0">{it.description.slice(0, 160)}{it.description.length > 160 ? '...' : ''}</p>
    <div class="meta">
      <span class="tag">{it.categories?.nama}</span>
      {it.author} - {tanggal(it.created_at)} - {it.answers?.[0]?.count ?? 0} jawaban
    </div>
  </article>
{/each}
