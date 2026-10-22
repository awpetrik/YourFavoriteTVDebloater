# YourFavoriteTVDebloater

Repo ini berisi versi modifikasi dari Free My TV, home screen pengganti untuk Android TV dan Google TV yang bebas iklan dan mematikan aplikasi promosi bawaan TV tanpa uninstall. Modifikasinya dikerjakan untuk TV TCL milik sendiri, dan fokusnya di satu bagian: deretan kartu rekomendasi TMDB di home screen. Penjelasan lengkap soal proyek aslinya ada di bagian [Credit](#credit) di bawah, satu-satunya tempat repo asli disebut di README ini.

Di luar perubahan di bawah, semua perilaku lain sama seperti versi aslinya: daftar aplikasi yang dimatikan, daftar aplikasi yang dilindungi, watchdog yang mengembalikan home screen bawaan kalau app crash berulang, dan tombol Original untuk kembali ke tampilan pabrik kapan saja.

## Yang berubah dari versi aslinya

Versi asli menampilkan 3 kartu rekomendasi per tab (Jelajahi, Film, Serial), dengan judul konten ditulis sebagai teks di bawah kartu. Di versi ini:

- Tiap tab menampilkan 10 kartu dalam satu deretan yang bisa digeser ke samping.
- Kartu yang sedang fokus melebar jadi banner (930x470 px), kartu di sampingnya menyusut jadi poster (313x470 px, rasio 2:3). Perpindahan fokus dianimasikan satu pegas (spring) yang menggerakkan lebar kartu, crossfade poster ke backdrop, dan posisi kamera deretan sekaligus, jadi tidak ada kartu yang melompat.
- Judul teks di bawah kartu dibuang. Sebagai gantinya, logo resmi judul dari TMDB digambar di dalam kartu: besar di pojok banner, kecil di tengah bawah poster. Meta (tahun, rating, genre) hanya tampil di dalam banner. Kalau sebuah judul tidak punya logo, judulnya ditulis sebagai teks di dalam kartu, bukan di bawahnya.
- Di bawah deretan ada indikator segmen dan penghitung posisi (misalnya 4 / 10).
- Background home screen mengikuti kartu yang sedang fokus, berganti setelah fokus berhenti 250 ms.
- Posisi kartu terakhir diingat per tab, jadi pindah tab dan kembali tidak mengulang dari kartu pertama.

Perubahan ini hanya di app Android (`HomeView.java`, `Recs.java`, `Backdrop.java`). Versi web untuk LG webOS dan Samsung Tizen masih memakai tampilan lama.

## Requirements

- TV dengan Google TV atau Android TV, Android 9 atau lebih baru. Teruji di TCL Smart TV Pro (Android 12).
- Komputer di Wi-Fi yang sama dengan TV.
- JDK 17, Android SDK command-line tools (build-tools 34, platform android-34), dan adb. Di Mac:

```bash
brew install --cask temurin android-commandlinetools android-platform-tools
sdkmanager "build-tools;34.0.0" "platforms;android-34"
```

## Install

1. Di TV, aktifkan developer mode: Settings, System, About, tekan *Android TV OS build* 7 kali. Lalu Settings, System, Developer options, nyalakan **USB debugging**. Catat alamat IP TV dari Settings, Network & Internet.
2. Ambil repo ini, lalu build:

```bash
git clone https://github.com/awpetrik/YourFavoriteTVDebloater && cd YourFavoriteTVDebloater
./build.sh
```

Run pertama membuat `config.properties`. Isi kota dan TMDB key kamu di file itu (lihat [Konfigurasi](#konfigurasi)), lalu jalankan `./build.sh` lagi sampai keluar `FreeMyTV.apk`.

3. Install ke TV:

```bash
adb connect 192.168.1.50:5555
adb install -r FreeMyTV.apk
```

Khusus TV TCL, izinkan app berjalan di background (default TCL memblokirnya):

```bash
adb shell "appops set io.freemytv AUTO_START allow || appops set io.freemytv APP_AUTO_START allow"
```

4. Di TV, buka *Free My TV*, lalu *Home Screen*, lalu *Free My TV*, dan terima prompt debugging (centang *Always allow*).

Untuk LG webOS dan Samsung Tizen, build web-nya ada di folder `webos/` dan `tizen/` (`python3 webos/build.py`, `python3 tizen/build.py`) dan memakai `config.properties` yang sama. Tampilannya masih versi lama seperti dijelaskan di atas.

## Konfigurasi

Nilai awal datang dari `config.properties`, dibuat oleh build pertama. Sebagian besar bisa diubah lagi dari TV lewat *Free My TV*, *Home Settings*, dan pilihan di TV yang menang.

| Key | Fungsi | Default |
|---|---|---|
| `tmdb_key` | Key TMDB gratis, untuk rekomendasi | kosong |
| `tmdb_region` | Kode negara dua huruf: judul ini ada di aplikasi apa | `US` |
| `watchmode_key` | Opsional, key Watchmode gratis: kartu membuka halaman judulnya di aplikasi streaming | kosong |
| `latitude`, `longitude` | Kota kamu, untuk cuaca dan jam matahari terbenam | New York |
| `city` | Nama yang tampil di kartu cuaca | `Weather` |
| `hide_animation` | Sembunyikan kartun dan anime dari rekomendasi | `false` |
| `language` | `auto` (ikut TV) atau kode bahasa yang tersedia | `auto` |
| `show_crypto` | Tampilkan harga kripto Binance di home dan screensaver | `true` |
| `currency` | Mata uang harga kripto (`USD`, `EUR`, `IDR`, ...) | `USD` |
| `temperature` | `C`, `F`, atau `auto` | `auto` |
| `disable_assistant` | Ikut matikan Google Assistant | `false` |

TMDB key gratis: daftar di [themoviedb.org](https://www.themoviedb.org/signup), lalu buka Settings, API, dan minta developer key. Tempel *API Read Access Token* (atau *API Key*) setelah `tmdb_key=`. File `config.properties` tidak masuk git, jadi key kamu tetap di komputer kamu.

`freemytv.keystore` juga tidak masuk git. File itu dibuat di komputer kamu dan dipakai menandatangani APK, jadi jangan dihapus: APK update hanya bisa ditimpa kalau tanda tangannya sama.

## Kalau mau balik

Di TV: *Free My TV*, *Home Screen*, *Original*. Home screen pabrik, screensaver, dan semua aplikasi kembali seperti semula, karena tidak ada yang di-uninstall. Kalau app baru ini crash tiga kali dalam lima menit, watchdog mengembalikan home screen bawaan sendiri. Mau hapus total: pindah ke Original dulu, baru uninstall *Free My TV* dari Settings, Apps.

## Status dan batasan

- Ini build personal, bukan rilis publik. Tidak ada APK jadi yang dibagikan, semua orang build sendiri dari source.
- Baru teruji penuh di satu TV (TCL Smart TV Pro, Android 12). Brand lain didukung kode aslinya, tapi perubahan deretan kartunya belum dicoba di TV lain.
- Versi webOS dan Tizen belum diselaraskan dengan deretan kartu baru.
- Fallback judul teks untuk konten tanpa logo TMDB belum pernah terlihat di TV sendiri, karena semua judul yang muncul sejauh ini punya logo. Kodenya ada, tampilannya belum terverifikasi.

## Credit

Proyek asli: [Free My TV](https://github.com/simbasjungles/free-my-tv) oleh simbasjungles. Repo ini adalah modifikasi dari proyek tersebut, dengan perubahan seperti dijelaskan di awal README. Terima kasih untuk pembuat dan kontributor aslinya.

Credit lain yang berlaku juga untuk versi ini:

- Versi LG dibangun di atas temuan [@danielthejoker18](https://github.com/danielthejoker18) di [FreeMyTvWebOS](https://github.com/danielthejoker18/FreeMyTvWebOS) (MIT): install lewat Developer Mode dan membuka aplikasi lewat service bus TV.
- Install di Samsung memakai [TizenBrew Installer](https://github.com/reisxd/TizenBrewInstaller) oleh [@reisxd](https://github.com/reisxd).
- Produk ini memakai TMDB API tapi tidak didukung atau disertifikasi oleh [TMDB](https://www.themoviedb.org/).
- Link judul ke aplikasi streaming oleh [Watchmode](https://api.watchmode.com/). Data cuaca oleh [Open-Meteo.com](https://open-meteo.com/) (CC BY 4.0). Harga dari API publik [Binance](https://www.binance.com/).
- Ikon: [Material Symbols](https://fonts.google.com/icons) (Apache 2.0). Font: [Inter](https://rsms.me/inter/) (SIL OFL 1.1). Logo koin: [cryptocurrency-icons](https://github.com/spothq/cryptocurrency-icons) (CC0). Logo aplikasi Samsung: [Simple Icons](https://simpleicons.org) (CC0).
- Tidak berafiliasi dengan Google, TCL, LG, Samsung, atau layanan streaming mana pun. Nama dan logo aplikasi milik pemiliknya masing-masing.

## License

MIT. Lihat [LICENSE](LICENSE).
