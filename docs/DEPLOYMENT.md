# Panduan Deploy ke Production

Cara meng-host aplikasi ini ke internet agar bisa diakses dari mana saja.

---

## 🌐 Opsi Deploy (Gratis & Mudah)

| Platform | Gratis | Custom Domain | HTTPS Auto | Cocok Untuk |
|----------|--------|---------------|------------|-------------|
| **GitHub Pages** | ✅ | ✅ | ✅ | Project open source, simple static site |
| **Netlify** | ✅ | ✅ | ✅ | Auto deploy dari Git, form handling, edge functions |
| **Vercel** | ✅ | ✅ | ✅ | Next.js/React tapi support static juga, DX terbaik |
| **Cloudflare Pages** | ✅ | ✅ | ✅ | Gratis unlimited bandwidth, Workers integration |
| **Firebase Hosting** | ✅ | ✅ | ✅ | Google ecosystem, CLI tooling bagus |

---

## 📋 Prasyarat Semua Platform

1. **Repo Git** (GitHub/GitLab/Bitbucket)
2. **Branch utama**: `main` atau `master`
3. **File `index.html` di root** (bukan di subfolder)
4. **HTTPS wajib** — kamera butuh secure context

---

## 🐙 GitHub Pages (Paling Sederhana)

### Cara 1: Via Settings (No GitHub Actions)

1. Buka repo di GitHub: `https://github.com/acarpl/deatech`
2. **Settings** → **Pages** (sidebar kiri)
3. **Source**: `Deploy from a branch`
4. **Branch**: `main` → `/ (root)`
5. Klik **Save**
6. Tunggu 1-2 menit → URL: `https://acarpl.github.io/deatech/`

### Cara 2: Via GitHub Actions (Recommended)

Buat file `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v4
      - uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      - uses: actions/deploy-pages@v4
```

**Keuntungan**: Auto deploy setiap push ke `main`, log build tersedia.

---

## 🌊 Netlify (Drag & Drop / Git Integration)

### Metode A: Drag & Drop (Tanpa Git)

1. Buka https://app.netlify.com/drop
2. Drag folder project `deatech` ke area drop
3. Selesai! URL random: `https://random-name.netlify.app`
4. (Opsional) Klik **Domain settings** → **Add custom domain**

### Metode B: Git Integration (Auto Deploy)

1. Login Netlify → **Add new site** → **Import an existing project**
2. Connect GitHub → pilih repo `acarpl/deatech`
3. Build settings:
   - **Build command**: (kosongkan / leave empty)
   - **Publish directory**: `.` (root)
4. **Deploy site**
5. Done! Auto deploy tiap push ke `main`.

**Netlify.toml (opsional, taruh di root):**
```toml
[build]
  publish = "."

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"
```

---

## ▲ Vercel (CLI / Git Integration)

### Via Vercel CLI

```bash
# Install CLI
npm i -g vercel

# Di folder project
vercel

# Ikuti prompt:
# ? Set up and deploy? [Y/n] y
# ? Which scope? (your account)
# ? Link to existing project? [y/N] n
# ? What's your project's name? deatech
# ? In which directory is your code located? ./
```

### Via Git Integration

1. Buka https://vercel.com/new
2. Import Git Repository → pilih `acarpl/deatech`
3. Framework Preset: **Other**
4. Build Command: (kosong)
5. Output Directory: `.`
6. **Deploy**

**vercel.json (opsional, di root):**
```json
{
  "cleanUrls": true,
  "trailingSlash": false,
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "DENY" }
      ]
    }
  ]
}
```

---

## ☁️ Cloudflare Pages

1. Login Cloudflare → **Workers & Pages** → **Create application** → **Pages**
2. **Connect to Git** → GitHub → pilih repo
3. Build settings:
   - **Build command**: (exit 0 / kosong)
   - **Build output directory**: `.`
4. **Save and Deploy**

**Keuntungan**: Unlimited bandwidth gratis, CDN global, gratis Workers untuk API.

---

## 🔥 Firebase Hosting

```bash
# Install CLI
npm install -g firebase-tools

# Login
firebase login

# Init (di folder project)
firebase init hosting

# Pilih:
# ? What do you want to use as your public directory? .
# ? Configure as a single-page app? No
# ? Set up automatic builds and deploys with GitHub? Yes/No

# Deploy
firebase deploy
```

**firebase.json (generated):**
```json
{
  "hosting": {
    "public": ".",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "headers": [
      {
        "source": "**",
        "headers": [
          { "key": "X-Content-Type-Options", "value": "nosniff" }
        ]
      }
    ]
  }
}
```

---

## 🔧 Konfigurasi Penting Semua Platform

### 1. Pastikan `index.html` di Root
```
deatech/
├── index.html        ← HARUS di root
├── dummy-bw/
├── docs/
└── ...
```

### 2. Model Lokal vs CDN

