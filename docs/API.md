# Referensi API & Arsitektur Teknis

Dokumentasi teknis untuk developer yang mau modifikasi/extend kode.

---

## 📦 Dependencies (CDN)

```html
<!-- TensorFlow.js Core -->
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@1.3.1/dist/tf.min.js"></script>

<!-- Teachable Machine Image Wrapper -->
<script src="https://cdn.jsdelivr.net/npm/@teachablemachine/image@0.8/dist/teachablemachine-image.min.js"></script>
```

| Package | Versi | Global Object | Fungsi |
|---------|-------|---------------|--------|
| `@tensorflow/tfjs` | 1.3.1 | `tf` | Tensor ops, model loading, backend WebGL |
| `@teachablemachine/image` | 0.8 | `tmImage` | Load model TM, Webcam wrapper, predict helper |

---

## ⚙️ Konfigurasi (Constants)

```javascript
const MODEL_URL = "https://teachablemachine.withgoogle.com/models/XXXX/";
const CLASS_TOO_CLOSE = "Too Close";
const CLASS_GOOD_DISTANCE = "Good Distance";
const CONFIDENCE_THRESHOLD = 0.85;  // 0.0 - 1.0
const COOLDOWN_MS = 1000;           // milliseconds
```

| Konstanta | Tipe | Default | Deskripsi |
|-----------|------|---------|-----------|
| `MODEL_URL` | `string` | TM demo | URL model (harus diakhiri `/`) |
| `CLASS_TOO_CLOSE` | `string` | `"Too Close"` | Nama class "dekat" di TM (case-sensitive) |
| `CLASS_GOOD_DISTANCE` | `string` | `"Good Distance"` | Nama class "jauh" di TM (case-sensitive) |
| `CONFIDENCE_THRESHOLD` | `number` | `0.85` | Min probability untuk trigger status |
| `COOLDOWN_MS` | `number` | `1000` | Jeda minimal antar ganti status (anti-flicker) |

---

## 🗂 State Variables (Global)

```javascript
let model, webcam;           // Model & Webcam instances
let isRunning = false;       // Flag loop aktif
let lastStatus = null;       // "too-close" | "good-distance" | null
let lastChangeTime = 0;      // Timestamp last status change (ms)
let audioContext = null;     // Web Audio API context
```

| Variabel | Tipe | Deskripsi |
|----------|------|-----------|
| `model` | `tmImage.CustomMobileNet` | Model hasil `tmImage.load()` |
| `webcam` | `tmImage.Webcam` | Wrapper webcam (canvas + video) |
| `isRunning` | `boolean` | Guard untuk `requestAnimationFrame` loop |
| `lastStatus` | `string|null` | Status UI terakhir yang diterapkan |
| `lastChangeTime` | `number` | `Date.now()` saat status terakhir berubah |
| `audioContext` | `AudioContext|null` | Singleton audio context |

---

## 🎯 DOM Elements (Cached)

```javascript
const stage = document.getElementById("stage");
const hint = document.getElementById("hint");
const statusText = document.getElementById("statusText");
const statusBadge = document.getElementById("statusBadge");
const label = document.getElementById("label");
const body = document.body;
```

| Element | ID | Fungsi |
|---------|-----|--------|
| `stage` | `#stage` | Container utama status (background color) |
| `hint` | `#hint` | Text petunjuk state netral |
| `statusText` | `#statusText` | Text besar ⚠️/✅ (class `.big`) |
| `statusBadge` | `#statusBadge` | Badge kecil di bawah text |
| `label` | `#label` | Real-time prediction: `"Class - XX.X%"` |
| `body` | `document.body` | Global background class toggle |

---

## 🔧 Core Functions

### `initAudio()`
```javascript
function initAudio(): void
```
Inisialisasi `AudioContext` (singleton). Harus dipanggil **setelah user interaction** (browser policy).

**Dipanggil di:** `start()` (onclick button) & `playWarningSound()` & `playTingSound()`

---

### `playWarningSound()`
```javascript
function playWarningSound(): void
```
Mainkan suara peringatan "Terlalu Dekat".

**Spesifikasi Audio:**
| Parameter | Nilai | Keterangan |
|-----------|-------|------------|
| `oscillator.type` | `"sawtooth"` | Gelombang bergerigi, timbre kasar |
| `frequency` | `220 Hz` | Nada A3 (rendah) |
| `gain envelope` | Attack 20ms → Sustain → Release 500ms | `exponentialRampToValueAtTime` |
| Durasi total | ~550ms | `oscillator.stop(now + 0.55)` |

