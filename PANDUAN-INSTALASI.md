# NexaERM v2.0 — Panduan Migrasi: Backend Apps Script + Frontend GitHub Pages

**Apa yang berubah?** Sebelumnya aplikasi disajikan oleh Apps Script (lalu dibungkus iframe di Blogger).
Sekarang:

| Bagian | Dulu | Sekarang |
|---|---|---|
| Tampilan (HTML/CSS/JS) | Di dalam proyek Apps Script / Blogger | **GitHub Pages** (situs statis, cepat, tanpa iframe) |
| Data & logika | Apps Script + Sheets + Drive | **Tetap** Apps Script + Sheets + Drive, kini sebagai **REST API (JSON)** |
| Login & hak akses | Token sesi | **Tetap sama** (data pasien tetap terlindungi login) |
| Tautan konsultasi pasien | `blogspot.com/p/k.html?k=KODE` | `https://USERNAME.github.io/NAMA-REPO/?k=KODE` |

Data Anda (Sheets, Drive, akun, invoice, rekam medis) **tidak berubah dan tidak perlu dipindah**.

---

## BAGIAN A — Perbarui Backend (Apps Script)

1. Buka proyek Apps Script NexaERM Anda → **Kode.gs** → hapus semua isinya → tempel isi `Kode.gs` versi 2.0 → **Simpan**.
2. (Disarankan) **Pengaturan proyek → centang "Tampilkan berkas manifes appsscript.json"**, buka `appsscript.json`, ganti dengan berkas `appsscript.json` yang diberikan → Simpan.
3. **Hapus** tiga berkas HTML lama dari proyek: `Index`, `Stylesheet`, `JavaScript` (klik titik tiga di samping nama berkas → Hapus). Sudah tidak dipakai.
4. Pilih fungsi **`siapkanV142`** → ▶ Run (sekali; memasang pemicu pre-warm 5 menit). Setujui izin bila diminta.
5. **Deploy → Kelola deployment → ✏️ Edit → Versi: Versi baru → Deploy.**
   Pastikan: *Jalankan sebagai* **Saya** · *Yang memiliki akses* **Siapa saja**.
   URL `/exec` **tetap sama**. Salin URL itu — Anda butuh di Bagian B.
6. Cek cepat: buka URL `/exec` di tab baru. Harus muncul teks JSON seperti
   `{"success":true,"data":{"app":"NexaERM","api":true,"versi":"2.0.0"},"message":"OK"}`.
   Kalau muncul halaman login Google atau error → akses deployment belum "Siapa saja".

---

## BAGIAN B — Pasang Frontend di GitHub Pages

> Folder kerja Anda adalah **`nexaerm-frontend`** (hasil ekstrak ZIP). Di dalamnya langsung terlihat
> `index.html`, `css`, `js`, `img`. **Di folder inilah** perintah `git init` dijalankan — bukan folder di atasnya.

### B1. Isi alamat backend
Buka `nexaerm-frontend\js\config.js` dengan Notepad, ganti:
```js
GAS_URL: 'GANTI_DENGAN_URL_EXEC_WEB_APP'
```
menjadi URL `/exec` dari Bagian A langkah 5 (tetap diapit tanda kutip). Simpan.

### B2. Pasang Git & buat akun GitHub (sekali saja)
- Windows: unduh Git dari https://git-scm.com/download/win (pengaturan default). Mac: ketik `git --version` di Terminal. Cek: `git --version`.
- Daftar akun di https://github.com. Username Anda menjadi bagian alamat situs (`USERNAME.github.io`).
- Identitas Git (sekali per komputer):
```bash
git config --global user.name "Nama Anda"
git config --global user.email "email-akun-github@contoh.com"
```

### B3. Buat repository
github.com → tombol **+** → **New repository** → nama mis. `nexaerm` → pilih **Public** → **jangan** centang README/.gitignore/license → **Create repository**.

> Public aman: isi repo hanya tampilan + alamat API. Tidak ada data pasien atau kata sandi di dalamnya.
> Semua data tetap di Google Sheets Anda dan hanya bisa diakses setelah login.

