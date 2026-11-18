# Báo cáo Day 08 — 2A202603286 - Dương Quốc Anh

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | mặc định 0.30; motorbike 0.25 | r01/r02; chọn ngưỡng trên `threshold_sweep.csv` của 4 ảnh gold nên chỉ dùng làm mốc in-sample |
| three_wheeler | YOLOE-26s, prompt `three_wheeler` | 0.15 | r03; bổ sung lớp mà YOLO26 thường gọi nhầm là car/truck |

- Lệnh `merge_prelabels.py` đã dùng: `merge_prelabels.py work\prelabel\yoloe_all\pre.coco.json work\prelabel\y26_all\pre.coco.json --thr "motorbike=0.25,three_wheeler=0.15" --out work\prelabel\merged_review\pre.coco.json`.
- **Phiên bản SAM bạn chọn:** SAM 2.1 base (`sam2.1_b.pt`) vì biên mask tốt hơn trong `img_004.jpg` và `img_017.jpg`; SAM 1 Mobile được chạy thêm để so sánh trên CPU. SAM 2.1 chậm hơn nhưng phù hợp ca khó.
- Lỗi không sửa được chỉ bằng đổi ngưỡng: YOLO26 có thể gọi tuk-tuk/xe ba bánh là `car` hoặc `truck`; xử lý bằng YOLOE prompt `three_wheeler`, merge ưu tiên YOLOE và kiểm tra lớp thủ công.
- Box điểm cao mà sai: `img_003.jpg`, object `16`, confidence 0.90 nhưng box ôm cả phần hàng cồng kềnh. Box điểm thấp cần kiểm lại: các motorbike confidence khoảng 0.28–0.36 trong `img_006.jpg` và `img_035.jpg` vẫn có thể là vật thật, nên không xoá chỉ theo confidence.
- Giới hạn: ngưỡng được chọn trên chính 4 ảnh gold nên precision/recall trên gold là số đo in-sample, có thể lạc quan; cần đọc cùng mẫu số và khoảng tin cậy.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 503 | 503/567 = 88.7% | 567 box máy |
| edited | 19 | 19/567 = 3.4% | 567 box máy |
| deleted | 45 | 45/567 = 7.9% | 567 box máy |
| added | 20 | 20/542 = 3.7% | 542 đối tượng sau duyệt |

- Lớp máy tệ nhất là `three_wheeler`: 21 box bị xoá, 7 bị sửa và chỉ 15 được giữ nguyên; lỗi chính là model đóng gán nhầm hoặc box lỏng quanh xe ba bánh. Ví dụ thấy ở `img_003.jpg` và các ca tuk-tuk trong CP5.
- So với gold: precision / recall của **máy** là **72.6% / 80.4%** (mẫu số: **56 đối tượng gold**, TP=45, FP=17, FN=11).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **6/12 = 50.0%**; lọt 3 bẫy `loose` (`img_021`, `img_029`, `img_025`), 2 bẫy `missing` (`img_035`, `img_027`) và 1 bẫy `wrong_class` (`img_033`). Nguyên nhân là bước quét ảnh độc lập trước khi nhìn nhãn máy chưa được thực hiện đủ chặt.
- Accepted khá cao (**503/567 = 88.7%**) nhưng bắt bẫy thấp (**6/12 = 50.0%**), cho thấy tôi có xu hướng chấp nhận box nhìn hợp lý mà chưa kiểm tra lại toàn cảnh, đặc biệt với box lỏng, vật bị bỏ sót và sai lớp.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): chưa ghi được chính xác trong lúc làm nên không dùng thời gian để kết luận; đây là hạn chế của log.
- Kết luận: **có, nhưng chỉ dùng làm pre-label cho người review**, ưu tiên `person`, `motorbike`, `car`; `three_wheeler` phải dùng YOLOE và kiểm tra thủ công. Căn cứ: accepted 503/567 = 88.7%, nhưng precision/recall gold chỉ 72.6%/80.4% và bắt bẫy 6/12 = 50.0%.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Quét toàn ảnh để tìm vật bị bỏ sót trước khi xem hoặc chấp nhận box máy.
  2. Kiểm lần lượt bỏ sót, box thừa, sai lớp, box lỏng rồi mới bấm Accept.
  3. Với box `can_review`, box lỏng và lớp `three_wheeler`, luôn đối chiếu ảnh gốc và ghi lại quyết định thay vì dựa vào confidence.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_004.jpg | SAM 2.1 base | 3 (2 dương, 1 âm) | Không thấy người lái bị phủ đáng kể; mask tập trung vào xe | Biên bánh và gầm xe còn hơi răng cưa, nhưng tách được xe khỏi người đứng bên phải | `tuktuk_img004_sam2.jpg` |
