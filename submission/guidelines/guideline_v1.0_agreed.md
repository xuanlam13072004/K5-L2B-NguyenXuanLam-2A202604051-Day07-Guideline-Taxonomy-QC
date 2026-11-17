# Guideline gán nhãn — v1.0 (agreed) — nhóm 1 người (101)

- Phiên bản: v1.0 (làm một mình, không có họp thống nhất; văn bản hóa sau khi gán, mô tả đúng quy tắc đã áp dụng khi gán)
- Ngày: 2026-11-17
- Người viết / duyệt: 101 (làm bài một mình, không có người duyệt thứ hai)

## 1. Mục tiêu
Nhãn bounding box 2D để huấn luyện mô hình phát hiện **người và phương tiện tham gia giao thông thật** trên ảnh đường phố (camera giao thông, hỗ trợ lái).
Sai nguy hiểm nhất: **bỏ sót người hoặc xe thật** (mô hình không "thấy" người đi đường). Sai thứ hai: **gán vật không thật** (tượng, đồ chơi, hình in, phản chiếu) khiến mô hình báo động giả.

## 2. Lớp và thuộc tính

### Lớp
| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Người thật nhìn thấy được: đi bộ, đứng, ngồi, người lái hoặc ngồi trên xe. | img_014 (2 người băng qua đường), img_026 (2 người trên 1 xe máy) | Tượng (img_017), ma-nơ-canh (img_033), người trên biển quảng cáo (img_018), linh vật (img_019) |
| `car` | Ô tô con thật chở người: sedan, hatchback, SUV, wagon, taxi, kể cả xe đang được chở trên xe khác. | img_015, img_023, các xe trên sàn img_002 | Mô hình xe trong tủ kính (img_038), xe trong gương (img_020) |
| `truck` | Xe chở hàng có thùng/khoang hàng tách khỏi cabin: pickup, xe tải nhỏ, xe tải thùng, xe chở ô tô. | img_001 (xe tải nhỏ), img_007, img_028, img_036, img_037 | Bia mộ hình pickup (img_027) |
| `van` | Xe thùng kín cỡ nhỏ một khối liền (cabin và khoang chung). | — (lô này không có ca chắc chắn) | Xe 16 chỗ img_005 → xem R-05 |
| `bus` | Xe chở khách cỡ lớn hoặc minibus nhiều hàng ghế, cửa lên xuống riêng. | img_009, img_031, img_037 | |
| `motorcycle` | Xe hai bánh có động cơ, kể cả xe gắn thùng bên. | img_006, img_021, img_022 (sidecar: 1 box) | Mô tô trang trí trưng bày → R-07 |
| `bicycle` | Phương tiện không động cơ đạp bằng chân: xe đạp, xe đạp chở hàng, xích lô. | img_006, img_029, img_023 (xích lô) | |
| `three_wheeler` | Xe ba bánh có động cơ chở khách/hàng: tuk-tuk, auto-rickshaw. | img_004, img_016, img_035, img_040 | Xích lô (không động cơ) → `bicycle` |
| `ignore_region` | Vùng không gán từng vật; bỏ sót trong vùng này không tính lỗi. | img_025, img_031 (cụm vật quá nhỏ ở xa) | |

Vật **không thuộc lớp nào** (cọc giao thông, động vật, xe đẩy tay, tượng, đồ chơi): **không gán box**.

### Thuộc tính
| Thuộc tính | Mức | Ngưỡng |
|---|---|---|
| `occluded_level` | `0` | bị che < 10% diện tích ước lượng của vật |
| | `1` | che 10–30% |
| | `2` | che 31–70% |
| | `3` | che > 70% nhưng vẫn nhận ra chắc chắn lớp |
| `truncated` | `yes` / `no` | `yes` khi bất kỳ phần nào của vật nằm ngoài khung ảnh |
| `can_review` | tick / không | Tick khi chưa chắc lớp hoặc chưa chắc vật là thật; không tick chỉ vì vật khó vẽ |

Hàng rào thưa, lưới, kính trong suốt: không tính là che (img_018).

