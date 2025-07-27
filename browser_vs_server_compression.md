# 📘 Documentation: Video Compression Using FFmpeg.wasm & Cloudflare Workers

---

## 🔰 Overview

This guide documents two main strategies for compressing user-uploaded video files:

1. **Browser-Side Compression** using `FFmpeg.wasm`
2. **Cloud-Based Compression** using **Cloudflare Workers + R2**

---

## 📍 Use Case 1: Browser-Side Compression (For Small to Medium Files)

### 🧠 Concept Flow

```
User selects video
↓
FFmpeg.wasm runs in browser
↓
Compressed video (in-memory)
↓
Upload to server (Cloudflare Worker)
```

### 🧪 Example Implementation

#### HTML

```html
<input type="file" id="videoInput" />
```

#### JavaScript

```js
import { createFFmpeg, fetchFile } from '@ffmpeg/ffmpeg';

const ffmpeg = createFFmpeg({ log: true });

async function compressInBrowser(file) {
  await ffmpeg.load();
  ffmpeg.FS('writeFile', 'input.mp4', await fetchFile(file));

  await ffmpeg.run('-i', 'input.mp4', '-vcodec', 'libx264', '-crf', '28', 'output.mp4');

  const data = ffmpeg.FS('readFile', 'output.mp4');
  return new Blob([data.buffer], { type: 'video/mp4' });
}
```

#### Upload to Worker

```js
const formData = new FormData();
formData.append('file', compressedBlob);

await fetch('https://your-worker-url.com/upload', {
  method: 'POST',
  body: formData,
});
```

### ⚠️ Limitation Example

If user uploads a **90 GB file**, and compressed output is **20 GB**:

- Total space required in browser: **90 GB (original) + 20 GB (compressed)**
- Risk: Browser crash or freeze

---

## 🛠️ Use Case 2: Cloud-Based Compression (for Large Files)

### 🔁 Recommended Flow

```
User → Upload to Cloudflare R2
                      ↓
             Trigger Cloudflare Worker
                      ↓
       FFmpeg runs on server (non-WASM)
                      ↓
     Save compressed video → Send download link
```

### Cloudflare Worker Example

```js
export default {
  async fetch(req) {
    const formData = await req.formData();
    const file = formData.get('file');

    await R2Bucket.put('original.mp4', file.stream());
    return new Response("Uploaded Successfully", { status: 200 });
  }
}
```

---

## 🖼️ Comparison Diagram

```
Small File:
User → Browser (FFmpeg.wasm) → Upload → Store

Large File:
User → Cloud Upload → Server-side FFmpeg → Store → Download Link
```

---

## 📌 Key Constraints

| Type              | Limit                             |
| ----------------- | --------------------------------- |
| FFmpeg.wasm       | \~1 GB (RAM/WebAssembly limit)    |
| Browser Storage   | 2–3 GB (browser/device-dependent) |
| Cloudflare Worker | Scalable for large file handling  |
| Cloudflare R2     | Ideal for cloud object storage    |

---

## ✅ Recommendations

| File Size | Approach                                |
| --------- | --------------------------------------- |
| < 500MB   | FFmpeg.wasm in browser ✅                |
| 500MB–1GB | Warn user, fallback to cloud ⚠️         |
| > 1GB     | Use cloud upload + server-side FFmpeg ❌ |

---

## 📦 To-Do (Optional Extensions)

-

---

Let me know if you need:

- A downloadable `.md` version
- Full backend + R2 upload/download pipeline
- Demo repo with UI + FFmpeg.wasm integration

