# DLH Mobile

Frontend mobile untuk penelitian **Aplikasi Mobile Crowdsourcing Pelaporan Penumpukan Sampah dengan Optimalisasi Rute Armada Menggunakan Algoritma K-Means Clustering**.

Dibangun dengan Flutter + Dart, Material 3, Riverpod, go_router, dan Dio.

> **Status pengujian terakhir:** `flutter analyze` → *No issues found!* dan
> `flutter test` → **44/44 tes lulus**. Rincian apa yang sudah terhubung API,
> masih memakai data contoh, atau belum selesai ada di bagian
> [Status integrasi backend](#status-integrasi-backend).

## Peran pengguna

| Role | Nilai `role` | Beranda |
|---|---|---|
| Masyarakat | `masyarakat` | `/citizen` |
| Admin / Koordinator DLH | `admin` | `/admin` |
| Petugas Armada | `petugas` | `/officer` |

Algoritma K-Means Clustering dan Nearest Neighbor **tidak** dihitung di aplikasi ini; aplikasi hanya menampilkan hasil dari backend.

## Menjalankan aplikasi

```bash
cd mobile_app
flutter pub get
flutter run
```

### Mode demo (default)

Aplikasi berjalan tanpa backend; data berasal dari repository lokal (`Demo*Repository`).
Gunakan email berikut dengan kata sandi bebas minimal 6 karakter:

| Email | Role yang didapat |
|---|---|
| `admin@dlh.id` | Admin / Koordinator DLH |
| `petugas@dlh.id` | Petugas Armada |
| `warga@dlh.id` | Masyarakat |

### Menyambung ke backend sungguhan

```bash
flutter run \
  --dart-define=DEMO_MODE=false \
  --dart-define=APP_ENV=production \
  --dart-define=API_BASE_URL=http://10.0.2.2:8000/api
```

`10.0.2.2` adalah localhost komputer host dari sudut pandang emulator Android.
Untuk perangkat fisik, gunakan IP LAN komputer, mis. `http://192.168.1.10:8000/api`.

Path endpoint terpusat di `lib/core/network/api_endpoints.dart` dan masih berupa
usulan — sesuaikan dengan kontrak API backend.

Trafik HTTP polos (cleartext) hanya dibuka pada build debug
(`android/app/src/debug/AndroidManifest.xml`) supaya backend pengembangan di
`http://10.0.2.2:8000/api` bisa diakses. Build release tetap mewajibkan HTTPS.

## Status integrasi backend

Tidak ada server backend di dalam repositori ini, jadi **belum ada pengujian
end-to-end terhadap server sungguhan**. Kolom status di bawah membedakan tiga
keadaan:

- **Terhubung API** — kode klien HTTP lengkap dan sudah diuji terhadap kontrak
  (path, metode, header, isi permintaan, pembacaan respons) memakai adaptor HTTP
  tiruan di `test/api_contract_test.dart`. Masih menunggu server nyata untuk
  verifikasi ujung-ke-ujung.
- **Mode demo** — memakai data contoh lokal (`Demo*Repository`); belum ada
  pemanggilan API.
- **Belum selesai** — belum tersedia atau masih menunggu kontrak backend.

### Sudah terhubung API

Seluruh pemakaian di bawah ini otomatis berpindah ke implementasi API saat
`DEMO_MODE=false`.

| Alur | Metode repository | Endpoint (usulan) | Uji kontrak |
|---|---|---|---|
| Masuk | `ApiAuthRepository.login` | `POST /auth/login` | ✔ |
| Daftar (khusus masyarakat) | `ApiAuthRepository.register` | `POST /auth/register` | ✔ |
| Pulihkan sesi / profil | `ApiAuthRepository.restoreSession` | `GET /auth/me` | ✔ |
| Kirim laporan (multipart) | `ApiReportRepository.createReport` | `POST /reports` | ✔ |
| Riwayat laporan saya | `ApiReportRepository.fetchMyReports` | `GET /reports?mine=true` | ✔ |
| Detail laporan | `ApiReportRepository.fetchReportDetail` | `GET /reports/{id}` | ✔ |
| Daftar laporan masuk (admin) | `ApiReportRepository.fetchIncomingReports` | `GET /reports/incoming` | — |
| Verifikasi laporan (admin) | `ApiReportRepository.reviewReport` | `POST /reports/{id}/review` | — |
| Titik laporan untuk peta | `ApiGisRepository.fetchReportPoints` | `GET /reports/points` | — |
| Hasil K-Means (tampil saja) | `ApiClusteringRepository.fetchClusterResults` | `GET /clustering/results` | — |
| Daftar armada (admin) | `ApiAdminRepository.fetchFleet` | `GET /fleet` | — |
| Terbitkan penugasan (admin) | `ApiAdminRepository.publishAssignments` | `POST /assignments/publish` | — |
| Daftar penugasan petugas | `ApiRoutingRepository.fetchAssignments` | `GET /assignments?mine=true` | — |
| Detail penugasan + geometri jalan | `ApiRoutingRepository.fetchAssignmentDetail` | `GET /assignments/{id}` | ✔ |
| Status satu titik kunjungan | `ApiRoutingRepository.updateStopStatus` | `PATCH /assignments/{id}/stops/{reportId}` | ✔ |
| Unggah foto bukti (multipart) | `ApiRoutingRepository.uploadStopProof` | `POST /assignments/{id}/stops/{reportId}/proof` | ✔ |
| Daftar pemberitahuan | `ApiNotificationRepository.fetchNotifications` | `GET /notifications` | — |
| Tandai semua dibaca | `ApiNotificationRepository.markAllRead` | `POST /notifications/read-all` | ✔ |

### Memakai perangkat, bukan mock

- **GPS**: `lib/features/reports/data/location_service.dart` memakai `geolocator`
  sungguhan (izin, timeout 15 detik, penanganan izin ditolak permanen) pada kedua
  mode. Saat izin ditolak, pengguna dapat memilih titik secara manual lewat peta
  pemilih lokasi.
- **Kamera & galeri**: `image_picker` sungguhan lewat `PhotoPickerService`.
- **Navigasi ke peta eksternal**: `url_launcher` (`NavigationLauncher`).

### Yang belum selesai

| Bagian | Keterangan |
|---|---|
| Verifikasi kontrak API | Seluruh path di `api_endpoints.dart` masih **usulan**. Wajib diselaraskan dengan backend sebelum mode non-demo dipakai; kesalahan bentuk respons hanya akan terlihat saat backend nyata tersedia. |
| Pengujian end-to-end | Belum ada. Uji yang ada memakai adaptor HTTP tiruan, bukan server. |
| Tandai dibaca per notifikasi | Belum ada `markAsRead(id)`; yang tersedia hanya "tandai semua dibaca". Menunggu kontrak endpoint dari backend. |
| Pusat notifikasi untuk admin/petugas | Rute `/citizen/notifikasi` hanya terdaftar untuk masyarakat; belum ada halaman notifikasi untuk admin/petugas. |
| Algoritma K-Means & Nearest Neighbor | Dijalankan di backend dan **tidak** dihitung di aplikasi ini. Aplikasi hanya menampilkan hasil. |

## Pengujian

```bash
flutter analyze
flutter test
```

Hasil terakhir: `flutter analyze` bersih, `flutter test` lulus 44/44. Berkas uji:

| Berkas | Cakupan |
|---|---|
| `test/app_flow_test.dart` | Splash, masuk, daftar, pemulihan sesi, penjagaan rute antar-role |
| `test/admin_flow_test.dart` | Navigasi seluruh halaman admin, detail cluster, alur verifikasi |
| `test/citizen_report_flow_test.dart` | Form pelaporan, izin lokasi, pemilih titik manual, riwayat, detail, notifikasi |
| `test/officer_route_flow_test.dart` | Rute GIS, status titik kunjungan, unggah foto bukti, navigasi eksternal |
| `test/api_contract_test.dart` | Kontrak klien HTTP: bentuk respons, autentikasi, penerjemahan error, multipart |

## Struktur folder `lib`

```
lib/
├── main.dart
├── app/            # konfigurasi aplikasi, router, tema
├── core/           # config, constants, networking, error, storage, utils
├── shared/         # widget yang dipakai ulang antar fitur
└── features/
    ├── auth/       # model, data, state, screen, widget
    ├── citizen/    # dashboard masyarakat
    ├── admin/      # dashboard admin DLH
    ├── officer/    # dashboard petugas armada
    ├── reports/    # laporan penumpukan sampah
    ├── gis/        # data spasial & peta
    ├── clustering/ # tampilan hasil K-Means
    ├── routing/    # penugasan & rute armada
    └── notifications/ # pusat pemberitahuan
```

Setiap fitur memisahkan `model/`, `data/`, `state/`, `screen/`, dan `widget/`.

## Mengganti sumber data

Semua akses data melalui kontrak repository, sehingga implementasi API dan demo
dapat ditukar di satu tempat saja (`features/*/state/*_providers.dart`):

| Fitur | Kontrak | Implementasi API | Implementasi demo |
|---|---|---|---|
| Auth | `auth_repository.dart` | `api_auth_repository.dart` | `demo_auth_repository.dart` |
| Laporan | `report_repository.dart` | `api_report_repository.dart` | `demo_report_repository.dart` |
| GIS | `gis_repository.dart` | `api_gis_repository.dart` | `demo_gis_repository.dart` |
| Clustering | `clustering_repository.dart` | `api_clustering_repository.dart` | `demo_clustering_repository.dart` |
| Routing | `routing_repository.dart` | `api_routing_repository.dart` | `demo_routing_repository.dart` |
| Admin (armada & penugasan) | `admin_repository.dart` | `api_admin_repository.dart` | `demo_admin_repository.dart` |
| Notifikasi | `notification_repository.dart` | `api_notification_repository.dart` | `demo_notification_repository.dart` |

Akses HTTP terpusat pada `core/network/api_client.dart` (base URL, timeout,
penyisipan token, dan penerjemahan error Dio menjadi `AppException`).

## Catatan peta dan rute

- Garis rute **hanya** digambar bila backend mengirim geometri jalan
  (`route_geometry` dengan minimal 2 verteks). Bila tidak ada, aplikasi
  menampilkan peringatan dan tidak menggambar apa pun.
- `AppMapPreview` (peta ringkas berbasis `CustomPaint`) hanya menggambar penanda
  titik, tanpa garis penghubung, supaya garis lurus antar titik tidak pernah
  disalahartikan sebagai jalur kendaraan.
- Urutan kunjungan dan nomor penanda berasal dari hasil Nearest Neighbor
  backend, bukan dihitung di aplikasi.

## Prasyarat Android

```bash
flutter doctor --android-licenses      # terima lisensi Android SDK
flutter emulators                       # daftar AVD
flutter emulators --launch <id_avd>     # jalankan emulator bila tersedia
flutter devices                         # daftar perangkat/emulator aktif
```
