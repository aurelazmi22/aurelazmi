<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import { supabase, user, namaDari } from '$lib/supabase';
  let cats = [], title = '', description = '', category_id = '', file = null, err = '', busy = false;

  onMount(async () => {
    const { data } = await supabase.from('categories').select('*').order('nama');
    cats = data || [];
  });

  async function kirim() {
    busy = true; err = '';
    let image_url = null;
    if (file) {
      if (file.size > 3 * 1024 * 1024) { err = 'Ukuran foto maksimal 3 MB.'; busy = false; return; }
      const path = `${$user.id}/${Date.now()}-${file.name.replace(/[^\w.-]/g, '_')}`;
      const up = await supabase.storage.from('question-images').upload(path, file);
      if (up.error) { err = up.error.message; busy = false; return; }
      image_url = supabase.storage.from('question-images').getPublicUrl(path).data.publicUrl;
    }
    const { data, error } = await supabase.from('questions')
      .insert({ title, description, category_id: Number(category_id), image_url, author: namaDari($user) })
      .select('id').single();
    busy = false;
    if (error) { err = error.message; return; }
    goto(`/question/${data.id}`);
  }
</script>

<h1>Ajukan pertanyaan</h1>
{#if !$user}
  <p>Anda perlu <a href="/login">masuk</a> dulu untuk bertanya.</p>
{:else}
  <form on:submit|preventDefault={kirim}>
    <label for="k">Kategori</label>
    <select id="k" required bind:value={category_id}>
      <option value="" disabled>Pilih kategori</option>
      {#each cats as c}<option value={c.id}>{c.nama}</option>{/each}
    </select>
    <label for="t">Judul</label>
    <input id="t" required maxlength="150" bind:value={title} placeholder="Contoh: Daun cabai keriting dan menguning, kenapa?" />
    <label for="d">Deskripsi</label>
    <textarea id="d" required rows="7" bind:value={description} placeholder="Jelaskan kondisi lahan, jenis tanaman, dan yang sudah Anda coba."></textarea>
    <label for="f">Foto (opsional, maks. 3 MB)</label>
    <input id="f" type="file" accept="image/*" on:change={(e) => (file = e.target.files[0] || null)} />
    {#if err}<p class="err">{err}</p>{/if}
    <button class="btn" disabled={busy}>{busy ? 'Mengirim...' : 'Kirim pertanyaan'}</button>
  </form>
{/if}
