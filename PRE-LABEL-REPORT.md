# Báo cáo thực hành PointPillars — Day 13

## Người thực hiện và lần chạy chính thức

- Hình thức: **Cá nhân**
- Người chạy: **Nguyễn Tuấn Minh — 2A202602194**
- Ngày chạy chính thức: **02/10/2026**, khoảng **09:22:20–09:23:02 UTC+7**
- Runner: `status=passed`; Linux `amd64`; 4 CPUs; 4 GB RAM.

## Môi trường, image và checkpoint

- Source tag: `day13-pointpillars:lc-20261001-amd64`
- Image ID: `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`
- Repo revision: `0831856d921609312d42c7582c366e5a311bb7b1`
- Checkpoint: `/opt/PointPillars/pretrained/epoch_160.pth`
- Checkpoint SHA256: `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`

## Input

- KITTI demo `000008`; `input/demo.pcd`; 17,238 points.
- PCD SHA256: `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`
- `z_ground = 0.075 m`.

## Ba lượt inference

| Lượt | delta | Pillar XY | n_boxes | mean_z | Phân bố class |
| --- | ---: | ---: | ---: | ---: | --- |
| A | 0 | 0.16 | 1 | 0.330 | vehicles: 1 |
| B | 1.73 | 0.16 | 13 | 1.034 | vehicles: 10; pedestrian: 2; two-wheels: 1 |
| C | 1.73 | 0.32 | 6 | 1.091 | pedestrian: 6 |

### A/B — chỉ đổi delta

Hai lượt giữ `Pillar XY = 0.16`, chỉ đổi `delta` từ `0` sang `1.73 m`. Prediction đổi từ 1 box sang 13 box và phân bố class cũng đổi. Đây là model chạy lại trên input đã biến đổi, **không phải** chỉ dịch cùng tập box của A theo z. Không thể kết luận chất lượng từng prediction khi chưa có Ground Truth/reference được duyệt.

### B/C — chỉ đổi Pillar XY

Hai lượt giữ `delta = 1.73 m`, chỉ đổi Pillar XY từ `0.16` sang `0.32`. Số prediction đổi từ 13 xuống 6 và phân bố class thay đổi rõ. Không thể kết luận B hoặc C tốt hơn chỉ từ số box hay `mean_z`; cần đối chiếu Ground Truth/reference và geometry 3D.

### Giới hạn khi đọc kết quả

Side view biểu diễn X-Z, phù hợp để kiểm tra chiều cao, đáy cuboid và lỗi z đồng loạt. Nó làm mất trục Y, nên không đủ để kết luận chính xác về yaw, vị trí ngang, object bị bỏ sót hay toàn bộ geometry 3D. Các JSON A/B/C chỉ là prediction/pre-label; trước khi import cần kiểm class, tâm, kích thước, yaw, độ bám mặt đất, false positive, missed object và geometry trong 3D/CVAT.

## QC có kiểm soát — training only

Nguồn QC là Run B: 13 box, `delta=1.73 m`, `z_ground=0.075 m`, do đó `height_offset = 1.805 m`.

| Case | Box bị shift | Shift z | Quyết định |
| --- | --- | --- | --- |
| `case-correct` | 0/13 | 0 m | Không có lỗi z có chủ đích; tiếp tục QC từng cuboid. |
| `case-batch-z` | 13/13 | -1.805 m | Dừng batch, kiểm tra/escalate pipeline và phép chuyển hệ tọa độ; không sửa thủ công từng cuboid. |
| `case-one-box-z` | 1/13 | -1.805 m | Không dừng batch; cô lập và kiểm tra object lỗi. |

QC cases là **TRAINING ONLY**, được tạo bằng biến đổi có chủ đích từ prediction, không phải Ground Truth và không phải inference mới. **Không import các QC case vào CVAT.**

## Nhận xét cá nhân

Tôi thực hiện bài lab cá nhân và trực tiếp chạy A/B/C cùng ba ca QC. Với `case-batch-z`, 13/13 box cùng lệch xuống `1.805 m`, là dấu hiệu lỗi hệ thống nên cần dừng batch và kiểm tra pipeline. Với `case-one-box-z`, chỉ 1/13 box lệch nên cần kiểm tra object đó riêng. Tôi chưa thể kết luận cấu hình nào tốt nhất chỉ từ `n_boxes`, `mean_z` hay Side view; các giá trị này không phải metric chất lượng thay cho reference và kiểm tra geometry.
