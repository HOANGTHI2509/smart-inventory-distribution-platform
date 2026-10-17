# TÀI LIỆU YÊU CẦU NGHIỆP VỤ CHI TIẾT (BUSINESS REQUIREMENTS SPECIFICATION)

**Đề tài:** Xây dựng nền tảng quản lý kho và phân phối hàng hóa thông minh ứng dụng AI dự báo nhu cầu và hỗ trợ tối ưu tồn kho  
**Loại tài liệu:** Đặc tả yêu cầu chức năng (Functional Requirements) và phi chức năng (Non-Functional Requirements)  
**Trạng thái tài liệu:** Bản dự thảo chính thức (Chuyển tiếp từ [`docs/01_problem_scope.md`](file:///d:/DO_AN_TOT_NGHIEP/docs/01_problem_scope.md))  
**Môi trường xác lập:** Môi trường doanh nghiệp giả lập; phát triển dựa trên mô hình nghiệp vụ chuẩn hóa và dữ liệu giả lập (synthetic data).

---

## Phần 1. Thông tin tài liệu

### 1.1. Mục đích tài liệu
Tài liệu này chuyển hóa các mục tiêu, ranh giới bài toán và nguyên tắc quản trị tồn kho đã xác định tại [`docs/01_problem_scope.md`](file:///d:/DO_AN_TOT_NGHIEP/docs/01_problem_scope.md) thành danh mục các yêu cầu nghiệp vụ cụ thể. Đây là căn cứ chính thức để:
- Thiết kế bảng quyết định nghiệp vụ và máy trạng thái giao dịch (`docs/03_business_rules.md`).
- Thiết kế mô hình dữ liệu và từ điển thực thể (`docs/04_data_dictionary.md`).
- Xây dựng ca kiểm thử (Test Cases) kiểm chứng chất lượng phần mềm trước khi nghiệm thu đồ án.

### 1.2. Phạm vi và Đối tượng sử dụng
- **Phạm vi tài liệu:** Bao phủ toàn bộ các phân hệ chức năng bắt buộc thuộc Core MVP, các phân hệ mở rộng có điều kiện thuộc Advanced MVP, và các yêu cầu phi chức năng phục vụ chạy thử nghiệm trong môi trường doanh nghiệp giả lập.
- **Đối tượng đọc:** Sinh viên thực hiện đề tài, giảng viên hướng dẫn, hội đồng phản biện đồ án tốt nghiệp, và người phát triển hệ thống.

### 1.3. Tài liệu tham chiếu
- [`GEMINI.md`](file:///d:/DO_AN_TOT_NGHIEP/GEMINI.md) — Quy tắc làm việc kỹ thuật và chuẩn mực trung thực trong đồ án.
- [`docs/01_problem_scope.md`](file:///d:/DO_AN_TOT_NGHIEP/docs/01_problem_scope.md) — Đặc tả bài toán, ranh giới phạm vi và tiêu chí nghiệm thu.

### 1.4. Thuật ngữ và Chữ viết tắt

| Thuật ngữ / Viết tắt | Tên tiếng Anh đầy đủ | Định nghĩa trong phạm vi đề tài |
| :--- | :--- | :--- |
| **SKU** | Stock Keeping Unit | Mã đơn vị phân loại sản phẩm lưu kho duy nhất. |
| **On-hand** | On-hand Inventory | Số lượng hàng vật lý thực tế đang có mặt trong kho. |
| **Allocated / Reserved** | Allocated / Reserved Stock | Số lượng hàng đã cam kết cho đơn xuất/chuyển kho nhưng chưa rời kho. |
| **Available** | Available-to-Promise (ATP) | Số lượng hàng sẵn sàng để bán hoặc chuyển: $\text{Available} = \text{On-hand} - \text{Allocated}$. |
| **On-order** | On-order Quantity | Số lượng hàng đã đặt mua từ NCC hoặc đang trên đường chuyển về kho. |
| **In-transit** | In-transit Stock | Hàng đã xuất khỏi kho gửi nhưng chưa nhập kho nhận trong lệnh điều chuyển. |
| **IP** | Inventory Position | Vị thế tồn kho dùng để ra quyết định: $\text{IP} = \text{Available} + \text{On-order}$. |
| **ROP** | Reorder Point | Điểm đặt hàng lại (ngưỡng kích hoạt xem xét bổ sung hàng). |
| **SS** | Safety Stock | Tồn kho an toàn phòng ngừa biến động nhu cầu trong thời gian giao hàng. |
| **Lead time ($L$)** | Replenishment Lead Time | Thời gian (tính bằng ngày) từ lúc tạo đơn bổ sung đến khi hàng nhập kho. |
| **PO** | Purchase Order | Đơn đặt hàng mua từ nhà cung cấp. |
| **GRN** | Goods Receipt Note | Phiếu xác nhận nhập kho thực tế (nhận đủ hoặc nhận một phần). |
| **GDN** | Goods Dispatch Note | Phiếu xác nhận xuất kho vật lý. |
| **CDC** | Central Distribution Center | Kho phân phối trung tâm (đóng vai trò nguồn cung chính). |
| **RDC** | Regional Distribution Center | Kho vệ tinh / kho khu vực (phục vụ nhu cầu phân phối địa phương). |
| **WAPE** | Weighted Absolute Percentage Error | Chỉ số đánh giá sai số dự báo có trọng số theo sản lượng tiêu thụ. |
| **MAE** | Mean Absolute Error | Sai số tuyệt đối trung bình giữa nhu cầu thực tế và dự đoán. |
| **RMSE** | Root Mean Squared Error | Căn bậc hai sai số bình phương trung bình. |
| **RBAC** | Role-Based Access Control | Kiểm soát truy cập và phân quyền người dùng theo vai trò. |

---

## Phần 2. Tác nhân và Quyền hạn

Hệ thống xác định 4 tác nhân con người chính tham gia vận hành và tác nghiệp. Bảng ma trận dưới đây quy định quyền hạn thao tác trên các phân hệ:

| Phân hệ chức năng | Quản lý kho (Warehouse Manager) | Nhân viên kho (Warehouse Operator) | Chuyên viên kế hoạch (Inventory Planner) | Quản trị viên (System Admin) |
| :--- | :---: | :---: | :---: | :---: |
| **Xác thực & Người dùng (AUTH)** | Xem thông tin cá nhân | Xem thông tin cá nhân | Xem thông tin cá nhân | Toàn quyền (CRUD User, Role) |
| **Danh mục cơ sở (MST)** | Xem | Xem | Xem | Toàn quyền (CRUD Kho, SKU, NCC) |
| **Số dư tồn kho (INV)** | Xem toàn bộ | Xem kho được phân công | Xem toàn bộ | Xem toàn bộ |
| **Đơn mua hàng PO (INB)** | Phê duyệt | Xem | Tạo, Cập nhật | Xem |
| **Nhập kho thực tế GRN (INB)** | Xem, Đối soát | Tạo, Xác nhận nhập | Xem | Xem |
| **Giữ chỗ & Xuất kho GDN (OUT)** | Phê duyệt xuất | Xác nhận xuất kho | Theo dõi điều phối | Xem |
| **Lệnh điều chuyển kho (TRF)** | Phê duyệt chuyển | Xuất kho gửi, Nhận kho đích | Tạo đề xuất chuyển | Xem |
| **Phân bổ đơn hàng (ORD)** | Giám sát phân bổ | Xem phiếu xuất | Cấu hình ưu tiên kho | Cấu hình tham số |
| **Dự báo AI (AI)** | Xem kết quả | Không truy cập | Xem, Chạy thử nghiệm | Giám sát kết nối service |
| **Khuyến nghị bổ sung (REC)** | Phê duyệt đề xuất | Không truy cập | Xem, Điều chỉnh, Trình duyệt | Xem cấu hình |
| **Nhật ký kiểm toán (AUD)** | Xem toàn bộ log | Xem log kho mình | Xem log điều phối | Xem toàn bộ log |
| **Bảng điều khiển (DASH)** | Xem tổng quan kho | Xem tác nghiệp ngày | Xem dự báo, cung cầu | Xem trạng thái hệ thống |

*Ghi chú:* Ký hiệu quyền: `Toàn quyền` (Tạo, Đọc, Sửa, Xóa); `Phê duyệt` (Chuyển trạng thái quyết định); `Xác nhận` (Ghi nhận thao tác thực tế); `Xem` (Chỉ đọc). Tuyệt đối không cấp quyền sửa/xóa đối với các bản ghi nhật ký kiểm toán (Audit Log).

---

## Phần 3. Danh mục Yêu cầu Chức năng (Functional Requirements)

Mỗi yêu cầu chức năng được xếp mức độ ưu tiên theo chuẩn MoSCoW:
- **Must (Bắt buộc):** Chức năng thuộc Core MVP, bắt buộc phải hoàn thành và kiểm thử được.
- **Should (Nên có):** Chức năng thuộc Advanced MVP, chỉ triển khai khi Core MVP đã ổn định.
- **Could (Có thể có):** Chức năng thuộc Nice-to-have, thực hiện nếu còn thời gian.

---

### PHÂN HỆ 1: XÁC THỰC VÀ PHÂN QUYỀN (FR-AUTH)

#### `FR-AUTH-001`: Đăng nhập và Xác thực người dùng
- **Tác nhân:** Tất cả người dùng.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Hệ thống xác thực danh tính người dùng bằng tên đăng nhập và mật khẩu; trả về mã phiên làm việc bảo mật (JWT Token).
- **Điều kiện trước:** Tài khoản đã được Quản trị viên kích hoạt trên hệ thống.
- **Luồng chính:**
  1. Người dùng nhập tên đăng nhập và mật khẩu trên giao diện Web.
  2. Hệ thống kiểm tra thông tin đăng nhập trong cơ sở dữ liệu (mật khẩu đã băm bằng BCrypt).
  3. Hệ thống sinh mã JWT Token chứa thông tin `user_id`, `username`, `roles` và trả về cho client.
- **Luồng ngoại lệ:** Nếu sai thông tin đăng nhập hoặc tài khoản bị khóa, hệ thống từ chối và hiển thị thông báo lỗi chung (không chỉ rõ sai tên hay sai mật khẩu để bảo mật).
- **Kết quả & Số dư:** Trạng thái đăng nhập thành công; không ảnh hưởng số dư tồn kho.
- **Tiêu chí nghiệm thu:** Đăng nhập đúng cấp token hợp lệ; đăng nhập sai 5 lần không làm lộ thông tin hệ thống.
- **Tham chiếu ca kiểm thử:** Kiểm tra chức năng bảo mật hệ thống.

#### `FR-AUTH-002`: Phân quyền truy cập theo vai trò (RBAC Enforcement)
- **Tác nhân:** Hệ thống.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Hệ thống kiểm tra vai trò (Role) của người dùng trước khi cho phép thực thi bất kỳ API tác nghiệp hoặc truy cập màn hình chức năng nào theo Ma trận quyền hạn tại Phần 2.
- **Điều kiện trước:** Request gửi kèm JWT Token hợp lệ trong Authorization Header.
- **Luồng chính:**
  1. Bộ lọc phân quyền (Security Filter) giải mã token, lấy danh sách vai trò của người dùng.
  2. Đối soát quyền hạn với endpoint API được gọi.
  3. Nếu hợp lệ, cho phép request đi tiếp vào Controller xử lý nghiệp vụ.
- **Luồng ngoại lệ:** Nếu vai trò không đủ quyền, hệ thống trả về mã lỗi HTTP 403 Forbidden và từ chối thực thi.
- **Kết quả & Số dư:** Bảo vệ an toàn dữ liệu; không ảnh hưởng số dư tồn kho.
- **Tiêu chí nghiệm thu:** Tài khoản `Warehouse Operator` gọi API phê duyệt đơn PO phải nhận về lỗi 403 Forbidden.
- **Tham chiếu ca kiểm thử:** Kiểm tra phân quyền truy cập.

---

### PHÂN HỆ 2: QUẢN LÝ DANH MỤC CƠ SỞ (FR-MST)

#### `FR-MST-001`: Quản lý danh mục sản phẩm (SKU Master)
- **Tác nhân:** Quản trị viên hệ thống.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Cho phép tạo mới, cập nhật và tra cứu thông tin sản phẩm lưu kho.
- **Điều kiện trước:** Người dùng có quyền Quản trị viên.
- **Luồng chính:**
  1. Nhập mã sản phẩm (`sku_code`), tên sản phẩm, đơn vị tính (`UOM`), danh mục phân loại (`category`).
  2. Hệ thống kiểm tra tính duy nhất của `sku_code`.
  3. Lưu bản ghi sản phẩm vào cơ sở dữ liệu ở trạng thái hoạt động (`ACTIVE`).
  4. Hệ thống tự động tạo các bản ghi số dư tồn kho ban đầu bằng 0 cho SKU này tại tất cả các kho đang hoạt động.
- **Luồng ngoại lệ:** Trùng mã `sku_code` $\implies$ Báo lỗi trùng lặp và yêu cầu nhập lại.
- **Kết quả & Số dư:** Thêm mới 1 SKU; bảng tồn kho khởi tạo các bản ghi số dư = 0.
- **Tiêu chí nghiệm thu:** Tạo thành công SKU mới; không thể tạo bản ghi thứ hai trùng mã `sku_code`.
- **Tham chiếu ca kiểm thử:** Kiểm tra danh mục sản phẩm.

#### `FR-MST-002`: Quản lý danh mục kho hàng cấp Tổng kho (Warehouse Master)
- **Tác nhân:** Quản trị viên hệ thống.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Cho phép quản lý danh mục kho hàng ở cấp Tổng kho (`Warehouse Level`); chưa phân cấp vị trí kệ/bin trong Core MVP.
- **Điều kiện trước:** Người dùng có quyền Quản trị viên.
- **Luồng chính:**
  1. Nhập mã kho (`warehouse_code`), tên kho, địa chỉ, loại kho (`CENTRAL` - Kho trung tâm, `REGIONAL` - Kho vệ tinh), thứ tự ưu tiên phân bổ đơn hàng (`priority_rank`).
  2. Hệ thống kiểm tra tính duy nhất của `warehouse_code`.
  3. Lưu thông tin kho vào cơ sở dữ liệu.
- **Luồng ngoại lệ:** Trùng mã kho $\implies$ Báo lỗi từ chối.
- **Kết quả & Số dư:** Thêm mới kho bãi sẵn sàng tiếp nhận giao dịch.
- **Tiêu chí nghiệm thu:** Tạo được kho `WH-CDC` (ưu tiên 2) và kho `WH-RDC` (ưu tiên 1).
- **Tham chiếu ca kiểm thử:** Kiểm tra danh mục kho hàng.

#### `FR-MST-003`: Quản lý danh mục đối tác (Partners)
- **Tác nhân:** Quản trị viên hệ thống.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Quản lý thông tin Nhà cung cấp (Supplier) và Khách hàng/Điểm nhận hàng (Customer).
- **Điều kiện trước:** Người dùng có quyền Quản trị viên.
- **Luồng chính:** Lưu thông tin mã đối tác, tên, loại đối tác (`SUPPLIER`/`CUSTOMER`), Lead time giao hàng mặc định của NCC (tính bằng ngày).
- **Kết quả & Số dư:** Danh mục đối tác sẵn sàng liên kết với PO và đơn xuất kho.
- **Tiêu chí nghiệm thu:** Tạo được 1 NCC có Lead time = 3 ngày và 1 Khách hàng.
- **Tham chiếu ca kiểm thử:** Kiểm tra danh mục đối tác.

#### `FR-MST-004`: Cấu hình tham số tồn kho ban đầu (Inventory Policy Config)
- **Tác nhân:** Chuyên viên kế hoạch tồn kho.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Thiết lập các tham số tồn kho tĩnh ban đầu cho từng cặp `(SKU, Warehouse)` làm căn cứ vận hành khi chưa có dữ liệu AI.
- **Điều kiện trước:** Đã tồn tại SKU và Kho trên hệ thống.
- **Luồng chính:** Lưu các giá trị: Lead time $L$ (ngày), Mức tồn an toàn tĩnh $SS_{\text{static}}$, Mức tồn tối đa mục tiêu $S$, Số lượng đặt tối thiểu $MOQ$.
- **Kết quả & Số dư:** Cập nhật bảng cấu hình tham số tồn kho.
- **Tiêu chí nghiệm thu:** Cấu hình thành công $L=3, SS=10, S=50, MOQ=10$ cho SKU1 tại Kho 1.
- **Tham chiếu ca kiểm thử:** Kiểm tra cấu hình tham số tồn kho.

---

### PHÂN HỆ 3: QUẢN LÝ SỐ DƯ TỒN KHO (FR-INV)

#### `FR-INV-001`: Tra cứu số dư tồn kho đa trạng thái thời gian thực
- **Tác nhân:** Quản lý kho, Nhân viên kho, Chuyên viên kế hoạch.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Hiển thị tức thời và chính xác 5 chỉ số tồn kho theo từng cặp `(SKU, Warehouse)`.
- **Điều kiện trước:** Người dùng đã đăng nhập và được cấp quyền xem kho tương ứng.
- **Luồng chính:**
  1. Người dùng chọn bộ lọc (SKU, Kho).
  2. Hệ thống truy vấn bảng số dư và trả về:
     - Tồn thực tế ($\text{On-hand}$).
     - Tồn giữ chỗ ($\text{Allocated}$).
     - Tồn khả dụng ($\text{Available} = \text{On-hand} - \text{Allocated}$).
     - Hàng đang về ($\text{On-order}$).
     - Hàng đang đi đường ($\text{In-transit}$).
- **Kết quả & Số dư:** Bảng số liệu hiển thị minh bạch; không có độ trễ số liệu sau mỗi giao dịch.
- **Tiêu chí nghiệm thu:** Thực hiện 1 giao dịch nhập kho $\implies$ giao diện tra cứu hiển thị số dư mới ngay lập tức.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 8 tại `01_problem_scope.md`.

#### `FR-INV-002`: Kiểm soát ràng buộc bất biến chống tồn âm (Stock Invariant Enforcement)
- **Tác nhân:** Hệ thống (tự động tại tầng ứng dụng và cơ sở dữ liệu).
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Ngăn chặn tuyệt đối mọi hành vi hoặc lỗi kỹ thuật làm số lượng tồn khả dụng hoặc tồn thực tế bị âm.
- **Điều kiện trước:** Bất kỳ thao tác xuất kho hoặc giữ chỗ nào phát sinh.
- **Luồng chính:**
  1. Ứng dụng kiểm tra điều kiện tiên quyết: $\text{Available} \ge Q_{\text{request}}$.
  2. Cơ sở dữ liệu kích hoạt Check Constraint: `CHECK (on_hand >= allocated AND on_hand >= 0 AND allocated >= 0)`.
- **Luồng ngoại lệ:** Nếu số lượng yêu cầu vượt quá $\text{Available}$, hệ thống hủy bỏ giao dịch (Rollback) và trả về lỗi HTTP 400 (`INSUFFICIENT_STOCK`).
- **Kết quả & Số dư:** Không có bản ghi xuất kho nào được tạo; số dư giữ nguyên.
- **Tiêu chí nghiệm thu:** Yêu cầu xuất 10 đơn vị khi $\text{Available} = 8 \implies$ bị chặn hoàn toàn, số dư giữ nguyên 8.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 1 tại `01_problem_scope.md`.

#### `FR-INV-003`: Tính toán và theo dõi Vị thế tồn kho (Inventory Position)
- **Tác nhân:** Hệ thống.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Tự động tính toán chỉ số $\text{IP} = \text{Available} + \text{On-order}$ phục vụ động cơ bổ sung hàng.
- **Điều kiện trước:** Có biến động về $\text{Available}$ hoặc $\text{On-order}$.
- **Luồng chính:** Hệ thống tự động tính toán lại $\text{IP}$ để so sánh với ngưỡng Điểm đặt hàng lại $\text{ROP}$.
- **Kết quả & Số dư:** Cung cấp đầu vào chính xác cho việc sinh cảnh báo thiếu hàng.
- **Tiêu chí nghiệm thu:** Kho có $\text{Available} = 5$, $\text{On-order} = 10 \implies \text{IP} = 15$.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 6 tại `01_problem_scope.md`.

---

### PHÂN HỆ 4: NGHIỆP VỤ NHẬP KHO TỪ NCC (FR-INB)

#### `FR-INB-001`: Tạo và quản lý đơn mua hàng từ Nhà cung cấp (Purchase Order)
- **Tác nhân:** Chuyên viên kế hoạch (tạo), Quản lý kho (duyệt).
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Lập và phê duyệt đơn mua hàng từ NCC; ghi nhận số lượng hàng đang về ($\text{On-order}$).
- **Điều kiện trước:** Đã có danh mục NCC và SKU.
- **Luồng chính:**
  1. Chuyên viên kế hoạch tạo đơn PO gồm danh sách SKU và số lượng đặt mua $Q_{\text{PO}}$, trạng thái `DRAFT`.
  2. Quản lý kho phê duyệt đơn PO $\implies$ Chuyển trạng thái sang `CONFIRMED`.
  3. Hệ thống tự động tăng hàng đang về tại kho đích: $\text{On-order} \mathrel{+}= Q_{\text{PO}}$.
  4. Tồn thực tế $\text{On-hand}$ và $\text{Available}$ tại kho đích **chưa thay đổi**.
- **Luồng ngoại lệ:** Nếu đơn PO bị hủy khi đang ở trạng thái `CONFIRMED` $\implies$ $\text{On-order} \mathrel{-}= Q_{\text{PO}}$, đơn chuyển `CANCELLED`.
- **Kết quả & Số dư:** $\text{On-order}$ tăng đúng bằng $Q_{\text{PO}}$; $\text{On-hand}$ không đổi.
- **Tiêu chí nghiệm thu:** Tạo và duyệt PO 100 cái $\implies$ $\text{On-order}$ tăng 100, $\text{On-hand}$ giữ nguyên.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 4 tại `01_problem_scope.md`.

#### `FR-INB-002`: Xác nhận phiếu nhập kho nhận đủ hàng (Full Goods Receipt)
- **Tác nhân:** Nhân viên kho.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Xác nhận hàng vật lý đã về kho đủ theo đơn PO.
- **Điều kiện trước:** Đơn PO ở trạng thái `CONFIRMED` hoặc `PARTIALLY_RECEIVED`; hàng về đủ $Q_{\text{actual}} = Q_{\text{PO}}$.
- **Luồng chính:**
  1. Nhân viên kho kiểm đếm, tạo phiếu GRN xác nhận nhận đủ $Q_{\text{actual}}$.
  2. Cập nhật số dư trong 1 transaction:
     - Tăng tồn thực tế: $\text{On-hand} \mathrel{+}= Q_{\text{actual}}$.
     - Giảm hàng đang về: $\text{On-order} \mathrel{-}= Q_{\text{actual}}$.
     - Tồn khả dụng $\text{Available}$ tự động tăng theo công thức.
  3. Đơn PO chuyển trạng thái sang `COMPLETED`.
  4. Ghi nhận audit log loại `INBOUND_RECEIPT`.
- **Kết quả & Số dư:** Hàng vào kho vật lý; $\text{On-hand}$ tăng, $\text{On-order}$ xóa trạng thái chờ.
- **Tiêu chí nghiệm thu:** Nhập đủ 100 cái theo PO $\implies$ $\text{On-hand} \mathrel{+}= 100, \text{On-order} \mathrel{-}= 100$, PO hoàn thành.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 8 tại `01_problem_scope.md`.

#### `FR-INB-003`: Xử lý nhập kho một phần từ Nhà cung cấp (Partial Goods Receipt)
- **Tác nhân:** Nhân viên kho.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Cho phép ghi nhận nhập kho khi NCC giao hàng làm nhiều đợt ($Q_{\text{actual}} < Q_{\text{PO}}$).
- **Điều kiện trước:** Đơn PO ở trạng thái `CONFIRMED`.
- **Luồng chính:**
  1. Nhân viên kho tạo phiếu GRN ghi nhận số lượng thực nhận đợt này là $Q_{\text{actual}}$.
  2. Hệ thống cập nhật số dư:
     - Tăng tồn thực tế: $\text{On-hand} \mathrel{+}= Q_{\text{actual}}$.
     - Giảm hàng đang về: $\text{On-order} \mathrel{-}= Q_{\text{actual}}$.
     - Số lượng còn lại ($Q_{\text{remaining}} = Q_{\text{PO}} - Q_{\text{actual}}$) tiếp tục được lưu giữ trong $\text{On-order}$.
  3. Đơn PO chuyển trạng thái sang `PARTIALLY_RECEIVED`.
  4. Ghi nhận audit log cho đợt nhập này.
- **Luồng ngoại lệ:** Nếu NCC thông báo không giao đợt tiếp, Quản lý kho có thể bấm "Đóng đơn sớm" $\implies$ hệ thống trừ nốt $Q_{\text{remaining}}$ khỏi $\text{On-order}$ và chuyển PO sang `CLOSED`.
- **Kết quả & Số dư:** $\text{On-hand}$ tăng đúng số thực nhận; $\text{On-order}$ giảm tương ứng, không bị mất dấu số lượng còn lại.
- **Tiêu chí nghiệm thu:** PO đặt 100, đợt 1 nhận 60 $\implies$ $\text{On-hand} \mathrel{+}= 60, \text{On-order}$ còn 40, PO là `PARTIALLY_RECEIVED`.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 4 tại `01_problem_scope.md`.

---

### PHÂN HỆ 5: NGHIỆP VỤ XUẤT KHO VÀ GIỮ CHỖ (FR-OUT)

#### `FR-OUT-001`: Tiếp nhận đơn xuất và Giữ chỗ tồn kho (Stock Reservation)
- **Tác nhân:** Hệ thống (tự động) hoặc Nhân viên điều phối.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Khóa giữ chỗ số lượng hàng trong kho khi đơn xuất được gán vào kho đó.
- **Điều kiện trước:** Kho được chỉ định có $\text{Available} \ge Q_{\text{order}}$.
- **Luồng chính:**
  1. Kiểm tra tồn khả dụng tại kho xuất.
  2. Thực hiện giữ chỗ trong transaction:
     - Tăng tồn giữ chỗ: $\text{Allocated} \mathrel{+}= Q_{\text{order}}$.
     - Tồn khả dụng giảm tương ứng: $\text{Available} \mathrel{-}= Q_{\text{order}}$.
     - Tồn thực tế $\text{On-hand}$ **chưa thay đổi** (hàng vẫn còn trong kho).
  3. Đơn xuất chuyển sang trạng thái `ALLOCATED`.
- **Luồng ngoại lệ:** Nếu $\text{Available} < Q_{\text{order}}$, từ chối giữ chỗ, chuyển đơn sang `UNFULFILLABLE` và báo thiếu hàng.
- **Kết quả & Số dư:** Hàng bị khóa độc quyền cho đơn hàng này, đơn khác không thể lấy.
- **Tiêu chí nghiệm thu:** Kho có $\text{Available} = 10$, giữ chỗ 4 $\implies \text{Allocated} = 4, \text{Available} = 6, \text{On-hand} = 10$.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 1 tại `01_problem_scope.md`.

#### `FR-OUT-002`: Hủy giữ chỗ khi hủy đơn xuất trước khi xuất vật lý (Reservation Cancellation)
- **Tác nhân:** Nhân viên điều phối, Quản lý kho.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Hoàn trả lại tồn khả dụng khi đơn xuất bị hủy trước khi đưa hàng ra khỏi kho.
- **Điều kiện trước:** Đơn xuất ở trạng thái `ALLOCATED` (chưa xuất kho vật lý).
- **Luồng chính:**
  1. Người dùng bấm lệnh Hủy đơn xuất.
  2. Hệ thống giải phóng giữ chỗ:
     - Giảm tồn giữ chỗ: $\text{Allocated} \mathrel{-}= Q_{\text{order}}$.
     - Phục hồi tồn khả dụng: $\text{Available} \mathrel{+}= Q_{\text{order}}$.
     - Tồn thực tế $\text{On-hand}$ giữ nguyên không đổi.
  3. Đơn xuất chuyển trạng thái sang `CANCELLED`.
  4. Ghi nhận audit log loại `RESERVATION_CANCELLED`.
- **Luồng ngoại lệ:** Nếu đơn đã ở trạng thái `DISPATCHED` (đã xuất vật lý), nút hủy bị vô hiệu hóa; hệ thống từ chối yêu cầu hủy.
- **Kết quả & Số dư:** $\text{Available}$ phục hồi nguyên vẹn; $\text{Allocated}$ giải phóng về 0.
- **Tiêu chí nghiệm thu:** Hủy đơn giữ chỗ 4 cái $\implies \text{Allocated}$ giảm 4, $\text{Available}$ tăng 4.
- **Tham chiếu ca kiểm thử:** Kiểm tra luồng hủy giữ chỗ đơn hàng.

#### `FR-OUT-003`: Xác nhận xuất kho vật lý (Goods Dispatch Note)
- **Tác nhân:** Nhân viên kho.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Xác nhận hàng thực tế đã được bốc dỡ ra khỏi kho giao cho đơn vị vận tải.
- **Điều kiện trước:** Đơn xuất ở trạng thái `ALLOCATED`.
- **Luồng chính:**
  1. Nhân viên kho xác nhận phiếu GDN hoàn tất xuất kho.
  2. Hệ thống cập nhật số dư đồng thời:
     - Giảm tồn thực tế: $\text{On-hand} \mathrel{-}= Q_{\text{order}}$.
     - Giải phóng phần giữ chỗ: $\text{Allocated} \mathrel{-}= Q_{\text{order}}$.
     - Tồn khả dụng $\text{Available}$ không đổi.
  3. Đơn xuất chuyển sang trạng thái `DISPATCHED`.
  4. Ghi nhận audit log loại `OUTBOUND_DISPATCH`.
- **Kết quả & Số dư:** Hàng thực tế rời kho; $\text{On-hand}$ giảm chính xác.
- **Tiêu chí nghiệm thu:** Xuất kho đơn 4 cái $\implies \text{On-hand}$ giảm 4, $\text{Allocated}$ giảm 4, $\text{Available}$ không đổi.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 8 tại `01_problem_scope.md`.

#### `FR-OUT-004`: Kiểm soát tương tranh xuất kho đồng thời (Concurrency Control)
- **Tác nhân:** Hệ thống.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Ngăn chặn race condition khi nhiều yêu cầu xuất hàng/giữ chỗ đồng thời cùng tác động vào một SKU tại cùng thời điểm.
- **Điều kiện trước:** 2 hoặc nhiều client gửi request trong khoảng chênh lệch thời gian nhỏ hơn 10ms.
- **Luồng chính:**
  1. Áp dụng cơ chế khóa bi quan (`SELECT ... FOR UPDATE`) trên bản ghi số dư tồn kho.
  2. Tuần tự hóa các giao dịch: Giao dịch 1 giữ khóa, kiểm tra $\text{Available}$, trừ số dư và commit.
  3. Giao dịch 2 nhận khóa, đọc số dư mới nhất. Nếu không còn đủ $\implies$ trả về lỗi HTTP 400.
- **Kết quả & Số dư:** Đúng 1 giao dịch thành công; tuyệt đối không làm âm tồn kho.
- **Tiêu chí nghiệm thu:** 2 tiến trình cùng đòi xuất 4 cái khi kho chỉ còn 5 cái $\implies$ 1 tiến trình thành công, 1 tiến trình bị từ chối; tồn kho còn 1.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 2 tại `01_problem_scope.md`.

---

### PHÂN HỆ 6: NGHIỆP VỤ ĐIỀU CHUYỂN KHO NỘI BỘ (FR-TRF)

#### `FR-TRF-001`: Lập lệnh điều chuyển kho nội bộ (Create Transfer Request)
- **Tác nhân:** Chuyên viên kế hoạch (tạo), Quản lý kho (duyệt).
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Lập yêu cầu luân chuyển hàng giữa 2 kho nội bộ (ví dụ: từ CDC sang RDC).
- **Điều kiện trước:** Kho gửi $A$ có $\text{Available}_A \ge Q_{\text{transfer}}$.
- **Luồng chính:**
  1. Tạo lệnh điều chuyển từ kho $A$ sang kho $B$ với số lượng $Q_{\text{transfer}}$.
  2. Khi lệnh được duyệt, hệ thống cập nhật:
     - Kho gửi $A$: Tăng giữ chỗ $\text{Allocated}_A \mathrel{+}= Q_{\text{transfer}} \implies \text{Available}_A$ giảm $Q_{\text{transfer}}$.
     - Kho nhận $B$: Ghi nhận hàng đang về $\text{On-order}_B \mathrel{+}= Q_{\text{transfer}}$.
     - Tồn thực tế $\text{On-hand}$ tại cả 2 kho **chưa thay đổi**.
  3. Lệnh chuyển ở trạng thái `REQUESTED`.
- **Luồng ngoại lệ:** Nếu $\text{Available}_A < Q_{\text{transfer}}$, từ chối tạo lệnh và báo lỗi không đủ hàng.
- **Kết quả & Số dư:** Hàng tại kho gửi bị khóa giữ chỗ; kho nhận biết có hàng sắp về.
- **Tiêu chí nghiệm thu:** Chuyển 10 cái từ A sang B $\implies \text{Allocated}_A \mathrel{+}= 10, \text{On-order}_B \mathrel{+}= 10, \text{On-hand}$ giữ nguyên.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 3 tại `01_problem_scope.md`.

#### `FR-TRF-002`: Hủy lệnh điều chuyển trước khi xuất kho gửi
- **Tác nhân:** Người tạo lệnh, Quản lý kho.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Hoàn trả số dư khi lệnh điều chuyển bị hủy trước khi bốc hàng lên xe.
- **Điều kiện trước:** Lệnh chuyển ở trạng thái `REQUESTED` (chưa xuất kho gửi).
- **Luồng chính:**
  1. Người dùng bấm Hủy lệnh chuyển.
  2. Hệ thống hoàn trả:
     - Kho gửi $A$: Giảm $\text{Allocated}_A \mathrel{-}= Q_{\text{transfer}} \implies \text{Available}_A$ phục hồi.
     - Kho nhận $B$: Giảm $\text{On-order}_B \mathrel{-}= Q_{\text{transfer}}$.
  3. Lệnh chuyển chuyển sang trạng thái `CANCELLED`.
- **Kết quả & Số dư:** Phục hồi $\text{Available}_A$; xóa hàng đang về ảo tại $B$.
- **Tiêu chí nghiệm thu:** Hủy lệnh chuyển 10 cái $\implies \text{Available}_A$ tăng lại 10, $\text{On-order}_B$ giảm 10.
- **Tham chiếu ca kiểm thử:** Kiểm tra luồng hủy lệnh điều chuyển.

#### `FR-TRF-003`: Xác nhận xuất kho gửi (Dispatch Transfer)
- **Tác nhân:** Nhân viên kho gửi ($W_A$).
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Xác nhận hàng thực tế rời kho gửi và chuyển sang trạng thái hàng đi đường.
- **Điều kiện trước:** Lệnh chuyển ở trạng thái `REQUESTED`.
- **Luồng chính:**
  1. Nhân viên kho gửi xác nhận xuất hàng.
  2. Hệ thống cập nhật:
     - Kho gửi $A$: Giảm tồn thực tế $\text{On-hand}_A \mathrel{-}= Q_{\text{transfer}}$ và giải phóng $\text{Allocated}_A \mathrel{-}= Q_{\text{transfer}}$.
     - Hệ thống: Tăng số lượng hàng đi đường $\text{In-transit} \mathrel{+}= Q_{\text{transfer}}$.
     - Kho nhận $B$: $\text{On-hand}_B$ và $\text{Available}_B$ **vẫn giữ nguyên chưa tăng**.
  3. Lệnh chuyển chuyển sang trạng thái `IN_TRANSIT`.
  4. Ghi nhận audit log loại `TRANSFER_DISPATCHED`.
- **Kết quả & Số dư:** Hàng rời kho gửi, chuyển sang $\text{In-transit}$; kho nhận chưa có hàng.
- **Tiêu chí nghiệm thu:** Xuất kho A 10 cái $\implies \text{On-hand}_A$ giảm 10, $\text{In-transit}$ tăng 10, $\text{On-hand}_B$ chưa đổi.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 3 tại `01_problem_scope.md`.

#### `FR-TRF-004`: Xác nhận nhập kho nhận đầy đủ (Full Transfer Receipt)
- **Tác nhân:** Nhân viên kho nhận ($W_B$).
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Xác nhận hàng chuyển đến nơi đầy đủ ($Q_{\text{received}} = Q_{\text{transfer}}$).
- **Điều kiện trước:** Lệnh chuyển ở trạng thái `IN_TRANSIT`.
- **Luồng chính:**
  1. Nhân viên kho nhận kiểm đếm đủ, xác nhận phiếu nhập chuyển kho.
  2. Cập nhật số dư:
     - Kho nhận $B$: Tăng tồn thực tế $\text{On-hand}_B \mathrel{+}= Q_{\text{received}}$.
     - Kho nhận $B$: Xóa hàng đang về $\text{On-order}_B \mathrel{-}= Q_{\text{transfer}}$.
     - Hệ thống: Xóa hàng đi đường $\text{In-transit} \mathrel{-}= Q_{\text{transfer}}$.
     - $\text{Available}_B$ tự động tăng theo công thức.
  3. Lệnh chuyển chuyển sang trạng thái `COMPLETED`.
  4. Ghi nhận audit log loại `TRANSFER_RECEIVED`.
- **Kết quả & Số dư:** Hàng chính thức vào kho nhận; đóng toàn bộ trạng thái đi đường.
- **Tiêu chí nghiệm thu:** Nhận đủ 10 cái $\implies \text{On-hand}_B$ tăng 10, $\text{In-transit}$ về 0, lệnh là `COMPLETED`.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 8 tại `01_problem_scope.md`.

#### `FR-TRF-005`: Xử lý nhận thiếu và ghi nhận hao hụt điều chuyển (Transfer Under-receiving & Discrepancy)
- **Tác nhân:** Nhân viên kho nhận ($W_B$).
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Xử lý chính xác trường hợp hàng đến nơi bị thiếu hụt hoặc hư hỏng ($Q_{\text{received}} < Q_{\text{transfer}}$).
- **Điều kiện trước:** Lệnh chuyển ở trạng thái `IN_TRANSIT`.
- **Luồng chính:**
  1. Nhân viên kho nhận nhập số thực nhận $Q_{\text{received}}$ và lý do thiếu.
  2. Hệ thống cập nhật số dư:
     - Kho nhận $B$: Chỉ tăng tồn thực tế đúng số thực nhận: $\text{On-hand}_B \mathrel{+}= Q_{\text{received}}$.
     - Kho nhận $B$: Xóa toàn bộ số lượng lệnh khỏi $\text{On-order}_B$ ($\mathrel{-}= Q_{\text{transfer}}$).
     - Hệ thống: Xóa toàn bộ số lượng lệnh khỏi $\text{In-transit}$ ($\mathrel{-}= Q_{\text{transfer}}$), không treo lơ lửng số dư đi đường.
  3. Tự động ghi 01 bản ghi vào bảng `discrepancy_log`:
     - $\Delta Q = Q_{\text{transfer}} - Q_{\text{received}}$ (số lượng thất thoát).
     - Mã lệnh chuyển tham chiếu, lý do và thời gian xác nhận.
  4. Lệnh chuyển chuyển sang trạng thái `COMPLETED_WITH_DISCREPANCY`.
- **Kết quả & Số dư:** Kho nhận tăng đúng thực nhận; xóa sạch $\text{In-transit}$; lưu vết hao hụt minh bạch.
- **Tiêu chí nghiệm thu:** Chuyển 10, nhận 9 $\implies \text{On-hand}_B \mathrel{+}= 9$, $\text{In-transit}$ về 0, phát sinh log thiếu 1 cái.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 3 tại `01_problem_scope.md`.

---

### PHÂN HỆ 7: PHÂN BỔ ĐƠN HÀNG ĐA KHO (FR-ORD)

#### `FR-ORD-001`: Phân bổ đơn hàng trọn gói từ 01 kho (Single-Warehouse Allocation)
- **Tác nhân:** Hệ thống (tự động khi đơn hàng vào hệ thống).
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Tự động chọn 01 kho xuất hàng trọn gói cho đơn hàng dựa trên danh sách thứ tự ưu tiên kho được cấu hình sẵn.
- **Điều kiện trước:** Đơn hàng có danh sách các SKU và số lượng $Q_{\text{order}, i}$.
- **Luồng chính:**
  1. Hệ thống duyệt danh sách kho theo thứ tự ưu tiên ($W_1 > W_2 > ...$).
  2. Kiểm tra điều kiện trọn gói: $\text{Available}_{W_k, i} \ge Q_{\text{order}, i} \quad (\forall i \in \text{đơn hàng})$.
  3. Chọn kho đầu tiên thỏa mãn:
     - Gán kho xuất cho đơn hàng.
     - Thực hiện giữ chỗ tồn kho tại kho đó (`FR-OUT-001`).
     - Đơn hàng chuyển sang trạng thái `ALLOCATED`.
- **Kết quả & Số dư:** Đơn hàng được giữ chỗ trọn vẹn tại 1 kho duy nhất; không bị xé lẻ kiện hàng.
- **Tiêu chí nghiệm thu:** Đơn cần 5 cái SKU1 và 3 cái SKU2; Kho 1 (ưu tiên 1) thiếu hàng, Kho 2 (ưu tiên 2) đủ hàng $\implies$ hệ thống tự động gán Kho 2.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 8 tại `01_problem_scope.md`.

#### `FR-ORD-002`: Xử lý đơn hàng không thể phân bổ tự động (Stockout Alert)
- **Tác nhân:** Hệ thống (tự động) và Nhân viên điều phối.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Xử lý an toàn khi không có kho nào đáp ứng được toàn bộ đơn hàng; tuyệt đối không tự ý chia nhỏ đơn hàng trong Core MVP.
- **Điều kiện trước:** Đã duyệt hết danh sách kho nhưng không kho nào có đủ toàn bộ mặt hàng.
- **Luồng chính:**
  1. Hệ thống từ chối phân bổ tự động.
  2. Chuyển trạng thái đơn hàng sang `UNFULFILLABLE`.
  3. Kích hoạt Cảnh báo thiếu hàng (Stockout Alert) hiển thị trên màn hình điều phối, nêu rõ SKU và số lượng thiếu.
  4. Nhân viên điều phối xem xét xử lý thủ công (chờ nhập hàng hoặc tạo lệnh điều chuyển bổ sung trước).
- **Kết quả & Số dư:** Đơn hàng được kiểm soát an toàn; không gây xuất âm hoặc giữ chỗ phân mảnh.
- **Tiêu chí nghiệm thu:** Đơn cần 10 cái, 2 kho mỗi kho chỉ có 6 cái $\implies$ đơn thành `UNFULFILLABLE` và kích hoạt cảnh báo thiếu hàng.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 8 tại `01_problem_scope.md`.

#### `FR-ORD-003`: Cơ chế tự động tách đơn hàng đa kho (Multi-warehouse Order Splitting)
- **Tác nhân:** Hệ thống.
- **Mức ưu tiên:** Should (Advanced MVP - Chỉ thực hiện sau khi Core MVP ổn định).
- **Mô tả:** Cho phép hệ thống tự động chia nhỏ đơn hàng thành nhiều kiện hàng con (Sub-orders) để xuất từ nhiều kho khác nhau khi không có kho nào đủ hàng trọn vẹn, nhằm tối đa hóa tỷ lệ đáp ứng đơn hàng.

---

### PHÂN HỆ 8: DỊCH VỤ AI DỰ BÁO NHU CẦU (FR-AI)

#### `FR-AI-001`: Sinh dữ liệu tiêu thụ tổng hợp có kiểm soát (Synthetic Data Generator)
- **Tác nhân:** Kỹ sư dữ liệu / Nhà phát triển.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Script sinh dữ liệu chuỗi thời gian tiêu thụ lịch sử theo ngày cho tập SKU và Kho mô phỏng.
- **Điều kiện trước:** Cấu hình tham số: số SKU (20–50), số Kho (2–3), thời gian (12–24 tháng), `random_seed`.
- **Luồng chính:**
  1. Sinh dữ liệu theo công thức: $D_t = \text{BaseLevel} + \text{Trend}(t) + \text{Seasonality}(t) + \epsilon_t$.
  2. Tích hợp kịch bản thực tế: ngày nhu cầu bằng 0 (intermittent demand), ngày đột biến tiêu thụ (spikes), nhiễu ngẫu nhiên Gaussian.
  3. Xuất ra định dạng chuẩn `(sku_id, warehouse_id, date, demand_quantity)`.
- **Kết quả & Số dư:** Bộ dữ liệu giả lập có quy luật thống kê rõ ràng, tái lập được 100% khi chạy lại cùng seed.
- **Tiêu chí nghiệm thu:** Chạy script 2 lần cùng seed 42 $\implies$ dữ liệu sinh ra giống hệt nhau.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 7 tại `01_problem_scope.md`.

#### `FR-AI-002`: Phân chia tập dữ liệu huấn luyện và đánh giá theo thời gian (Temporal Split)
- **Tác nhân:** AI Service Pipeline.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Chia chuỗi thời gian tuần tự theo mốc thời gian để loại trừ nguy cơ rò rỉ dữ liệu tương lai (Data Leakage).
- **Điều kiện trước:** Dữ liệu lịch sử đã được sắp xếp tăng dần theo `date`.
- **Luồng chính:**
  1. Cắt tập dữ liệu theo tỷ lệ:
     - Train set: 70–80% thời gian đầu.
     - Validation set: 10–15% thời gian tiếp theo.
     - Test set: 10–15% thời gian cuối cùng.
  2. Tuyệt đối không dùng k-fold ngẫu nhiên hoặc xáo trộn (shuffle) dòng.
- **Kết quả & Số dư:** 3 tập dữ liệu độc lập về thời gian.
- **Tiêu chí nghiệm thu:** $\max(\text{date}_{\text{train}}) < \min(\text{date}_{\text{val}}) < \min(\text{date}_{\text{test}})$.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 7 tại `01_problem_scope.md`.

#### `FR-AI-003`: Huấn luyện mô hình dự báo và So sánh Baseline
- **Tác nhân:** AI Service Pipeline.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Huấn luyện mô hình học máy ứng viên và so sánh đối đầu với mô hình baseline truyền thống trên cùng tập Test set.
- **Điều kiện trước:** Tập dữ liệu Train và Test set; khoảng dự báo $H = 7 \text{ đến } 14$ ngày tới.
- **Luồng chính:**
  1. Huấn luyện mô hình Baseline: Rolling Moving Average (7 ngày / 14 ngày) và Naive (chu kỳ tuần trước).
  2. Huấn luyện mô hình ứng viên (ví dụ: LightGBM Regressor với các đặc trưng lag, rolling mean, thứ trong tuần).
  3. Dự báo trên tập Test set cho cùng khoảng thời gian $H$.
- **Kết quả & Số dư:** Xuất mô hình đã huấn luyện; không mặc định mô hình nào tốt nhất trước khi có số liệu thực nghiệm.
- **Tiêu chí nghiệm thu:** Pipeline chạy hoàn tất, lưu được model artifact `.pkl`.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 7 tại `01_problem_scope.md`.

#### `FR-AI-004`: Đo lường và Báo cáo sai số dự báo (Forecast Error Evaluation)
- **Tác nhân:** AI Service Pipeline.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Đo lường độ lệch giữa nhu cầu dự đoán và thực tế trên tập Test bằng các chỉ số thống kê chuẩn mực.
- **Điều kiện trước:** Chuỗi thực tế $y_t$ và chuỗi dự báo $\hat{y}_t$ trên tập Test.
- **Luồng chính:**
  1. Tính toán các chỉ số:
     - $\text{MAE} = \frac{1}{N} \sum |y_t - \hat{y}_t|$
     - $\text{RMSE} = \sqrt{\frac{1}{N} \sum (y_t - \hat{y}_t)^2}$
     - $\text{WAPE} = \frac{\sum |y_t - \hat{y}_t|}{\sum y_t}$ (khắc phục nhược điểm chia cho 0 của MAPE).
  2. Xuất bảng so sánh đối đầu giữa mô hình ứng viên và các baseline.
- **Kết quả & Số dư:** Bảng báo cáo sai số minh bạch, trung thực; gắn nhãn rõ là kết quả trên dữ liệu giả lập.
- **Tiêu chí nghiệm thu:** Bảng báo cáo hiển thị đầy đủ MAE, RMSE, WAPE cho từng mô hình; WAPE tính chính xác.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 7 tại `01_problem_scope.md`.

#### `FR-AI-005`: Cung cấp API suy luận dự báo nhu cầu (Inference API)
- **Tác nhân:** Backend Service gọi sang AI Service.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Cung cấp endpoint REST API nhận mã SKU, Kho và trả về mảng dự báo nhu cầu theo từng ngày trong tương lai.
- **Điều kiện trước:** Dịch vụ Python AI đang chạy; mô hình đã được nạp.
- **Luồng chính:**
  1. Backend gửi request HTTP POST `/api/v1/predict` kèm `{"sku_id": "SP01", "warehouse_id": "WH01", "horizon_days": 7}`.
  2. AI Service thực thi suy luận và trả về mảng dự báo theo ngày: `[{"date": "2026-10-17", "demand": 5.2}, ...]`.
- **Luồng ngoại lệ:** Nếu lỗi kết nối hoặc chưa có mô hình, trả về mã lỗi HTTP 500/404 để Backend kích hoạt cơ chế Fallback tĩnh.
- **Kết quả & Số dư:** Trả về dữ liệu dự báo định dạng JSON trong thời gian < 2 giây.
- **Tiêu chí nghiệm thu:** Gọi API suy luận thành công, nhận đúng mảng dự báo cho 7 ngày tới.
- **Tham chiếu ca kiểm thử:** Kiểm tra tích hợp API Backend - AI.

---

### PHÂN HỆ 9: ĐỘNG CƠ HỖ TRỢ QUYẾT ĐỊNH BỔ SUNG TỒN KHO (FR-REC)

#### `FR-REC-001`: Tính toán Điểm đặt hàng lại (ROP) và Tồn an toàn (SS) định lượng
- **Tác nhân:** Động cơ quy tắc tồn kho (Inventory Rule Engine).
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Tự động tính toán ngưỡng ROP và SS dựa trên đầu vào dự báo của AI và tham số Lead time.
- **Điều kiện trước:** Có kết quả dự báo $\hat{d}_{t+k}$ trong $L$ ngày tới từ AI và tham số $L, Z, \sigma_D$.
- **Luồng chính:**
  1. Tính tổng nhu cầu dự báo trong thời gian giao hàng: $\hat{D}_{\text{lead}} = \sum_{k=1}^{L} \hat{d}_{t+k}$.
  2. Tính tồn an toàn: $\text{SS} = Z \times \sigma_D \times \sqrt{L}$.
  3. Tính Điểm đặt hàng lại: $\text{ROP} = \hat{D}_{\text{lead}} + \text{SS}$.
- **Kết quả & Số dư:** Giá trị ROP và SS được cập nhật động theo dự báo mới nhất.
- **Tiêu chí nghiệm thu:** $L=3$, dự báo 3 ngày là $[5, 6, 4] \implies \hat{D}_{\text{lead}} = 15$. Với $\text{SS} = 10 \implies \text{ROP} = 25$.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 6 tại `01_problem_scope.md`.

#### `FR-REC-002`: Quét Vị thế tồn kho và Kích hoạt Cảnh báo thiếu hàng
- **Tác nhân:** Hệ thống (tự động qua Cron job hoặc trigger sau mỗi giao dịch xuất kho).
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** So sánh vị thế tồn kho $\text{IP}$ với ngưỡng $\text{ROP}$ để phát hiện sớm nguy cơ thiếu hàng.
- **Điều kiện trước:** Đã tính toán xong $\text{IP} = \text{Available} + \text{On-order}$ và $\text{ROP}$.
- **Luồng chính:**
  1. So sánh điều kiện: Nếu $\text{IP} \le \text{ROP}$:
     - Đánh dấu trạng thái SKU tại kho là `REORDER_NEEDED`.
     - Kích hoạt cờ cảnh báo trên màn hình điều hành.
- **Kết quả & Số dư:** Cảnh báo xuất hiện kịp thời; không sinh cảnh báo sai khi hàng đang về đã đủ bù đắp.
- **Tiêu chí nghiệm thu:** $\text{IP} = 15, \text{ROP} = 25 \implies$ kích hoạt cảnh báo; nếu $\text{IP} = 26 \implies$ không cảnh báo.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 6 tại `01_problem_scope.md`.

#### `FR-REC-003`: Tính toán Lượng đề xuất bổ sung ($Q_{\text{suggest}}$)
- **Tác nhân:** Động cơ quy tắc tồn kho.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Tính toán số lượng hàng cần bổ sung theo chính sách Order-up-to level $S$.
- **Điều kiện trước:** Đã kích hoạt cảnh báo $\text{IP} \le \text{ROP}$; có tham số mức tồn tối đa $S$.
- **Luồng chính:**
  1. Tính lượng đề xuất: $Q_{\text{suggest}} = \max(\text{MOQ}, S - \text{IP})$.
  2. Lưu bản ghi khuyến nghị vào bảng `replenishment_recommendation` gồm: SKU, Kho, Ngày đề xuất, $\text{IP}, \text{ROP}, \hat{D}_{\text{lead}}, \text{SS}, Q_{\text{suggest}}$, Loại đề xuất (`PURCHASE` từ NCC hoặc `TRANSFER` từ kho trung tâm).
- **Kết quả & Số dư:** Bản ghi khuyến nghị hiển thị minh bạch công thức và lý do tính toán.
- **Tiêu chí nghiệm thu:** $S = 50, \text{IP} = 15 \implies Q_{\text{suggest}} = 35$ (không lấy $S - \text{Available}$ vì sẽ bị đề xuất thừa hàng đang về).
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 6 tại `01_problem_scope.md`.

#### `FR-REC-004`: Cơ chế Fallback về Quy tắc tĩnh khi thiếu dữ liệu (Cold-start Policy)
- **Tác nhân:** Hệ thống.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Tự động chuyển sang sử dụng tham số tĩnh cấu hình thủ công khi SKU chưa có đủ dữ liệu chuỗi thời gian cho AI (dưới 30 ngày).
- **Điều kiện trước:** SKU mới hoặc AI service tạm thời mất kết nối.
- **Luồng chính:**
  1. Hệ thống phát hiện thiếu dữ liệu AI.
  2. Lấy tham số $\text{ROP}_{\text{static}}$ và $\text{SS}_{\text{static}}$ từ danh mục (`FR-MST-004`).
  3. Tính toán đề xuất bổ sung và hiển thị nhãn: *"Đề xuất tính theo tham số tĩnh do SKU chưa đủ lịch sử dữ liệu cho AI"*.
- **Kết quả & Số dư:** Vận hành kho không bị gián đoạn; người dùng nhận biết rõ căn cứ tính toán.
- **Tiêu chí nghiệm thu:** SKU mới tạo chưa có dữ liệu AI vẫn sinh được đề xuất bổ sung theo tham số tĩnh.
- **Tham chiếu ca kiểm thử:** Kiểm tra cơ chế Fallback tĩnh.

#### `FR-REC-005`: Phê duyệt đề xuất bổ sung (Human-in-the-loop Approval)
- **Tác nhân:** Chuyên viên kế hoạch (xem xét), Quản lý kho (phê duyệt).
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Cho phép con người xem xét, điều chỉnh số lượng và bấm nút chuyển khuyến nghị thành giao dịch mua hàng hoặc điều chuyển.
- **Điều kiện trước:** Có khuyến nghị ở trạng thái `PENDING_APPROVAL`.
- **Luồng chính:**
  1. Người dùng mở danh sách khuyến nghị, xem xét căn cứ tính toán.
  2. Người dùng có thể điều chỉnh số lượng $Q$ nếu cần.
  3. Bấm "Phê duyệt mua hàng" $\implies$ Tự động sinh đơn mua PO ở trạng thái `DRAFT` (`FR-INB-001`).
  4. Bấm "Phê duyệt điều chuyển" $\implies$ Tự động sinh Lệnh chuyển kho ở trạng thái `REQUESTED` (`FR-TRF-001`).
  5. Khuyến nghị chuyển sang trạng thái `APPROVED`.
- **Luồng ngoại lệ:** Người dùng bấm "Từ chối" kèm lý do $\implies$ Khuyến nghị chuyển sang `REJECTED`.
- **Kết quả & Số dư:** Bảo đảm con người giữ quyền quyết định cuối cùng; hệ thống chỉ đóng vai trò trợ lý hỗ trợ.
- **Tiêu chí nghiệm thu:** Duyệt khuyến nghị 35 cái mua hàng $\implies$ sinh ra đúng đơn PO với số lượng 35.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 8 tại `01_problem_scope.md`.

#### `FR-REC-006`: Mô phỏng đánh giá chính sách tồn kho (Inventory Policy Simulation)
- **Tác nhân:** Script mô phỏng độc lập.
- **Mức ưu tiên:** Should (Advanced MVP - Chỉ thực hiện sau khi Core MVP ổn định).
- **Mô tả:** Chạy mô phỏng vòng lặp vận hành theo từng ngày trên tập Test set: Giả lập việc nhận hàng, bán hàng, kích hoạt ROP và bổ sung theo gợi ý của AI. So sánh với kịch bản áp dụng chính sách tồn kho cố định truyền thống để đo lường tỷ lệ đáp ứng đơn hàng (Service Level), số ngày thiếu hàng (Stockout Days) và lượng tồn trữ bình quân (Average Holding Inventory). Ghi rõ toàn bộ kết quả là trên môi trường giả lập.

---

### PHÂN HỆ 10: NHẬT KÝ KIỂM TOÁN VÀ IDEMPOTENCY (FR-AUD)

#### `FR-AUD-001`: Ghi nhận nhật ký biến động kho bất biến (Stock Movement Audit Log)
- **Tác nhân:** Hệ thống (tự động ghi nhận trong mọi transaction thay đổi số dư).
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Ghi lại 100% các biến động số dư kho vào bảng nhật ký chỉ ghi (`APPEND-ONLY`), không cho phép sửa đổi hoặc xóa.
- **Điều kiện trước:** Bất kỳ thao tác nào làm thay đổi `On-hand`, `Allocated`, `On-order`, `In-transit`.
- **Luồng chính:**
  1. Trong cùng transaction nghiệp vụ, hệ thống ghi 01 bản ghi vào bảng `stock_movement_log` gồm: mã giao dịch, SKU, Kho, thời gian, loại biến động (`INBOUND_RECEIPT`, `OUTBOUND_DISPATCH`, `TRANSFER_DISPATCH`, ...), số lượng thay đổi, số dư sau thay đổi, mã phiếu tham chiếu, mã người thực hiện.
  2. Bảng nhật ký từ chối mọi lệnh `UPDATE` hoặc `DELETE` thông qua quyền hạn cơ sở dữ liệu.
- **Kết quả & Số dư:** Dữ liệu lịch sử biến động minh bạch 100%, phục vụ đối soát và kiểm toán.
- **Tiêu chí nghiệm thu:** Thực hiện 1 lệnh xuất kho $\implies$ kiểm tra bảng log có đúng 1 bản ghi ghi nhận số lượng giảm và mã phiếu tương ứng.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 8 tại `01_problem_scope.md`.

#### `FR-AUD-002`: Cơ chế khóa Idempotency Key chống ghi nhận giao dịch trùng lặp
- **Tác nhân:** Hệ thống API.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Ngăn chặn việc gửi lại request API do người dùng click đúp hoặc lỗi mạng retry làm cộng/trừ số dư nhiều lần.
- **Điều kiện trước:** Client gửi request kèm header `Idempotency-Key` duy nhất.
- **Luồng chính:**
  1. Kiểm tra mã key trong bảng `idempotency_keys`.
  2. Nếu key chưa có: Lưu key với trạng thái `PROCESSING`, thực thi nghiệp vụ kho, lưu kết quả phản hồi HTTP và cập nhật trạng thái `COMPLETED`.
  3. Nếu key đã có ở trạng thái `COMPLETED`: Bỏ qua việc thực thi logic kho; trả về ngay lập tức kết quả của lần thực thi trước đó.
- **Kết quả & Số dư:** Số dư tồn kho chỉ tăng/giảm đúng một lần duy nhất dù gửi request bao nhiêu lần.
- **Tiêu chí nghiệm thu:** Gửi 2 request xác nhận nhập kho cùng key liên tiếp $\implies$ tồn kho chỉ tăng 1 lần; request 2 trả về kết quả thành công cũ.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 5 tại `01_problem_scope.md`.

---

### PHÂN HỆ 11: BẢNG ĐIỀU KHIỂN VÀ CẢNH BÁO (FR-DASH)

#### `FR-DASH-001`: Bảng điều khiển giám sát tồn kho và Cảnh báo thiếu hàng
- **Tác nhân:** Quản lý kho, Chuyên viên kế hoạch.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Cung cấp màn hình dashboard trực quan hiển thị tổng quan số dư các kho, danh sách các mặt hàng chạm ngưỡng nguy hiểm ($IP \le ROP$) và danh sách khuyến nghị chờ duyệt.
- **Điều kiện trước:** Người dùng đã đăng nhập hệ thống.
- **Luồng chính:**
  1. Hiển thị các thẻ KPI: Tổng SKU quản lý, Số SKU an toàn, Số SKU cần bổ sung, Tổng giá trị tồn kho giả lập.
  2. Bảng cảnh báo thiếu hàng nổi bật các dòng SKU có $\text{IP} \le \text{ROP}$.
  3. Widget danh sách khuyến nghị bổ sung hàng cho phép bấm nhanh vào duyệt.
- **Kết quả & Số dư:** Cung cấp bức tranh toàn cảnh giúp người quản trị ra quyết định nhanh chóng.
- **Tiêu chí nghiệm thu:** SKU chạm ROP lập tức xuất hiện trong danh sách cảnh báo trên Dashboard.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 6 và 8 tại `01_problem_scope.md`.

#### `FR-DASH-002`: Trực quan hóa chuỗi thời gian tiêu thụ và Dự báo AI
- **Tác nhân:** Chuyên viên kế hoạch, Quản lý kho.
- **Mức ưu tiên:** Must (Core MVP).
- **Mô tả:** Hiển thị biểu đồ chuỗi thời gian tiêu thụ lịch sử, đường dự báo tương lai của AI và đường so sánh baseline cho từng SKU được chọn.
- **Điều kiện trước:** Người dùng chọn 1 SKU và 1 Kho cụ thể.
- **Luồng chính:**
  1. Hệ thống tải dữ liệu lịch sử và mảng dự báo từ cơ sở dữ liệu.
  2. Vẽ biểu đồ đường (Line Chart):
     - Đoạn quá khứ: Đường tiêu thụ thực tế.
     - Đoạn tương lai: Đường dự báo AI (nét liền) đối đầu đường Moving Average baseline (nét đứt).
  3. Hiển thị bảng tóm tắt chỉ số sai số WAPE và MAE của mô hình.
- **Kết quả & Số dư:** Trực quan hóa dữ liệu minh bạch, giải thích được lý do đằng sau các con số đề xuất.
- **Tiêu chí nghiệm thu:** Biểu đồ vẽ thông suốt, phân định rõ ràng giữa đoạn lịch sử và đoạn dự báo tương lai.
- **Tham chiếu ca kiểm thử:** Ca kiểm thử 7 tại `01_problem_scope.md`.

---

## Phần 4. Đặc tả các Luồng Nghiệp vụ (Business Workflows)

Phần này mô tả chi tiết trạng thái trước/sau và tác động số dư của 7 luồng nghiệp vụ xương sống:

### 4.1. Luồng Nhập hàng từ Nhà cung cấp (Nhận đủ và Nhận một phần)
- **Tác nhân:** Chuyên viên kế hoạch (tạo PO), Quản lý kho (duyệt PO), Nhân viên kho (tạo GRN).
- **Tiền điều kiện:** Đã có danh mục NCC và SKU.
- **Trình tự thực hiện:**
  1. *Bước 1 (Lập PO):* Tạo PO $\rightarrow$ Duyệt PO $\implies$ Trạng thái PO: `CONFIRMED`.  
     *Tác động tồn kho:* Kho đích tăng $\text{On-order} \mathrel{+}= Q_{\text{PO}}$. Tồn $\text{On-hand}$ và $\text{Available}$ không đổi.
  2. *Bước 2 (Giao nhận vật lý):* Hàng về kho. Nhân viên kiểm đếm số lượng thực nhận $Q_{\text{actual}}$.
  3. *Bước 3a (Nếu nhận đủ - $Q_{\text{actual}} = Q_{\text{PO}}$):*  
     Tạo GRN hoàn tất $\implies$ PO chuyển `COMPLETED`.  
     *Tác động tồn kho:* $\text{On-hand} \mathrel{+}= Q_{\text{actual}}$, $\text{On-order} \mathrel{-}= Q_{\text{PO}}$, $\text{Available}$ tăng $Q_{\text{actual}}$.
  4. *Bước 3b (Nếu nhận một phần - $Q_{\text{actual}} < Q_{\text{PO}}$):*  
     Tạo GRN đợt này $\implies$ PO chuyển `PARTIALLY_RECEIVED`.  
     *Tác động tồn kho:* $\text{On-hand} \mathrel{+}= Q_{\text{actual}}$, $\text{On-order} \mathrel{-}= Q_{\text{actual}}$. Phần còn lại $(Q_{\text{PO}} - Q_{\text{actual}})$ vẫn nằm trong $\text{On-order}$ chờ đợt sau.

### 4.2. Luồng Giữ chỗ và Xuất hàng (Order Reservation & Dispatch)
- **Tác nhân:** Hệ thống phân bổ (tự động), Nhân viên kho (xuất GDN).
- **Tiền điều kiện:** Đơn hàng được phân bổ vào kho có $\text{Available} \ge Q_{\text{order}}$.
- **Trình tự thực hiện:**
  1. *Bước 1 (Giữ chỗ):* Hệ thống khóa giữ chỗ $\implies$ Trạng thái đơn: `ALLOCATED`.  
     *Tác động tồn kho:* $\text{Allocated} \mathrel{+}= Q_{\text{order}} \implies \text{Available} \mathrel{-}= Q_{\text{order}}$. Tồn thực tế $\text{On-hand}$ giữ nguyên.
  2. *Bước 2 (Gom hàng & Đóng gói):* Nhân viên gom hàng vật lý theo phiếu nhặt hàng (Pick list).
  3. *Bước 3 (Xác nhận xuất kho GDN):* Hàng bốc lên xe $\implies$ Trạng thái đơn: `DISPATCHED`.  
     *Tác động tồn kho:* $\text{On-hand} \mathrel{-}= Q_{\text{order}}$, $\text{Allocated} \mathrel{-}= Q_{\text{order}}$. Tồn khả dụng $\text{Available}$ không đổi.

### 4.3. Luồng Hủy giao dịch trước khi xuất vật lý (Cancellation Flow)
- **Tác nhân:** Quản lý kho, Chuyên viên điều phối.
- **Tiền điều kiện:** Đơn xuất ở trạng thái `ALLOCATED` hoặc Lệnh chuyển ở trạng thái `REQUESTED` (chưa xuất kho vật lý).
- **Trình tự thực hiện:**
  1. Người dùng bấm Hủy giao dịch trên giao diện.
  2. Hệ thống kiểm tra trạng thái: nếu chưa xuất vật lý $\implies$ Cho phép hủy.
  3. *Tác động tồn kho đơn xuất:* $\text{Allocated} \mathrel{-}= Q_{\text{order}} \implies \text{Available} \mathrel{+}= Q_{\text{order}}$. Đơn chuyển `CANCELLED`.
  4. *Tác động tồn kho lệnh chuyển:* Kho gửi giải phóng $\text{Allocated}$, $\text{Available}$ phục hồi; Kho nhận giảm $\text{On-order}$. Lệnh chuyển chuyển `CANCELLED`.
  5. *Ngoại lệ:* Nếu hàng đã ở trạng thái `DISPATCHED` hoặc `IN_TRANSIT` $\implies$ Từ chối hủy; bắt buộc qua quy trình nhập trả.

### 4.4. Luồng Điều chuyển hàng từ Kho gửi đến Kho nhận (Transfer Flow)
- **Tác nhân:** Chuyên viên kế hoạch (tạo), Nhân viên kho gửi (xuất), Nhân viên kho nhận (nhập).
- **Tiền điều kiện:** Kho gửi $A$ có $\text{Available}_A \ge Q_{\text{transfer}}$.
- **Trình tự thực hiện:**
  1. *Bước 1 (Lập lệnh):* Lập lệnh chuyển $A \rightarrow B$ $\implies$ Trạng thái lệnh: `REQUESTED`.  
     *Tác động:* Kho $A$ tăng $\text{Allocated}_A \mathrel{+}= Q$, giảm $\text{Available}_A \mathrel{-}= Q$. Kho $B$ tăng $\text{On-order}_B \mathrel{+}= Q$. $\text{On-hand}$ cả 2 kho chưa đổi.
  2. *Bước 2 (Xuất kho gửi):* Kho $A$ xuất hàng $\implies$ Trạng thái lệnh: `IN_TRANSIT`.  
     *Tác động:* Kho $A$ giảm $\text{On-hand}_A \mathrel{-}= Q$, giảm $\text{Allocated}_A \mathrel{-}= Q$. Hệ thống tăng $\text{In-transit} \mathrel{+}= Q$. Kho $B$ giữ nguyên $\text{On-hand}_B$.
  3. *Bước 3 (Nhập kho nhận):* Kho $B$ nhận đủ hàng $\implies$ Trạng thái lệnh: `COMPLETED`.  
     *Tác động:* Kho $B$ tăng $\text{On-hand}_B \mathrel{+}= Q$, giảm $\text{On-order}_B \mathrel{-}= Q$. Hệ thống giảm $\text{In-transit} \mathrel{-}= Q$. $\text{Available}_B$ tự động tăng tương ứng.

### 4.5. Luồng Xử lý nhận thiếu và Ghi nhận hao hụt điều chuyển (Under-receiving Flow)
- **Tác nhân:** Nhân viên kho nhận.
- **Tiền điều kiện:** Lệnh điều chuyển đang ở trạng thái `IN_TRANSIT`; thực nhận $Q_{\text{received}} < Q_{\text{transfer}}$.
- **Trình tự thực hiện:**
  1. Nhân viên kiểm đếm, nhập số lượng thực nhận $Q_{\text{received}}$ và lý do thiếu hụt (vỡ, mất mát).
  2. Hệ thống cập nhật:
     - Kho $B$: Chỉ tăng tồn thực tế đúng số thực nhận: $\text{On-hand}_B \mathrel{+}= Q_{\text{received}}$, $\text{Available}_B \mathrel{+}= Q_{\text{received}}$.
     - Kho $B$: Xóa toàn bộ số lượng lệnh khỏi $\text{On-order}_B$ ($\mathrel{-}= Q_{\text{transfer}}$).
     - Hệ thống: Xóa toàn bộ số lượng lệnh khỏi $\text{In-transit}$ ($\mathrel{-}= Q_{\text{transfer}}$) để đóng lệnh chuyển, không để treo số dư đi đường.
  3. Ghi bản ghi vào `discrepancy_log`: $\Delta Q = Q_{\text{transfer}} - Q_{\text{received}}$, mã lệnh chuyển, lý do, người xác nhận.
  4. Trạng thái lệnh chuyển: `COMPLETED_WITH_DISCREPANCY`.

### 4.6. Luồng Phân bổ đơn hàng (Single-Warehouse Allocation Flow)
- **Tác nhân:** Hệ thống phân bổ tự động.
- **Tiền điều kiện:** Đơn hàng mới tiếp nhận từ khách hàng.
- **Trình tự thực hiện:**
  1. Hệ thống lấy danh sách kho sắp xếp theo thứ tự ưu tiên cấu hình trước: $W_1 > W_2 > ...$
  2. Lặp qua từng kho $W_k$: Kiểm tra nếu $\text{Available}_{W_k, i} \ge Q_{\text{order}, i}$ với mọi SKU $i$ trong đơn.
  3. *Trường hợp tìm thấy kho thỏa mãn:*  
     - Gán kho xuất là $W_k$.
     - Khóa giữ chỗ tại $W_k$ (`Allocated` tăng, `Available` giảm).
     - Trạng thái đơn: `ALLOCATED`. Kết thúc luồng thành công.
  4. *Trường hợp duyệt hết danh sách không kho nào đủ trọn vẹn:*  
     - Không tự động tách đơn trong Core MVP.
     - Trạng thái đơn: `UNFULFILLABLE`.
     - Kích hoạt Cảnh báo thiếu hàng hiển thị lên màn hình điều phối.

### 4.7. Luồng Sinh và Phê duyệt khuyến nghị bổ sung hàng (Replenishment Flow)
- **Tác nhân:** Động cơ quy tắc tồn kho (sinh), Chuyên viên kế hoạch & Quản lý kho (duyệt).
- **Tiền điều kiện:** Hệ thống đã chạy suy luận dự báo nhu cầu từ AI.
- **Trình tự thực hiện:**
  1. *Bước 1 (Tính toán & Quét):* Hệ thống tính $\text{IP} = \text{Available} + \text{On-order}$ và $\text{ROP} = \hat{D}_{\text{lead}} + \text{SS}$.
  2. *Bước 2 (Kích hoạt đề xuất):* Nếu $\text{IP} \le \text{ROP}$, hệ thống tự động tính $Q_{\text{suggest}} = \max(\text{MOQ}, S - \text{IP})$ và lưu bản ghi khuyến nghị ở trạng thái `PENDING_APPROVAL`.
  3. *Bước 3 (Người dùng xem xét):* Chuyên viên kế hoạch xem danh sách khuyến nghị, kiểm tra lý do tính toán ($\text{IP}, \text{ROP}, \hat{D}, L, \text{SS}$), có thể điều chỉnh số lượng $Q$.
  4. *Bước 4 (Phê duyệt):*
     - Nếu duyệt mua: Hệ thống sinh đơn PO ở trạng thái `DRAFT`.
     - Nếu duyệt chuyển: Hệ thống sinh Lệnh chuyển kho ở trạng thái `REQUESTED`.
     - Khuyến nghị chuyển sang `APPROVED`.

---

## Phần 5. Yêu cầu Chức năng Dịch vụ AI (AI Requirements)

### 5.1. Quy cách Dữ liệu Đầu vào và Đầu ra
- **Dữ liệu đầu vào của dịch vụ AI:**
  - Chuỗi thời gian lịch sử tiêu thụ theo định dạng bảng: `(sku_id, warehouse_id, date, demand_quantity)`.
  - Tham số khoảng dự báo: `horizon_days` (mặc định từ 7 đến 14 ngày tới).
- **Dữ liệu đầu ra của dịch vụ AI:**
  - Mảng giá trị dự báo theo từng ngày trong tương lai: `[(date_{t+1}, demand_{t+1}), ..., (date_{t+H}, demand_{t+H})]`.
  - Báo cáo sai số mô hình trên tập kiểm thử: `{"WAPE": 0.18, "MAE": 3.42, "RMSE": 4.85}`.

### 5.2. Nguyên tắc Huấn luyện và Chia tập Dữ liệu
- **Chia tập theo thời gian bắt buộc (Temporal Split):** Bắt buộc phân chia tuần tự theo trục thời gian: 70–80% thời gian đầu làm Train, 10–15% thời gian kế tiếp làm Validation, 10–15% thời gian cuối làm Test. Tuyệt đối không xáo trộn dòng ngẫu nhiên để loại trừ rò rỉ thông tin tương lai về quá khứ (Data Leakage).
- **So sánh Baseline bắt buộc:** Mô hình ứng viên (như LightGBM hoặc Random Forest) phải được so sánh đối đầu trên cùng một tập Test set và cùng khoảng dự báo với ít nhất 01 mô hình baseline truyền thống: Rolling Moving Average (7 ngày / 14 ngày) và Naive Forecast.
- **Tính khách quan:** Không mặc định bất kỳ mô hình nào là tốt nhất trước khi có số liệu thực nghiệm thực tế chạy trên máy tính.

### 5.3. Tiêu chuẩn Đánh giá và Tách biệt Hai tầng Chỉ số
Hệ thống phân định rạch ròi giữa hai khái niệm:
1. **Độ chính xác dự báo (Forecast Accuracy):** Đo lường sai số toán học giữa nhu cầu dự đoán và thực tế qua các chỉ số $\text{MAE}, \text{RMSE}$ và đặc biệt là $\text{WAPE} = \frac{\sum |y_t - \hat{y}_t|}{\sum y_t}$ (khắc phục lỗi chia cho 0 khi nhu cầu = 0).
2. **Hiệu quả vận hành tồn kho (Inventory Operational Performance):** Chỉ được khẳng định thông qua bài toán mô phỏng (Simulation) trên tập test: đo lường Tỷ lệ đáp ứng đơn (Service Level), Số ngày thiếu hàng (Stockout Days), và Tồn kho bình quân (Average Holding).
3. **Giới hạn dữ liệu giả lập:** Toàn bộ kết quả thử nghiệm chỉ phản ánh hiệu quả trong **kịch bản giả lập có kiểm soát**, không tuyên bố là hiệu quả trên dữ liệu sản xuất của doanh nghiệp thực tế.

### 5.4. Cơ chế Fallback và Nguyên tắc Con người Kiểm soát
- **Cơ chế Fallback tĩnh (Cold-start):** Khi một SKU có lịch sử dưới 30 ngày hoặc dịch vụ AI gặp sự cố kỹ thuật, hệ thống tự động chuyển sang sử dụng tham số $ROP$ và $SS$ cấu hình thủ công trong danh mục, có nhãn cảnh báo rõ ràng trên giao diện.
- **Nguyên tắc con người phê duyệt (Human-in-the-loop):** AI tuyệt đối không tự ý ra quyết định đặt hàng hay tự ý thay đổi số dư kho. Toàn bộ khuyến nghị phải được hiển thị minh bạch công thức và chờ con người bấm nút phê duyệt.

---

## Phần 6. Yêu cầu Phi Chức năng (Non-Functional Requirements)

### 6.1. Tính toàn vẹn giao dịch và Chống tồn âm (NFR-INT)
- **`NFR-INT-001` (Giao dịch ACID):** 100% các thao tác nhập, xuất, chuyển kho phải được thực thi trong một Database Transaction có mức cô lập tối thiểu là `READ_COMMITTED`. Nếu có bất kỳ lỗi nào xảy ra trong chuỗi xử lý, toàn bộ transaction phải được Rollback nguyên trạng.
- **`NFR-INT-002` (Ràng buộc toàn vẹn cơ sở dữ liệu):** Cài đặt Check Constraint cứng ở mức bảng cơ sở dữ liệu: `CHECK (on_hand >= allocated AND on_hand >= 0 AND allocated >= 0)`.

### 6.2. Kiểm soát Tương tranh và Hiệu năng (NFR-PRF & NFR-CON)
- **`NFR-CON-001` (Khóa dữ liệu chống tranh chấp):** Áp dụng khóa bi quan (`SELECT ... FOR UPDATE`) có timeout ngắn tại tầng Repository khi truy vấn số dư tồn kho phục vụ giữ chỗ hoặc xuất kho, bảo đảm chịu tải đồng thời tối thiểu 20 requests cùng lúc không gây Deadlock hoặc race condition.
- **`NFR-PRF-001` (Thời gian phản hồi nghiệp vụ kho):** 95% các yêu cầu API tra cứu tồn kho và ghi nhận phiếu kho phản hồi dưới **500ms** trên môi trường local.
- **`NFR-PRF-002` (Thời gian phản hồi AI):** API suy luận dự báo nhu cầu cho 01 SKU trong khoảng 7–14 ngày phản hồi dưới **2 giây**.

### 6.3. Tính Bất biến và Chống trùng lặp (NFR-IDM & NFR-AUD)
- **`NFR-IDM-001` (Idempotency):** 100% API xác nhận phiếu kho (GRN, GDN, Transfer) phải hỗ trợ header `Idempotency-Key` kết hợp máy trạng thái, ngăn chặn việc người dùng double click hoặc retry mạng làm ghi nhận trùng lặp số dư.
- **`NFR-AUD-001` (Audit Trail Append-only):** Mọi giao dịch làm biến động số dư kho đều sinh 01 bản ghi lịch sử trong bảng `stock_movement_log` với quyền chỉ ghi (`APPEND-ONLY`).

### 6.4. Bảo mật và Khả năng chịu lỗi (NFR-SEC & NFR-REL)
- **`NFR-SEC-001` (Xác thực và Phân quyền):** Xác thực qua JWT; mật khẩu băm bằng thuật toán an toàn (BCrypt); phân quyền chặt chẽ theo 4 vai trò tác nhân tại Phần 2.
- **`NFR-REL-001` (Khả năng chịu lỗi kết nối AI):** Khi dịch vụ Python AI không phản hồi hoặc timeout quá 3 giây, Backend tự động kích hoạt cơ chế Fallback tĩnh (`FR-REC-004`) để đảm bảo các hoạt động xuất/nhập kho vẫn diễn ra bình thường.

### 6.5. Khả năng kiểm thử và Triển khai (NFR-ENV)
- **`NFR-ENV-001` (Tái lập dữ liệu thử nghiệm):** Script sinh dữ liệu giả lập phải nhận tham số `random_seed` cố định để bảo đảm dữ liệu và kết quả thử nghiệm có thể tái lập được 100%.
- **`NFR-ENV-002` (Triển khai cục bộ):** Toàn bộ hệ thống (Backend Java 21 Spring Boot, Database PostgreSQL, AI Service Python 3.10+, Web Frontend) có thể khởi chạy và hoạt động thông suốt trên môi trường phát triển cục bộ (Local environment).

---

## Phần 7. Ma trận Truy vết Yêu cầu (Traceability Matrix)

Bảng dưới đây đối soát toàn bộ các yêu cầu chức năng với nguồn bài toán tại [`docs/01_problem_scope.md`](file:///d:/DO_AN_TOT_NGHIEP/docs/01_problem_scope.md) và các Ca kiểm thử nghiệm thu (Acceptance Tests):

| Mã yêu cầu (Req ID) | Tên tóm tắt yêu cầu | Nguồn mục tại `01_problem_scope.md` | Mã Ca kiểm thử liên quan (`01_problem_scope.md`) | Mức ưu tiên | Trạng thái xác nhận |
| :--- | :--- | :--- | :--- | :---: | :---: |
| **FR-AUTH-001** | Đăng nhập và xác thực JWT | Mục 1.3 (Mục tiêu bảo mật) | Kiểm thử bảo mật | Must | Đã xác nhận |
| **FR-AUTH-002** | Phân quyền truy cập theo vai trò | Mục 2 (Đối tượng sử dụng) | Kiểm thử phân quyền | Must | Đã xác nhận |
| **FR-MST-001** | Quản lý danh mục sản phẩm (SKU) | Mục 6.1 (Danh mục cơ bản) | Ca kiểm thử 8 (E2E Workflow) | Must | Đã xác nhận |
| **FR-MST-002** | Quản lý kho cấp Tổng kho | Mục 6.1 (Kho cấp tổng) | Ca kiểm thử 8 (E2E Workflow) | Must | Đề xuất chờ xác nhận |
| **FR-MST-003** | Quản lý đối tác NCC và Khách hàng | Mục 6.1 (Đối tác cơ bản) | Ca kiểm thử 4 & 8 | Must | Đã xác nhận |
| **FR-MST-004** | Cấu hình tham số tồn kho ban đầu | Mục 4.4 (Tham số tồn kho) | Ca kiểm thử 6 (Replenishment) | Must | Đã xác nhận |
| **FR-INV-001** | Tra cứu số dư thời gian thực 5 trạng thái | Mục 3.1 (5 chỉ số tồn kho) | Ca kiểm thử 8 (E2E Workflow) | Must | Đã xác nhận |
| **FR-INV-002** | Ràng buộc bất biến chống tồn âm | Mục 3.1 & 7 (Tình huống 1) | **Ca kiểm thử 1 (Ngăn tồn âm)** | Must | Đã xác nhận |
| **FR-INV-003** | Tính toán vị thế tồn kho ($\text{IP}$) | Mục 3.1 & 4.4 (Vị thế tồn kho) | Ca kiểm thử 6 (Replenishment) | Must | Đã xác nhận |
| **FR-INB-001** | Tạo đơn mua PO tăng On-order | Mục 3.2.A (Nhập kho từ NCC) | Ca kiểm thử 4 & 8 | Must | Đã xác nhận |
| **FR-INB-002** | Xác nhận phiếu nhập kho nhận đủ | Mục 3.2.A (Nhập đủ GRN) | Ca kiểm thử 8 (E2E Workflow) | Must | Đã xác nhận |
| **FR-INB-003** | Xử lý nhập kho một phần từ NCC | Mục 3.2.A & 7 (Tình huống 4) | **Ca kiểm thử 4 (Nhập một phần)** | Must | Đã xác nhận |
| **FR-OUT-001** | Giữ chỗ tồn kho khi có đơn xuất | Mục 3.2.B (Xuất kho - Giữ chỗ) | Ca kiểm thử 1 & 8 | Must | Đã xác nhận |
| **FR-OUT-002** | Hủy giữ chỗ phục hồi Available | Mục 3.3 (Hủy giao dịch) | Kiểm thử hủy giữ chỗ | Must | Đã xác nhận |
| **FR-OUT-003** | Xác nhận xuất kho vật lý (GDN) | Mục 3.2.B (Xuất kho vật lý) | Ca kiểm thử 8 (E2E Workflow) | Must | Đã xác nhận |
| **FR-OUT-004** | Kiểm soát tương tranh xuất đồng thời | Mục 3.3 & 7 (Tình huống 2) | **Ca kiểm thử 2 (Concurrency)** | Must | Đã xác nhận |
| **FR-TRF-001** | Lập lệnh điều chuyển kho nội bộ | Mục 3.2.C (Điều chuyển kho) | Ca kiểm thử 3 & 8 | Must | Đã xác nhận |
| **FR-TRF-002** | Hủy lệnh điều chuyển trước xuất kho | Mục 3.2.C & 3.3 (Hủy điều chuyển) | Kiểm thử hủy điều chuyển | Must | Đã xác nhận |
| **FR-TRF-003** | Xác nhận xuất kho gửi tăng In-transit | Mục 3.2.C (Xuất kho gửi) | Ca kiểm thử 3 (Under-receiving) | Must | Đã xác nhận |
| **FR-TRF-004** | Xác nhận nhập kho nhận đủ | Mục 3.2.C (Nhập đủ kho nhận) | Ca kiểm thử 8 (E2E Workflow) | Must | Đã xác nhận |
| **FR-TRF-005** | Nhận thiếu và ghi nhận hao hụt | Mục 3.2.C & 7 (Tình huống 3) | **Ca kiểm thử 3 (Under-receiving)** | Must | Đã xác nhận |
| **FR-ORD-001** | Phân bổ Single-Warehouse theo ưu tiên | Mục 5.1 (Quy tắc phân bổ MVP) | Ca kiểm thử 8 (E2E Workflow) | Must | Đề xuất chờ xác nhận |
| **FR-ORD-002** | Xử lý thiếu hàng (Stockout Alert) | Mục 5.1 (Xử lý khi thiếu hàng) | Kiểm thử đơn unfulfillable | Must | Đã xác nhận |
| **FR-ORD-003** | Tự động tách đơn hàng đa kho | Mục 6.2 (Chức năng nâng cao) | Kiểm thử tách đơn (Advanced) | Should | Đã xác nhận (Advanced) |
| **FR-AI-001** | Sinh dữ liệu tổng hợp có seed | Mục 4.2 (Dữ liệu tổng hợp) | Ca kiểm thử 7 (Đánh giá AI) | Must | Đã xác nhận |
| **FR-AI-002** | Chia tập dữ liệu theo thời gian | Mục 4.3 (Temporal Split) | Ca kiểm thử 7 (Đánh giá AI) | Must | Đã xác nhận |
| **FR-AI-003** | Huấn luyện AI và so sánh Baseline | Mục 4.3 (So sánh Baseline) | Ca kiểm thử 7 (Đánh giá AI) | Must | Đã xác nhận |
| **FR-AI-004** | Đo lường sai số WAPE, MAE, RMSE | Mục 4.3 (Chỉ số sai số) | Ca kiểm thử 7 (Đánh giá AI) | Must | Đã xác nhận |
| **FR-AI-005** | Cung cấp API suy luận dự báo | Mục 4.1 & 8.1 (Inference API) | Kiểm thử tích hợp Backend-AI | Must | Đề xuất chờ xác nhận |
| **FR-REC-001** | Tính toán ROP và SS định lượng | Mục 4.4 (Công thức ROP, SS) | Ca kiểm thử 6 (Replenishment) | Must | Đã xác nhận |
| **FR-REC-002** | Quét IP sinh cảnh báo thiếu hàng | Mục 4.4 (Cảnh báo $IP \le ROP$) | Ca kiểm thử 6 (Replenishment) | Must | Đã xác nhận |
| **FR-REC-003** | Tính lượng đề xuất $Q_{\text{suggest}}$ | Mục 4.4 (Công thức $Q$) | Ca kiểm thử 6 (Replenishment) | Must | Đã xác nhận |
| **FR-REC-004** | Cơ chế Fallback khi thiếu dữ liệu AI | Mục 4.5 (Cold-start Policy) | Kiểm thử Fallback tĩnh | Must | Đã xác nhận |
| **FR-REC-005** | Phê duyệt đề xuất sinh PO/Transfer | Mục 4.4 (Human-in-the-loop) | Ca kiểm thử 8 (E2E Workflow) | Must | Đã xác nhận |
| **FR-REC-006** | Mô phỏng chính sách tồn kho | Mục 6.2 (Inventory Simulation) | Kiểm thử Simulation (Advanced) | Should | Đã xác nhận (Advanced) |
| **FR-AUD-001** | Ghi nhật ký biến động kho bất biến | Mục 3.3 & 6.1 (Audit log) | Ca kiểm thử 8 (E2E Workflow) | Must | Đã xác nhận |
| **FR-AUD-002** | Khóa Idempotency Key chống trùng lặp | Mục 3.3 & 7 (Tình huống 5) | **Ca kiểm thử 5 (Idempotency)** | Must | Đã xác nhận |
| **FR-DASH-001**| Bảng điều khiển tồn kho và cảnh báo | Mục 6.1 (Giao diện Web) | Ca kiểm thử 6 & 8 | Must | Đã xác nhận |
| **FR-DASH-002**| Trực quan hóa dự báo và Baseline | Mục 4.3 & 6.1 (Dashboard AI) | Ca kiểm thử 7 (Đánh giá AI) | Must | Đã xác nhận |

---

## Phần 8. Các Vấn đề Chờ Xác nhận (Pending Decisions)

Bảng tổng hợp các vấn đề kỹ thuật và nghiệp vụ chưa thể chốt chính thức trong tài liệu này do cần phản hồi phê duyệt từ người dùng hoặc cần dữ liệu khảo sát sâu hơn:

| STT | Vấn đề nghiệp vụ / kỹ thuật | Phương án đề xuất mặc định của AI | Tác động nếu thay đổi | Trạng thái |
| :---: | :--- | :--- | :--- | :---: |
| 1 | **Phạm vi quản lý vị trí kho Core MVP** | Quản lý ở cấp Tổng kho (`Warehouse Level`); chưa quản lý vị trí Kệ/Ô (`Bin Level`). | Nếu chuyển sang quản lý Kệ/Bin ngay trong Core MVP, số lượng bảng DB và giao diện tác nghiệp sẽ tăng gấp đôi, gây rủi ro tiến độ 6 tháng. | `Pending Decision (Chờ người dùng xác nhận)` |
| 2 | **Chu kỳ và Tầm dự báo AI** | Dự báo theo ngày (Daily), tầm dự báo từ 7 đến 14 ngày tới. | Nếu chuyển sang dự báo theo tuần (Weekly), sẽ giảm số điểm dữ liệu và khó phản ánh Lead time ngắn (2–3 ngày). | `Pending Decision (Chờ người dùng xác nhận)` |
| 3 | **Phương thức giao tiếp Backend – AI** | REST API đồng bộ cho suy luận online; kết hợp Cron Job định kỳ chạy batch dự báo lưu DB. | Nếu dùng Message Queue (RabbitMQ/Kafka), hệ thống sẽ tăng độ phức tạp hạ tầng triển khai local. | `Pending Decision (Chờ người dùng xác nhận)` |
| 4 | **Lựa chọn Framework Web Frontend** | Khuyến nghị React (Vite) hoặc Vue (Vite) xây dựng Single Page Application. | Cần chốt để chuẩn hóa thiết kế API giao tiếp và thư viện vẽ biểu đồ. | `Pending Decision (Chờ người dùng xác nhận)` |
| 5 | **Quy mô kịch bản doanh nghiệp giả lập** | Mô phỏng 2–3 kho hàng (1 CDC, 1–2 RDC) và khoảng 20–50 SKU đại diện. | Nếu mô phỏng hàng nghìn SKU, thời gian huấn luyện AI và tải hệ thống local sẽ kéo dài. | `Pending Decision (Chờ người dùng xác nhận)` |
| 6 | **Công thức chi phí tồn trữ và đặt hàng (EOQ)** | Tạm thời áp dụng chính sách Min-Max ($S - \text{IP}$) và MOQ; chưa đưa công thức EOQ giải tích vào Core MVP do chưa có chi phí thực tế. | Sẽ được định mức tham số giả định trong tài liệu quy tắc nghiệp vụ `docs/03_business_rules.md`. | `Pending Decision (Cần nghiên cứu ở docs/03)` |
| 7 | **Hàm mục tiêu chi phí vận chuyển đa kho** | Áp dụng thứ tự ưu tiên kho cố định cho Core MVP; chưa tối ưu hóa chi phí vận tải mạng lưới (Multi-echelon linear programming). | Chuyển sang giai đoạn Advanced MVP khi có ma trận cước vận chuyển giả định. | `Pending Decision (Chuyển sang Advanced MVP)` |
