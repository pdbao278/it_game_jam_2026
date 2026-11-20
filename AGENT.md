# HƯỚNG DẪN DÀNH CHO AGENT (AGENT.MD) - PHÁT TRIỂN GAME 2D & 3D

Tài liệu này là bộ quy chuẩn và chỉ dẫn hành động bắt buộc dành cho mọi Trợ lý AI (Agent) khi tham gia lập trình, thiết kế kiến trúc, sửa lỗi và tối ưu hóa dự án game (hỗ trợ cả **2D Phaser** và **3D Three.js**).

---

## 1. TỔNG QUAN DỰ ÁN & TECH STACK

- **Môi trường & Bundler:** Vite (ES Modules), Node.js.
- **Engine 2D:** Phaser (Phiên bản v4.x - cấu hình tại `src/engine/phaser-game.js`).
- **Engine 3D:** Three.js (Phiên bản 0.187.x / WebGL & WebGPU).
- **Framework Kiểm thử:** Vitest (`npm run test`).
- **Kho Tri Thức & Kỹ Năng:** Toàn bộ hướng dẫn chuyên sâu nằm tại thư mục `.agents/skills/`.
- **Ngôn ngữ phản hồi & ghi chú:** Bắt buộc sử dụng **Tiếng Việt**.

---

## 2. NGUYÊN TẮC CỐT LÕI (CORE PRINCIPLES)

1. **Tách biệt Logic và Đồ họa (Decoupled Architecture):**
   - Không nhồi nhét toàn bộ logic vào vòng lặp `update()` của Scene.
   - Tách biệt logic trạng thái game (Game State / Business Logic) thành các class hoặc module thuần JS/TS để dễ dàng viết Unit Test độc lập với Canvas/WebGL.
2. **Khắc phục lỗi thời của AI (No Deprecated APIs):**
   - **Phaser:** Tuân thủ chuẩn Phaser 4 (sử dụng RenderNodes, Filters, không dùng cú pháp cũ của Phaser 2/3 đã bị loại bỏ).
   - **Three.js:** Tuyệt đối không dùng `Geometry` (phải dùng `BufferGeometry`), luôn cấu hình `SRGBColorSpace`, bật `ACESFilmicToneMapping` để tránh hiện tượng màu sắc bị rửa trôi (washed-out).
3. **Phòng chống tràn bộ nhớ (Zero Memory Leaks):**
   - Mọi Texture, Sound, Mesh, Geometry, Filter và Event Listener khi khởi tạo đều phải có phương thức hủy (`destroy()`, `dispose()`, `off()`) khi chuyển Scene hoặc dọn dẹp game.

---

## 3. QUY CHUẨN PHÁT TRIỂN GAME 2D (PHASER 4)

Khi thực hiện các tác vụ liên quan đến 2D, Agent phải kích hoạt và tham chiếu các skill tương ứng trong thư mục `.agents/skills/`:

### A. Vòng đời Scene & Quản lý trạng thái

- **Khai báo chuẩn:** Mỗi màn chơi hoặc giao diện là một Scene kế thừa `Phaser.Scene`.
- **Thứ tự thực thi:** `init(data)` -> `preload()` -> `create()` -> `update(time, delta)`.
- **Đa Scene song song:** Tách riêng `GameScene` (chứa gameplay) và `UIScene` (chứa máu, điểm số, HUD) chạy đè lên nhau.
- *Tra cứu nhanh:* `.agents/skills/scenes/SKILL.md` (kèm bảng tra cứu API tại `scenes/references/REFERENCE.md`), `.agents/skills/game-setup-and-config/SKILL.md`, `.agents/skills/data-manager/SKILL.md`.

### B. Vật lý, Va chạm & Điều khiển (Physics & Controls)

- Ưu tiên sử dụng **Arcade Physics** (`this.physics.add`) cho các game tốc độ cao, va chạm hình chữ nhật/hình tròn nhẹ nhàng.
- Chỉ dùng **Matter.js Physics** khi game đòi hỏi tính toán vật lý phức tạp (khớp nối, cơ học vật rắn đa giác).
- Luôn nhân tốc độ di chuyển với `delta` time hoặc dùng vận tốc chuẩn của hệ vật lý để đảm bảo game chạy đồng đều trên màn hình 60Hz lẫn 144Hz.
- *Tra cứu nhanh:* `.agents/skills/physics-arcade/SKILL.md`, `.agents/skills/physics-matter/SKILL.md`, `.agents/skills/input-keyboard-mouse-touch/SKILL.md` (kèm bảng sự kiện tại `input-keyboard-mouse-touch/references/REFERENCE.md`).

### C. Đồ họa, Animation & Hiệu ứng 2D

- Tổ chức tài nguyên đồ họa bằng Spritesheet hoặc Texture Atlas.
- Tạo hiệu ứng nổ, đạn bay, ánh sáng dạ quang bằng hệ thống hạt (Particles) thay vì sinh quá nhiều Sprite thủ công.
- Sử dụng Tweens để tạo chuyển động UI mượt mà, đàn hồi.
- *Tra cứu nhanh:* `.agents/skills/sprites-and-images/SKILL.md` (kèm API tại `sprites-and-images/references/REFERENCE.md`), `.agents/skills/animations/SKILL.md`, `.agents/skills/particles/SKILL.md` (kèm API tại `particles/references/REFERENCE.md`), `.agents/skills/tweens/SKILL.md`, `.agents/skills/filters-and-postfx/SKILL.md`.