---

### `playTingSound()`
```javascript
function playTingSound(): void
```
Mainkan suara notifikasi "Jarak Baik".

**Spesifikasi Audio:**
| Parameter | Nilai | Keterangan |
|-----------|-------|------------|
| `oscillator.type` | `"sine"` | Gelombang sinus, timbre murni |
| `frequency` | `880 Hz` | Nada A5 (tinggi, "ting") |
| `gain envelope` | Attack 10ms → Release 150ms | Sangat cepat |
| Durasi total | ~160ms | `oscillator.stop(now + 0.16)` |

---

### `updateUI(status, confidence)`
```javascript
function updateUI(status: string, confidence: number): void
```
**Fungsi utama update tampilan.** Dipanggil setiap frame dari `predict()`.

**Parameter:**
| Param | Tipe | Deskripsi |
|-------|------|-----------|
| `status` | `string` | `top.className` dari prediksi (mis. `"Too Close"`) |
| `confidence` | `number` | `top.probability` (0.0 - 1.0) |

**Logika:**
1. **Cooldown check**: `if (now - lastChangeTime < COOLDOWN_MS) return;`
2. **Determine newStatus**:
   - `status === CLASS_TOO_CLOSE && confidence >= THRESHOLD` → `"too-close"`
   - `status === CLASS_GOOD_DISTANCE && confidence >= THRESHOLD` → `"good-distance"`
   - Else → `null` (netral)
3. **If changed**: Update DOM classes, text, background, play sound
4. **Always**: Update `label.textContent = `${status} - ${confidence}%``

**Side Effects:**
- `body.classList.add/remove("too-close", "good-distance")`
- `stage.style.background` → `#8B0000` / `#006400` / `""`
- `statusText` / `statusBadge` show/hide + content
- `hint` show/hide
- Audio: `playWarningSound()` / `playTingSound()`

---

### `start()` (Event Handler)
```javascript
document.getElementById("startBtn").onclick = async function start(): Promise<void>
```
**Flow:**
1. `initAudio()` — unlock audio context
2. `label.textContent = "memuat model..."`
3. `model = await tmImage.load(model.json, metadata.json)` — **load model**
4. `webcam = new tmImage.Webcam(200, 200, true)` — width, height, flip
5. `await webcam.setup()` — request camera permission (`getUserMedia`)
6. `await webcam.play()` — start video stream
7. `document.getElementById("cam").appendChild(webcam.canvas)` — render ke DOM
8. Disable button, update label, `isRunning = true`
9. `requestAnimationFrame(loop)` — kick off loop

**Error Handling:** Try/catch di load model → tampilkan error ke `label`

---

### `loop()`
```javascript
async function loop(): Promise<void>
```
**Main render loop** (60fps ideal via `requestAnimationFrame`).

```javascript
async function loop() {
  if (!isRunning) return;
  webcam.update();        // Capture frame ke canvas internal
  await predict();        // Inferensi
  requestAnimationFrame(loop); // Schedule next frame
}
```

---

### `predict()`
```javascript
async function predict(): Promise<void>
```
**Inferensi + post-processing.**

```javascript
async function predict() {
  const predictions = await model.predict(webcam.canvas);
  // predictions: Array<{className: string, probability: number}>
  
  // Find top-1
  let top = predictions[0];
  for (const p of predictions) {
    if (p.probability > top.probability) top = p;
  }
  
  updateUI(top.className, top.probability);
}
```

**Return `model.predict()`:**
```typescript
type Prediction = {
  className: string;   // Nama class dari metadata.json
  probability: number; // 0.0 - 1.0
}[]
```

---

## 🎨 CSS Classes (State-Driven)

| Class | Selector | Trigger | Visual |
|-------|----------|---------|--------|
| `.too-close` | `body.too-close` | `newStatus === "too-close"` | Background `#8B0000` (dark red) |
| `.good-distance` | `body.good-distance` | `newStatus === "good-distance"` | Background `#006400` (dark green) |
| `.status-too-close` | `.status-badge.status-too-close` | Too close | Red badge `#FF4A1C` |
| `.status-good-distance` | `.status-badge.status-good-distance` | Good distance | Green badge `#00C851` |
| `[hidden]` | `[hidden]` | Native HTML attribute | `display: none !important` |

---

## 🔄 Event Flow Diagram

