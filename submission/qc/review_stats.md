# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 567; đối tượng sau duyệt: 542; ngưỡng accepted: IoU >= 0.9
- Đã **loại 20 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 503 | 503/567 = 88.7% | 567 box máy |
| edited (sửa) | 19 | 19/567 = 3.4% | 567 box máy |
| deleted (xoá) | 45 | 45/567 = 7.9% | 567 box máy |
| added (vẽ thêm) | 20 | 20/542 = 3.7% | 542 đối tượng sau duyệt |

Trong số deleted + added, **4 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 16.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 285 | 3 | 5 | 5 |
| bicycle | 5 | 0 | 3 | 3 |
| motorbike | 140 | 7 | 6 | 4 |
| car | 52 | 2 | 7 | 5 |
| bus | 3 | 0 | 0 | 2 |
| truck | 3 | 0 | 3 | 0 |
| three_wheeler | 15 | 7 | 21 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 56 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 45 | 17 | 11 | 72.6% (KTC 95%: 60–82%) | 80.4% (KTC 95%: 68–89%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 6/12 = 50.0% (KTC 95%: 25–75%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - loose: 0/3
  - wrong_class: 2/3
  - missing: 1/3
  - lọt: img_021.jpg (loose), img_029.jpg (loose), img_025.jpg (loose), img_035.jpg (missing), img_027.jpg (missing), img_033.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