---

## 4. QUY CHUẨN PHÁT TRIỂN GAME 3D (THREE.JS & WEBGPU)

Khi làm việc với các thành phần 3D, Agent phải áp dụng các tiêu chuẩn chất lượng hình ảnh cao cấp:

### A. Thiết lập Renderer & Ánh sáng Chuẩn Studio

```javascript
// Thiết lập chuẩn không gian màu và ánh sáng
renderer.outputColorSpace = THREE.SRGBColorSpace;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.0;
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
```

- Phối hợp ánh sáng theo quy tắc 3 điểm: Key Light (đèn chính đổ bóng), Fill Light (đèn phụ làm mềm bóng) và Ambient/Hemisphere Light (ánh sáng môi trường).
- *Tra cứu nhanh:* `.agents/skills/threejs-exposure-color-grading/references/scene-referred-color-pipeline.md`, `.agents/skills/threejs-shadow-systems/references/cached-clipmap-shadows.md`, `.agents/skills/threejs-bloom/references/hdr-bloom-system.md`.

### B. Camera & Điều khiển 3D

- Quản lý góc nhìn người chơi mượt mà (Third-Person Chase Cam, First-Person hoặc Isometric).
- Xử lý chuyển động camera có độ trễ mượt (damped/interpolated motion) thay vì gắn cứng (hard-attachment) vào nhân vật.
- *Tra cứu nhanh:* `.agents/skills/threejs-camera-direction/references/camera-rig-and-cinematic-systems.md`.

### C. Tối ưu Hiệu năng 3D (Performance Guardrails)

- Khi hiển thị số lượng lớn vật thể cùng loại (cây cối, đạn, quái vật), bắt buộc sử dụng `THREE.InstancedMesh`.
- Giới hạn kích thước texture (tối đa 2K cho web) và luôn bật mipmap.
- Khi xóa vật thể khỏi Scene, bắt buộc dọn dẹp tài nguyên GPU:

  ```javascript
  mesh.geometry.dispose();
  if (Array.isArray(mesh.material)) {
    mesh.material.forEach(m => m.dispose());
  } else {
    mesh.material.dispose();
  }
  ```

- *Tra cứu nhanh:* `.agents/skills/threejs-procedural-vfx/references/procedural-vfx-system.md`, `.agents/skills/threejs-water-optics/references/water-surface-system.md`, `.agents/skills/threejs-visual-validation/references/graphics-validation-protocol.md`.

---

## 5. KIẾN TRÚC KẾT HỢP HYBRID 2D + 3D (NẾU CẦN)

Nếu dự án kết hợp cả 2 nền tảng trong cùng một game:

1. **Mô hình Layered Canvas:**
   - **Canvas 3D (Three.js):** Nằm ở lớp dưới cùng (`z-index: 1`), hiển thị nền không gian 3D, biển nước hoặc hiệu ứng khí quyển.
   - **Canvas 2D (Phaser):** Nằm ở lớp trên (`z-index: 2`, cấu hình `transparent: true`), hiển thị gameplay 2D, nhân vật pixel, UI và HUD.
2. **Đồng bộ Tọa độ:** Sử dụng phép chiếu `Vector3.project(camera)` để ánh xạ vị trí từ không gian 3D lên tọa độ màn hình 2D của Phaser.

---

## 6. QUY TRÌNH LÀM VIỆC CỦA AGENT KHI NHẬN TÁC VỤ

Mỗi khi người dùng đưa ra một yêu cầu mới, Agent phải thực hiện theo quy trình:

1. **Phân tích Phạm vi:** Xác định tác vụ thuộc 2D, 3D, Logic nghiệp vụ hay Cấu trúc hệ thống.
2. **Kích hoạt Kỹ năng (Tra cứu trúng đích - Tuyệt đối không đọc toàn bộ):**
   - Không duyệt quét đệ quy toàn bộ thư mục `.agents/skills/` hoặc đọc toàn bộ các file code/shader nặng nề.
   - Tra cứu tại **Mục 8 (Bản đồ cấu trúc file các skill)** để mở trực tiếp file lý thuyết (`SKILL.md`), file tra cứu API (`references/...`), hoặc code mẫu chạy được (`examples/...`).
3. **Triển khai Từng bước (Incremental Implementation):**
   - Viết code sạch, module hóa, có chú thích rõ ràng bằng Tiếng Việt.
   - Đảm bảo không làm hỏng các tính năng cũ đang chạy tốt.
4. **Kiểm tra & Rà soát:**
   - Chạy lệnh test (`npm run test`) nếu có thay đổi logic.
   - Đảm bảo cú pháp tương thích với bundler Vite hiện tại.

---

## 7. ĐỊNH HƯỚNG CÔNG NGHỆ: NEXT.JS, VITE, REACT & HTML THUẦN

