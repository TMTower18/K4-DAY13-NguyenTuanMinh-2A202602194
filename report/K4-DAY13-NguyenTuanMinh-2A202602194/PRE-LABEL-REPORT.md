# Báo cáo thực hành PointPillars — Day 13

Giữ bản đã điền ngoài Git, trong thư mục private do LC thu. Đây là kiểm tra formative; không ghi điểm của người khác.

## Nhóm và provenance

- Mã nhóm/phòng: **[Điền theo yêu cầu của LC nếu có]**
- Thành viên: **Nguyễn Tuấn Minh — MSSV: 2A202602194**
- Hình thức thực hiện: **Cá nhân**
- Trạng thái: `executed-by-group` **[template hiện không có lựa chọn executed-individually; xác nhận lại với LC nếu cần]**
- Người thực sự chạy; ngày/giờ; hệ máy/architecture:
  - Người chạy: **Nguyễn Tuấn Minh**
  - Ngày chạy chính thức: **02/10/2026**
  - Thời gian: khoảng **09:22:20 – 09:23:02 (UTC+7)**
  - Host: **Windows 11 / PowerShell**
  - Docker runtime: **Linux `amd64`**
  - Giới hạn runner: **4 CPUs, 4 GB RAM**
  - Trạng thái toàn bộ runner: **`passed`**

- Image tag và image ID; phiên bản repo:
  - Source tag: `day13-pointpillars:lc-20261001-amd64`
  - Image ID: `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`
  - Repo revision: `0831856d921609312d42c7582c366e5a311bb7b1`
  - Bundle: `working_tree_dirty=true`

- PCD được cấp / frame_id; nơi được phép chạy; fingerprint:
  - File: `input/demo.pcd`
  - frame_id: `demo`
  - Dataset: **KITTI**
  - Nguồn: KITTI Vision Benchmark Suite / MMDetection3D demo `000008`
  - Số điểm: **17,238**
  - PCD SHA256: `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`
  - Source SHA256: `3b9de6cc966534900f6a1bdc93b21772e47a334eb2ef18082021956520d902d1`
  - License: `CC-BY-NC-SA-3.0`
  - Nơi chạy: **máy cá nhân của Nguyễn Tuấn Minh bằng Docker offline**

- Checkpoint:
  - PointPillars KITTI có sẵn trong Docker image.
  - Path: `/opt/PointPillars/pretrained/epoch_160.pth`
  - SHA256: `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`

- Phạm vi: **front-window**
- Score threshold: **[chưa xác nhận từ runner/source nên chưa tự điền]**

- Giả định kênh thứ tư/intensity và nguồn `z_ground`:
  - PCD được chuyển đổi từ KITTI demo `000008`.
  - Reflectance gốc bị loại bỏ; provenance của bundle ghi sử dụng constant channels trong model adapter.
  - `z_ground = 0.075 m`
  - Source provenance ghi `z_offset_m = 1.73 m`

## Ba lượt inference thật

A/B/C được chạy trên cùng PCD bằng một lần chạy chính thức của runner vào ngày 02/10/2026.  
Số hộp và `mean_z` được lấy từ output `run-A/B/C/summary.csv`.

| Lượt | delta | Pillar XY | Số hộp | mean_z | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | --- | --- | --- | --- | --- | --- |
| A | 0 | 0.16 | **1** | **0.330** | `run-A/boxes-demo-delta-0-voxel-0.16.json`; `run-A/side-demo-delta-0-voxel-0.16.png`; `run-A/summary.csv` | Model sinh 1 prediction, terminal ghi `vehicles=1`. Side view cho thấy một cuboid ở vùng khoảng x ≈ 11–15 m. |
| B | 1.73 | 0.16 | **13** | **1.034** | `run-B/boxes-demo-delta-1.73-voxel-0.16.json`; `run-B/side-demo-delta-1.73-voxel-0.16.png`; `run-B/summary.csv` | Model sinh 13 prediction gồm `vehicles=10`, `pedestrian=2`, `two-wheels=1`. Prediction xuất hiện ở nhiều vùng hơn so với A. |
| C | 1.73 | 0.32 | **6** | **1.091** | `run-C/boxes-demo-delta-1.73-voxel-0.32.json`; `run-C/side-demo-delta-1.73-voxel-0.32.png`; `run-C/summary.csv` | Model sinh 6 prediction, terminal ghi `pedestrian=6`. Tập prediction thay đổi rõ so với B khi Pillar XY tăng từ 0.16 lên 0.32. |

