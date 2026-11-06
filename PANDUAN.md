# Panduan Live Streaming YouTube lewat GitHub

Paket ini berisi file yang **persis sama dengan yang sedang berjalan sekarang**
untuk live streaming channel — bukan contoh, bukan versi percobaan.
Satu-satunya file mesinnya adalah `stream.yml`, yang ditaruh di dalam
repository GitHub pada folder `.github/workflows/`.

Dua hal yang **sengaja tidak ada di paket ini**, karena memang tidak boleh
beredar sebagai file:

1. **Video yang diputar** — tersimpan di repository sebagai aset Release
   (tag `v1`, nama file `video-album-01.mp4`, ukuran ±1 GB).
2. **Stream key YouTube** — tersimpan sebagai *Secret* terenkripsi di
   GitHub dengan nama `YT_STREAM_KEY`. Nilainya tidak pernah ditulis di
   file mana pun.

---

## 1. Cara kerjanya (gambaran besar)

```
Video (di Release GitHub)
        │
        ▼
GitHub Actions menjalankan stream.yml:
  1. memasang ffmpeg di server GitHub
  2. mengunduh video dari Release
  3. ffmpeg memutar video berulang-ulang (loop)
     dan mendorongnya ke server live YouTube
        │
        ▼
YouTube menerima aliran ──► siaran LIVE di channel
        │
        ▼
Durasi tercapai ──► aliran berhenti ──► siaran selesai
dan rekamannya otomatis tersimpan sebagai video di channel
```

Artinya: **tidak ada komputer yang harus menyala di rumah.** Yang bekerja
adalah server GitHub. Komputer/HP hanya dipakai untuk menekan tombol mulai.

## 2. Yang dibutuhkan

- Satu akun GitHub dan satu repository (repository ini: `live-youtube-2`).
- Satu channel YouTube yang akses live-nya sudah aktif.
- Satu siaran (broadcast) di YouTube Studio untuk ditumpangi aliran.
- Stream key channel (dari YouTube Studio, menu Live streaming).
- Satu file video MP4 sebagai bahan putar.

## 3. Isi file stream.yml — apa tugas tiap bagian

| Bagian | Tugasnya |
|---|---|
| `workflow_dispatch` + `minutes` | Workflow hanya jalan saat dijalankan manual / dipicu, dengan isian durasi dalam menit. Nilai bawaan di file = 55; saat menjalankan selalu diisi sesuai kebutuhan (mis. 240 untuk 4 jam). |
| `timeout-minutes: 355` | Batas keras satu pekerjaan GitHub. Satu job GitHub Actions memang berhenti sendiri pada ±6 jam — ini batas platform, bukan setelan kita. |
| Langkah *Install ffmpeg* | Memasang ffmpeg di server GitHub. **Jangan dihapus** — tanpa langkah ini run pertama dulu gagal total ("ffmpeg: command not found"). |
| Langkah *Download loop video* | Mengunduh `video-album-01.mp4` dari Release `v1` di repository yang sama. |
| Langkah *Stream to YouTube Live* | Menjalankan ffmpeg: membaca video dengan `-stream_loop -1` (putar ulang tanpa henti), video 2500 kbps, audio AAC 128 kbps, dikirim ke alamat RTMPS YouTube memakai stream key dari Secret `YT_STREAM_KEY`. Perintah `timeout "<minutes>m"` menghentikan aliran tepat saat durasi yang diminta habis. |

Angka yang dipakai pola harian saat ini: **sesi 4 jam = `minutes` diisi 240.**
Video album berdurasi ±50 menit, jadi selama 4 jam ia berputar ±5 kali.

## 4. Persiapan di YouTube Studio (sekali per sesi)

Setiap sesi live membutuhkan **satu broadcast baru** — broadcast yang sudah
berakhir tidak bisa dipakai lagi.

1. Buka YouTube Studio → **Buat → Live streaming**.
2. Jadwalkan untuk jam mulai sesi, lalu isi: judul, deskripsi,
   visibilitas **Publik**, kategori **Musik**, "Tidak dibuat untuk anak-anak".
3. Pakai **Default stream key**. **Jangan pernah menekan tombol Reset**
   kecuali kunci bocor — mereset kunci memutuskan semua setelan yang
   memakai kunci lama.
4. Simpan. Broadcast siap menerima aliran pada jamnya.

## 5. Menjalankan live (praktiknya)

Cara manual dari halaman GitHub (bisa dari HP):