Khi tư vấn hoặc thiết kế kiến trúc Web Game, Agent tuân thủ các định hướng sau:

### A. So sánh các lựa chọn nền tảng

* **Vite + Vanilla JS / TS (hoặc HTML thuần có Bundler):**
  - **Lựa chọn tối ưu nhất** cho game thuần túy (mini-game, game arcade 2D Phaser, 3D Three.js).
  - Khởi động nhanh, bundle siêu nhẹ, không gánh overhead của framework UI, dễ đóng gói lên các nền tảng web game (itch.io, Poki, CrazyGames) hoặc mobile.
- **Vite + React:**
  - Lựa chọn hoàn hảo khi game có hệ thống giao diện phức tạp (Túi đồ, Shop, Cài đặt, Khung chat, Quản lý tài khoản) hoặc làm game 3D với **React Three Fiber (R3F)**.
- **Next.js:**
  - **Không dùng cho phần lõi render game**, vì engine game chạy 100% ở Client (Canvas/WebGL), SSR của Next.js không có tác dụng và còn dễ gây lỗi thiếu đối tượng `window`/`document`.
  - **Chỉ dùng khi dự án là Cổng thông tin (Web Portal) game lớn**: Cần SEO trang chủ, blog tin tức, hệ thống Auth/Thanh toán, API backend lưu điểm/leaderboard. Khi đó, game chỉ là một component Canvas client-only (`'use client'` hoặc `dynamic import` với `ssr: false`) nhúng bên trong.

### B. Kiến trúc chuẩn khi tích hợp React với Game Engine

1. **Mô hình Phân tách (UI Overlay + Canvas Gameplay):**
   - **Canvas / WebGL (Phaser / Three.js):** Đảm nhiệm 100% vòng lặp game (`update`), đồ họa, va chạm, vật lý ở 60 FPS.
   - **React (DOM / HTML / Tailwind):** Nằm ở lớp đè phía trên (`pointer-events: none` cho nền, `pointer-events: auto` cho nút bấm/menu) để hiển thị HUD, Menu Pause, Inventory, Dialog.
2. **Nguyên tắc State sống còn:**
   - **Tuyệt đối không dùng `useState` cho vòng lặp game:** Không lưu tọa độ `(x, y)` hay vận tốc nhân vật vào React state mỗi khung hình (16ms) vì sẽ kích hoạt re-render 60 lần/giây gây sập FPS.
   - **Tách biệt State:** Tọa độ, chuyển động giữ trong Engine/Canvas hoặc `useRef`. Chỉ bắn sự kiện cập nhật React State khi có thay đổi trạng thái gián đoạn (mất máu, nhặt vật phẩm, chuyển màn, thắng/thua).
3. **Khi nào làm game hoàn toàn bằng React DOM (dùng `<div>`):**
   - Chỉ phù hợp với game thẻ bài (Card games), cờ (Chess, Caro), giải đố theo lượt (2048, Sudoku, Wordle), tiểu thuyết tương tác (Visual Novel). Không dùng cho game hành động hoặc game cần vật lý liên tục.

---

## 8. BẢN ĐỒ CẤU TRÚC FILE CÁC SKILL (SKILL FILE MAP & FAST-LOOKUP)

> **QUY TẮC SỐNG CÒN DÀNH CHO AGENT:** Tuyệt đối không đọc toàn bộ thư mục `.agents/skills/` hoặc đọc tuần tự từ đầu đến cuối một skill. Nhiều skill Three.js chứa hàng chục file shader, binary và thư viện phụ trợ nặng hàng megabyte. Hãy dùng bảng mục lục dưới đây để mở **chính xác duy nhất 1 file** giải quyết vấn đề của bạn:
>
> - **Khi cần nắm cách dùng, cú pháp & checklist:** Đọc file `SKILL.md` của skill đó.
> - **Khi cần tra cứu bảng API, thông số, sự kiện chi tiết:** Đọc file trong thư mục `references/`.
> - **Khi cần lấy code mẫu chuẩn chạy ngay:** Đọc file code chính trong thư mục `examples/`.
> - **Khi cần công cụ tự động hóa:** Dùng script trong thư mục `scripts/`.

---

### 8.1. NHÓM KỸ NĂNG PHASER 4 (GAME 2D)

#### A. Các skill có tệp tra cứu API chi tiết (`references/REFERENCE.md`)

*Mở `SKILL.md` để nắm logic tổng quát; mở `references/REFERENCE.md` khi cần tra cứu thông số, method, property hoặc event.*