### A/B — chỉ đổi delta

A có **1 hộp**, B có **13 hộp**.

Hai lượt giữ nguyên:

`Pillar XY = 0.16`

và chỉ thay:

`delta: 0 → 1.73 m`

Run A ghi nhận:

`vehicles = 1`

Run B ghi nhận:

`vehicles = 10`  
`pedestrian = 2`  
`two-wheels = 1`

Như vậy thay đổi `delta` trước inference làm thay đổi rõ tập prediction. Đây là **chạy lại model trên input được biến đổi**, không phải lấy hộp của A rồi đơn giản dịch chúng thêm `1.73 m`.

Side view cũng cho thấy B có prediction tại nhiều vùng theo trục x hơn A.

Điều chưa chắc chắn là độ chính xác thực sự của từng prediction vì chưa có Ground Truth/reference được duyệt để đối chiếu.

### B/C — chỉ đổi Pillar XY

B có **13 hộp**, C có **6 hộp**.

Hai lượt giữ:

`delta = 1.73 m`

nhưng thay:

`Pillar XY: 0.16 → 0.32`

Run B:

- 10 vehicles
- 2 pedestrian
- 1 two-wheels

Run C:

- 6 pedestrian

Như vậy thay đổi kích thước pillar làm thay đổi cả **số lượng prediction và phân bố class**.

Không thể kết luận B hoặc C tốt hơn chỉ dựa trên số hộp hoặc `mean_z`. Cần Ground Truth/reference và kiểm tra geometry để đánh giá chất lượng.

### Giới hạn ROI và góc Side ảnh hưởng cách đọc miss/yaw thế nào?

Side view chủ yếu biểu diễn mặt phẳng **X-Z**, vì vậy hữu ích để kiểm tra:

- vị trí theo chiều cao;
- đáy cuboid;
- lỗi dịch đồng loạt theo trục z.

Tuy nhiên trục Y bị mất trong hình chiếu. Các object có vị trí Y khác nhau có thể chồng lên nhau.

Vì vậy không thể chỉ dựa vào Side view để kết luận chính xác về:

- yaw;
- vị trí ngang;
- object bị miss;
- toàn bộ geometry 3D.

Cần kiểm tra thêm bằng BEV/3D viewer hoặc các nguồn dữ liệu được bài thực hành cho phép.

### JSON nào còn chưa đủ cơ sở để import? Cần kiểm gì tiếp?

Cả ba JSON A/B/C hiện là **prediction/pre-label của model**, chưa phải Ground Truth.

Không nên coi số lượng box hoặc Side view là đủ bằng chứng để import trực tiếp thành annotation chính thức.

Trước khi sử dụng cần kiểm tra:

- class;
- tâm `(x, y, z)`;
- kích thước cuboid;
- yaw;
- độ bám mặt đất;
- false positive;
- missed object;
- geometry trong 3D/CVAT.

## Ca QC có kiểm soát — không import CVAT

Ba ca QC được tạo từ prediction của **Run B**:

`boxes-demo-delta-1.73-voxel-0.16.json`

Thông tin nguồn:

- Source prediction SHA256:  
  `c2a8db247353f0ef00299ff50acff997b7b4a87bdcfcefe792815acb5652cc80`
- `delta_m = 1.73`
- `z_ground_m = 0.075`
- `height_offset_m = 1.805 m`
- Tổng số box nguồn: **13**
- `training_only = true`

Trong đó:

`height_offset = delta + z_ground`

`1.73 + 0.075 = 1.805 m`

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Quyết định | Bằng chứng |
| --- | --- | --- | --- | --- | --- |
| `case-correct` | **0 / 13** | `0 m` | Không có biến đổi có chủ đích; đây là copy của source-frame prediction | Không dừng vì lỗi z; tiếp tục QC từng cuboid | `case-correct.json`, `side-correct.png` |
| `case-batch-z` | **13 / 13** | `-1.805 m` | Helper chỉ dịch z của toàn bộ 13 box; class/x/y/yaw không bị thay đổi có chủ đích | **Dừng batch và kiểm tra/escalate pipeline** | `case-batch-z.json`, `side-batch-z.png` |
| `case-one-box-z` | **1 / 13** | `-1.805 m` ở box đầu tiên | Helper chỉ dịch z của box đầu tiên; các thuộc tính khác không bị thay đổi có chủ đích | **Không dừng toàn batch; kiểm tra riêng object lỗi** | `case-one-box-z.json`, `side-one-box-z.png` |

