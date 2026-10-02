<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import { page } from '$app/stores';
  import { supabase, user, tanggal, namaDari } from '$lib/supabase';
  let qn = null, answers = [], votes = [], body = '', err = '', busy = false, loading = true;
  const id = $page.params.id;

  $: urut = answers
    .map((a) => ({
      ...a,
      skor: votes.filter((v) => v.answer_id === a.id).length,
      saya: votes.some((v) => v.answer_id === a.id && v.user_id === $user?.id)
    }))
    .sort((x, y) => (y.id === qn?.best_answer_id) - (x.id === qn?.best_answer_id) || y.skor - x.skor);
  $: pemilik = $user && qn && $user.id === qn.user_id;

  async function load() {
    const a = await supabase.from('questions').select('*,categories(nama)').eq('id', id).maybeSingle();
    qn = a.data;
    const b = await supabase.from('answers').select('*').eq('question_id', id).order('created_at');
    answers = b.data || [];
    if (answers.length) {
      const v = await supabase.from('answer_votes').select('answer_id,user_id').in('answer_id', answers.map((x) => x.id));
      votes = v.data || [];
    } else votes = [];
    loading = false;
  }
  async function jalan(p) {
    err = '';
    const { error } = await p;
    if (error) err = error.message; else await load();
  }
  async function jawab() {
    busy = true;
    await jalan(supabase.from('answers').insert({ question_id: id, body, author: namaDari($user) }));
    if (!err) body = '';
    busy = false;
  }
  async function hapusPertanyaan() {
    if (!confirm('Hapus pertanyaan ini beserta semua jawabannya?')) return;
    const { error } = await supabase.from('questions').delete().eq('id', id);
    if (error) err = error.message; else goto('/');
  }
  const hapusJawaban = (a) => confirm('Hapus jawaban ini?') && jalan(supabase.from('answers').delete().eq('id', a.id));
  const terbaik = (a) => jalan(supabase.from('questions').update({ best_answer_id: qn.best_answer_id === a.id ? null : a.id }).eq('id', id));
  const vote = (a) => jalan(a.saya
    ? supabase.from('answer_votes').delete().eq('answer_id', a.id).eq('user_id', $user.id)
    : supabase.from('answer_votes').insert({ answer_id: a.id }));
  onMount(load);
</script>

{#if loading}<p>Memuat...</p>
{:else if !qn}<p>Pertanyaan tidak ditemukan. <a href="/">Kembali ke beranda</a></p>
{:else}
  <h1>{qn.title}</h1>
  <p class="meta"><span class="tag">{qn.categories?.nama}</span> {qn.author} - {tanggal(qn.created_at)}</p>
  <pre class="isi">{qn.description}</pre>
  {#if qn.image_url}<img class="foto" src={qn.image_url} alt="Foto dari penanya" />{/if}
  {#if pemilik}<div class="aksi"><button class="kecil hapus" on:click={hapusPertanyaan}>Hapus pertanyaan</button></div>{/if}
  {#if err}<p class="err">{err}</p>{/if}

  <h2 style="margin-top:2rem">{answers.length} jawaban</h2>
  {#each urut as a (a.id)}
    <div class="card jawab">
      {#if a.id === qn.best_answer_id}<span class="tag best">Jawaban terbaik</span>{/if}
      <pre class="isi">{a.body}</pre>
      <div class="meta">{a.author} - {tanggal(a.created_at)}</div>
      <div class="aksi">
        <button class="kecil" class:on={a.saya} disabled={!$user} on:click={() => vote(a)}
          title={$user ? '' : 'Masuk untuk memberi suara'}>Berguna {a.skor}</button>
        {#if pemilik}
          <button class="kecil" on:click={() => terbaik(a)}>{a.id === qn.best_answer_id ? 'Batalkan terbaik' : 'Tandai terbaik'}</button>
        {/if}
        {#if $user?.id === a.user_id}
          <button class="kecil hapus" on:click={() => hapusJawaban(a)}>Hapus</button>
        {/if}
      </div>
    </div>
  {:else}
    <p>Belum ada jawaban. Bagikan pengalaman Anda.</p>
  {/each}

  <h2>Jawaban Anda</h2>
  {#if !$user}
    <p><a href="/login">Masuk</a> untuk menjawab.</p>
  {:else}
    <form on:submit|preventDefault={jawab}>
      <textarea required rows="5" bind:value={body} aria-label="Jawaban Anda"></textarea>
      <button class="btn" disabled={busy}>Kirim jawaban</button>
    </form>
  {/if}
{/if}