1. Buka repository → tab **Actions**.
2. Di daftar workflow, pilih **stream**.
3. Tekan **Run workflow**.
4. Isi **minutes** dengan durasi sesi dalam menit (4 jam = `240`).
5. Tekan **Run workflow** sekali lagi untuk mengesahkan.
6. Tunggu ±1–2 menit. Buka halaman siaran di YouTube Studio:
   siaran berubah menjadi **LIVE**, penghitung durasi berjalan, dan
   kesehatan stream menunjukkan "Sangat baik".

Sesudah itu tidak ada yang perlu dijaga. Aliran berhenti sendiri tepat
saat durasi habis, siaran selesai, dan rekamannya tersimpan otomatis
sebagai video publik di tautan yang sama.

> Repository ini dijalankan manual dari tab Actions — belum ada
> jadwal otomatis yang terpasang. Contoh pola yang terbukti aman:
> satu sesi 4 jam = `minutes` diisi `240`, satu run utuh tanpa
> disambung run lain.

## 6. Menghentikan live lebih awal

1. Buka repository → **Actions** → run yang sedang berjalan.
2. Tekan **Cancel workflow**.
3. Aliran berhenti; sesudah ±1 menit siaran di YouTube ikut selesai
   dan rekamannya diproses sebagai video.

## 7. Mengganti video yang diputar

1. Buka repository → **Releases** → release `v1` → **Edit**.
2. Hapus aset video lama, unggah video MP4 baru.
   - Nama file harus **persis sama** dengan yang dicari workflow
     (`video-album-01.mp4`), atau ubah juga nama itu di `stream.yml`.
3. Simpan. Run berikutnya otomatis memakai video baru.
   Run yang **sedang berjalan** tetap memakai video yang diunduhnya
   di awal — penggantian berlaku mulai run berikutnya.

## 8. Aturan dan batas yang wajib diingat

- **Satu stream key = satu penerbit.** Jangan menjalankan dua run
  bersamaan; aliran kedua akan ditolak YouTube atau saling menabrak.
- **Satu job GitHub maksimal ±6 jam.** Untuk sesi lebih panjang dari
  itu, sesi harus disambung run baru — pola kita memakai 4 jam, jadi
  satu run selalu cukup dan paling aman.
- **Rekaman tersimpan otomatis hanya bila siaran < 12 jam** dan
  visibilitasnya Publik. Di atas 12 jam rekaman bisa tidak terarsip
  dan jam tayangnya hilang.
- **Stream key = kata sandi siaran.** Siapa pun yang memegangnya bisa
  live di channel. Jangan kirim lewat chat, jangan tampilkan di layar
  saat direkam, jangan tulis di file. Kalau bocor: Reset di Studio,
  lalu ganti nilai Secret `YT_STREAM_KEY` di GitHub dengan kunci baru.
- Karena siarannya berupa **putaran rekaman**, tulis itu dengan jujur
  di deskripsi siaran (mis. "live radio rotation of the recorded album")
  — jangan mengklaim tampil langsung.

## 9. Kalau ada masalah

| Gejala | Penyebab paling mungkin | Yang dilakukan |
|---|---|---|
| Run gagal dalam < 1 menit, log menyebut ffmpeg tidak ditemukan | Langkah *Install ffmpeg* hilang/berubah | Kembalikan file `stream.yml` persis seperti di paket ini |
| Run berjalan lama tapi YouTube tidak pernah LIVE | Secret `YT_STREAM_KEY` kosong/salah, atau broadcast belum dibuat, atau kunci pernah di-Reset di Studio | Periksa Secret di Settings → Secrets and variables → Actions; pastikan broadcast sesi itu ada dan memakai Default stream key |
| Run gagal di langkah unduh video | Nama aset atau tag Release tidak cocok | Pastikan Release bertag `v1` berisi `video-album-01.mp4` |
| Siaran berhenti di tengah sebelum durasi habis | Run GitHub terputus | Jalankan run baru dengan `minutes` = sisa waktu sesi; ia menempel ke broadcast yang sama selama broadcast belum diakhiri manual di Studio |
| Mau memastikan benar-benar live | Halaman watch YouTube tidak selalu bisa jadi bukti dari semua jaringan | Bukti penentu = halaman siaran di **YouTube Studio**: lencana LIVE + penghitung berjalan + kesehatan stream |

## 10. Mengganti stream key di GitHub (bila kunci di-Reset)

1. Salin stream key baru dari YouTube Studio (tombol **Copy** di menu
   Live streaming).
2. Buka repository → **Settings → Secrets and variables → Actions**.
3. Pilih `YT_STREAM_KEY` → **Update** → tempel kunci baru → simpan.
4. Run berikutnya sudah memakai kunci baru.

---

*Paket ini dibuat dari setelan yang sedang berjalan. File `stream.yml`
di dalamnya identik dengan yang terpasang di repository sekarang.*
