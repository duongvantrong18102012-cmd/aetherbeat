# AetherBeat — Liquid Glass Audio-Reactive Studio

> **Trình phát nhạc & studio thị giác âm thanh thời gian thực (Audio-Reactive Studio)** kết hợp hệ thống vật liệu **Liquid Glass 2.5** khúc xạ ánh sáng bằng **WebGL (Snell's Law FBO)**, bộ phân tích phổ tần số **Web Audio API (Zero-GC)** và các bộ dựng hình **Canvas 2D** hiệu năng cao 60–120 FPS — đóng gói gọn trong **một tệp `index.html` duy nhất** không cần build step.

---

## Mục Lục

1. [Tổng Quan Kiến Trúc](#-tổng-quan-kiến-trúc)
2. [Tính Năng Nổi Bật](#-tính-năng-nổi-bật)
3. [Hệ Thống Quang Học Liquid Glass (WebGL Engine)](#-hệ-thống-quang-học-liquid-glass-webgl-engine)
4. [Bộ Phân Tích Âm Thanh & Bắt Nhịp Thời Gian Thực (Web Audio Engine)](#-bộ-phân-tích-âm-thanh--bắt-nhịp-thời-gian-thực-web-audio-engine)
5. [Hệ Thống Visualizer Tiền Cảnh (Canvas 2D High-FPS)](#-hệ-thống-visualizer-tiền-cảnh-canvas-2d-high-fps)
6. [Ảnh Bìa Mặc Định Procedural & Tải Tệp Cục Bộ](#-ảnh-bìa-mặc-định-procedural--tải-tệp-cục-bộ)
7. [Giao Diện, Tùy Chỉnh & Phím Tắt](#-giao-diện-tùy-chỉnh--phím-tắt)
8. [Tối Ưu Hóa Hiệu Năng (60–120 FPS)](#-tối-ưu-hóa-hiệu-năng-60120-fps)
9. [Hướng Dẫn Chạy & Triển Khai](#-hướng-dẫn-chạy--triển-khai)
10. [Cấu Trúc Dự Án & Công Nghệ Sử Dụng](#-cấu-trúc-dự-án--công-nghệ-sử-dụng)
11. [Giấy Phép](#-giấy-phép)

---

## Tổng Quan Kiến Trúc

AetherBeat được thiết kế theo mô hình **4 lớp kết hợp (4-Layer Composited Architecture)** hoàn toàn chạy trên GPU/CPU phía trình duyệt, tách biệt rõ ràng giữa **nền tĩnh tuyệt đối**, **lớp khúc xạ thủy tinh lỏng**, **visualizer phản ứng nhạc** và **giao diện HUD tương tác**:

```text
┌──────────────────────────────────────────────────────────────────────────┐
│  Lớp 4: Interactive HUD & Slide-Over Studio Drawer (HTML + Tailwind)    │
│         • Top Telemetry Bar (BPM, Beat 4/4, Live IOR, FPS)              │
│         • Hero Stage (Cover Frame, Dual-Tone Title, Seekbar, Controls)  │
│         • Studio Customization Drawer & Liquid Toasts                   │
├──────────────────────────────────────────────────────────────────────────┤
│  Lớp 3: Foreground Canvas 2D Visualizers (VisualizersRenderer)          │
│         • #ring-canvas: Vòng Astrolabe + 64 gai FFT + Shockwave + Sparks│
│         • #spectrum-canvas: Equalizer 64 cột treo ngược dưới thanh tua  │
│         • #bottom-wave-canvas: Sóng năng lượng kép (Bass / Mid-Snare)   │
├──────────────────────────────────────────────────────────────────────────┤
│  Lớp 2: Real-Time WebGL Snell Refraction (AudioReactiveLiquidGlassEngine)│
│         • Tính toán SDF bo góc (sdRoundBox) cho từng bề mặt kính        │
│         • Khúc xạ theo định luật Snell + Tán sắc lăng kính RGB           │
│         • Sóng xung kích quang học lan tỏa khi có Kick / Beat           │
├──────────────────────────────────────────────────────────────────────────┤
│  Lớp 1: Static Baked Backdrop Texture (u_envTex)                        │
│         • Render 1 lần duy nhất vào Offscreen Canvas -> WebGL Texture   │
│         • Đảm bảo 100% tĩnh bên ngoài các khối kính                     │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Tính Năng Nổi Bật

- **Liquid Glass 2.5 (Zero `backdrop-filter: blur`)**: Thay vì dùng `backdrop-filter` gây nặng GPU và mờ đục, AetherBeat dùng **WebGL Fragment Shader** để tính toán khúc xạ ánh sáng thực tế qua các bề mặt kính, viền cong meniscus đàn hồi và tán sắc RGB.
- **Nền Tĩnh Tuyệt Đối (Static Backdrop FBO)**: Mọi chuyển động thị giác chỉ diễn ra bên trong lớp kính và hệ thống visualizer, giữ cho nền phía sau ổn định, sắc nét và không gây rối mắt.
- **Tích Hợp Sẵn Nhạc Mẫu Synthwave 124 BPM**: Phát ngay lập tức bằng bộ tổng hợp âm thanh Web Audio API (Kick, Snare/Clap, Hi-hat, Bassline A–F–G–E, Lead Synth) mà không cần tải tệp ngoài.
- **Hỗ Trợ Nhạc & Ảnh Bìa Cục Bộ (Kéo Thả / Chọn Tệp)**: Hỗ trợ kéo thả trực tiếp tệp âm thanh (`MP3`, `WAV`, `OGG`, `FLAC`, `M4A`...) và ảnh bìa (`PNG`, `JPG`, `WebP`) vào bất kỳ đâu trên màn hình.
- **Phát Hiện Nhịp & Đo BPM Thời Gian Thực**: Thuật toán **Multi-Band Spectral Flux** tách riêng Kick (Bass) và Snare (Mid), tự động ước lượng BPM trực tiếp bằng bộ lọc trung vị.
- **Chế Độ Overdrive & Trình Chiếu Sạch**: Bật/tắt cường độ phản ứng xung kích cực đại (`O`) và ẩn toàn bộ thanh công cụ trên cùng (`H`) để quay video, stream hoặc trưng bày.

---

## Hệ Thống Quang Học Liquid Glass (WebGL Engine)

Class `AudioReactiveLiquidGlassEngine` quản lý canvas toàn màn hình `#optical-canvas` thông qua WebGL 1.0:

### 1. Bake Nền Tĩnh (`bakeStaticBackdrop`)
Nền được vẽ một lần duy nhất lên `backdrop2D` (offscreen canvas) và nạp vào texture `u_envTex` với 3 chế độ:
- **Đêm Noir (`noir`)**: Bầu trời đêm tương phản cao, ánh trăng mờ, quầng sáng đỏ chân trời, cột đèn đường silhouette, lưới kiến trúc quang học và vệt sáng chéo giúp các mép kính có chi tiết sắc nét để bẻ cong ánh sáng.
- **Ảnh Bìa Tĩnh (`cover`)**: Tự động lấy ảnh bìa hiện tại phủ toàn màn hình (aspect-fill) kết hợp lớp phủ vignette tối dịu để giữ độ tương phản cho chữ và HUD.
- **Studio Sáng (`studio-light`)**: Chuyển giao diện sang Light Theme (`body.theme-light`), nền xám xanh sáng cùng các đường gân kiến trúc và luồng sáng chéo.

### 2. Thuật Toán Khúc Xạ Trong Fragment Shader (`lensFS`)
- **Quản lý thấu kính không gây Layout Thrashing**: Tối đa `maxLenses` (6 đến 24 phần tử tùy `MAX_FRAGMENT_UNIFORM_VECTORS` của GPU) được lấy từ `#main-player-stage`, `#top-hud-bar`, `#studio-drawer.is-open`, `.liquid-control`, `.liquid-toast`. Tọa độ DOM chỉ được đo lại khi `resize`, `scroll` hoặc đóng/mở drawer (`cacheLensElements()` / `measureLenses()`).
- **Khoảng cách hình học có dấu (`sdRoundBox`)**: Xác định chính xác điểm ảnh nằm trong hay ngoài khối kính bo góc.
- **Độ cong viền Meniscus & Lò xo đàn hồi**: Biến `u_elastic` mô phỏng vật lý lò xo (`springForce = -0.26 * elasticPulse`, damping `0.72`) phản ứng khi nhấn nút hoặc khi có nhịp Kick/Snare.
- **Chiết suất động (`dynamicIOR`)**:
  $$\text{IOR}_{\text{live}} = \text{baseIOR} + \text{bass} \times 0.68 \times \text{reactivity} + \text{beat} \times 0.62 \times \text{reactivity}$$
- **Sóng xung kích quang học (`u_shockwave`, `u_shockAmp`)**: Khi phát hiện cú đánh Kick, một vòng sóng Gauss lan tỏa từ tâm ảnh bìa (`u_stageCenter`) làm biến dạng tia khúc xạ và tạo ánh lóe bên trong các tấm kính.
- **Tán sắc lăng kính RGB (`chromaSpread`)**: Lấy mẫu 3 kênh Đỏ – Lục – Lam ở 3 tọa độ UV lệch nhau dọc theo vector khúc xạ.
- **Viền Fresnel theo con trỏ chuột (`u_pointer`)**: Tính góc giữa pháp tuyến cạnh kính và vector hướng tới con trỏ để tạo vệt sáng specular trượt theo chuột.

---

## Bộ Phân Tích Âm Thanh & Bắt Nhịp Thời Gian Thực (Web Audio Engine)

Class `RealtimeAudioStudioEngine` sử dụng `AudioContext` và `AnalyserNode` (`fftSize = 1024`, tức **512 bin tần số**, `smoothingTimeConstant = 0.68`):

### 1. Phân Tích Đa Băng Tần & Spectral Flux
Mỗi khung hình (`analyzeFrame`), dữ liệu phổ `freqData` được so sánh với khung hình trước (`prevFreqData`) để tính năng lượng và độ biến thiên dương (spectral flux):

| Dải Tần | Khoảng Bin (`fftSize=1024`) | Tần Số Xấp Xỉ | Vai Trò |
| :--- | :--- | :--- | :--- |
| **Bass / Kick** | Bin `1 .. 6` | ~43 Hz – 260 Hz | Phát hiện cú đánh trống Kick, kích hoạt Shockwave & độ nảy Stage |
| **Mid / Snare** | Bin `7 .. 42` | ~300 Hz – 1.8 kHz | Phát hiện tiếng Snare/Clap & Vocal, kích hoạt rung đàn hồi phụ |
| **Treble / Air** | Bin `43 .. 160` | ~1.8 kHz – 6.9 kHz | Điều khiển độ lấp lánh viền sáng Specular và sóng phụ bên phải |

### 2. Phát Hiện Nhịp (Onset Detection) & Ước Lượng BPM
- Sử dụng **bộ đệm vòng tròn cố định (Circular Ring Buffer)** dài 36 mẫu (`bassRing`, `midRing`) để tính ngưỡng thích nghi (adaptive mean) mà **không tạo bất kỳ rác bộ nhớ (Zero-GC)**.
- **Kick Hit**: Kích hoạt khi `kickSignal > avgBass * 1.22` và `rawBass > 0.18` (giãn cách tối thiểu `235 ms`).
- **Ước lượng BPM**: Khoảng thời gian giữa hai nhịp liên tiếp (IBI trong khoảng `250–1200 ms`) được quy đổi về dải `76–178 BPM`, lưu vào mảng `ibiRing` 12 phần tử và lọc trung vị bằng thuật toán sắp xếp chèn (`insertion sort`) tại chỗ.

### 3. Bộ Tổng Hợp Nhạc Mẫu 124 BPM (`triggerDemoStep`)
Khi chưa tải tệp nhạc ngoài, bộ đếm bước 16th-note (`stepMs = (60 / 124 / 4) * 1000`) tổng hợp trực tiếp 5 lớp âm thanh qua `masterGain`:
1. **Sub Kick**: Sóng `sine` quét tần số từ `165 Hz` xuống `34 Hz`.
2. **Snare / Clap**: Nhiễu trắng qua bộ lọc `bandpass` tại `1850 Hz` (`Q = 0.8`) ở phách 4 và 12.
3. **Hi-Hat**: Nhiễu trắng qua bộ lọc `highpass` tại `7200 Hz` ở các phách lẻ.
4. **Dark-Pop Bassline**: Sóng `sawtooth` qua bộ lọc `lowpass` theo vòng hợp âm Am – F – G – E (`55.0`, `43.65`, `49.0`, `41.20 Hz`).
5. **Lead Synth Pluck**: Sóng `triangle` đánh giai điệu arpeggio theo từng ô nhịp.

---

## Hệ Thống Visualizer Tiền Cảnh (Canvas 2D High-FPS)

Class `VisualizersRenderer` quản lý 3 canvas 2D riêng biệt, **loại bỏ hoàn toàn `shadowBlur`** (vốn rất chậm trên trình duyệt) và thay bằng các kỹ thuật vẽ gộp (batched strokes/fills) và cộng sáng (`globalCompositeOperation = 'lighter'`):

1. **Vòng Astrolabe Xoay Quanh Ảnh Bìa (`#ring-canvas`)**:
   - Vòng tròn nét đứt + vòng phụ đồng tâm xoay thuận chiều kim đồng hồ, tăng tốc theo `energy` và `beat`.
   - **64 gai phổ tần số hướng tâm**: Vẽ gộp thành 2 lần `stroke()` (thân trắng và đầu nhọn màu nhấn khi biên độ > `0.34`).
   - **8 ngôi sao kim cương (Diamond Stars)**: Phóng to theo nhịp Kick & Snare.
   - **4 cung tròn đối nghịch**: Xoay ngược chiều ở vòng ngoài.
   - **Pool Shockwave & Hạt Crystalline Sparks**: Pre-allocate `6` vòng sóng xung kích và `52` hạt kim cương bắn tỏa từ chu vi vòng tròn mỗi khi có Beat.
2. **Thanh Equalizer Treo Ngược (`#spectrum-canvas`)**:
   - **64 cột** treo từ mép dưới thanh tiến độ xuống phía dưới.
   - Ánh xạ tần số phi tuyến (`Math.pow(norm, 1.8)`) giúp dải Bass và Mid hiển thị cân đối.
   - Đầu cột màu nhấn + **Peak-hold cap** rơi tự do theo trọng lực giả lập + lớp glow cộng sáng nhẹ cho các cột năng lượng cao.
3. **Sóng Năng Lượng Kép Cạnh Dưới (`#bottom-wave-canvas`)**:
   - Hai ngọn đồi sóng lồng vào nhau ở đáy màn hình: **Sóng Trắng (trái)** phản ứng theo Bass/Kick và **Sóng Màu Nhấn (phải)** phản ứng theo Mid/Snare/High, kết hợp đường viền đáy sắc nét.

---

## Ảnh Bìa Mặc Định Procedural & Tải Tệp Cục Bộ

- **Procedural Noir Cover (`generateDefaultCoverBlobUrl`)**: Ảnh bìa mặc định 720×720px được vẽ hoàn toàn bằng Canvas 2D và xuất ra `Blob URL` (`image/png`), không dùng ảnh ngoài hay chuỗi Base64 dài. Tác phẩm mô phỏng không gian đêm mưa tối giản với cột đèn đường, ký tự neon đỏ và nhân vật silhouette.
- **Tải ảnh bìa & file nhạc**:
  - Nhấn trực tiếp vào khung ảnh bìa, nút **Ảnh bìa** / **File nhạc** trên thanh công cụ, trong Drawer, hoặc **kéo thả** tệp thẳng vào cửa sổ trình duyệt.
  - Khi tải file nhạc mới, tên bài hát sẽ tự động trích xuất từ tên tệp (loại bỏ phần mở rộng và ký tự `_`/`-`) nếu người dùng chưa tự đặt tên trước đó.
  - Tự động giải phóng `URL.createObjectURL` cũ khi thay ảnh/nhạc mới để tránh rò rỉ bộ nhớ.

---

## Giao Diện, Tùy Chỉnh & Phím Tắt

### Thanh Công Cụ & HUD (`#top-hud-bar`)
- **Telemetry thời gian thực**: Hiển thị **BPM**, **4 chấm nhịp (Beat 4/4)**, **Chiết suất IOR trực tiếp**, và **FPS thực tế** (cập nhật mỗi 500 ms).
- **Overdrive ON / Balanced**: Chuyển đổi giữa chế độ phản ứng xung kích cực đại (`1.28x` hệ số loop, `1.35x` lực nảy beat) và chế độ cân bằng mượt mà (`0.88x` / `0.85x`).
- **Chuyển nguồn phát**: Chuyển qua lại giữa **Nhạc mẫu 124 BPM** và **File nhạc của bạn**.

### Ngăn Tùy Chỉnh Studio (`#studio-drawer`)
Nhấn nút **Tùy chỉnh** (hoặc nhấn trực tiếp vào tên bài hát / tên kênh để mở nhanh):
- **Thông tin hiển thị**:
  - *Tên bài hát*: Ký tự đầu tiên giữ màu trắng/tối sắc nét, các ký tự còn lại mang màu nhấn phát sáng.
  - *Tên kênh / Nghệ sĩ*: Hiển thị ở góc dưới bên phải kèm đường gạch chân gradient (`Yukii Music` mặc định).
- **Nền Tĩnh Phía Sau**: `Đêm Noir`, `Ảnh Bìa Tĩnh`, `Studio Sáng`.
- **5 Bảng Màu Nhấn (Accent Presets)**:
  - Crimson Noir (`#ef1a2d`)
  - Optical Cyan (`#38bdf8`)
  - Violet Prism (`#a855f7`)
  - Emerald Glass (`#10b981`)
  - Amber Caustic (`#f59e0b`)
- **Thông Số Quang Học & Âm Thanh**:
  - *Chiết suất kính gốc (Base IOR)*: `1.10` – `2.50` (mặc định `1.52`).
  - *Cường độ phản ứng nhạc*: `40%` – `250%` (mặc định `155%`).
  - *Hiện biểu tượng giao diện trên ảnh bìa*: Bật/tắt cụm icon sóng/pin và thanh icon mạng xã hội trang trí trên ảnh bìa.
- **Bộ Giám Sát 3 Dải Tần Trực Tiếp**: Thanh đo `%` thời gian thực cho `BASS (20–160Hz)`, `MID / VOCAL`, và `TREBLE / AIR`.

### Phím Tắt (Keyboard Shortcuts)

| Phím | Chức Năng |
| :--- | :--- |
| `Space` | Phát / Tạm dừng (Play / Pause) |
| `←` / `→` | Tua lùi / Tua tới `5 giây` |
| `O` | Bật / Tắt chế độ **Overdrive** |
| `H` | Bật / Tắt chế độ **Trình chiếu sạch (Presentation Mode)** |
| `M` | Tắt / Bật tiếng (Mute / Unmute) |
| `Esc` | Đóng bảng tùy chỉnh Studio |

*(Lưu ý: Phím tắt tự động vô hiệu hóa khi con trỏ đang nhập liệu trong ô `INPUT` hoặc `TEXTAREA`.)*

---

## Tối Ưu Hóa Hiệu Năng (60–120 FPS)

AetherBeat áp dụng nhiều kỹ thuật tối ưu hóa cấp trình duyệt để giữ tốc độ khung hình ổn định ngay cả trên màn hình độ phân giải cao:
1. **Zero Per-Frame DOM Layout Reflow**: Không đọc `getBoundingClientRect()` hay sửa `width`/`top`/`left` trong vòng lặp `requestAnimationFrame`. Tọa độ kính được cache; thanh tiến độ dùng `transform: scaleX(...)`; độ nảy stage/cover dùng `transform: translate3d(...) scale(...) rotate(...)` và `opacity` (chạy hoàn toàn trên GPU compositor).
2. **Zero-GC Audio & Particle Loops**: Tất cả mảng FFT, lịch sử Beat, mảng sắp xếp IBI, Shockwave pool và Spark pool đều dùng `TypedArray` (`Float32Array`, `Uint8Array`, `Uint16Array`) cấp phát một lần duy nhất khi khởi tạo.
3. **Adaptive DPR Cap**: Giới hạn `devicePixelRatio` ở mức `1.25` (màn hình rộng `> 1440px`) hoặc `1.5` cho cả WebGL lẫn Canvas 2D để giảm tải fill-rate trên màn hình 4K/Retina.
4. **Throttled HUD Text Updates**: Chỉ cập nhật text DOM (`timeCurrent`, `BPM`, `IOR`, thanh monitor) mỗi `4` frame hoặc chỉ khi chuỗi thời gian thực sự thay đổi.

---

## Hướng Dẫn Chạy & Triển Khai

Vì toàn bộ ứng dụng nằm trong một tệp `index.html` duy nhất:

### Chạy trực tiếp trên máy
- Mở trực tiếp tệp `index.html` bằng trình duyệt hiện đại (**Chrome**, **Edge**, **Firefox**, **Safari**) hỗ trợ **WebGL** và **Web Audio API**.
- Hoặc chạy qua một static server bất kỳ:
  ```bash
  # Với Python
  python -m http.server 8080

  # Hoặc với Node.js (npx)
  npx serve .
  ```
  Sau đó truy cập `http://localhost:8080`.

### Triển khai lên GitHub Pages
1. Vào **Settings → Pages** của repository trên GitHub.
2. Tại **Build and deployment → Source**, chọn **Deploy from a branch**.
3. Chọn nhánh `main` và thư mục `/ (root)`, nhấn **Save**.

---

## Cấu Trúc Dự Án & Công Nghệ Sử Dụng

```text
aetherbeat/
├── index.html   # Toàn bộ mã nguồn: HTML, CSS (Liquid Glass 2.5), WebGL Shader, Web Audio & Canvas 2D
├── README.md    # Tài liệu chi tiết kiến trúc và hướng dẫn sử dụng
└── LICENSE      # Giấy phép mã nguồn mở MIT
```

- **HTML5 & CSS3**: Giao diện chuẩn ngữ nghĩa, biến CSS động (`--accent`, `--lg-*`), GPU compositing.
- **Tailwind CSS (CDN)**: Bố cục responsive linh hoạt trên Desktop & Mobile.
- **WebGL 1.0 (GLSL ES)**: Shader khúc xạ Snell, tán sắc RGB và sóng xung kích quang học.
- **Web Audio API**: Bộ tổng hợp 124 BPM tích hợp & phân tích phổ FFT 1024 điểm.
- **Canvas 2D API**: Bộ dựng hình Astrolabe Ring, Hanging Spectrum, Dual Bottom Waves và Procedural Artwork.
- **Google Fonts**: `Inter` & `JetBrains Mono`.

---

## Giấy Phép

Dự án được phát hành theo giấy phép **MIT License** — xem chi tiết trong tệp [LICENSE](LICENSE).