| Tên Skill | Đường dẫn file chính | File tra cứu chuyên sâu (`references/`) | Khi nào cần đọc file tra cứu chi tiết |
| --- | --- | --- | --- |
| **cameras** | `.agents/skills/cameras/SKILL.md` | `references/REFERENCE.md` | Tra cứu API `BaseCamera`, bounds, viewport, ma trận camera, công thức shake/zoom/fade/follow. |
| **game-object-components** | `.agents/skills/game-object-components/SKILL.md` | `references/REFERENCE.md` | Tra cứu mixin components: Transform, Alpha, Tint, Flip, Origin, Depth, Mask, Bounds, Lighting v4. |
| **input-keyboard-mouse-touch** | `.agents/skills/input-keyboard-mouse-touch/SKILL.md` | `references/REFERENCE.md` | Tra cứu mã sự kiện con trỏ Pointer, Keyboard Key codes, HitArea tương tác, Drag & Drop, Gamepad. |
| **loading-assets** | `.agents/skills/loading-assets/SKILL.md` | `references/REFERENCE.md` | Tra cứu danh sách định dạng file nạp được, Loader events, pack files JSON, quản lý Cache bộ nhớ. |
| **particles** | `.agents/skills/particles/SKILL.md` | `references/REFERENCE.md` | Tra cứu toàn bộ cấu hình ParticleEmitter: emission zones, death zones, gravity wells, blend modes. |
| **scenes** | `.agents/skills/scenes/SKILL.md` | `references/REFERENCE.md` | Tra cứu API SceneManager, quản lý vòng đời, sleep/wake, scene transitions, truyền data giữa các Scene. |
| **sprites-and-images** | `.agents/skills/sprites-and-images/SKILL.md` | `references/REFERENCE.md` | Tra cứu bảng phương thức Sprite, Image, tint modes, frame handling, animation binds. |
| **v4-new-features** | `.agents/skills/v4-new-features/SKILL.md` | `references/REFERENCE.md` | Tra cứu chi tiết các tính năng mới độc quyền của v4: RenderNodes, Filters, SpriteGPULayer, Noise, Gradient. |

#### B. Các skill 2D độc lập (Chỉ có duy nhất 1 file `SKILL.md` - Đọc trực tiếp)

*Tất cả các skill dưới đây chỉ chứa file `SKILL.md`, đọc trực tiếp file này là đủ:*

- `actions-and-utilities/SKILL.md`: Căn chỉnh (align), bố cục dạng lưới (grid layout), thao tác batch trên nhóm GameObject.
- `animations/SKILL.md`: AnimationManager, tạo animation từ Spritesheet/Atlas, play/stop/chain, frame callbacks.
- `audio-and-sound/SKILL.md`: Tích hợp âm thanh Web Audio, SoundManager, nhạc nền lặp, hiệu ứng không gian (spatial audio).
- `curves-and-paths/SKILL.md`: Quỹ đạo chuyển động, đường cong Bézier, Spline, PathFollower cho đường đạn/quái bay.
- `data-manager/SKILL.md`: Quản lý trạng thái bằng DataManager (`setData`, `getData`), bắt sự kiện khi dữ liệu thay đổi.
- `events-system/SKILL.md`: Hệ thống sự kiện decoupled (EventEmitter, scene events, game-wide events).
- `filters-and-postfx/SKILL.md`: Áp dụng bộ lọc hậu kỳ (Bloom, Blur, Glow, Color Matrix, custom GLSL pipeline v4).
- `game-setup-and-config/SKILL.md`: Khởi tạo `new Phaser.Game(config)`, căn chỉnh Canvas, cấu hình pixelArt, FPS, ScaleMode.
- `geometry-and-math/SKILL.md`: Tiện ích toán học, Vector2, Rectangle, Circle, Polygon, Lerp, khoảng cách và góc quay.
- `graphics-and-shapes/SKILL.md`: Vẽ vector runtime bằng Graphics: đường viền, hình chữ nhật bo góc, đa giác, gradient.
- `groups-and-containers/SKILL.md`: Object Pooling (hồi sinh đạn/quái bằng `getFirstDead`), Container gom nhóm vị trí/góc xoay.
- `physics-arcade/SKILL.md`: Arcade Physics v4: trọng lực, vận tốc, ma sát, va chạm (collider), vùng kích hoạt (overlap).
- `physics-matter/SKILL.md`: Matter.js: vật lý vật rắn phức tạp, đa giác lồi/lõm, khớp nối lò xo/thanh giằng (constraints).
- `render-textures/SKILL.md`: DynamicTexture & RenderTexture: vẽ đối tượng lên texture ngoài màn hình, chụp snapshot màn hình.
- `scale-and-responsive/SKILL.md`: ScaleManager: co giãn màn hình FIT/RESIZE/ENVELOP, tự động căn giữa, chế độ toàn màn hình.
- `text-and-bitmaptext/SKILL.md`: Hiển thị chữ Text và BitmapText (phông chữ bitmap nhẹ cho pixel game), word-wrap, căn lề.
- `tilemaps/SKILL.md`: Bản đồ Tilemap từ Tiled JSON, tạo layer địa hình, cấu hình va chạm tile theo ID/property.
- `time-and-timers/SKILL.md`: Quản lý thời gian, TimerEvent, `time.delayedCall`, bộ đếm lặp vô hạn, timeScale làm chậm thời gian.
- `tweens/SKILL.md`: Chuyển động mượt bằng Tween, hàm đàn hồi (easing), chuỗi tween (tween chain), stagger dịch chuyển lệch pha.
- `v3-to-v4-migration/SKILL.md`: Hướng dẫn chuyển đổi code cũ từ Phaser 3 lên chuẩn Phaser 4 (sửa đổi renderer, pipeline, filters).

---

