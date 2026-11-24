# OH SDI Operation Control — Android Enterprise 2.0

Project Capacitor untuk membangun APK Android offline dari aplikasi monitoring CWM dan DAK/WDM.

## Yang dipertahankan dari workbook sumber

Workbook asli disimpan tanpa diedit di `data/Dashboard_OH_SDI_Monitoring_CWM_DAK-WDM.xlsx`. Sumber memiliki enam sheet:

1. `ROTASI_ITEM_RAWAT`
2. `DATA_BLOK`
3. `REKAP_STATUS`
4. `DASHBOARD_ANALITIK`
5. `LOG_CIRCLE_MANUAL`
6. `LOG_DAK_WDM`

Aplikasi menggunakan master 36 blok dan data log yang berhasil dibaca dari workbook pada companion web app: 1 record CWM dan 25 record DAK/WDM. Workbook penuh tetap disertakan sebagai sumber resmi. Formula dan layout Excel tidak ditulis ulang oleh build APK.

## Fitur project

- Aplikasi Android berbasis Capacitor.
- Penyimpanan SQLite lokal pada Android; browser dev memiliki fallback localStorage.
- Riwayat audit lokal untuk aktivitas baru dan penggabungan backup.
- Input CWM dan DAK/WDM, dashboard, monitoring blok, riwayat dan alert.
- Backup JSON, restore penuh, ekspor CSV.
- Gabungkan backup dari perangkat lain dengan menambahkan aktivitas yang belum ada.
- GitHub Actions untuk menghasilkan APK debug.

## Build APK melalui GitHub (bisa dikelola dari browser HP)

1. Buat repository GitHub baru atau unggah isi folder project ini ke repository yang sudah ada.
2. Pastikan file `.github/workflows/build-android.yml` ikut terunggah.
3. Commit dan push ke branch `main`.
4. Buka tab **Actions** pada repository GitHub.
5. Pilih **Build OH SDI Android APK** lalu tekan **Run workflow**. Workflow juga berjalan saat perubahan terkait di-push ke `main`.
6. Setelah selesai, buka run terbaru dan unduh artifact `OH-SDI-Operation-Control-debug-apk`.
7. Ekstrak ZIP artifact dan instal `app-debug.apk` pada Android. Jika diminta, izinkan instalasi aplikasi dari sumber tersebut.

APK debug cocok untuk pengujian internal. Untuk distribusi resmi, buat APK release yang ditandatangani dengan keystore dan GitHub Secrets; jangan membagikan keystore atau password di repository.

## Pengembangan lokal

Memerlukan Node.js 22, Android Studio, Android SDK dan JDK 21. Project memakai Vite (`www/` sebagai source dan `dist/` sebagai output).

```bash
npm install
npm run build
npx cap add android
npx cap sync android
npx cap open android
```

Build dari terminal:

```bash
cd android
./gradlew assembleDebug
```

Hasil APK berada di `android/app/build/outputs/apk/debug/app-debug.apk`.

## Multi-perangkat saat offline: cara kerja dan batasan

Tidak ada database bersama secara live jika setiap Android sepenuhnya offline dan tidak terhubung ke server/jaringan lokal. Versi ini mendukung **offline multi-device dengan pertukaran backup**:

1. Tiap perangkat menyimpan database SQLite sendiri.
2. Perangkat A membuat backup JSON.
3. Kirim file melalui Bluetooth, Quick Share/Nearby Share, kabel USB atau media penyimpanan.
4. Perangkat B memilih **Gabungkan backup perangkat lain**.
5. Aktivitas baru digabungkan; log lokal yang sudah ada tidak dihapus.

Untuk sinkronisasi langsung beberapa HP dalam satu jaringan Wi-Fi tanpa internet, perlu modul LAN sync tambahan (perangkat host, discovery, autentikasi dan resolusi konflik). Itu bukan bagian dari build awal ini.

## Data, backup dan keamanan

- SQLite memakai database lokal tanpa enkripsi pada build starter ini. Gunakan PIN perangkat dan pengamanan Android; enkripsi database perlu ditambahkan sebelum penggunaan data sensitif.
- Backup JSON juga belum dienkripsi. Simpan dan kirim melalui kanal tepercaya.
- Restore penuh mengganti database lokal. Gunakan Gabungkan Backup untuk menggabungkan log tanpa mengganti data lokal.
- Workbook Excel asli tidak dimodifikasi oleh aplikasi. Ekspor CSV dapat di-append ke sheet log setelah diperiksa.
- Perhitungan ringkas di aplikasi tidak menggantikan seluruh formula/rekap resmi dalam workbook.

## Uji penerimaan sebelum dipakai operasional

- Instal APK di Android dan aktifkan mode pesawat.
- Tutup paksa aplikasi, buka kembali, pastikan data tetap ada.
- Input satu record uji CWM dan satu DAK/WDM; periksa dashboard, log dan alert.
- Buat backup dari HP A, pindahkan ke HP B, gabungkan dan pastikan record masuk satu kali.
- Ulangi import backup yang sama untuk memastikan duplikat tidak ditambahkan.
- Ekspor CSV dan cocokkan tanggal, blok, rotasi, HK serta Ha dengan log.
- Jangan gunakan hasil sebagai laporan resmi sebelum diverifikasi dengan workbook sumber dan data lapangan.
