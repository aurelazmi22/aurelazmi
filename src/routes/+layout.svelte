<script>
  import '../app.css';
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import { supabase, user } from '$lib/supabase';

  onMount(() => {
    supabase.auth.getSession().then(({ data }) => user.set(data.session?.user ?? null));
    const { data } = supabase.auth.onAuthStateChange((_e, s) => user.set(s?.user ?? null));
    return () => data.subscription.unsubscribe();
  });
  async function keluar() { await supabase.auth.signOut(); goto('/'); }
</script>

<svelte:head><title>Tani Tanya - Tanya Jawab Pertanian</title></svelte:head>

<header>
  <div class="wrap">
    <a class="brand" href="/">Tani Tanya</a>
    <nav>
      <a href="/">Beranda</a>
      <a href="/ask">Tanya</a>
      {#if $user}
        <button on:click={keluar}>Keluar</button>
      {:else}
        <a href="/login">Masuk</a>
      {/if}
    </nav>
  </div>
</header>

<main class="wrap"><slot /></main>

<footer>
  <div class="wrap">Butuh bantuan? Hubungi Azkhia Aurelia Azmi di
    <a href="https://wa.me/6282261817052">082261817052</a>.</div>
</footer>