```
User Click "Mulai Kamera"
         │
         ▼
    initAudio() ──────► AudioContext created/resumed
         │
         ▼
   tmImage.load() ────► Load model.json + metadata.json
         │
         ▼
   new tmImage.Webcam(200,200,true)
         │
         ▼
   webcam.setup() ────► navigator.mediaDevices.getUserMedia()
         │
         ▼
   webcam.play() ────► Video stream playing
         │
         ▼
   Append canvas to #cam
         │
         ▼
   isRunning = true
         │
         ▼
requestAnimationFrame(loop)
         │
         ▼
┌─────────────────────────────────────┐
│           LOOP (per frame)          │
├─────────────────────────────────────┤
│ webcam.update()                     │
│   │                                 │
│   ▼                                 │
│ model.predict(canvas)               │
│   │                                 │
│   ▼                                 │
│ Find top prediction                 │
│   │                                 │
│   ▼                                 │
│ updateUI(className, probability)    │
│   │                                 │
│   ├── Cooldown check                │
│   ├── Determine newStatus           │
│   ├── If changed: DOM + Audio       │
│   └── Always: label update          │
│   │                                 │
│   ▼                                 │
│ requestAnimationFrame(loop) ◄───────┘
└─────────────────────────────────────┘
```

---

## 🧪 Extending: Tambah Class / Logika Baru

### Tambah Class Ketiga (mis. "Sedang")

1. **TM**: Tambah class "Sedang" di Teachable Machine, retrain, export
2. **Config**:
```javascript
const CLASS_SEDANG = "Sedang";
```
3. **updateUI**: Tambah branch
```javascript
} else if (status === CLASS_SEDANG && confidence >= CONFIDENCE_THRESHOLD) {
  newStatus = "sedang";
}
```
4. **CSS**: Tambah `.sedang` styles
5. **DOM**: Update UI untuk state baru

### Ganti Model ke Object Detection

Ganti `@teachablemachine/image` → `@teachablemachine/object` dan adaptasi `predict()` untuk handle bounding boxes.

---

## 📊 Performance Notes

| Metrik | Nilai Tipis | Catatan |
|--------|-------------|---------|
| Model size | ~1-3 MB | MobileNet-based |
| Inference time | 20-50ms/frame | Bergantung GPU/WebGL |
| FPS target | 30-60 fps | `requestAnimationFrame` |
| Memory | ~50-100 MB | Tensor buffers + video |

**Optimasi:**
- Kurangi resolusi webcam: `new tmImage.Webcam(160, 160, true)`
- Naikkan `COOLDOWN_MS` kurangi DOM updates
- Gunakan `tf.setBackend('webgl')` (default) — lebih cepat dari CPU

---

## 🔐 Browser Compatibility

| Feature | Chrome | Firefox | Safari | Edge |
|---------|--------|---------|--------|------|
| `getUserMedia` | ✅ 53+ | ✅ 36+ | ✅ 11+ | ✅ 79+ |
| WebGL (TF.js) | ✅ | ✅ | ✅ | ✅ |
| Web Audio API | ✅ 14+ | ✅ 25+ | ✅ 14+ | ✅ 79+ |
| `requestAnimationFrame` | ✅ | ✅ | ✅ | ✅ |
| ES6 Modules | ✅ 61+ | ✅ 60+ | ✅ 11+ | ✅ 79+ |

**Catatan:** Butuh **HTTPS atau localhost** untuk `getUserMedia`.

---

## 📝 TypeScript Definitions (Reference)

```typescript
// tmImage types (subset)
declare namespace tmImage {
  interface CustomMobileNet {
    predict(canvas: HTMLCanvasElement): Promise<Prediction[]>;
  }
  
  interface Webcam {
    new (width: number, height: number, flip: boolean): Webcam;
    setup(): Promise<void>;
    play(): Promise<void>;
    update(): void;
    canvas: HTMLCanvasElement;
  }
  
  function load(modelUrl: string, metadataUrl: string): Promise<CustomMobileNet>;
  
  type Prediction = {
    className: string;
    probability: number;
  };
}
```

---

## 🔗 Related Docs

- [Tutorial Lengkap](TUTORIAL.md) — Step-by-step pengguna
- [Deploy Guide](DEPLOYMENT.md) — Production deployment
- [Custom Model](CUSTOM_MODEL.md) — Teachable Machine deep dive
- [TF.js API](https://www.tensorflow.org/js/api_docs) — Official TensorFlow.js docs