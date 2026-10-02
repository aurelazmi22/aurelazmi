# Tani Tanya - Tanya Jawab Pertanian

Website tanya jawab pertanian (SvelteKit + Supabase), siap deploy ke Vercel.
Contact person: Azkhia Aurelia Azmi - 082261817052

## 1. Siapkan Supabase
1. Buat project di https://supabase.com.
2. Buka **SQL Editor**, tempel isi `supabase/schema.sql`, lalu **Run**. Setelah itu jalankan juga `supabase/update-1.sql` (jawaban terbaik dan voting). Ini membuat tabel, kategori, aturan keamanan (RLS), dan bucket foto `question-images`.
3. Buka **Authentication > Providers**, pastikan **Email** aktif. Untuk uji coba cepat, matikan "Confirm email" (opsional).
4. Buka **Project Settings > API**, salin **Project URL** dan **anon public key**.

## 2. Jalankan lokal (opsional)
```
npm install
cp .env.example .env   # isi dua nilai dari Supabase
npm run dev
```

## 3. Deploy ke Vercel
Cara A (paling mudah, lewat GitHub):
1. Ekstrak zip, upload isinya ke repository GitHub baru.
2. Di https://vercel.com klik **Add New > Project**, pilih repository tadi.
3. Framework terdeteksi **SvelteKit**. Pada **Environment Variables** tambahkan:
   - `PUBLIC_SUPABASE_URL`
   - `PUBLIC_SUPABASE_ANON_KEY`
4. Klik **Deploy**.

Cara B (Vercel CLI): `npm i -g vercel`, lalu jalankan `vercel` di folder proyek dan isi dua environment variable yang sama (`vercel env add`).

## 4. Setelah deploy
Di Supabase **Authentication > URL Configuration**, isi **Site URL** dengan alamat Vercel Anda (misalnya `https://nama-app.vercel.app`) agar link konfirmasi email dan link reset kata sandi benar. Tambahkan juga `https://nama-app.vercel.app/reset` di **Redirect URLs**.

## Masalah umum
- Build gagal "PUBLIC_SUPABASE_URL is not exported": environment variable belum diisi di Vercel.
- Gagal unggah foto: pastikan `schema.sql` sudah dijalankan penuh (bucket dan policy storage).
