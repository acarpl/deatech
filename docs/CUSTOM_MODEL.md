# Panduan Membuat Model Kustom (Teachable Machine)

Cara membuat, melatih, dan mengintegrasikan model AI sendiri tanpa coding.

---

## 🎯 Kenapa Teachable Machine?

| Keuntungan | Detail |
|------------|--------|
| **No-code** | UI visual, drag & drop, cocok non-programmer |
| **Gratis** | Tanpa biaya, host model gratis di Google Cloud |
| **Cepat** | Train < 1 menit untuk ~100 sampel |
| **Export TF.js** | Langsung pakai di browser, no server needed |
| **Iteratif** | Bisa tambah sampel → retrain → export ulang kapan saja |

---

## 📋 Persiapan

### Hardware
- **Webcam** (built-in laptop / external USB)
- **Pencahayaan cukup** (hindari backlight / terlalu gelap)
- **Background bersih** (kurangi noise visual)

### Software
- Browser modern (Chrome direkomendasikan)
- Akun Google (untuk simpan project di Teachable Machine)

---

## 🚀 Langkah 1: Buat Project Baru

1. Buka https://teachablemachine.withgoogle.com
2. Klik **"Get Started"**
3. Pilih **"Image Project"** → **"Standard Image Model"**
4. (Opsional) Sign in Google untuk auto-save ke Drive

---

## 🏷️ Langkah 2: Desain Class (Label)

### Untuk Detektor Jarak (2 Class Minimum)

| Class Name | Deskripsi | Jumlah Sampel Minimum |
|------------|-----------|----------------------|
| `Terlalu Dekat` | Wajah mengisi >80% frame, jarak ~10-20cm | 50-100 |
| `Jarak Baik` | Wajah proporsional, jarak ~50-70cm | 50-100 |

### Tips Naming Class
- **Case-sensitive** — harus persis sama di kode
- **Deskriptif** — `"Terlalu Dekat"` lebih baik dari `"Class 1"`
- **Hindari karakter khusus** — pakai huruf, angka, spasi saja

### Variasi Class Lain (Ideas)
| Use Case | Class Names |
|----------|-------------|
| Posture | `"Tegang"`, `"Santai"`, `"Miring"` |
| Activity | `"Membaca"`, `"Mengetik"`, `"Istirahat"` |
| Presence | `"Ada Orang"`, `"Tidak Ada"` |
| Multi-distance | `"Sangat Dekat"`, `"Dekat"`, `"Normal"`, `"Jauh"` |

---

## 📸 Langkah 3: Rekam Sampel (Data Collection)

### Best Practices

| Tips | Mengapa |
|------|---------|
| **Min 50-100 sampel/class** | Model butuh variasi untuk generalisasi |
| **Variasi pencahayaan** | Siang, malam, lampu kuning, lampu putih |
| **Variasi posisi** | Kiri, kanan, tengah, dekat, jauh |
| **Variasi ekspresi** | Senyum, datar, kaget, ngantuk |
| **Variasi background** | Dinding polos, rak buku, jendela |
| **Jangan terlalu seragam** | Overfitting ke kondisi spesifik |

### Cara Rekam di TM

1. Klik class `"Terlalu Dekat"`
2. Posisikan wajah **sangat dekat** ke kamera (isi frame)
3. Klik **"Hold to Record"** → gerakkan sedikit wajah (kiri/kanan/atas/bawah)
4. Lepas → otomatis capture ~100+ images
5. Ulangi untuk class `"Jarak Baik"` — duduk normal

### Keyboard Shortcut (Lebih Cepat)
- **Space** = Hold to Record
- **Release Space** = Stop

---

## 🏋️ Langkah 4: Training

1. Pastikan 2 class punya sampel (indikator hijau ✓)
2. Klik **"Train Model"** (tombol biru kanan)
3. Tunggu progress bar selesai (10-60 detik)
4. **Preview panel** (kanan) akan live-test kamera

### Evaluasi Hasil Training