### 8.2. NHÓM KỸ NĂNG THREE.JS & WEBGPU (GAME 3D & KỸ XẢO)

*Mỗi skill 3D được thiết kế module hóa cao. Để không lãng phí token, hãy chọn đọc đúng tài liệu lý thuyết (`references/`) hoặc file code mẫu thực tế (`examples/`) theo nhu cầu bên dưới:*

#### A. Môi trường, Bầu trời & Khí quyển

- **`threejs-atmosphere-aerial-perspective`**: Tán xạ khí quyển theo mô hình vật lý Rayleigh/Mie.
  - `SKILL.md`: Quy trình và khái niệm cơ bản.
  - `references/atmosphere-system-contract.md`: Hợp đồng thông số kỹ thuật quang phổ, hướng mặt trời, bầu trời sao.
  - `examples/lut-aerial-perspective/atmosphere-effect.js`: Code mẫu tích hợp hiệu ứng khí quyển qua bảng tra cứu LUT.
- **`threejs-volumetric-clouds`**: Hệ thống mây thể tích động thời gian thực.
  - `SKILL.md`: Bounded raymarching mây theo thời tiết.
  - `references/weather-volume-and-reconstruction.md`: Cấu trúc 3D noise (Perlin/Worley), ánh sáng xuyên mây (silver lining), tái tạo khử nhiễu.
  - `examples/weather-volume-clouds/cloud-effect.js`: Code mẫu pass mây thể tích hoàn chỉnh.
- **`threejs-procedural-planets`**: Tạo hành tinh thủ tục nhìn từ không gian xuống mặt đất.
  - `SKILL.md`: Sinh địa hình hình cầu, lục địa, sinh quyển, LOD độ cao.
  - `references/planet-field-and-atmosphere-systems.md`: Công thức tạo trường địa hình và khí quyển hành tinh.
  - `examples/procedural-planet-surface/planet-system.js`: Mẫu khởi tạo mesh và vật liệu hành tinh.

#### B. Mặt nước & Đại dương

- **`threejs-water-optics`**: Nước quang học phân tích, sông dạ quang và hồ bơi.
  - `SKILL.md`: Khái niệm quang học mặt nước, phản xạ Fresnel, hấp thụ Beer-Lambert.
  - `references/water-surface-system.md`: Nguyên lý tổng hợp sóng đa tần và khúc xạ màn hình.
  - `references/photoreal-water.md` + `examples/photoreal-water/photoreal-water.js`: Mặt nước hồ tả thực kèm vệt tụ quang (caustics).
  - `references/luminescent-water.md` + `examples/luminescent-water/luminescent-water.js`: Dòng sông dạ quang phát sáng rực rỡ khi dòng chảy cuộn sóng.
  - `examples/interactive-pool-volume/water-volume-system.js`: Hồ bơi tương tác tạo sóng lan truyền khi vật thể chạm mặt nước.
- **`threejs-spectral-ocean`**: Biển khơi và sóng vỗ bờ mô phỏng phổ FFT.
  - `SKILL.md`: Mô phỏng đại dương quy mô lớn bằng WebGPU/TSL.
  - `references/spectral-cascade-ocean-system.md` + `examples/spectral-cascade-ocean/ocean-system.js`: Biển sâu sóng lớn đa dải bước sóng FFT, bọt trắng đỉnh sóng (Jacobian foam).
  - `references/coastal-breaker-and-swash-ocean.md` + `examples/coastal-breaker-ocean/coastal-breaker-ocean.js`: Sóng cuộn vỡ bờ ven biển và màng nước dạt trên cát ướt.
  - `examples/ocean-beach-waves/ocean-beach-waves.js`: Sóng nước nông tràn bờ bãi tắm.
  - `examples/submerged-snell-ocean/underwater-snell-ocean.ts`: Khung nhìn thợ lặn dưới đáy biển (Cửa sổ Snell, tụ quang đáy biển).
  - `examples/stylized-above-below-ocean/stylized-ocean-material.js`: Nước biển phong cách stylized nhìn xuyên thấu từ trên xuống dưới.

#### C. Bề mặt, Vật liệu Thủ tục & Thời tiết

- **`threejs-procedural-materials`**: Vật liệu PBR thủ tục cao cấp (Shader thủ tục không phụ thuộc texture nặng).
  - `SKILL.md`: Tổng quan các công thức PBR thủ tục, derivative normals.
  - `references/procedural-pbr-system.md`: Nguyên tắc vật liệu PBR và chống răng cưa phản xạ (specular AA).
  - *Kính & Thạch dẻo:* `references/dielectric-glass-optics.md` + `examples/spectral-dispersive-glass/` (Thủy tinh tán sắc); `references/softbody-jelly.md` + `examples/softbody-jelly/` (Thạch dẻo trong suốt kèm caustics).
  - *Sinh vật phát quang:* `references/luminescent-butterfly.md` (Bướm phát quang); `references/luminescent-jellyfish.md` (Sứa phát quang mô phỏng xúc tu).
  - *Mô phỏng mô mềm & Vải:* `references/simulated-cloth-system.md` + `examples/simulated-cloth/` (Vải GPU XPBD); `references/simulated-tearable-denim.md` + `examples/simulated-tearable-denim/` (Vải jean rách xơ từng sợi); `references/simulated-fur-system.md` + `examples/simulated-fur/` (Lông thú tương tác chuyển động).
  - *Hiệu ứng đặc biệt:* `references/simulated-paper-burning.md` + `examples/simulated-paper-burning/` (Giấy cháy lan ra than hồng); `references/thin-film-soap-bubble-system.md` + `examples/thin-film-soap-bubbles/` (Bong bóng xà phòng giao thoa ngũ sắc Airy); `references/physical-diffraction-grating.md` (Cách tử tán xạ đĩa CD); `examples/lava-flow-surface/` (Dung nham nóng chảy); `examples/raytraced-diamond/` (Kim cương khúc xạ); `examples/deformable-sand/` (Cát lún biến dạng).
