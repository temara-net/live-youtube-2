# Live Streaming YouTube lewat GitHub — Channel Kedua

Mesin live streaming yang sama dengan yang sudah terbukti berjalan:
server GitHub menjalankan ffmpeg, memutar satu video berulang-ulang,
dan mendorongnya ke siaran live YouTube. Tidak ada komputer yang
harus menyala di rumah.

File mesinnya: `.github/workflows/stream.yml` — panduan lengkapnya
ada di `PANDUAN.md`.

## Yang harus diisi sendiri oleh pemilik (belum terisi)

1. **Stream key channel** — di repository ini:
   Settings → Secrets and variables → Actions → New repository secret
   - Name: `YT_STREAM_KEY`
   - Secret: stream key dari YouTube Studio channel ini
     (menu Live streaming → tombol Copy). Jangan bagikan ke siapa pun.
2. **Video yang diputar** — di repository ini:
   Releases → Draft a new release → tag `v1` → lampirkan file video
   MP4 dengan nama persis `video-album-01.mp4` → Publish release.
3. **Siaran di YouTube Studio channel ini** — setiap sesi butuh satu
   broadcast baru (judul, deskripsi, Publik, kategori, Default stream
   key, jangan tekan Reset).

Sesudah 1–3 terisi: tab **Actions** → workflow **stream** →
**Run workflow** → isi `minutes` (durasi sesi dalam menit,
4 jam = 240) → Run. Siaran menjadi LIVE dalam ±1–2 menit,
berhenti sendiri saat durasi habis, dan rekamannya tersimpan
otomatis sebagai video di channel.

Syarat channel: akses live streaming YouTube harus sudah aktif
di channel ini (verifikasi nomor HP + masa tunggu 24 jam sesudah
pengaktifan pertama, sesuai aturan YouTube).