| img_013.jpg | SAM 1 Mobile | 3 (2 dương, 1 âm) | Có thể dính một phần người lái trong vùng cabin | Mask phủ khá đầy đủ thân/khung xe nhưng cần thêm điểm âm quanh người lái để tách sạch | `tuktuk_img013_sam1.jpg` |
| img_013.jpg | SAM 2.1 base | 3 (2 dương, 1 âm) | Có dính người lái và một số vùng trong cabin | Có lỗ/đốm trong vùng che khuất; cần thêm điểm âm hoặc sửa thủ công trước khi dùng làm mask chuẩn | `tuktuk_img013_sam2.jpg` |
| img_017.jpg | SAM 2.1 base | 3 (2 dương, 1 âm) | Không thấy hai người đi bộ bên trái bị lấy vào mask | Tách xe khá sát; còn vài điểm rỗng nhỏ ở vùng cửa/khoang xe | `tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: click dương đặt trên thân/khung xe giúp SAM giữ đúng vật; click âm đặt trên người lái giúp tách người khỏi xe. Với `img_013.jpg`, một điểm âm chưa đủ vì người lái nằm trong vùng cabin bị che khuất, nên cần thêm điểm âm hoặc sửa mask thủ công. So sánh định tính: SAM 2.1 cho biên tốt ở `img_004.jpg` và `img_017.jpg`, nhưng ở cảnh khó `img_013.jpg` vẫn dính người lái; SAM 1 nhẹ hơn nhưng cũng cần kiểm tra thủ công.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không dùng trong bài; dùng offline `sam2_points.py` vì có thể lưu rõ điểm dương/âm và file mask trong `submission/masks/`.

## 6. Calibration (Part 6)

- Số quyết định: **3** (`calibration_decisions.csv`); quan trọng nhất là `CAL-01`: xe ba bánh có mui/khung vẫn gán `three_wheeler`, người lái gán riêng.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: tách hàng cồng kềnh khỏi box phương tiện khi nhô quá khoảng 20%, xử lý xe ba bánh và người lái riêng, đồng thời thêm điểm âm SAM khi mask dính người lái.
- Nguyên nhân gốc: model đóng chưa có lớp `three_wheeler`, guideline v1.0 còn câu hỏi mở về hàng cồng kềnh và người review chưa quét ảnh độc lập đủ kỹ.

## 7. QC và quyết định (Part 6)

- Mẫu: **5 ảnh** (0 ảnh `can_review` + 5 ảnh ngẫu nhiên, seed 42), **104 đối tượng**; người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: **2/104 = 1.9%** · theo **ảnh**: **2/5 = 40.0%** (ngưỡng Release ≤ 2%).
- Top lỗi: **sai vị trí/kích thước: 2 lỗi**, ví dụ `img_003.jpg` object `16` và `img_031.jpg` object `473`; cả hai box ôm phần hàng/cồng kềnh vượt ra ngoài phần phương tiện.
- **Giới hạn mẫu nhỏ**: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **77%**; mẫu chỉ có 5 ảnh nên chưa thể chứng minh đạt ngưỡng Release 2%.
- Script kiểm được cấu trúc JSON/CSV, lớp, box nằm trong ảnh, box quá nhỏ, trạng thái review và thống kê bẫy. Script không tự kiểm được box có ôm sát vật không, hàng cồng kềnh có cần tách không, hoặc lỗi cùng bị bỏ sót với máy; các điểm này phải kiểm bằng mắt theo checklist.

**Quyết định: Rework**, ngày **2026-11-18**. Lý do: tỉ lệ ảnh có lỗi là **2/5 = 40.0% > 2%** và bắt bẫy chỉ **6/12 = 50.0% < 80%**; vì vậy chưa đủ bằng chứng để Release.
Việc cần làm tiếp: duyệt lại 6 bẫy bị lọt (`img_021`, `img_029`, `img_025` lỗi loose; `img_035`, `img_027` lỗi missing; `img_033` lỗi wrong_class), sửa các box tương ứng, chạy lại `review_stats.py`, `qc_sample.py`/`qc_report.py` và chỉ Release khi kết quả QC đạt ngưỡng.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** dùng Codex để đọc lỗi script, giải thích guideline và kiểm tra file; mọi nhãn cuối, QC và nhận xét mask đều được đối chiếu thủ công trên ảnh.
- Gold có bị nhiễm không: không; gold được gán và lưu trước khi dùng pre-label để so sánh.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows PowerShell, CPU không CUDA; gặp lỗi thiếu `numpy`, PowerShell chặn `Activate.ps1` và phải chạy trực tiếp bằng `.venv\Scripts\python.exe`.
- Dùng cache `data/cache/` thay vì tự chạy: không; chạy các script local trong môi trường `.venv`.
- Lỗi / bug script đã gặp: thiếu `numpy`, đường dẫn JSON sai và lệnh merge thiếu các file positional; đã sửa bằng cài dependency và dùng đúng đường dẫn/lệnh.
- Một điều tôi làm khác lần sau: ghi thời gian gold và batch ngay khi bắt đầu, đồng thời quét ảnh trước khi xem nhãn máy để giảm automation bias.
