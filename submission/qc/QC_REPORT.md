# Báo cáo QC — 101 (làm một mình)

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và nhãn bbox 2D (người, phương tiện) trên 40 ảnh đường phố cho bài thực hành; chỉ một người gán, chưa đo đồng thuận nên KHÔNG dùng để huấn luyện / đánh giá mô hình.
- Nguồn và phép sử dụng dữ liệu: data/ATTRIBUTION.csv (CC BY, CC BY-SA, CC0, public domain); giữ nguyên ghi công.
- Manifest và checksum nhãn cuối: Chưa có nhãn cuối / batch_manifest.json (build_final.py cần IAA). Nhãn hiện có: submission/labels/101.json, SHA-256 6439fce9919767342164f49b3e25d15b2627d0e41c4fa51f4c8de1203e46262c.
- Guideline và taxonomy khi gán / khi bàn giao: Gán bằng bộ 9 lớp trên CVAT; bàn giao guideline_v1.0_agreed.md + taxonomy.json v1.0 (9 lớp, rules giữ nguyên). Không có v1.1 vì chưa calibration.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 40/40 lượt của 101; lượt thứ hai 0/40. split_assign.py yêu cầu ≥ 2 thành viên nên không tạo được assignment.csv.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: N/A — 0 cặp người gán.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: N/A. Thống kê một người: 518 annotation (person 269, motorcycle 126, car 76, bicycle 18, truck 13, three_wheeler 8, bus 6, ignore_region 2); 5 ảnh không có vật (img_017, 020, 027, 033, 038).
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Không kiểm được (không có người thứ hai, chưa đối chiếu ca tham chiếu của coach).

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Chưa có — không có calibration_candidates.csv.
- Tác động v1.1 lên nhãn và ảnh: Không có v1.1.
- Tham chiếu bằng chứng khi Rework: Không áp dụng.

## 4. Kết quả duyệt

- Vòng QC: 1 (chưa thực hiện bằng script)
- Người review và ngày: 101, 2026-11-17 (tự kiểm cấu trúc, không phải review độc lập)
- Đã duyệt / cả lô: 0/40 theo phiếu qc_review.csv (chưa duyệt)
- Ảnh lỗi / ảnh đã duyệt: N/A (0 ảnh đã duyệt)
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: N/A (0 annotation đã duyệt)
- Vật bỏ sót, ảnh và vùng: Chưa duyệt.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Tự kiểm phát hiện cần xem lại: lớp van có 0 box, xe 16 chỗ img_005 gán bus (R-05); can_review tick ở 256/518 box (49%), quá cao so với mục đích cờ; img_024 "xe" vàng gán car (R-07 vs R-03).
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy kiểm được: 518 box đều hợp lệ về lớp, thuộc tính, tọa độ, không box nào cao < 20 px (kiểm bằng hàm label_errors của common.py với taxonomy v1.0). Cần người: box có ôm sát, có đúng là vật thật (img_011, img_024) hay không — script không biết.

## 5. Quyết định bàn giao

- Quyết định: Hold
- Lý do, người nhận và phạm vi sử dụng: Thiếu lượt gán độc lập thứ hai nên không có IAA, calibration, nhãn cuối và vòng QC; chỉ nộp làm bằng chứng học tập cho coach, không dùng huấn luyện.
- Nếu còn việc: Cần ≥ 1 người gán độc lập 40 ảnh theo guideline_v1.0_agreed.md (người xử lý: do coach chỉ định), rồi chạy split_assign → qc_compare → build_final → qc_sample/qc_report.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca img_002 → R-06 (guideline_v1.0_agreed.md mục 3) → submission/labels/101.json có 1 truck (id 15) + 7 car; khớp quy tắc.
