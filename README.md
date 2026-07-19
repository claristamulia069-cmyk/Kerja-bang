# Sistem Manajemen KKN — Panduan Setup

Aplikasi web 1 file (`index.html`) untuk mengelola kepanitiaan KKN 1 bulan:
anggota & divisi, tugas, keuangan, jadwal, dan dokumen. Backend memakai
Firebase (Auth, Firestore, Storage). Siap deploy ke Railway sebagai situs statis.

## Role & sifat akun

- **Super Admin** — akun tunggal, akses penuh ke semua data, dan satu-satunya
  yang boleh membuat akun anggota lain atau mengubah divisi seseorang.
  Cocok dipakai untuk testing karena bisa mengakses semua fitur.
- **BPH** — akses penuh ke data operasional (tugas, keuangan, jadwal, dokumen
  semua divisi), tapi **tidak bisa** mengubah akun/divisi anggota lain.
- **Sie** (Akademi, Humas, PDD, Acara) — hanya bisa mengelola tugas/jadwal/
  dokumen milik divisinya sendiri, dan boleh mencatat transaksi keuangan.

**Akun bersifat tetap ("paten")**: tidak ada pendaftaran terbuka. Semua akun
—termasuk BPH dan sie— dibuat oleh Super Admin lewat menu **Anggota → Tambah
Anggota** di dalam aplikasi. Anggota biasa tidak bisa mengubah divisinya
sendiri; hanya Super Admin yang bisa.

**Ganti sandi dibatasi satu kali**: setiap akun (selain lewat reset manual
oleh Anda di Firebase Console) hanya bisa mengganti sandinya sendiri sekali,
lewat menu **Ganti Sandi** di sidebar. Setelah dipakai, opsi ini terkunci.
*Catatan jujur:* ini adalah pembatasan di level tampilan aplikasi (memakai
penanda `sandiDiganti` di Firestore), bukan jaminan keamanan mutlak — Firebase
Authentication sendiri tidak punya konsep "ganti sandi maksimal N kali", jadi
seseorang yang cukup teknis secara teori bisa memanggil API Firebase Auth
langsung untuk melewatinya. Untuk penggunaan tim KKN yang saling percaya,
pembatasan ini sudah cukup memadai.

---

## 1. Buat Project Firebase

1. Buka https://console.firebase.google.com → **Add project**.
2. Buka **Build → Authentication → Get started** → aktifkan sign-in method
   **Email/Password**.
3. Buka **Build → Firestore Database → Create database** (mode production).
4. Buka **Build → Storage → Get started**.
5. Buka **Project settings (ikon gerigi) → General → Your apps → Add app → Web (</>)**.
   Salin objek `firebaseConfig` yang muncul.

## 2. Isi Konfigurasi ke index.html

Buka `index.html`, cari bagian ini:

```js
const firebaseConfig = {
  apiKey: "GANTI_DENGAN_API_KEY_ANDA",
  authDomain: "GANTI.firebaseapp.com",
  projectId: "GANTI",
  storageBucket: "GANTI.appspot.com",
  messagingSenderId: "GANTI",
  appId: "GANTI"
};
```

Ganti semua nilai `"GANTI..."` dengan nilai asli dari Firebase Console.

Di dekatnya ada juga:

```js
const SUPER_SETUP_CODE = "KKN-SUPER-2026";
```

**Ganti kode ini dengan kode rahasia Anda sendiri** sebelum deploy — kode ini
dipakai satu kali di form "Setup Super Admin" pada halaman login untuk
membuat akun Super Admin pertama. Siapa pun yang tahu kode ini bisa membuat
akun Super, jadi jangan pakai kode contoh di atas dan jangan sebar ke orang
lain selain Anda.

## 3. Pasang Security Rules

**Firestore:** Firebase Console → Firestore Database → tab **Rules** → hapus
isi default, tempel isi file `firestore.rules` dari folder ini → **Publish**.

**Storage:** Firebase Console → Storage → tab **Rules** → hapus isi default,
tempel isi file `storage.rules` → **Publish**.

Rules ini membatasi akses hanya untuk pengguna yang sudah login, dan hanya
Super Admin yang boleh mengubah data akun anggota lain.

## 4. Buat Akun Super Admin (untuk testing & kelola akun)

1. Buka `index.html` di browser.
2. Klik **Setup Super Admin** di halaman login.
3. Masukkan `SUPER_SETUP_CODE` yang sudah Anda ganti di langkah 2, isi nama,
   email, dan sandi Anda sendiri → **Buat Akun Super Admin**.
4. Setelah berhasil login sebagai Super Admin, buat akun BPH dan tiap sie
   lewat menu **Anggota → Tambah Anggota**. Sandi awal yang Anda tentukan
   akan ditampilkan sekali di layar setelah akun dibuat — catat dan berikan
   ke anggota terkait secara aman (chat pribadi, bukan grup terbuka).
5. **Setelah semua akun dibuat**, sebaiknya ganti lagi nilai `SUPER_SETUP_CODE`
   di `index.html` menjadi string acak yang tidak dipakai lagi (atau hapus
   seluruh blok "Setup Super Admin" dari HTML), lalu push ulang ke GitHub,
   supaya tidak ada orang lain yang bisa membuat akun Super baru.

## 5. Deploy ke Railway

1. Upload `index.html` ke root repository GitHub Anda (file rules & README
   ini tidak wajib ikut di-deploy, cukup untuk referensi Anda sendiri).
2. Di Railway: **New Project → Deploy from GitHub repo** → pilih repo Anda.
3. Railway otomatis mendeteksi `index.html` dan men-deploy sebagai situs statis.
4. Setiap kali Anda push perubahan ke GitHub, Railway akan auto-deploy ulang.

## Struktur Data Firestore

| Koleksi     | Field utama                                                          |
|-------------|------------------------------------------------------------------------|
| `users`     | nama, email, role, jabatan, sandiDiganti, createdAt                   |
| `tasks`     | judul, deskripsi, divisi, penanggungJawab, status, deadline           |
| `finance`   | tipe (masuk/keluar), keterangan, jumlah, tanggal, createdBy           |
| `timeline`  | kegiatan, tanggal, waktu, divisi, createdBy                           |
| `documents` | nama, divisi, url, storagePath, uploadedBy, createdAt                 |

## Lupa sandi?

Karena ganti sandi dibatasi satu kali dari dalam aplikasi, kalau ada anggota
lupa sandi dan sudah pernah ganti sebelumnya, Anda (sebagai pemilik project
Firebase) bisa mereset manual lewat **Firebase Console → Authentication →
klik anggota terkait → Reset password**, atau hapus & buat ulang akunnya
lewat menu Tambah Anggota di aplikasi.

## Troubleshooting

- **`auth/api-key-not-valid`** → `firebaseConfig` belum diganti dengan nilai asli.
- **Kode setup salah** → pastikan `SUPER_SETUP_CODE` di `index.html` sama
  dengan yang Anda masukkan di form.
- **Data tidak muncul / permission denied** → pastikan Firestore Rules sudah
  di-publish sesuai `firestore.rules`.
- **Upload dokumen gagal** → cek Storage Rules sudah dipasang, dan ukuran
  file di bawah 20MB.