**Pakai Model CDN (Default):**
- `MODEL_URL` = URL Teachable Machine (butuh internet)
- File lebih kecil, tapi butuh koneksi saat load

**Pakai Model Lokal (Offline-Ready):**
1. Copy folder `dummy-bw/` ke repo
2. Edit `index.html`:
```javascript
const MODEL_URL = "dummy-bw/";  // Relative path, harus diakhiri "/"
```
3. Deploy — jalan tanpa internet (kecuali load TF.js dari CDN)

### 3. Fully Offline (Termasuk TF.js)

Download library & host lokal:

```bash
# Di folder project
mkdir -p lib
cd lib

# Download TF.js
curl -O https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.3.1/dist/tf.min.js

# Download Teachable Machine
curl -O https://cdn.jsdelivr.net/npm/@teachablemachine/image@0.8/dist/teachablemachine-image.min.js
```

Edit `index.html`:
```html
<script src="lib/tf.min.js"></script>
<script src="lib/teachablemachine-image.min.js"></script>
```

---

## 🔒 Security Headers (Recommended)

Tambahkan ke konfigurasi platform masing-masing:

| Header | Value | Fungsi |
|--------|-------|--------|
| `Content-Security-Policy` | `default-src 'self'; script-src 'self' 'unsafe-inline' https://cdn.jsdelivr.net; connect-src 'self' https://teachablemachine.withgoogle.com; img-src 'self' data:; media-src 'self';` | Batasi resource loading |
| `X-Frame-Options` | `DENY` | Prevent clickjacking |
| `X-Content-Type-Options` | `nosniff` | Prevent MIME sniffing |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Privacy referrer |
| `Permissions-Policy` | `camera=self; microphone=()` | Hanya izinkan camera, blok mic |

---

## ✅ Checklist Sebelum Deploy

- [ ] `index.html` di root folder
- [ ] `MODEL_URL` benar (CDN atau lokal `dummy-bw/`)
- [ ] Class names di config match dengan model (`CLASS_TOO_CLOSE`, `CLASS_GOOD_DISTANCE`)
- [ ] Test lokal jalan: `python -m http.server` → `localhost:8000`
- [ ] Kamera minta izin & video muncul
- [ ] Deteksi berubah warna + suara
- [ ] Push ke GitHub: `git add . && git commit -m "Ready for deploy" && git push`
- [ ] Deploy ke platform pilihan
- [ ] Test URL production di HP & laptop lain
- [ ] (Opsional) Custom domain dikonfigurasi

---

## 🐛 Troubleshooting Deploy

### "Camera not working" di production
- **Penyebab**: Deploy ke HTTP (bukan HTTPS) / domain tidak secure
- **Solusi**: Semua platform di atas gratis HTTPS. Pastikan akses via `https://`

### Model tidak load (404 / CORS)
- **Penyebab**: `MODEL_URL` salah / model TM tidak public
- **Solusi**: 
  - Cek Network tab (F12) → lihat request ke `model.json` & `metadata.json`
  - Pastikan model TM di-set **Public** saat export
  - Atau pakai model lokal `dummy-bw/`

### "tf is not defined" / Library tidak load
- **Penyebab**: CDN blocked / CSP terlalu ketat
- **Solusi**: Tambah `https://cdn.jsdelivr.net` ke `script-src` CSP, atau host library lokal

### Deploy stuck / build error
- **GitHub Pages**: Cek tab **Actions** → lihat log error
- **Netlify/Vercel**: Cek **Deploy log** di dashboard
- **Common**: File terlalu besar (>100MB gratis), symlink, case-sensitive filename

---

## 📊 Monitoring & Analytics (Opsional)

Tambahkan ke `<head>` `index.html`:

```html
<!-- Google Analytics (Ganti G-XXXXXXXXXX) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');
</script>

<!-- Atau Plausible (privacy-friendly) -->
<script defer data-domain="domainmu.com" src="https://plausible.io/js/script.js"></script>
```

---

## 🔄 Update & Maintenance

| Aksi | Perintah |
|------|----------|
| Update kode | Edit lokal → `git add . && git commit -m "msg" && git push` |
| Update model | Ganti file di `dummy-bw/` → commit push |
| Rollback | `git revert <commit-hash>` → push |
| Custom domain | Setup di platform (CNAME/A record) → verifikasi |

---

## 💡 Tips Pro

1. **Preview Deploy**: Netlify/Vercel bikin **Deploy Preview** tiap PR — test sebelum merge
2. **Cache Busting**: Tambah `?v=2` ke script src kalau update library
3. **PWA**: Tambah `manifest.json` + service worker → installable di HP
4. **Custom Domain**: Beli domain → tambah CNAME ke platform → enable HTTPS auto

---

**Next:** [Buat Model Kustom](CUSTOM_MODEL.md) | [Kembali ke README](README.md)