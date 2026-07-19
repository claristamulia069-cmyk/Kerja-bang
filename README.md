# Sistem Manajemen KKN — Panduan Setup

Aplikasi web 1 file (`index.html`) untuk mengelola kepanitiaan KKN 1 bulan:
anggota & divisi, tugas, keuangan, jadwal, dan dokumen. Backend memakai
Firebase (Auth, Firestore, Storage). Siap deploy ke Railway sebagai situs statis.

Role yang didukung: **BPH**, **Sie Akademi**, **Sie Humas**, **Sie PDD**, **Sie Acara**.
- **BPH**: akses penuh ke semua data (bisa edit/hapus semua tugas, jadwal,
  dokumen, transaksi keuangan, dan mengubah divisi anggota lain).
- **Sie**: bisa lihat semua data, tapi hanya bisa mengedit/menghapus tugas,
  jadwal, dan dokumen milik divisinya sendiri. Semua sie bisa mencatat
  transaksi keuangan (hapus transaksi hanya bisa oleh BPH, sebagai kontrol kas).

---

## 1. Buat Project Firebase

1. Buka https://console.firebase.google.com → **Add project**.
2. Setelah project dibuat, buka **Build → Authentication → Get started**
   → aktifkan sign-in method **Email/Password**.
3. Buka **Build → Firestore Database → Create database** (mode production).
4. Buka **Build → Storage → Get started**.
5. Buka **Project settings (ikon gerigi) → General → Your apps → Add app → Web (</>)**.
   Salin objek `firebaseConfig` yang muncul.

## 2. Isi Konfigurasi ke index.html

Buka `index.html`, cari bagian ini di dekat awal tag `<script>` bawah:

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

## 3. Pasang Security Rules

**Firestore:**
Firebase Console → Firestore Database → tab **Rules** → hapus isi default,
tempel isi file `firestore.rules` dari folder ini → klik **Publish**.

**Storage:**
Firebase Console → Storage → tab **Rules** → hapus isi default, tempel isi
file `storage.rules` → klik **Publish**.

Rules ini membatasi akses hanya untuk pengguna yang sudah login, dan
membatasi hak edit/hapus sesuai divisi masing-masing — jauh lebih aman
dibanding rules `allow read, write: if true` yang membuka akses ke siapa saja.

## 4. Buat Akun Pertama (BPH)

1. Buka `index.html` di browser (bisa langsung dobel klik untuk uji coba lokal,
   atau setelah di-deploy).
2. Klik **Daftar di sini**, isi data, pilih divisi **BPH** untuk akun admin pertama.
3. Setelah anggota lain mendaftar dengan divisi sendiri-sendiri, akun BPH bisa
   mengubah divisi siapa pun lewat menu **Anggota → Ubah Divisi** bila ada
   kesalahan input saat pendaftaran.

> Aplikasi ini tidak memakai akun demo bawaan (`ketua@panitia.com` dll) —
> setiap anggota mendaftar dengan email aslinya sendiri, lebih aman untuk
> data kepanitiaan yang sesungguhnya.

## 5. Deploy ke Railway

Ikuti panduan deploy yang sudah Anda miliki (GitHub → Railway):

1. Upload `index.html` ke root repository GitHub Anda (file rules & README
   ini tidak wajib ikut di-deploy, cukup untuk referensi Anda sendiri).
2. Di Railway: **New Project → Deploy from GitHub repo** → pilih repo Anda.
3. Railway otomatis mendeteksi `index.html` dan men-deploy sebagai situs statis.
4. Setiap kali Anda push perubahan ke GitHub, Railway akan auto-deploy ulang.

## Struktur Data Firestore

| Koleksi     | Field utama                                                        |
|-------------|----------------------------------------------------------------------|
| `users`     | nama, email, role, jabatan, createdAt                               |
| `tasks`     | judul, deskripsi, divisi, penanggungJawab, status, deadline          |
| `finance`   | tipe (masuk/keluar), keterangan, jumlah, tanggal, createdBy          |
| `timeline`  | kegiatan, tanggal, waktu, divisi, createdBy                          |
| `documents` | nama, divisi, url, storagePath, uploadedBy, createdAt                |

## Troubleshooting

- **`auth/api-key-not-valid`** → `firebaseConfig` belum diganti dengan nilai asli.
- **Data tidak muncul / permission denied** → pastikan Firestore Rules sudah
  di-publish sesuai `firestore.rules`, dan pastikan koleksi `users` punya
  dokumen dengan field `role` yang sesuai untuk akun yang login.
- **Upload dokumen gagal** → cek Storage Rules sudah dipasang, dan ukuran
  file di bawah 20MB.