- **`threejs-temporal-surfaces`**: Bề mặt tương tác tích lũy theo thời gian.
  - `references/ping-pong-accumulation.md` + `examples/touch-history-frost/frost-surface-effect.js`: Băng tuyết tan dần khi có dấu tay miết qua.
  - `references/refractive-window-rain.md` + `examples/refractive-window-rain/window-rain-effect.js`: Giọt mưa trượt trên cửa kính ô tô làm nhòe và khúc xạ quang cảnh phía sau.
- **`threejs-precipitation-surfaces`**: Tương tác bề mặt mưa tuyết.
  - `references/precipitation-surface-systems.md`: Nguyên lý tích tụ tuyết trên bề mặt hướng lên và vũng nước mưa.
  - `examples/snow-accumulation/snow-system.js`: Phủ tuyết tự nhiên lên mô hình 3D.
  - `examples/wet-puddle-rain/rain-puddle-system.js`: Mặt đường ướt mưa có vũng nước gợn sóng phản chiếu.
- **`threejs-parallax-occlusion-mapping`**: Nổi khối bề mặt parallax chuyên sâu.
  - `references/silhouette-relief-contract.md` + `examples/silhouette-relief/ParallaxOcclusion.js`: Kỹ thuật POM xử lý viền phẳng và cong chống lẹm hình.

#### D. Dựng hình Thủ tục (Procedural Geometry) & Kiến trúc

- **`threejs-procedural-geometry`**: Dựng mô hình 3D bằng code chuẩn kỹ thuật (không lỗi hở lưới, coplanar flicker).
  - `SKILL.md`: Tổng quan dựng hình hard-surface và mô hình phức tạp.
  - `references/geometry-craft-workflow.md`: Kỹ thuật bo góc (bevel), vát góc (fillet), đục khoét (aperture), độ dày vỏ.
  - `references/geometry-quality-gates.md`: Bộ tiêu chuẩn nghiệm thu chất lượng lưới (manifold, pháp tuyến nhất quán).
  - `references/profile-sweeps-and-mesh-writers.md`: Kỹ thuật đùn biên dạng, quét cung tròn và loft mặt cong.
  - `references/vehicle-loft-and-projector-contract.md`: Thiết kế vỏ xe và phương tiện khí động học.
  - `references/complete-submarine-assembly.md`: Quy trình lắp ráp tàu ngầm hoàn chỉnh.
  - `references/procedural-optimus-humanoid-assembly.md`: Quy trình lắp ráp robot hình người humanoid.
  - `examples/`: Code dựng hình mẫu đầy đủ: `formula-one-race-car`, `sport-motorcycle`, `porcelain-brass-submarine`, `procedural-optimus-humanoid`, `sculpted-gallery-frame`.
- **`threejs-procedural-architecture`**: Sinh công trình kiến trúc và tòa nhà thành phố.
  - `references/grammar-and-mesh-compiler.md` + `examples/procedural-financial-tower/building-system.js`: Ngữ pháp phân tầng mặt tiền, ô cửa sổ, phào chỉ và biên dịch mesh tối ưu draw call.
- **`threejs-procedural-vegetation`**: Thực vật, cây xanh và thảm thực vật.
  - `references/structured-ash-growth-system.md` + `examples/structured-ash-growth/tree-system.js`: Cây tần bì phân nhánh đệ quy (thân, cành, cụm lá).
  - `references/gpu-culled-flower-field.md` + `examples/gpu-culled-flower-field/`: Cánh đồng hoa bạt ngàn tối ưu GPU culling.
  - `examples/gpu-computed-grass/gpu-grass-system.js`: Thảm cỏ tính toán GPU đu đưa theo gió.
  - `examples/procedural-surface-ivy/ivy-effect.js`: Dây leo bám tường đá tự nhiên.
  - `examples/stylized-meadow-grass/grass-system.js`: Cỏ đồng quê stylized mượt mà.

#### E. Kỹ xảo Trực quan (VFX) & Không gian Vũ trụ

