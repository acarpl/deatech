# Tutorial Lengkap: Detektor Jarak Layar

Panduan step-by-step dari nol sampai pakai model sendiri.

---

## 📋 Daftar Isi

1. [Persiapan](#persiapan)
2. [Jalankan Pertama Kali](#jalankan-pertama-kali)
3. [Pahami Kode](#pahami-kode)
4. [Kustomisasi Dasar](#kustomisasi-dasar)
5. [Buat Model Sendiri (Level 2)](#buat-model-sendiri-level-2)
6. [Troubleshooting](#troubleshooting)

---

## 🔧 Persiapan

### Yang Dibutuhkan

| Software | Versi Minimum | Link Download |
|----------|---------------|---------------|
| Browser Modern | Chrome 80+, Firefox 75+, Edge 80+, Safari 14+ | Sudah terinstall biasanya |
| Python | 3.7+ | [python.org](https://python.org) |
| **ATAU** VS Code | 1.60+ | [code.visualstudio.com](https://code.visualstudio.com) |
| Git (opsional) | 2.30+ | [git-scm.com](https://git-scm.com) |

### Clone / Download Project

**Opsi A: Git (direkomendasikan)**
```bash
git clone https://github.com/acarpl/deatech.git
cd deatech
```

**Opsi B: Download ZIP**
1. Buka https://github.com/acarpl/deatech
2. Klik **Code** → **Download ZIP**
3. Ekstrak ke folder mana saja

---

## ▶️ Jalankan Pertama Kali

### Metode 1: VS Code + Live Server (Paling Mudah)

1. Buka folder project di VS Code: `File` → `Open Folder` → pilih folder `deatech`
2. Install ekstensi **Live Server**:
   - Klik ikon Extensions (Ctrl+Shift+X)
   - Cari "Live Server" by Ritwick Dey
   - Klik **Install**
3. Klik kanan file `index.html` → **Open with Live Server**
4. Browser otomatis buka di `http://localhost:5500`
5. Klik tombol **"▶ Mulai Kamera"** → izinkan akses kamera

### Metode 2: Terminal + Python

```bash
# Masuk ke folder project
cd deatech

# Jalankan server
python -m http.server
# Atau kalau python3:
python3 -m http.server

# Buka browser ke:
# http://localhost:8000
```

### Metode 3: Node.js (http-server)

```bash
# Install sekali saja
npm install -g http-server

# Jalankan
http-server -p 8080
# Buka http://localhost:8080
```

---

## ✅ Verifikasi Berjalan

Setelah browser terbuka:

1. Klik **"▶ Mulai Kamera"**
2. Browser minta izin kamera → klik **Izinkan / Allow**
3. Video webcam muncul di layar
4. Gerakkan wajah ke depan/belakang layar
5. Status berubah:
   - 🔴 **Merah + "⚠️ TERLALU DEKAT"** + suara peringatan (nada rendah)
   - 🟢 **Hijau + "✅ JARAK BAIK"** + suara ting (nada tinggi)
   - ⚪ **Netral** = belum yakin / confidence < 85%

---

## 🧠 Pahami Kode

Buka `index.html` di editor. Kode terbagi jadi 3 bagian:

### 1. HTML + CSS (Baris 1–67)
- Struktur halaman: judul, tombol, area kamera, area status
- Styling: dark mode default, transisi warna smooth

### 2. Konfigurasi (Baris 87–98) — **EDIT DI SINI**
```javascript
const MODEL_URL = "https://teachablemachine.withgoogle.com/models/S75C_fwK3/";
const CLASS_TOO_CLOSE = "Too Close";
const CLASS_GOOD_DISTANCE = "Good Distance";
const CONFIDENCE_THRESHOLD = 0.85;
const COOLDOWN_MS = 1000;
```

### 3. JavaScript Logic (Baris 100–278)
| Fungsi | Deskripsi |
|--------|-----------|
| `initAudio()` | Inisialisasi Web Audio API (wajib user interaction) |
| `playWarningSound()` | Suara "Terlalu Dekat" — sawtooth 220Hz |
| `playTingSound()` | Suara "Jarak Baik" — sine 880Hz |
| `updateUI(status, confidence)` | Update tampilan + cooldown + audio |
| `start()` | Load model + setup webcam + mulai loop |
| `loop()` | RequestAnimationFrame loop |
| `predict()` | Inferensi model + ambil top prediction |

---

## ⚙️ Kustomisasi Dasar

### Ubah Threshold Confidence
```javascript
// Lebih ketat (kurang sensitif): 0.9 - 0.95
// Lebih longgar (lebih sensitif): 0.7 - 0.8
const CONFIDENCE_THRESHOLD = 0.85;
```

### Ubah Cooldown (Anti-Flicker)
```javascript
// Lebih cepat respons: 500ms
// Lebih stabil: 2000ms (2 detik)
const COOLDOWN_MS = 1000;
```

### Ubah Suara
Edit fungsi `playWarningSound()` dan `playTingSound()`:
- `oscillator.type`: `"sine"`, `"square"`, `"sawtooth"`, `"triangle"`
- `oscillator.frequency.value`: Frekuensi Hz (note nada)
- Envelope (gain): Atur attack/sustain/release

### Pakai Model Lokal (Tanpa Internet)
1. Model sudah ada di folder `dummy-bw/`
2. Ganti `MODEL_URL`:
```javascript
const MODEL_URL = "dummy-bw/";  // Harus diakhiri "/"
```

---

## 🎓 Buat Model Sendiri (Level 2)

### Langkah 1: Buka Teachable Machine
1. Buka https://teachablemachine.withgoogle.com
2. Klik **"Get Started"** → **Image Project** → **Standard Image Model**

### Langkah 2: Buat 2 Class
| Class Name (Contoh) | Deskripsi | Tips Rekam |
|---------------------|-----------|------------|
| `Terlalu Dekat` | Wajah sangat dekat ke kamera | Isi frame penuh wajah, jarak ~10-20cm |
| `Jarak Baik` | Wajah jarak normal | Duduk normal ~50-70cm dari layar |

**Tips rekam:**
- Minimal 50-100 sampel per class
- Variasi: posisi kiri/kanan, pencahayaan, ekspresi
- Klik **"Hold to Record"** sambil gerakkan wajah

### Langkah 3: Train Model
1. Klik **"Train Model"** (tunggu selesai)
2. Test di panel **Preview** (kanan)
3. Pastikan akurasi bagus di kedua class

### Langkah 4: Export
1. Klik **"Export Model"**
2. Pilih tab **TensorFlow.js**
3. Pilih **Upload (shareable link)**
4. Klik **Upload** → tunggu selesai
5. **Copy link** yang diberikan (format: `https://teachablemachine.withgoogle.com/models/XXXXXXXX/`)

### Langkah 5: Update Kode
Edit `index.html` bagian konfigurasi:
```javascript
// Ganti dengan link Anda (HARUS diakhiri "/")
const MODEL_URL = "https://teachablemachine.withgoogle.com/models/XXXXXXXX/";

// Ganti sesuai nama class di TM (case-sensitive!)
const CLASS_TOO_CLOSE = "Terlalu Dekat";
const CLASS_GOOD_DISTANCE = "Jarak Baik";
```

### Langkah 6: Test
Refresh browser → klik Mulai Kamera → test deteksi.

---

## 🐛 Troubleshooting

### Kamera tidak muncul / error "NotAllowedError"
- **Penyebab**: Browser blokir kamera / user deny permission
- **Solusi**: 
  - Klik ikon 🔒 di address bar → izinkan kamera
  - Atau: Settings → Privacy → Camera → allow localhost

### Model gagal load / "❌ Gagal memuat model"
- **Penyebab**: `MODEL_URL` salah / model tidak public / network error
- **Solusi**:
  - Pastikan URL diakhiri `/`
  - Cek console (F12) untuk error detail
  - Pastikan model di TM di-set "Public" saat export

### Deteksi tidak akurat / selalu salah
- **Penyebab**: Model belum cukup dilatih / class imbalance / pencahayaan beda
- **Solusi**:
  - Tambah sampel di TM (lebih variatif)
  - Turunkan `CONFIDENCE_THRESHOLD` ke 0.7-0.75
  - Rekam ulang dengan pencahayaan mirip saat pakai

### Suara tidak keluar
- **Penyebab**: Browser policy — AudioContext butuh user interaction
- **Solusi**: Sudah ditangani di kode (`initAudio()` dipanggil saat klik tombol Start)
- **Cek**: Klik tombol Start dulu, baru suara bisa main

### Flickering (status bolak-balik cepat)
- **Solusi**: Naikkan `COOLDOWN_MS` ke 2000 (2 detik) atau lebih

### Error "tf is not defined" / "tmImage is not defined"
- **Penyebab**: CDN gagal load / offline
- **Solusi**: 
  - Pastikan online (butuh internet load library)
  - Atau download library & host lokal

---

## 🎯 Next Steps

- [Deploy ke GitHub Pages](DEPLOYMENT.md#github-pages) — gratis, custom domain
- [Deploy ke Netlify/Vercel](DEPLOYMENT.md#netlify-vercel) — auto HTTPS, CDN
- [Pelajari API Detail](API.md) — modifikasi logika deteksi
- [Buat Model Lebih Lanjut](CUSTOM_MODEL.md) — multi-class, object detection, dll

---

**Butuh bantuan?** Buka [Issues GitHub](https://github.com/acarpl/deatech/issues) atau cek [referensi Morari Studio](https://morari.studio/kelas-c/session-3/build/).