| Indikator | Baik | Perlu Perbaikan |
|-----------|------|-----------------|
| **Preview akurat** | Deteksi benar saat test | Sering salah klasifikasi |
| **Confidence tinggi** | >0.9 di kondisi ideal | <0.7 atau fluktuatif |
| **Transition smooth** | Mulus saat gerak | Flickering / loncat-loncat |

### Jika Hasil Buruk
1. **Tambah sampel** di class yang lemah
2. **Hapus sampel outlier** (klik thumbnail → delete)
3. **Balance class** — jumlah sampel kira-kira sama
4. **Retrain** (klik Train Model lagi)

---

## 📤 Langkah 5: Export Model

### Opsi A: Upload ke Cloud (Recommended - Mudah)

1. Klik **"Export Model"**
2. Tab **"TensorFlow.js"**
3. Pilih **"Upload (shareable link)"**
4. Klik **"Upload my model"**
5. Tunggu upload selesai
6. **Copy link** yang muncul:
   ```
   https://teachablemachine.withgoogle.com/models/XXXXXXXXXX/
   ```
   ⚠️ **WAJIB diakhiri `/`**

### Opsi B: Download (Untuk Hosting Sendiri / Offline)

1. Tab **"TensorFlow.js"** → **"Download"**
2. Ekstrak ZIP → dapat folder:
   ```
   my-model/
   ├── model.json
   ├── metadata.json
   └── weights.bin
   ```
3. Host di server kamu / taruh di folder project (`dummy-bw/`)

---

## 🔗 Langkah 6: Integrasi ke Kode

### Pakai Model Cloud (Default)

Edit `index.html` bagian konfigurasi:
```javascript
// Ganti dengan link Anda (HARUS diakhiri "/")
const MODEL_URL = "https://teachablemachine.withgoogle.com/models/XXXXXXXXXX/";

// Ganti sesuai nama class di TM (case-sensitive!)
const CLASS_TOO_CLOSE = "Terlalu Dekat";
const CLASS_GOOD_DISTANCE = "Jarak Baik";
```

### Pakai Model Lokal (Offline)

1. Copy folder model ke project:
   ```
   deatech/
   ├── index.html
   ├── my-model/          ← folder hasil download
   │   ├── model.json
   │   ├── metadata.json
   │   └── weights.bin
   └── ...
   ```
2. Edit config:
```javascript
const MODEL_URL = "my-model/";  // Relative path, diakhiri "/"
```

---

## 🧪 Langkah 7: Test & Iterasi

1. Refresh browser → test deteksi
2. Amati:
   - Apakah class name di label match? (`label.textContent`)
   - Confidence berapa? (target >0.85)
   - Apakah flickering?
3. **Iterasi**: Balik ke TM → tambah sampel → retrain → export ulang → replace link

---

## 🎓 Advanced: Tips Pro

### 1. Data Augmentation (Manual)
Rekam dengan variasi sengaja:
- Pakai kacamata / tidak
- Rambut terikat / terbuka
- Masker / tidak
- Berbagai jarak antar 2 class

### 2. Hard Negative Mining
Tambah class `"Bukan Wajah"` / `"Background"`:
- Rekam kamera kosong, tangan, objek acak
- Bantu model belajar "bukan wajah" → kurangi false positive

### 3. Multi-Class Distance (4 Class)
| Class | Jarak | Use Case |
|-------|-------|----------|
| `Sangat Dekat` | <20cm | Warning kritis |
| `Dekat` | 20-40cm | Warning ringan |
| `Normal` | 40-70cm | Ideal |
| `Jauh` | >70cm | Info saja |

Update kode: tambah `CLASS_*` baru, branch di `updateUI()`, CSS baru.

### 4. Versioning Model
Setiap export dapat URL baru. Simpan log:
| Versi | Tanggal | Class | Sampel | Akurasi Preview | Link |
|-------|---------|-------|--------|-----------------|------|
| v1.0 | 2026-09-28 | 2 | 80/80 | 95% | `.../models/ABC123/` |
| v1.1 | 2026-09-29 | 2 | 120/120 | 98% | `.../models/DEF456/` |

---

## 🐛 Troubleshooting Model

### "Model tidak load / 404"
- Link salah copy? Harus diakhiri `/`
- Model di TM di-set **Private**? Harus **Public**
- Coba buka link di browser → harus download `model.json`