- **`threejs-procedural-vfx`**: Hệ thống hiệu ứng kỹ xảo thời gian thực chuẩn phim ảnh.
  - `references/procedural-vfx-system.md`: Quản lý pool VFX, cấp bậc phát sáng HDR, vòng đời hạt.
  - `references/filmic-lens-flare.md` + `examples/filmic-lens-flare/filmic-lens-flare.js`: Vệt lóe sáng ống kính điện ảnh (Lens Flare).
  - `references/volumetric-aurora-curtains.md` + `examples/raymarched-aurora-curtains/aurora-curtains.js`: Dải cực quang uốn lượn raymarched.
  - `references/volumetric-fluid-fire.md` + `examples/volumetric-fluid-fire/volumetric-fluid-fire.js`: Lửa và khói chất lưu thể tích WebGPU.
  - `examples/holographic-shape-transition/`: Quét hologram phân rã và biến đổi hình dáng vật thể.
  - `examples/reentry-plasma/`: Lớp plasma rực lửa bao quanh tàu không gian khi xuyên khí quyển.
- **`threejs-raymarched-space-effects`**: Hiện tượng vũ trụ & vật lý thiên văn tương đối tính.
  - `references/curved-ray-integrators.md`: Raymarching tích phân quỹ đạo tia sáng cong trong trường hấp dẫn.
  - `references/lensed-celestial-spheres.md`: Bầu trời sao và thiên hà biến dạng qua thấu kính hấp dẫn.
  - `examples/schwarzschild-geodesic-black-hole/`: Hố đen Schwarzschild bẻ cong ánh sáng.
  - `examples/traversable-wormhole-transit/`: Lỗ sâu không gian xuyên thời không (wormhole).
  - `examples/curved-ray-accretion-volume/`: Đĩa bồi tụ phát quang rực rỡ quanh miệng hố đen.

#### F. Rendering Pipeline, Camera, Ánh sáng & Kiểm thử 3D

- **`threejs-bloom`**: `references/hdr-bloom-system.md` (Cấu hình Selective Bloom 2 lớp, ngưỡng sáng và khôi phục vật liệu an toàn).
- **`threejs-camera-direction`**: `references/camera-rig-and-cinematic-systems.md` (Chase rig bám đuôi, giảm chấn quán tính mượt mà, quaternion xoay mượt, chống kẹt camera vào tường).
- **`threejs-exposure-color-grading`**: `references/scene-referred-color-pipeline.md` (Đo sáng độ chói trung bình thích ứng mắt người, tone mapping duy nhất, 3D LUT grading 32-cube).
- **`threejs-image-pipeline`**: `references/production-image-pipeline.md` (Thứ tự chuẩn kết hợp các pass: Depth, Normal, GTAO, Bloom, Tone Mapping).
- **`threejs-procedural-animation`**: `references/procedural-motion-and-docking-systems.md` (Quỹ đạo bay, docking vào trạm xoay, lò xo giảm chấn không phụ thuộc FPS).
- **`threejs-procedural-fields`**: `references/field-stack-recipes.md` (Công thức xếp chồng các tầng noise cho địa hình, độ gồ ghề, mài mòn, độ ẩm).
- **`threejs-screen-space-ambient-occlusion`**: `references/gtao-bent-normal-pipeline.md` (Quy trình GTAO nửa độ phân giải, bent normal, bilateral filter khử nhiễu).
- **`threejs-shadow-systems`**: `references/cached-clipmap-shadows.md` (Bóng đổ thế giới lớn, clipmap cache và ổn định texel chống nhấp nháy bóng khi di chuyển).
- **`threejs-visual-validation`**: `references/graphics-validation-protocol.md` (Bộ quy chuẩn kiểm định đồ họa: góc camera cố định, baseline không hậu kỳ, đo đạc GPU budget).
- **`threejs-skill-router`**: `SKILL.md` (Cây quyết định phân luồng tác vụ 3D để chọn đúng skill chuyên biệt).

---

### 8.3. NHÓM KỸ NĂNG TÀI NGUYÊN, Ý TƯỞNG & QUY TRÌNH KỸ THUẬT

#### A. Kỹ năng Quản lý Tài nguyên & Ý tưởng (Có nhiều file tài liệu & script)

- **`create-game-assets`**: Quy trình thiết kế, sinh và kiểm tra tài nguyên đồ họa game.
  - `SKILL.md`: Nguyên tắc tổng quan và tiêu chuẩn nghệ thuật.
  - `references/art-direction.md`: Định hướng nghệ thuật, phong cách thị giác và bảng màu.
  - `references/raster-pipeline.md`: Quy trình xử lý ảnh 2D raster/pixel, xử lý nền giả trong suốt (fake transparency).
  - `references/three-d-pipeline.md`: Quy chuẩn tài nguyên 3D (lưới, map PBR, định dạng file).
  - `references/provenance.md`: Quản lý bản quyền, giấy phép (license) và kê khai Steam AI disclosure.
  - `assets/art-direction-brief.md`: Mẫu lập brief định hướng nghệ thuật cho game.
  - `scripts/build_preview_sheet.py`: Script Python tự động ghép sprite thành ảnh preview sheet.
  - `scripts/asset_report.py`: Script Python quét và xuất báo cáo kiểm tra tài nguyên.
