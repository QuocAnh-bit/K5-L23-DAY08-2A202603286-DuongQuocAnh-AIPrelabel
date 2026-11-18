# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603286 - Dương Quốc Anh

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe ba bánh có mui/khung và xe chở hàng ba bánh thuộc lớp nào? | Gán `three_wheeler`; người lái là một `person` riêng, không gộp vào box xe. | `img_013.jpg`, `img_004.jpg` | `CAL-01` |
| 2 | Hàng hoặc giỏ cồng kềnh có nằm trong box phương tiện không? | Nếu phần hàng nhô quá khoảng 20% chiều dài phương tiện, box chỉ ôm phần phương tiện nhìn thấy; không ôm hàng rời. | `img_003.jpg` | `CAL-02` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_003.jpg` | Box xe ba bánh ôm cả phần hàng và bạt lớn | Pre-label giữ box lớn quanh toàn bộ xe và hàng | Thu box về phần phương tiện; đánh dấu `can_review` nếu ranh giới hàng/xe không rõ | Mục 4.1.5: hàng cồng kềnh nhô quá khoảng 20% không nằm trong box phương tiện |
| `img_031.jpg` | Box bicycle ôm cả hai giỏ hàng | Box `bicycle` bao trùm cả xe và hàng ở hai bên | Chỉ giữ phần xe đạp; không kéo box theo giỏ/hàng nhô rộng | Mục 4.1.5: hàng cồng kềnh tách khỏi box xe |
| `img_013.jpg` | Mask tuk-tuk dính người lái | SAM 1 và SAM 2.1 đều có vùng mask chạm người lái trong cabin | Thêm điểm âm trên người lái hoặc sửa polygon/mask; người lái vẫn là nhãn riêng | Mục 4.2: click âm để tách người lái khỏi xe |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt lớp `three_wheeler`, quy tắc hàng cồng kềnh và thao tác điểm âm SAM khi dính người lái. | `CAL-01`, `CAL-02`, `CAL-03`; ví dụ `img_013.jpg`, `img_003.jpg`, `img_031.jpg` |
