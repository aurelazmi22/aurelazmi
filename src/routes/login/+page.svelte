<script>
  import { goto } from '$app/navigation';
  import { supabase } from '$lib/supabase';
  let daftar = false, email = '', password = '', pesan = '', err = '', busy = false;

  async function lupa() {
    err = ''; pesan = '';
    if (!email) { err = 'Isi email Anda dulu, lalu klik Lupa kata sandi.'; return; }
    const { error } = await supabase.auth.resetPasswordForEmail(email, { redirectTo: location.origin + '/reset' });
    if (error) err = error.message; else pesan = 'Link atur ulang kata sandi sudah dikirim ke email Anda.';
  }
  async function kirim() {
    busy = true; err = ''; pesan = '';
    const fn = daftar ? supabase.auth.signUp : supabase.auth.signInWithPassword;
    const { data, error } = await fn.call(supabase.auth, { email, password });
    busy = false;
    if (error) { err = error.message; return; }
    if (daftar && !data.session) { pesan = 'Pendaftaran berhasil. Cek email Anda untuk konfirmasi, lalu masuk.'; return; }
    goto('/');
  }
</script>

<h1>{daftar ? 'Daftar akun' : 'Masuk'}</h1>
<form on:submit|preventDefault={kirim} style="max-width:380px">
  <label for="e">Email</label>
  <input id="e" type="email" required bind:value={email} autocomplete="email" />
  <label for="p">Kata sandi (min. 6 karakter)</label>
  <input id="p" type="password" required minlength="6" bind:value={password} autocomplete={daftar ? 'new-password' : 'current-password'} />
  {#if err}<p class="err">{err}</p>{/if}
  {#if pesan}<p class="ok">{pesan}</p>{/if}
  <button class="btn" disabled={busy}>{daftar ? 'Daftar' : 'Masuk'}</button>
</form>
{#if !daftar}<p><a href="#lupa" on:click|preventDefault={lupa}>Lupa kata sandi?</a></p>{/if}
<p>
  {daftar ? 'Sudah punya akun?' : 'Belum punya akun?'}
  <a href="#ganti" on:click|preventDefault={() => (daftar = !daftar)}>{daftar ? 'Masuk' : 'Daftar'}</a>
</p>
