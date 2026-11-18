# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 5 ảnh (0 ca khó + 5 ngẫu nhiên), 104 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 2/104 = 1.9% | 104 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 2/5 = 40.0% | 5 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 503/567 = 88.7% / 19/567 = 3.4% / 45/567 = 7.9% | 567 box máy |
| Máy: added | 20/542 = 3.7% | 542 đối tượng sau duyệt |
| Bắt lỗi bẫy | 6/12 = 50.0% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| sai_vi_tri_kich_thuoc | 2 | img_003.jpg, img_031.jpg |

## Giới hạn thống kê
- 2 ảnh lỗi trong 5 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **77%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 40.0% > 2%
- bắt bẫy 50% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
