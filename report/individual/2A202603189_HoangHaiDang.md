# Báo cáo thực hành Day 3 — Segmentation

- Mã học viên theo lớp: 2A202603189
- Ngày / CVAT local: 11/11/2026 / CVAT local
- Công cụ đã dùng: CVAT local (Brush để tự vẽ object đầu tiên, SAM3 trong CVAT cho các object còn lại)

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | submissions/easy_semantic.zip | 3 / 3 | |
| medium_instance | submissions/medium_instance.zip | 3 / 3 | |
| hard_panoptic | submissions/hard_panoptic.zip | 2 / 2 |  |
| cp1_holes | submissions/cp1_holes.zip | 1 / 1 |  |
| cp2_slice | submissions/cp2_slice.zip | 1 / 1 |  |
| cp5_occlusion | submissions/cp5_occlusion.zip | 1 / 1 |  |
| cp3_thin | submissions/cp3_thin.zip | 1 / 1 |  |
| cp4_curb | submissions/cp4_curb.zip | 1 / 1 |  |
| cp6_coverage | submissions/cp6_coverage.zip | 1 / 1 | |
| **Tổng tối đa** | | |  |

**Ghi chú export:** `easy_semantic.zip` và `medium_instance.zip` được export từ task dùng chung danh sách 12 label của `hard_panoptic`, nên `labelmap.txt` (Easy) và `categories` (Medium) có thêm label ngoài `classes.json` của task. Khi đó `scripts/inspect_submissions.py` báo LỖI "label/category ngoài classes.json". Mask thực tế chỉ dùng đúng class của task:
- Easy: chỉ có road, sidewalk, building, vegetation, sky (không có pixel nào thuộc label thừa).
- Medium: chỉ gồm person, car, motorcycle, bus, truck (không có annotation nào thuộc label thừa).

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: ảnh `000000181542.jpg`, người phụ nữ mặc áo dài trắng đang đi qua vạch sang đường ở giữa ảnh.
- Class và quy tắc tôi dùng để chọn biên: `person`. Vẽ theo phần nhìn thấy, gồm cả tóc, hai tay và tà áo dài; dừng biên ở chỗ người tiếp giáp với xe máy phía sau, không tô sang xe. Người này là một object riêng, tách khỏi người lái và xe máy xung quanh.
- Nếu dùng gợi ý sau đó: dùng SAM3 cho các object còn lại, rồi kiểm lại class và số object trong danh sách Objects. Giữ người lái và xe máy là hai object riêng (ví dụ người lái #20 và xe máy #10 ở ảnh `000000181542.jpg`), vì quy tắc instance tính mỗi vật là một mask. Ở mép trái ảnh `000000373353.jpg`, vật bị trùm bạt chỉ lộ bánh xe được giữ class `motorcycle`, vì phần nhìn thấy là bánh và hình dáng xe máy.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `cp3_thin` / `839f7736-abe28069.jpg`, cánh tay ngang của các cột đèn đường: một đèn bên trái (cạnh cột đèn giao thông) và cụm đèn ở giữa bên phải, phía sau ô tô
- Lỗi thuộc loại: thiếu nét mảnh (phủ vùng)
- Bằng chứng tôi nhìn thấy: xem lại mask bản export đầu, thân cột đèn đã là `pole` nhưng các cánh tay đèn mảnh vươn ngang ra đường bị bỏ trống, phần lớn nằm ở vùng chưa gán nhãn sát rặng cây.
- Quy tắc và hành động sửa: trạm `cp3_thin` yêu cầu tô nét mảnh ở zoom lớn bằng brush nhỏ. Tôi phóng to, dùng brush nhỏ tô thêm các cánh tay đèn thành `pole` (khoảng 2.100 pixel), không đổi các vùng `sky`, `road`, `traffic sign` khác.
- Sau sửa đã Save và export lại chưa? 

Kết quả tự đánh giá (nếu có): chưa có điểm.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `cp1_holes` / `000000144300.jpg`: nhà caravan trắng bên trái và xe van xanh phía sau xe máy | (a) gán `truck` vì là phương tiện lớn có bánh; (b) caravan là nhà kéo, không thuộc class nào nên để trống; van có thể là `car` | Trạm chỉ có 6 class person/bicycle/car/motorcycle/bus/truck; không có class trailer/van riêng | Gán cả hai là `truck` (2 object). Hỏi coach: caravan không có đầu kéo có nên để trống không? |
| `cp4_curb` / `7d83710e-4697c3b2.jpg`: khu bê tông có SUV trắng đỗ, bên phải vỉa hè | (a) `road` vì là chỗ cho xe vào đỗ; (b) `sidewalk` vì cùng mặt bê tông, cùng độ cao với vỉa hè | Quy tắc: ranh theo chức năng/bó vỉa, không theo màu; đây là bãi đỗ riêng, không phải lòng đường hay lối đi bộ | Để trống, chỉ tô road bên trái bó vỉa và dải vỉa hè đi bộ. Hỏi coach: bãi đỗ xe nên tính là road hay để trống? |
| `cp6_coverage` / `7daa6479-67988f3f.jpg`: xe buýt MTA ở giữa và xe tải CAPITAL bên trái | (a) gán `car` cho đủ phủ vùng vì là phương tiện; (b) để trống vì trạm không có class bus/truck | `classes.json` của cp6 chỉ có road, sidewalk, building, vegetation, sky, car, person; phiếu quy tắc: "không ép đoán cho đủ coverage" | Để trống xe buýt và xe tải; chỉ tô `car` cho xe con (xe đen bên trái, taxi ở xa). |
