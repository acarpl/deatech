# Dokumentasi Detektor Jarak Layar

Selamat datang di dokumentasi **Screen Distance Detector** — aplikasi web yang mendeteksi jarak pengguna dari layar menggunakan AI (Teachable Machine + TensorFlow.js).

---

## 📚 Daftar Isi

| Dokumen | Deskripsi |
|---------|-----------|
| [Tutorial Memulai](TUTORIAL.md) | Panduan langkah demi langkah untuk menjalankan & mengkustomisasi |
| [Referensi API](API.md) | Dokumentasi teknis fungsi, variabel, dan arsitektur kode |
| [Deploy ke Production](DEPLOYMENT.md) | Cara deploy ke GitHub Pages, Netlify, Vercel, dll |
| [Model Kustom](CUSTOM_MODEL.md) | Cara buat & latih model Teachable Machine sendiri |

---

## 🚀 Quick Start

```bash
# 1. Clone repo
git clone https://github.com/acarpl/deatech.git
cd deatech

# 2. Jalankan local server (pilih salah satu)
# Opsi A: Python
python -m http.server
# -> Buka http://localhost:8000

# Opsi B: VS Code + Live Server extension
# Klik kanan index.html -> "Open with Live Server"
```

⚠️ **WAJIB** pakai `localhost` — double-klik file HTML tidak akan jalan (kamera butuh secure context).

---

## 🎯 Fitur Utama

- ✅ Deteksi real-time: "Terlalu Dekat" vs "Jarak Baik"
- ✅ Visual feedback: Background merah/hijau + badge status
- ✅ Audio feedback: Suara peringatan (rendah) & notifikasi (tinggi)
- ✅ Cooldown anti-flicker (default 1 detik)
- ✅ Threshold confidence configurable (default 0.85)
- ✅ Model kustom via Teachable Machine (no-code)
- ✅ Offline-ready (model bisa disimpan lokal di folder `dummy-bw/`)

---

## 🛠 Tech Stack

| Teknologi | Versi | Fungsi |
|-----------|-------|--------|
| TensorFlow.js | 1.3.1 | Runtime ML di browser |
| @teachablemachine/image | 0.8 | Wrapper load model TM |
| Web Audio API | Native | Suara sintetis (oscillator) |
| Webcam API | Native | Akses kamera via `getUserMedia` |
| Vanilla JS | ES6+ | Logika aplikasi tanpa framework |

---

## 📁 Struktur Project

```
deatech/
├── index.html          # Aplikasi utama (HTML + CSS + JS)
├── dummy-bw/           # Model default (TF.js format)
│   ├── model.json      # Arsitektur & weights
│   ├── metadata.json   # Labels, image size, versi
│   └── weights.bin     # Bobot model (binary)
├── image.jpg           # Gambar contoh (bisa diganti)
├── README.txt          # Instruksi singkat (legacy)
└── docs/               # Dokumentasi ini
    ├── README.md       # Index ini
    ├── TUTORIAL.md     # Tutorial lengkap
    ├── API.md          # Referensi teknis
    ├── DEPLOYMENT.md   # Panduan deploy
    └── CUSTOM_MODEL.md # Buat model sendiri
```

---

## 🤝 Kontribusi

1. Fork repo
2. Buat branch: `git checkout -b fitur-baru`
3. Commit: `git commit -m "Tambah fitur X"`
4. Push: `git push origin fitur-baru`
5. Buat Pull Request

---

## 📄 Lisensi

MIT License — bebas gunakan, modifikasi, distribusi.

---

## 🔗 Link Berguna

- [Teachable Machine](https://teachablemachine.withgoogle.com) — Buat model tanpa coding
- [TensorFlow.js Docs](https://www.tensorflow.org/js) — Referensi TF.js
- [Web Audio API MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API) — Dokumentasi audio
- [Repo GitHub](https://github.com/acarpl/deatech) — Source code & issues