- **`idea-refine`**: Tinh chỉnh và sàng lọc ý tưởng game.
  - `SKILL.md`: Quy trình hội tụ và phân kỳ ý tưởng.
  - `frameworks.md`: Các khung đánh giá tiềm năng gameplay và core loop.
  - `refinement-criteria.md`: Bộ tiêu chí mài giũa tính khả thi của trò chơi.
  - `examples.md`: Kịch bản mẫu quá trình tinh chỉnh ý tưởng game thực tế.
  - `scripts/idea-refine.sh`: Script thực thi quy trình idea-refine.

#### B. Các kỹ năng Quy trình Kỹ thuật & Chất lượng Mã nguồn (Chỉ có duy nhất `SKILL.md` - Đọc trực tiếp)

*Tất cả các skill sau đây chỉ cần mở trực tiếp file `SKILL.md` khi đối mặt với tác vụ tương ứng:*

| Tên Skill | Đường dẫn | Khi nào cần kích hoạt |
| --- | --- | --- |
| **frontend-ui-engineering** | `.agents/skills/frontend-ui-engineering/SKILL.md` | Khi thiết kế giao diện game (HUD, Menu, Dialog), đảm bảo responsive và trải nghiệm người dùng cao cấp. |
| **api-and-interface-design** | `.agents/skills/api-and-interface-design/SKILL.md` | Khi thiết kế ranh giới module, tạo contract giữa UI frontend và logic game lõi. |
| **test-driven-development** | `.agents/skills/test-driven-development/SKILL.md` | Khi triển khai logic nghiệp vụ mới, viết test Vitest trước hoặc song song để đảm bảo tính đúng đắn. |
| **debugging-and-error-recovery** | `.agents/skills/debugging-and-error-recovery/SKILL.md` | Khi gặp lỗi sập game, bug logic khó hiểu; áp dụng phương pháp truy tìm nguyên nhân gốc rễ (Root Cause). |
| **performance-optimization** | `.agents/skills/performance-optimization/SKILL.md` | Khi game bị giật lag tụt FPS, rò rỉ bộ nhớ (leak memory), hoặc số lượng draw calls quá lớn. |
| **doubt-driven-development** | `.agents/skills/doubt-driven-development/SKILL.md` | Khi chuẩn bị đưa ra quyết định kiến trúc rủi ro cao hoặc thay đổi lớn khó đảo ngược. |
| **incremental-implementation** | `.agents/skills/incremental-implementation/SKILL.md` | Khi thực hiện tính năng chạm vào nhiều file, chia nhỏ thành các bước kiểm chứng được. |
| **planning-and-task-breakdown** | `.agents/skills/planning-and-task-breakdown/SKILL.md` | Khi nhận yêu cầu phức tạp từ người dùng, lập kế hoạch và phân rã các đầu việc tuần tự. |
| **git-workflow-and-versioning** | `.agents/skills/git-workflow-and-versioning/SKILL.md` | Khi commit code, tách nhánh tính năng hoặc giải quyết xung đột mã nguồn. |
| **code-review-and-quality** | `.agents/skills/code-review-and-quality/SKILL.md` | Khi rà soát mã nguồn tự viết hoặc trước khi xác nhận hoàn thành tác vụ lớn. |
| **code-simplification** | `.agents/skills/code-simplification/SKILL.md` | Khi tái cấu trúc mã nguồn (refactoring) để code sáng sủa, dễ bảo trì, loại bỏ phức tạp thừa. |
| **security-and-hardening** | `.agents/skills/security-and-hardening/SKILL.md` | Khi xử lý dữ liệu người dùng, chống hack/cheat client-side, bảo vệ state game. |
| **browser-testing-with-devtools** | `.agents/skills/browser-testing-with-devtools/SKILL.md` | Khi kiểm thử thực nghiệm trên trình duyệt, soi DOM, console error hoặc network request qua DevTools. |
| **ci-cd-and-automation** | `.agents/skills/ci-cd-and-automation/SKILL.md` | Khi thiết lập pipeline tự động kiểm thử và đóng gói bản build. |
| **shipping-and-launch** | `.agents/skills/shipping-and-launch/SKILL.md` | Khi chuẩn bị đưa game lên các cổng phát hành (itch.io, web hosting, mobile). |
| **source-driven-development** | `.agents/skills/source-driven-development/SKILL.md` | Khi cần tra cứu tài liệu chính thống của framework để tránh code theo giả định sai lệch. |
| **spec-driven-development** | `.agents/skills/spec-driven-development/SKILL.md` | Khi yêu cầu chưa rõ ràng, lập văn bản đặc tả yêu cầu chi tiết trước khi code. |
| **documentation-and-adrs** | `.agents/skills/documentation-and-adrs/SKILL.md` | Khi ghi lại các quyết định kiến trúc quan trọng cho dự án (ADR). |
| **deprecation-and-migration** | `.agents/skills/deprecation-and-migration/SKILL.md` | Khi loại bỏ hoặc chuyển đổi các module/API cũ sang module mới. |
| **context-engineering** | `.agents/skills/context-engineering/SKILL.md` | Khi tối ưu hóa không gian ngữ cảnh (context window) của AI agent. |
| **using-agent-skills** | `.agents/skills/using-agent-skills/SKILL.md` | Meta-skill hướng dẫn cách phát hiện và điều hướng các kỹ năng trong hệ thống. |
