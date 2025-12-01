# LAPORAN PROYEK VTUBER TRACKING APPLICATION

## Pendahuluan

Proyek ini merupakan aplikasi pelacakan gerakan real-time untuk model VTuber menggunakan OpenCV dan Mediapipe, juga teknologi computer vision dan Live2D. Aplikasi mampu menangkap gerakan wajah, tubuh, tangan, dan kaki pengguna melalui webcam, kemudian mentranslasikan gerakan tersebut ke model Live2D secara real-time.

## Tujuan Proyek

Mengembangkan sistem tracking komprehensif yang dapat mengendalikan avatar melalui gerakan tubuh pengguna dengan akurasi tinggi dan performa yang stabil. Sistem dirancang untuk memberikan pengalaman interaktif dalam aplikasi streaming atau content creation.

## Teknologi yang Digunakan

### Library Utama

1. OpenCV - Untuk pengambilan dan pemrosesan frame dari webcam
2. MediaPipe - Framework machine learning untuk deteksi landmark wajah, pose tubuh, dan tangan
3. Live2D SDK v3 - Rendering dan kontrol model avatar 2D
4. Pygame - Manajemen window dan event handling
5. PyOpenGL - Rendering grafis dan texture background
6. NumPy - Operasi matematis dan array processing

### Komponen MediaPipe

- Face Mesh: Deteksi 468 landmark pada wajah dengan refinement untuk iris
- Pose Detection: Tracking 33 titik landmark pada tubuh
- Hand Tracking: Deteksi 21 landmark per tangan dengan dukungan maksimal 2 tangan

## Fitur Utama

### Tracking Kepala

Sistem menghitung rotasi kepala dalam tiga sumbu menggunakan algoritma Perspective-n-Point:

- Yaw: Rotasi kiri-kanan
- Pitch: Rotasi atas-bawah  
- Roll: Kemiringan kepala

Implementasi menggunakan 6 titik landmark kunci pada wajah yang dipetakan ke model 3D untuk menghitung matriks rotasi. Hasil rotasi dikalibrasi dalam rentang -30 hingga 30 derajat untuk mencegah gerakan ekstrem.

### Tracking Mata

Deteksi kedipan mata menggunakan metrik Eye Aspect Ratio yang menghitung rasio antara jarak vertikal dan horizontal kelopak mata. Sistem juga melacak posisi bola mata dengan mendeteksi iris dan membandingkannya dengan sudut mata untuk mendapatkan arah pandangan.

Kalibrasi EAR:
- Nilai maksimal: 0.37 (mata terbuka penuh)
- Nilai minimal: 0.15 (mata tertutup)

### Tracking Mulut

Apertura mulut diukur berdasarkan jarak vertikal antara bibir atas dan bawah, kemudian dinormalisasi untuk parameter animasi model.

### Tracking Tubuh

Sistem mendeteksi orientasi tubuh berdasarkan posisi bahu:

- Body Tilt: Kemiringan horizontal tubuh
- Body Roll: Rotasi tubuh
- Body Pitch: Gerakan maju-mundur

Untuk mode dinamis, sistem juga melacak posisi 2D model berdasarkan pergerakan bahu pengguna dan mengimplementasikan fitur zoom otomatis berdasarkan kedalaman subjek dari kamera.

### Tracking Lengan

Posisi tangan dipetakan ke dua parameter per lengan:

- Shoulder parameter: Gerakan naik-turun tangan
- Elbow parameter: Gerakan kiri-kanan tangan

Mapping menggunakan interpolasi linear dari koordinat normalized hand landmark.

### Tracking Kaki

Sistem melacak posisi lutut untuk menggerakkan parameter kaki model. Gerakan vertikal lutut dikonversi menjadi animasi kaki dengan rentang -10 hingga 10.

## Sistem Smoothing

Seluruh parameter tracking menggunakan exponential moving average untuk menghaluskan gerakan dan mengurangi jitter:

Formula: nilai_baru = alpha x nilai_raw + (1 - alpha) x nilai_lama