### B4. Buka terminal di folder yang benar
Buka folder `nexaerm-frontend` di File Explorer → klik address bar → ketik `powershell` → Enter.
Verifikasi:
```powershell
dir
```
**Wajib terlihat `index.html`, `css`, `js`, `img` di daftar teratas.** Kalau yang terlihat justru folder `nexaerm-frontend`
lagi, masuk dulu dengan `cd nexaerm-frontend`. Jangan lanjut sebelum `index.html` terlihat.

### B5. Push pertama
Satu per satu:
```bash
git init
git add .
git commit -m "NexaERM v2.0 frontend"
git branch -M main
git remote add origin https://github.com/USERNAME/NAMA-REPO.git
git push -u origin main
```
(`git add .` — ada titik di akhir.) Saat diminta *Username* isi username GitHub; *Password* isi
**Personal Access Token** (bukan password akun). Layar tidak menampilkan apa pun saat paste token — itu normal; tekan Enter.

**Membuat token:** https://github.com/settings/tokens → *Generate new token (classic)* → Note bebas →
Expiration 90 days → centang **repo** → Generate → salin `ghp_...` (hanya tampil sekali).
Alternatif visual tanpa terminal: **GitHub Desktop** → Add Local Repository → pilih `nexaerm-frontend` → Publish repository.

Tanda sukses: `Writing objects: 100%` dan `* [new branch] main -> main`.

### B6. Aktifkan GitHub Pages
Repo di github.com → **Settings → Pages** → *Source*: **Deploy from a branch** → Branch **main** · **/ (root)** → **Save**.
Tunggu 1–2 menit, refresh. Muncul: *Your site is live at `https://USERNAME.github.io/NAMA-REPO/`*. Pastikan **Enforce HTTPS** tercentang.

### B7. Uji
Buka alamat situs → layar login NexaERM tampil → masuk dengan akun Anda.

---

## BAGIAN C — Setelan Setelah Pindah

1. Masuk sebagai Superadmin → **Master Data → Pengaturan → Konsultasi Online → "Alamat halaman konsultasi"** isi:
   `https://USERNAME.github.io/NAMA-REPO/` → Simpan.
   Tautan pasien otomatis menjadi `…/NAMA-REPO/?k=KODE`. (Kolom ini kini **wajib** terisi sebelum mendaftarkan konsultasi.)
2. Tautan konsultasi lama (Blogger) tidak lagi berlaku: klik **Kirim / buat ulang tautan** pada konsultasi yang masih aktif.
3. Blog Blogger/tema lama boleh dihentikan. Tidak ada lagi yang bergantung padanya.

## Update di kemudian hari
- **Tampilan berubah:** ganti berkas di folder `nexaerm-frontend` (jaga `GAS_URL` di `js/config.js`), lalu
  `git add .` → `git commit -m "update"` → `git push`. Tunggu 1–2 menit, tekan **Ctrl+Shift+R**.
- **Backend berubah:** tempel `Kode.gs` baru → Deploy → Kelola deployment → Edit → Versi baru (URL tidak berubah).

## Catatan penting
- Data yang diedit langsung di Google Sheets (bukan lewat aplikasi) tampil di aplikasi paling lama 5 menit (cache).
- Pemicu **pemanasan** (tiap 5 menit, jam 06–22) menjaga Apps Script tetap "hangat" supaya respons cepat.
- Situs memakai `noindex` agar tidak muncul di Google. Jangan menautkannya di tempat umum.
- Token sesi tersimpan di peramban staf (localStorage); keluar otomatis setelah 8 jam atau saat **Keluar**.

## Pemecahan masalah