## 3. Quy tắc vẽ box
- **R-01 Phần nhìn thấy:** box ôm sát phần nhìn thấy của vật, không đoán phần bị che.
- **R-02 Người và xe tách riêng:** người đi xe máy / xe đạp / tuk-tuk = 1 box `person` cho mỗi người + 1 box cho xe. Box người gồm cả mũ bảo hiểm; box xe gồm bánh và thân xe, được phép chồng lên chân người (img_026: 2 `person` + 1 `motorcycle`).
- **R-03 Vật không thật:** tượng, mộ, ma-nơ-canh, đồ chơi, mô hình, hình in/quảng cáo, tranh tường, linh vật: **không gán** (img_017, 018, 019, 027, 028, 033, 038).
- **R-04 Phản chiếu và xe chủ:** vật trong gương, trên kính, mặt đường ướt và thân xe của người chụp: **không gán** (img_015, 020, 031, 033).
- **R-05 Xe chở người cỡ lớn:** xe 16 chỗ / minibus chở khách → `bus` (img_005, img_009). `van` chỉ dùng cho xe thùng kín chở hàng.
- **R-06 Xe chở xe / chở hàng:** xe chở = 1 box `truck` (cabin + rơ-moóc), mỗi ô tô thật trên sàn = 1 box `car` riêng (img_002). Hàng nhô ra khỏi xe **không** nằm trong box xe (img_029, img_035). Vật không thật trên xe thật: chỉ gán xe (img_028).
- **R-07 Xe thật nhưng trang trí / trưng bày:** vẫn gán theo lớp nếu là xe thật có bánh và có thể chạy, tick `can_review` (img_011, img_019, img_024).
- **R-08 Vật nhỏ:** cao < 20 px → không vẽ box lớp; cụm vật nhỏ ở xa → 1 `ignore_region` (img_025, img_031).
- **R-09 Đám đông:** còn tách được từng cá thể ≥ 20 px thì gán từng vật (img_021, img_032); không tách được thì `ignore_region`.
- **R-10 Ảnh nhòe / tối / đen trắng:** vẫn gán nếu nhận ra lớp; không chắc thì tick `can_review` (img_013, 014, 022).

## 4. Ca khó
| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_002 | Xe chở 7 ô tô | R-06 | 1 `truck` + từng `car` | Ô tô trên sàn là xe thật, mô hình cần thấy |
| img_005 | Xe 16 chỗ màu bạc | R-05 | `bus` | Chở khách nhiều hàng ghế; **cần hỏi reviewer** vì dễ nhầm `van` |
| img_011 | Hình nhân lớn + mô tô trang trí | R-03, R-07 | Không gán hình nhân; mô tô gán + `can_review` | **Cần hỏi reviewer** |
| img_017 | Tượng người cưỡi ngựa | R-03 | Không gán | Không phải người thật |
| img_020 | Ô tô trong gương chiếu hậu | R-04 | Không gán | Phản chiếu |
| img_023 | Xích lô | — | `bicycle` | Không động cơ |
| img_024 | "Xe" vàng giả trên nóc xe đen | R-07 | `car` + `can_review` | **Cần hỏi reviewer**: có thể là vỏ xe giả (R-03) |
| img_027 | Bia mộ hình pickup | R-03 | Không gán | Không phải xe |
| img_028 | Pickup thật chở tượng ngựa | R-03, R-06 | Chỉ gán `truck` | Tượng không gán |
| img_038 | Tủ kính mô hình ô tô | R-03 | Không gán | Đồ chơi |

## 5. QC và phiên bản
Ngưỡng QC theo `taxonomy.json` (`rules`, giữ nguyên). Ca mơ hồ: tick `can_review`, ghi vào bảng mục 4.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v1.0 | Thêm `bus`, `bicycle`, `three_wheeler`; ngưỡng % cho `occluded_level`; R-01…R-10 | img_004, 009, 023, 035 (xe ngoài taxonomy); img_017, 020, 027, 038 (vật không thật) |