Nilai alpha berbeda untuk setiap parameter:
- Parameter kepala: 0.4
- Parameter mata: 0.6 (lebih responsif)
- Parameter bola mata: 0.2 (lebih halus)
- Parameter tubuh: 0.2-0.3
- Parameter lengan: 0.6
- Parameter kaki: 0.4

## Background Rendering

Aplikasi menggunakan background statis yang dimuat dari file gambar. Gambar diproses dan dikonversi menjadi texture OpenGL yang dirender sebagai layer belakang model Live2D.

Proses:
1. Load gambar menggunakan OpenCV
2. Resize ke resolusi display (640x480)
3. Konversi color space BGR ke RGB
4. Flip vertikal untuk koordinat OpenGL
5. Upload ke GPU sebagai texture 2D

## Mode Operasi

### Mode Statis

Model tetap di posisi tengah dengan skala default. Hanya orientasi tubuh yang berubah tanpa pergerakan posisi 2D atau zoom.

### Mode Dinamis

Model dapat bergerak mengikuti pergerakan tubuh pengguna dengan fitur:
- Translasi horizontal dan vertikal
- Auto-zoom berdasarkan jarak dari kamera
- Offset vertikal adaptif sesuai skala

Toggle antara mode dilakukan dengan menekan tombol S.

## Kalibrasi Parameter

### Hand Tracking
- Y minimum (tangan atas): 0.1
- Y maksimum (tangan bawah): 0.8
- X minimum (tangan kiri): 0.2
- X maksimum (tangan kanan): 0.8

### Knee Tracking
- Y minimum: 0.5
- Y maksimum: 0.9

### Zoom System
- Default shoulder depth: -0.5
- Zoom sensitivity: 4.0
- Scale range: 0.8 - 3.0

## Struktur Program

### Inisialisasi

Program dimulai dengan setup MediaPipe solutions, inisialisasi pygame window dengan OpenGL context, loading model Live2D, dan pembuatan smoother objects untuk semua parameter.

### Main Loop

Alur eksekusi per frame:

1. Capture frame dari webcam
2. Flip horizontal untuk efek mirror
3. Proses frame dengan MediaPipe (face, pose, hands)
4. Reset parameter ke nilai default
5. Ekstraksi data dari detection results
6. Hitung nilai parameter dengan smoothing
7. Update parameter Live2D model
8. Render background texture
9. Update dan draw Live2D model
10. Display ke window
11. Handle keyboard events

### Error Handling

Sistem mengimplementasikan fault handler dan pengecekan availability untuk setiap landmark yang digunakan, mencegah crash akibat detection failure.

## Performa

Target framerate: 30 FPS
Resolusi display: 640 x 480 pixels
MediaPipe model complexity: 1 (balanced)
Confidence threshold: 0.5 untuk detection dan tracking

## Debugging

Program mencetak informasi real-time ke console setiap frame:
- Status mode (Static/Dynamic)
- Nilai parameter kepala
- Nilai parameter tubuh
- Nilai parameter lengan
- Nilai parameter kaki
- Skala model

## Limitasi

1. Memerlukan pencahayaan yang baik untuk tracking optimal
2. Tracking tangan bergantung pada visibility penuh
3. Background harus kontras dengan subjek untuk pose detection yang akurat
4. Memerlukan GPU untuk performa optimal pada rendering
5. Model Live2D harus memiliki parameter ID yang sesuai

## Pengembangan Masa Depan

Potensi peningkatan sistem:

- Implementasi multi-model support
- Recording dan playback motion data
- Kalibrasi otomatis per user
- Support untuk full body tracking dengan sensor tambahan
- Integration dengan software streaming
- Optimisasi performa untuk low-end hardware
- Machine learning untuk prediksi gerakan
- Support untuk ekspresi wajah yang lebih kompleks

## Kesimpulan

Aplikasi berhasil mengintegrasikan multiple tracking systems untuk menciptakan pengalaman VTuber yang responsif dan natural. Kombinasi antara computer vision state-of-the-art dan rendering real-time menghasilkan sistem yang dapat digunakan untuk streaming atau content creation profesional. Sistem smoothing yang comprehensive memastikan gerakan model tetap halus dan realistis meskipun terdapat noise pada detection.