| Gejala | Penyebab | Solusi |
|---|---|---|
| Halaman 404 di GitHub Pages | `index.html` tidak di root repo (folder induk ikut ter-push) | Buka repo di github.com: jika terlihat folder `nexaerm-frontend` di root, ulangi B4–B5 dari **dalam** folder itu dengan `git push -u origin main --force` |
| Halaman tanpa warna/tata letak, Console: 404 css/js | File rata di root / folder `css` `js` tidak ada | Pastikan struktur seperti di README; jangan upload lewat tombol "Upload files" di web GitHub |
| "Alamat server belum diatur…" | `GAS_URL` masih berisi `GANTI_…` | Edit `js/config.js`, push ulang |
| "Server tidak membalas dengan benar…" | Deployment bukan "Siapa saja", atau belum versi baru | Bagian A langkah 5; buka URL `/exec` di tab baru, harus tampil JSON |
| "Koneksi bermasalah" | Internet putus, atau Apps Script sedang dingin/penuh kuota | Coba lagi beberapa detik; cek Executions di Apps Script |
| Console: CORS error | Frontend mengirim `application/json` (sudah diatasi: kode memakai `text/plain`) atau URL `/dev` dipakai | Pakai URL `/exec` dari Deploy, bukan `/dev` |
| Perubahan tak muncul setelah push | Cache | Ctrl+Shift+R / jendela Incognito |
| `Password authentication is not supported` | GitHub butuh token | Buat Personal Access Token (B5) |
| `remote origin already exists` | Sudah pernah disambungkan | Lewati, lanjut `git push`; salah URL: `git remote set-url origin <url>` |
| `Updates were rejected…` | Repo punya file yang tak ada di lokal | `git pull --rebase origin main` lalu `git push` |
| `LF will be replaced by CRLF` | Peringatan format baris Windows | Abaikan |

---

## UPDATE v2.1 — Percepatan (pasang di atas v2.0)

**Yang berubah**
- **Pemanasan saat halaman dibuka:** server dibangunkan selagi Anda mengetik username/password, jadi cold start tidak lagi menunda login.
- **Login = 1 panggilan:** data awal ikut dikirim bersama login (dulu 2 panggilan berurutan).
- **Cache di memori** untuk Beranda, Master Data, Laporan, dan Tagihan (20–60 detik). Pindah menu bolak-balik tidak memanggil server lagi. Setiap ada penyimpanan data, cache otomatis dibuang. *Data pasien tidak disimpan ke disk perangkat.*
- **Anti-putus:** panggilan baca diulang otomatis (maks 3×) bila jaringan putus; panggilan tulis diulang 1× dengan nomor permintaan yang sama dan **tidak akan tersimpan dobel**.
- Lencana konsultasi dicek tiap 45 detik (dulu 20 detik) untuk menghemat jatah eksekusi Apps Script.
- `login` tidak lagi menahan kunci global saat menyiapkan data awal.

**Cara pasang**
1. **Backend:** tempel `Kode.gs` baru → Simpan → Deploy → Kelola deployment → ✏️ Edit → **Versi baru** → Deploy (URL tidak berubah).
2. **Frontend:** dari ZIP baru, salin **`index.html`, folder `css`, `js/app.js`, dan `img`** ke folder repo Anda. **Jangan timpa `js/config.js`** (berisi URL Anda).
3. Di terminal, dalam folder repo:
```bash
git add .
git commit -m "v2.1 percepatan"
git push
```
4. Tunggu 1–2 menit, tekan **Ctrl+Shift+R**.

**Catatan**
- Perubahan data dari staf lain tampil dalam ≤ 20–60 detik di menu yang di-cache (atau langsung setelah Anda sendiri menyimpan sesuatu).
- Hasil terbaik: tetap jalankan pemicu `pemanasan` (lihat bagian Backend) agar server tetap hangat di jam kerja.

---

## UPDATE v2.2 — Pendaftaran fleksibel

Pada **Jadwal Kunjungan baru** dan **Daftarkan Konsultasi** kini ada dua pilihan di bagian Pasien:
**Pasien terdaftar** (cari dari daftar) atau **Pasien baru** (isi nama, tanggal lahir, jenis kelamin, alamat, WhatsApp; alergi opsional).
Pasien baru otomatis masuk Data Pasien dengan No. RM, lalu langsung dijadwalkan / dibuatkan tautan konsultasi.
Sistem memperingatkan bila ada kemungkinan pasien ganda (nama + tanggal lahir sama).

Update ini **hanya frontend** — Kode.gs tidak berubah, tidak perlu deploy ulang backend.
Ganti isi repo dengan folder baru, tetapi **pertahankan `js/config.js` Anda** (berisi alamat /exec asli).