Ba ca QC này do helper tạo biến đổi có chủ đích từ prediction của Run B để phục vụ bài tập QC.

**Chúng không phải Ground Truth, không phải ba lượt inference mới, không dùng để đưa ra quality claim và không import vào CVAT.**

### Diễn giải quyết định QC

#### `case-correct`

Không có lỗi z được helper cố ý thêm vào.

Từ "correct" ở đây chỉ có nghĩa là prediction nguồn **không bị helper làm sai thêm**, không có nghĩa model prediction là Ground Truth hoặc chắc chắn chính xác.

#### `case-batch-z`

Cả **13/13 box** đều bị dịch xuống:

`1.805 m`

Manifest mô tả đây là trường hợp:

> Every box z is shifted down by delta + z_ground, simulating a missed inverse conversion.

Đây là pattern lỗi có tính **toàn batch / hệ thống**.

Quyết định:

**Dừng batch → kiểm tra pipeline / phép chuyển hệ tọa độ → không sửa thủ công từng cuboid.**

#### `case-one-box-z`

Chỉ **1/13 box** bị dịch xuống:

`1.805 m`

Manifest mô tả đây là object-local error.

Quyết định:

**Không dừng toàn batch ngay.**

Cần cô lập object đó, kiểm tra JSON/geometry và xác định nguyên nhân trước khi sửa hoặc loại box.

## Nhận xét cá nhân

### Nguyễn Tuấn Minh — MSSV: 2A202602194

- **Vai trò đã làm:**  
  Tôi thực hiện bài lab cá nhân. Tôi đã kiểm tra môi trường, xác minh SHA256 của `student-prelabel-amd64.zip`, kiểm tra Docker và trực tiếp chạy Student Bundle. Lần chạy chính thức ngày 02/10/2026 đã hoàn thành A/B/C và ba ca QC với trạng thái `passed`.

- **Một quan sát A/B/C có dẫn file hoặc vùng:**  
  Run A (`run-A/side-demo-delta-0-voxel-0.16.png`, `run-A/summary.csv`) chỉ sinh **1 box**, `mean_z=0.330`.  
  Run B (`run-B/side-demo-delta-1.73-voxel-0.16.png`, `run-B/summary.csv`) sinh **13 box**, `mean_z=1.034`, gồm 10 vehicles, 2 pedestrian và 1 two-wheels.  
  Điều này cho thấy thay đổi `delta` trước inference có thể làm thay đổi tập prediction chứ không đơn giản dịch cùng một tập box theo z.

- **Diễn giải phép z thuận/ngược:**  
  `delta` và `z_ground` liên quan đến phép chuyển đổi hệ tọa độ trước/sau inference. Khi prediction được đưa trở lại source frame cần thực hiện đúng inverse conversion. Bộ QC mô phỏng hậu quả nếu bước này bị sai hoặc bị bỏ sót. Với `delta=1.73 m` và `z_ground=0.075 m`, offset được helper sử dụng là:

  `1.73 + 0.075 = 1.805 m`

  `case-batch-z` cố ý dịch z của toàn bộ box xuống `1.805 m` để mô phỏng lỗi hệ thống.

- **Một quyết định lỗi batch và hành động:**  
  Với `case-batch-z`, **13/13 box** cùng lệch xuống `1.805 m`. Tôi sẽ **dừng batch và kiểm tra/escalate pipeline**, không sửa từng cuboid bằng tay.

  Với `case-one-box-z`, chỉ **1/13 box** bị lệch z. Tôi sẽ không kết luận toàn pipeline bị lỗi mà sẽ cô lập và kiểm tra object đó trước.

- **Điều chưa chắc:**  
  Chưa thể kết luận Run A, B hay C có chất lượng detection tốt nhất chỉ dựa trên `n_boxes`, `mean_z` và Side view. `mean_z` không phải metric chất lượng. Cần Ground Truth/reference được phép sử dụng và kiểm tra geometry, class và yaw bằng các góc nhìn phù hợp.

## LC ghi nhận riêng

- Quyền dùng PCD/image và đúng ca: **[LC điền]**
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung: **[LC điền]**
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT: **[LC điền]**
- Nhận xét cá nhân và quyết định dừng pipeline: **[LC điền]**
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do: **[LC điền]**