### "Class name mismatch"
- Error: `label` nampilin nama class tapi UI tidak berubah
- Cek: `CLASS_TOO_CLOSE` di kode **persis sama** dengan TM (spasi, kapitalisasi)
- Debug: `console.log(top.className)` di `predict()`

### Confidence selalu rendah (<0.5)
- Sampel terlalu sedikit / terlalu seragam
- Pencahayaan training ≠ pencahayaan pakai
- Tambah sampel di kondisi real-world

### Flickering parah
- Naikkan `CONFIDENCE_THRESHOLD` ke 0.9
- Naikkan `COOLDOWN_MS` ke 2000
- Tambah sampel transisi (antara dekat & jauh)

### Model bias ke satu class
- Jumlah sampel tidak balance
- Hapus sampel class dominan / tambah class lemah
- Gunakan **"Underrepresented class"** warning di TM (jika ada)

---

## 📊 Evaluasi Kuantitatif (Opsional)

Untuk evaluasi lebih objektif, gunakan **Confusion Matrix**:

1. Siapkan test set terpisah (20% data, tidak dipakai training)
2. Di TM: klik **"Advanced"** → **"Show confusion matrix"** (setelah train)
3. Target:
   - Diagonal > 90%
   - Off-diagonal < 10%

Atau manual di browser:
```javascript
// Tambah di console untuk logging
console.log(`True: ${trueLabel}, Pred: ${top.className}, Conf: ${top.probability}`);
```

---

## 🔄 Continuous Improvement Workflow

```
┌─────────────┐
│  Collect    │  Rekam sampel baru di kondisi edge-case
│  New Data   │
└──────┬──────┘
       ▼
┌─────────────┐
│  Retrain    │  Klik "Train Model" di TM
│  Model      │
└──────┬──────┘
       ▼
┌─────────────┐
│  Evaluate   │  Test di Preview + Real browser
│  Results    │
└──────┬──────┘
       ▼
┌─────────────┐
│  Export &   │  Upload → copy link → update MODEL_URL
│  Deploy     │
└──────┬──────┘
       ▼
┌─────────────┐
│  Monitor    │  Catat false pos/neg → loop ke Collect
│  Production │
└─────────────┘
```

---

## 📁 Struktur File Model (Reference)

### `metadata.json`
```json
{
  "tfjsVersion": "1.3.1",
  "tmVersion": "2.4.7",
  "packageVersion": "0.8.4",
  "packageName": "@teachablemachine/image",
  "timeStamp": "2026-09-28T00:00:00.000Z",
  "modelName": "my-distance-model",
  "labels": ["Terlalu Dekat", "Jarak Baik"],
  "imageSize": 224
}
```
- `labels` array → urutan class = urutan output `predictions`
- `imageSize` → input resolution (224x224 default MobileNet)

### `model.json`
- Arsitektur MobileNet + custom head
- Referensi ke `weights.bin` (shards)

### `weights.bin`
- Binary weights (bisa >1 file: `weights.bin`, `weights.1.bin`, ...)

---

## 🔒 Privacy & Ethics

| Aspek | Rekomendasi |
|-------|-------------|
| **Data pengguna** | Jangan upload video/user data ke TM tanpa consent |
| **Model bias** | Test di berbagai etnis, usia, gender, aksesoris |
| **Transparansi** | Info ke user: "AI mendeteksi jarak via kamera" |
| **Opt-out** | Tombol "Stop Camera" / disable fitur |
| **Lokal first** | Prefer model lokal (`dummy-bw/`) biar data tidak keluar |

---

## 📚 Resources

- [Teachable Machine Tutorials](https://teachablemachine.withgoogle.com/train)
- [TF.js Model Optimization](https://www.tensorflow.org/js/guide/conversion)
- [MobileNet Architecture](https://arxiv.org/abs/1704.04861)
- [Dataset Best Practices](https://developers.google.com/machine-learning/guides/data-preparation)

---

**Next:** [Deploy ke Production](DEPLOYMENT.md) | [API Reference](API.md) | [Kembali ke README](README.md)