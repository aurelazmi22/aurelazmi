# Progress

## Sudah dibuat
- Kerangka SvelteKit (Svelte 4) dengan adapter Vercel, mode client-side (ssr dimatikan)
- Tema pertanian warna coklat, responsif, fokus keyboard terlihat
- Autentikasi email dan kata sandi (daftar, masuk, keluar) lewat Supabase Auth
- Lupa kata sandi: kirim link ke email, halaman `/reset` untuk kata sandi baru
- Beranda: daftar pertanyaan, filter kategori, pencarian judul
- 10 kategori pertanian awal (SQL)
- Ajukan pertanyaan: judul, deskripsi, kategori, foto opsional (maks. 3 MB) ke Supabase Storage
- Detail pertanyaan: daftar jawaban, form jawab (wajib login)
- Hapus pertanyaan (pemilik) dan hapus jawaban (penulis)
- Jawaban terbaik: ditandai oleh penanya, tampil paling atas
- Voting "Berguna" per jawaban (satu suara per user, bisa dibatalkan), urut dari suara terbanyak
- Skema database, RLS, dan storage di `supabase/schema.sql`; tambahan di `supabase/update-1.sql`
- README setup Supabase dan deploy Vercel
- Kontak: Azkhia Aurelia Azmi, 082261817052 (footer)

## Belum dibuat
- Edit pertanyaan/jawaban
- Halaman profil
- Paginasi dan pratinjau foto sebelum unggah
- Build belum diuji di sini (jaringan terbatas): jalankan `npm install && npm run build`
