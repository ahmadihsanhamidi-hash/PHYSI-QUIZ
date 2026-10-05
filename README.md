# PHYSI-QUIZ — Online Supabase Version

Versi ini memindahkan data inti PHYSI-QUIZ dari localStorage ke Supabase Postgres + Auth.

## 1. Setup Supabase

1. Buat project di Supabase.
2. Buka SQL Editor dan jalankan `supabase/schema.sql`.
3. Pastikan Email/Password Authentication aktif.
4. Buat satu akun guru melalui Authentication → Users.
5. Salin UUID user guru tersebut, lalu jalankan:
   ```sql
   update public.profiles
   set role = 'teacher', class_level = 'X'
   where id = 'UUID_GURU';
   ```
6. Buat akun siswa lewat halaman Daftar Siswa pada website.

## 2. Hubungkan website

Edit `config.js`:

```js
window.PHYSI_CONFIG = {
  SUPABASE_URL: 'https://PROJECT.supabase.co',
  SUPABASE_PUBLISHABLE_KEY: 'sb_publishable_xxx'
};
```

Gunakan publishable key, bukan `service_role`. Supabase mendesain client browser untuk menggunakan publishable key bersama RLS.

## 3. Jalankan lokal

Karena halaman memakai JavaScript module/CDN dan Supabase Auth, lebih aman menjalankan melalui HTTP server.

Contoh:

```bash
python -m http.server 8080
```

Lalu buka `http://localhost:8080` di folder ini.

## 4. Deploy Vercel

Upload folder ini sebagai project statis atau hubungkan repository GitHub.

Tidak diperlukan build step. `index.html` menjadi entry point.

## 5. Data yang sudah disediakan

Seed awal berisi:

- Pengukuran — kelas X
- GLB & GLBB — kelas X
- Listrik Dinamis — kelas XI
- Gelombang — kelas XI
- Relativitas — kelas XII
- 13 soal contoh dengan level Dasar, Menengah, dan HOTS
- media YouTube/PhET sesuai materi

Progress tidak diisi dengan angka palsu. Progress terbentuk dari `attempts` + `attempt_answers` ketika siswa benar-benar mengerjakan latihan.

## 6. Alur production

Siswa:

Login/Daftar → pilih kelas → pilih materi → pelajari video/PhET → latihan → feedback → nilai tersimpan → progress diperbarui.

Guru:

Login → dashboard → CRUD materi/soal/media → buat paket kuis → pantau hasil → analisis kesalahan → kirim feedback.

## 7. Data progres

Tidak ada angka kemajuan siswa yang dipalsukan sebagai data nyata. Seed database berisi materi, media, dan soal awal. Setiap kali siswa mengerjakan latihan, baris baru masuk ke `attempts` dan `attempt_answers`; dashboard kemudian menghitung rata-rata, akurasi, progress materi, dan error rate dari data tersebut.

## 8. Keamanan

RLS diaktifkan pada tabel inti. Siswa hanya dapat membuat dan membaca attempt miliknya sendiri. Guru yang profilnya memiliki role `teacher` dapat mengelola materi, media, soal, kuis, hasil, dan feedback.

Jangan pernah menaruh service role key pada frontend.
