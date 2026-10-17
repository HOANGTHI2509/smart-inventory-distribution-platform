# TÀI LIỆU TỪ ĐIỂN DỮ LIỆU VÀ CẤU TRÚC THỰC THỂ (DATA DICTIONARY SPECIFICATION)

**Đề tài:** Xây dựng nền tảng quản lý kho và phân phối hàng hóa thông minh ứng dụng AI dự báo nhu cầu và hỗ trợ tối ưu tồn kho  
**Loại tài liệu:** Đặc tả Từ điển dữ liệu (Data Dictionary), Cấu trúc thực thể và Ràng buộc toàn vẹn cơ sở dữ liệu  
**Trạng thái tài liệu:** Bản dự thảo chính thức (Chuyển tiếp từ [`docs/01_problem_scope.md`](file:///d:/DO_AN_TOT_NGHIEP/docs/01_problem_scope.md), [`docs/02_business_requirements.md`](file:///d:/DO_AN_TOT_NGHIEP/docs/02_business_requirements.md) và [`docs/03_business_rules.md`](file:///d:/DO_AN_TOT_NGHIEP/docs/03_business_rules.md))  
**Hệ quản trị cơ sở dữ liệu mục tiêu:** PostgreSQL 15+ (Hỗ trợ chuẩn ANSI SQL, ACID, Check constraints, Stored Generated Columns và Pessimistic Locking).

---

## Phần A — Quy ước Dữ liệu (Data Conventions & Standards)

### A.1. Quy tắc Đặt tên (Naming Conventions)
1. **Tên bảng (Table Names):**
   - Viết bằng chữ thường, số nhiều, các từ phân tách bằng dấu gạch dưới (`snake_case`).
   - Ví dụ: `warehouses`, `products`, `purchase_orders`, `stock_movement_logs`.
2. **Tên cột (Column Names):**
   - Viết bằng chữ thường, số ít, phân tách bằng dấu gạch dưới (`snake_case`).
   - Khóa chính luôn là `id`.
   - Khóa ngoại có cấu trúc: `{tên_bảng_số_ít}_id` (ví dụ: `warehouse_id`, `sku_id`, `supplier_id`).
   - Cột chỉ cờ trạng thái boolean bắt đầu bằng `is_` hoặc `has_` (ví dụ: `is_active`, `is_zero_filled`).
   - Cột thời gian kết thúc bằng `_at` (ví dụ: `created_at`, `updated_at`, `dispatched_at`).
   - Cột ngày không giờ kết thúc bằng `_date` (ví dụ: `order_date`, `forecast_date`).
3. **Tên khóa và chỉ mục (Constraints & Indexes):**
   - Khóa chính (Primary Key): `pk_{tên_bảng}`.
   - Khóa ngoại (Foreign Key): `fk_{bảng_nguồn}_{bảng_đích}_{tên_cột}`.
   - Ràng buộc duy nhất (Unique Constraint): `uq_{tên_bảng}_{cột}`.
   - Ràng buộc kiểm tra (Check Constraint): `chk_{tên_bảng}_{quy_tắc}`.
   - Chỉ mục thông thường (Index): `idx_{tên_bảng}_{cột}`.

### A.2. Quy ước Khóa chính: So sánh UUID vs BIGINT Identity
- **Đánh giá kỹ thuật:**
  - `BIGINT GENERATED ALWAYS AS IDENTITY` (8 bytes): Hiệu năng B-Tree tối ưu, tiết kiệm dung lượng bộ nhớ đệm (buffer pool), tốc độ ghi tuần tự vượt trội, truy vấn khóa ngoại nhanh. Nhược điểm: có thể lộ số lượng giao dịch nếu dùng trực tiếp trên API công khai.
  - `UUIDv4` (16 bytes): Tính duy nhất toàn cầu, không phụ thuộc cơ sở dữ liệu. Nhược điểm: ngẫu nhiên hoàn toàn làm phân mảnh chỉ mục B-Tree (index page splits), suy giảm hiệu năng ghi hàng loạt.
  - `UUIDv7` (16 bytes): Kết hợp dấu thời gian và ngẫu nhiên, sắp xếp tuần tự tốt hơn UUIDv4, nhưng PostgreSQL cần thư viện/extension bên ngoài để sinh trực tiếp.
- **Quyết định kiến trúc đề xuất (`PROPOSED`):**
  - Sử dụng **`BIGINT GENERATED ALWAYS AS IDENTITY`** làm Khóa chính (`id`) nội bộ cho tất cả các bảng trong cơ sở dữ liệu để đạt hiệu năng index và transaction cao nhất.
  - Bổ sung các cột **Mã định danh nghiệp vụ (Business Code)** có kiểu `VARCHAR(50) UNIQUE NOT NULL` phục vụ giao tiếp API và hiển thị cho người dùng: `sku_code`, `warehouse_code`, `po_number`, `order_number`, `transfer_number`, `grn_number`, `gdn_number`.
  - Riêng bảng idempotency sử dụng chuỗi UUID (`VARCHAR(100)` hoặc `UUID`) do client truyền lên qua HTTP Header.

### A.3. Quy ước Kiểu Dữ liệu Số lượng và Đơn giá
1. **Số lượng tồn kho, xuất, nhập, chuyển (`quantity`):**
   - Sử dụng kiểu **`INTEGER`** (4 bytes, phạm vi $-2.147.483.648$ đến $+2.147.483.647$).
   - *Lý do:* Hệ thống quy chuẩn đơn vị tính là đơn vị nguyên chiếc ("Cái", "Hộp", "Thùng"). Tuyệt đối **cấm sử dụng `FLOAT` hoặc `DOUBLE PRECISION`** để tránh sai số dấu phẩy động nhị phân gây lệch số dư tồn kho.
2. **Đơn giá và Thành tiền (`unit_price`, `total_amount`):**
   - Sử dụng kiểu **`NUMERIC(15, 2)`** (cho phép lưu tối đa 999 nghìn tỷ, với 2 chữ số thập phân).
3. **Tham số sai số và xác suất AI (`mae`, `rmse`, `wape`, `service_level`):**
   - Sử dụng **`NUMERIC(8, 4)`** hoặc **`DOUBLE PRECISION`** cho các chỉ số đánh giá mô hình.

### A.4. Quy ước Thời gian và Múi giờ
- Toàn bộ các cột thời gian giao dịch, kiểm toán sử dụng kiểu: **`TIMESTAMPTZ` (`TIMESTAMP WITH TIME ZONE`)**.
- Dữ liệu lưu trữ ở tầng database luôn ở chuẩn UTC. Tầng ứng dụng backend (Spring Boot) chịu trách nhiệm định dạng múi giờ hiển thị sang múi giờ nghiệp vụ Việt Nam (UTC+7, `Asia/Ho_Chi_Minh`).
- Dữ liệu chuỗi thời gian dự báo AI theo ngày sử dụng kiểu **`DATE`** thuần túy (không kèm giờ) để chuẩn hóa theo ngày nghiệp vụ.
- Mọi bảng đều bắt buộc có 2 trường kiểm toán hệ thống:
  - `created_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP`
  - `updated_at TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP`

### A.5. Cách Biểu diễn Trạng thái và Enum
- Sử dụng kiểu **`VARCHAR(30)` kết hợp `CHECK (status IN (...))`**.
- *Lý do kỹ thuật:* Không sử dụng PostgreSQL native `CREATE TYPE enum` vì gây khó khăn khi migration schema (việc bổ sung/đổi tên giá trị enum trong PostgreSQL đòi hỏi quyền quản trị và cơ chế lock bảng). `VARCHAR(30)` kết hợp CHECK constraint cho phép dễ dàng mở rộng và tương thích tự nhiên với `@Enumerated(EnumType.STRING)` trong Spring Boot JPA.

### A.6. Quy ước Xóa dữ liệu: Soft Delete vs Cấm Xóa Tuyệt Đối
1. **Đối với Dữ liệu Danh mục (Master Data):**
   - Áp dụng **Soft Delete** thông qua cờ `is_active BOOLEAN NOT NULL DEFAULT TRUE`.
   - Khi ngừng hoạt động một SKU, Kho hoặc Nhà cung cấp, hệ thống chuyển `is_active = FALSE`. Tuyệt đối cấm xóa vật lý (`HARD DELETE`) để không làm đứt gãy khóa ngoại tới các giao dịch lịch sử.
2. **Đối với Dữ liệu Giao dịch và Kiểm toán (Transactional & Audit Data):**
   - **TUYỆT ĐỐI CẤM XÓA (NO DELETE):** Cơ sở dữ liệu thiết lập quyền `REVOKE DELETE` đối với các bảng `stock_movement_logs`, `purchase_orders`, `goods_receipt_notes`, `outbound_orders`, `transfer_orders`, `discrepancy_logs`, `inventory_balances`.
   - Mọi thao tác hủy bỏ phải chuyển trạng thái sang `CANCELLED` hoặc `CLOSED` và sinh giao dịch bù trừ (compensating record).

---

## Phần B — Danh mục Thực thể (Entity Catalog)

Hệ thống được tổ chức thành **9 nhóm thực thể logic** với tổng cộng **26 bảng cơ sở dữ liệu**, được chuẩn hóa ở dạng chuẩn 3NF:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                                 DANH MỤC THỰC THỂ HỆ THỐNG                              │
├───────────────────────────────┬────────────────────────────────────────────────────────┤
│ Nhóm 1: Hệ thống & Người dùng  │ 1. users                                               │
│                               │ 2. user_roles                                          │
│                               │ 3. user_activity_logs (Nhật ký bảo mật & đăng nhập)    │
├───────────────────────────────┼────────────────────────────────────────────────────────┤
│ Nhóm 2: Danh mục Cơ sở (Master)│ 4. warehouses                                          │
│                               │ 5. suppliers                                           │
│                               │ 6. products (SKU)                                      │
│                               │ 7. sku_warehouse_configs (Tham số tĩnh & Lead time)    │
├───────────────────────────────┼────────────────────────────────────────────────────────┤
│ Nhóm 3: Số dư & Nhật ký kho   │ 8. inventory_balances                                  │
│                               │ 9. stock_movement_logs (Audit log vật lý & giữ chỗ)    │
├───────────────────────────────┼────────────────────────────────────────────────────────┤
│ Nhóm 4: Nhập kho NCC (Inbound)│ 10. purchase_orders                                    │
│                               │ 11. purchase_order_items                               │
│                               │ 12. goods_receipt_notes (Phiếu nhập kho GRN)           │
│                               │ 13. goods_receipt_items                                │
├───────────────────────────────┼────────────────────────────────────────────────────────┤
│ Nhóm 5: Xuất kho (Outbound)   │ 14. outbound_orders                                    │
│                               │ 15. outbound_order_items                               │
│                               │ 16. stock_reservations (Giữ chỗ tồn kho)               │
│                               │ 17. goods_dispatch_notes (Phiếu xuất kho GDN)          │
│                               │ 18. goods_dispatch_items                               │
├───────────────────────────────┼────────────────────────────────────────────────────────┤
│ Nhóm 6: Điều chuyển (Transfer)│ 19. transfer_orders                                    │
│                               │ 20. transfer_order_items                               │
│                               │ 21. discrepancy_logs (Nhật ký thất thoát/sai lệch)     │
├───────────────────────────────┼────────────────────────────────────────────────────────┤
│ Nhóm 7: Dữ liệu & Dự báo AI   │ 22. daily_demand_history (Chuỗi thời gian liên tục)     │
│                               │ 23. ai_model_runs (Lịch sử huấn luyện & đánh giá)      │
│                               │ 24. forecast_values (Dự báo nhu cầu đa bước)           │
├───────────────────────────────┼────────────────────────────────────────────────────────┤
│ Nhóm 8: Động cơ Bổ sung tồn   │ 25. replenishment_recommendations                     │
├───────────────────────────────┼────────────────────────────────────────────────────────┤
│ Nhóm 9: Kỹ thuật & Giao dịch  │ 26. idempotency_records                                │
└───────────────────────────────┴────────────────────────────────────────────────────────┘
```

### Đánh giá Gộp/Tách Thực thể:
- **Tách `goods_receipt_notes` khỏi `purchase_orders`:** Cho phép một đơn PO có thể được giao và kiểm đếm thành nhiều đợt độc lập (1-n), giải quyết hoàn chỉnh kịch bản nhập kho từng phần (`BR-INB-003`, `BR-INB-004`).
- **Tách `stock_reservations`:** Đảm bảo việc giữ chỗ đơn xuất được theo dõi tường minh, có mốc thời gian giữ và giải phóng, hỗ trợ phục hồi tồn khả dụng ngay khi hủy đơn (`BR-OUT-003`).
- **Tách `discrepancy_logs`:** Quản lý độc lập các biến cố hao hụt/thất thoát trên đường chuyển kho, không trộn lẫn vào dữ liệu vận hành thông thường (`BR-TRF-005`).
- **Gộp tham số tĩnh vào `sku_warehouse_configs`:** Thay vì lưu phân tán trong bảng sản phẩm hay kho, bảng này định nghĩa chính sách tồn kho chuyên biệt cho từng cặp `(sku, warehouse)` (bao gồm $MOQ, S, \text{SS}_{\text{static}}, \text{ROP}_{\text{static}}, L$).

---

## Phần C — Đặc tả Chi tiết Từng Bảng (Detailed Table Specifications)

### Nhóm 1: Quản trị Hệ thống và Phân quyền

#### 1. Bảng `users`
Lưu trữ thông tin tài khoản người dùng đăng nhập hệ thống.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính định danh người dùng | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_users` | Hệ thống | `FR-AUTH-001` |
| `username` | Tên đăng nhập | `VARCHAR(50)` | `UK` | `NOT NULL` | - | `LENGTH(username) >= 4` | `uq_users_username` | Admin tạo | `FR-AUTH-001` |
| `password_hash` | Mật khẩu mã hóa BCrypt | `VARCHAR(255)` | - | `NOT NULL` | - | - | - | User / Admin | `FR-AUTH-001` |
| `full_name` | Họ và tên người dùng | `VARCHAR(100)` | - | `NOT NULL` | - | - | - | Admin tạo | `FR-AUTH-001` |
| `email` | Địa chỉ email liên hệ | `VARCHAR(100)` | `UK` | `NOT NULL` | - | `email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'` | `uq_users_email` | Admin tạo | `FR-AUTH-001` |
| `is_active` | Trạng thái tài khoản | `BOOLEAN` | - | `NOT NULL` | `TRUE` | - | `idx_users_active` | Admin | `FR-AUTH-001` |
| `created_at` | Thời điểm tạo tài khoản | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-AUTH-001` |
| `updated_at` | Thời điểm cập nhật cuối | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-AUTH-001` |

#### 2. Bảng `user_roles`
Bảng liên kết phân quyền người dùng và vai trò theo mô hình RBAC chuẩn.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `user_id` | Định danh người dùng | `BIGINT` | `PK, FK -> users(id)` | `NOT NULL` | - | - | `fk_user_roles_user` | Admin | `FR-AUTH-002` |
| `role_name` | Tên vai trò được cấp | `VARCHAR(30)` | `PK` | `NOT NULL` | - | `role_name IN ('ADMIN', 'WAREHOUSE_MANAGER', 'WAREHOUSE_OPERATOR', 'INVENTORY_PLANNER')` | - | Admin | `FR-AUTH-002` |
| `granted_at` | Thời điểm cấp quyền | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-AUTH-002` |

#### 3. Bảng `user_activity_logs`
Lưu trữ nhật ký hoạt động bảo mật, đăng nhập, đổi mật khẩu và phân quyền của người dùng (đáp ứng yêu cầu `FR-AUTH-003`).

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính bản ghi nhật ký | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_user_activity_logs` | Hệ thống | `FR-AUTH-003` |
| `user_id` | Định danh người dùng thực hiện | `BIGINT` | `FK -> users(id)` | `NOT NULL` | - | - | `idx_ual_user_time` | Auth context | `FR-AUTH-003` |
| `action` | Hành vi thực hiện | `VARCHAR(50)` | - | `NOT NULL` | - | `action IN ('LOGIN_SUCCESS', 'LOGIN_FAILED', 'LOGOUT', 'PASSWORD_CHANGE', 'ROLE_ASSIGNED', 'ROLE_REVOKED')` | `idx_ual_action` | Auth / Admin | `FR-AUTH-003` |
| `ip_address` | Địa chỉ IP của client | `VARCHAR(45)` | - | `NULL` | - | - | - | HTTP Request | `FR-AUTH-003` |
| `status` | Kết quả thực hiện | `VARCHAR(20)` | - | `NOT NULL` | - | `status IN ('SUCCESS', 'FAILURE')` | - | Auth / Admin | `FR-AUTH-003` |
| `details` | Chi tiết bổ sung (JSON hoặc text) | `TEXT` | - | `NULL` | - | - | - | Hệ thống | `FR-AUTH-003` |
| `created_at` | Thời điểm ghi nhận | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | `idx_ual_user_time` | Hệ thống | `FR-AUTH-003` |

---

### Nhóm 2: Danh mục Cơ sở (Master Data)

#### 4. Bảng `warehouses`
Quản lý danh sách các điểm kho trong mạng lưới phân phối (CDC và RDC).

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính kho hàng | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_warehouses` | Hệ thống | `FR-MST-001` |
| `warehouse_code` | Mã viết tắt của kho | `VARCHAR(30)` | `UK` | `NOT NULL` | - | `LENGTH(warehouse_code) >= 2` | `uq_warehouses_code` | Admin | `FR-MST-001` |
| `warehouse_name` | Tên kho đầy đủ | `VARCHAR(100)` | - | `NOT NULL` | - | - | - | Admin | `FR-MST-001` |
| `warehouse_type` | Loại hình kho | `VARCHAR(20)` | - | `NOT NULL` | - | `warehouse_type IN ('CDC', 'RDC')` | `idx_warehouses_type` | Admin | `FR-MST-001` |
| `address` | Địa chỉ thực tế | `VARCHAR(255)` | - | `NOT NULL` | - | - | - | Admin | `FR-MST-001` |
| `priority_order` | Thứ tự ưu tiên phân bổ đơn | `INTEGER` | - | `NOT NULL` | `1` | `priority_order >= 1` | `idx_warehouses_priority` | Admin / Planner | `FR-ORD-003`, `BR-ORD-002` |
| `is_active` | Trạng thái hoạt động | `BOOLEAN` | - | `NOT NULL` | `TRUE` | - | `idx_warehouses_active` | Admin | `FR-MST-001` |
| `created_at` | Thời điểm tạo bản ghi | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-MST-001` |
| `updated_at` | Thời điểm cập nhật | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-MST-001` |

#### 4. Bảng `suppliers`
Lưu trữ thông tin các nhà cung cấp hàng hóa cho hệ thống kho.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính nhà cung cấp | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_suppliers` | Hệ thống | `FR-MST-003` |
| `supplier_code` | Mã nhà cung cấp | `VARCHAR(30)` | `UK` | `NOT NULL` | - | - | `uq_suppliers_code` | Admin / Manager | `FR-MST-003` |
| `supplier_name` | Tên đầy đủ của nhà cung cấp | `VARCHAR(150)` | - | `NOT NULL` | - | - | - | Admin / Manager | `FR-MST-003` |
| `contact_email` | Email liên hệ đặt hàng | `VARCHAR(100)` | - | `NULL` | - | - | - | Admin / Manager | `FR-MST-003` |
| `contact_phone` | Số điện thoại liên hệ | `VARCHAR(20)` | - | `NULL` | - | - | - | Admin / Manager | `FR-MST-003` |
| `default_lead_time_days` | Lead time cung ứng mặc định | `INTEGER` | - | `NOT NULL` | `3` | `default_lead_time_days >= 1` | - | Admin / Manager | `FR-MST-003`, `BR-REC-001` |
| `is_active` | Trạng thái hợp tác | `BOOLEAN` | - | `NOT NULL` | `TRUE` | - | `idx_suppliers_active` | Admin / Manager | `FR-MST-003` |
| `created_at` | Thời điểm tạo | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-MST-003` |
| `updated_at` | Thời điểm cập nhật | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-MST-003` |

#### 5. Bảng `products`
Danh mục các mặt hàng (SKU) được quản lý trong hệ thống phân phối.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính sản phẩm | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_products` | Hệ thống | `FR-MST-002` |
| `sku_code` | Mã SKU định danh duy nhất | `VARCHAR(50)` | `UK` | `NOT NULL` | - | `LENGTH(sku_code) >= 3` | `uq_products_sku` | Admin / Planner | `FR-MST-002` |
| `product_name` | Tên sản phẩm | `VARCHAR(200)` | - | `NOT NULL` | - | - | `idx_products_name` | Admin / Planner | `FR-MST-002` |
| `category` | Ngành hàng / Phân loại | `VARCHAR(50)` | - | `NOT NULL` | - | - | `idx_products_category` | Admin / Planner | `FR-MST-002` |
| `unit_of_measure` | Đơn vị tính cơ sở | `VARCHAR(20)` | - | `NOT NULL` | `'Cái'` | - | - | Admin / Planner | `FR-MST-002` |
| `default_supplier_id`| Nhà cung cấp ưu tiên chính | `BIGINT` | `FK -> suppliers(id)`| `NULL` | - | - | `fk_products_supplier` | Admin / Planner | `FR-MST-002` |
| `is_active` | Trạng thái kinh doanh | `BOOLEAN` | - | `NOT NULL` | `TRUE` | - | `idx_products_active` | Admin / Planner | `FR-MST-002` |
| `created_at` | Thời điểm tạo | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-MST-002` |
| `updated_at` | Thời điểm cập nhật | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-MST-002` |

#### 6. Bảng `sku_warehouse_configs`
Cấu hình tham số tồn kho tĩnh và thời gian cung ứng riêng biệt cho từng cặp `(sku, warehouse)`.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính cấu hình | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_sku_wh_config` | Hệ thống | `FR-MST-004` |
| `sku_id` | Định danh sản phẩm | `BIGINT` | `FK -> products(id)` | `NOT NULL` | - | - | `fk_config_sku` | Planner | `FR-MST-004` |
| `warehouse_id` | Định danh kho hàng | `BIGINT` | `FK -> warehouses(id)` | `NOT NULL` | - | - | `fk_config_wh` | Planner | `FR-MST-004` |
| `lead_time_days` | Lead time cung ứng tại kho này ($L$) | `INTEGER` | - | `NOT NULL` | `3` | `lead_time_days >= 1` | - | Planner | `BR-REC-001` |
| `min_order_qty` | Lượng đặt hàng tối thiểu ($\text{MOQ}$) | `INTEGER` | - | `NOT NULL` | `1` | `min_order_qty >= 1` | - | Planner | `BR-REC-004` |
| `max_stock_level` | Mức tồn mục tiêu tối đa ($S$) | `INTEGER` | - | `NOT NULL` | `100` | `max_stock_level >= min_order_qty` | - | Planner | `BR-REC-004` |
| `static_safety_stock` | Tồn an toàn tĩnh dự phòng ($\text{SS}_{\text{static}}$) | `INTEGER` | - | `NOT NULL` | `0` | `static_safety_stock >= 0` | - | Planner | `BR-REC-005` |
| `static_reorder_point`| Điểm đặt hàng tĩnh ($\text{ROP}_{\text{static}}$) | `INTEGER` | - | `NOT NULL` | `0` | `static_reorder_point >= static_safety_stock` | - | Planner | `BR-REC-005` |
| `is_active` | Trạng thái áp dụng | `BOOLEAN` | - | `NOT NULL` | `TRUE` | - | - | Planner | `FR-MST-004` |
| `created_at` | Thời điểm tạo | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-MST-004` |
| `updated_at` | Thời điểm cập nhật | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-MST-004` |

*Ràng buộc kết hợp:* `uq_sku_warehouse_config UNIQUE (sku_id, warehouse_id)`.

---

### Nhóm 3: Số dư Tồn kho và Nhật ký Kiểm toán

#### 7. Bảng `inventory_balances`
Lưu trữ trạng thái số dư hiện hành tức thời của từng SKU tại từng kho hàng.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính số dư | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_inventory_balances` | Hệ thống | `FR-INV-001` |
| `sku_id` | Định danh sản phẩm | `BIGINT` | `FK -> products(id)` | `NOT NULL` | - | - | `fk_inv_sku` | Hệ thống | `FR-INV-001` |
| `warehouse_id` | Định danh kho hàng | `BIGINT` | `FK -> warehouses(id)` | `NOT NULL` | - | - | `fk_inv_wh` | Hệ thống | `FR-INV-001` |
| `on_hand` | Tồn vật lý trong kho | `INTEGER` | - | `NOT NULL` | `0` | `on_hand >= 0` | - | Giao dịch kho | `BR-INV-001` |
| `allocated` | Tồn đang giữ chỗ | `INTEGER` | - | `NOT NULL` | `0` | `allocated >= 0` | - | Giữ chỗ xuất/chuyển | `BR-INV-001` |
| `available` | Tồn khả dụng tính toán | `INTEGER` | `GENERATED` | `NOT NULL` | - | `STORED: (on_hand - allocated)` | `idx_inv_available` | Tự động sinh | `BR-INV-001` |
| `on_order` | Hàng đang về kho | `INTEGER` | - | `NOT NULL` | `0` | `on_order >= 0` | - | Duyệt PO/Lệnh chuyển | `BR-INV-002` |
| `version` | Khóa phiên bản phục vụ kiểm soát | `BIGINT` | - | `NOT NULL` | `0` | - | - | Hibernate/JPA | `NFR-CON-001` |
| `updated_at` | Thời điểm cập nhật cuối | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-INV-001` |

*Ràng buộc toàn vẹn bắt buộc cấp Database:*
1. `uq_inventory_balance UNIQUE (sku_id, warehouse_id)`.
2. `chk_inventory_invariant CHECK (on_hand >= allocated AND on_hand >= 0 AND allocated >= 0 AND on_order >= 0)`.

#### 9. Bảng `stock_movement_logs`
Sổ cái kiểm toán biến động kho bất biến (Append-only). Nguồn sự thật duy nhất cho mọi thay đổi số dư tồn kho vật lý và giữ chỗ.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính bản ghi log | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_stock_movement_logs` | Hệ thống | `FR-AUD-001` |
| `sku_id` | Sản phẩm biến động | `BIGINT` | `FK -> products(id)` | `NOT NULL` | - | - | `idx_sml_sku_wh` | Transaction kho | `FR-AUD-001` |
| `warehouse_id` | Kho phát sinh biến động | `BIGINT` | `FK -> warehouses(id)` | `NOT NULL` | - | - | `idx_sml_sku_wh` | Transaction kho | `FR-AUD-001` |
| `movement_type` | Loại nghiệp vụ biến động | `VARCHAR(30)` | - | `NOT NULL` | - | `movement_type IN ('INBOUND_RECEIPT', 'OUTBOUND_RESERVE', 'RESERVE_CANCEL', 'OUTBOUND_DISPATCH', 'TRANSFER_RESERVE', 'TRANSFER_CANCEL', 'TRANSFER_DISPATCH', 'TRANSFER_RECEIPT', 'DISCREPANCY_LOSS')` | `idx_sml_type` | Transaction kho | `FR-AUD-001`, `BR-INV-004` |
| `delta_on_hand` | Lượng thay đổi tồn vật lý | `INTEGER` | - | `NOT NULL` | `0` | - | - | Transaction kho | `BR-INV-004` |
| `delta_allocated` | Lượng thay đổi tồn giữ chỗ | `INTEGER` | - | `NOT NULL` | `0` | - | - | Transaction kho | `BR-INV-004` |
| `balance_on_hand_after` | Tồn vật lý sau biến động | `INTEGER` | - | `NOT NULL` | - | `balance_on_hand_after >= 0` | - | Snapshot sau TX | `FR-AUD-001` |
| `balance_allocated_after`| Tồn giữ chỗ sau biến động | `INTEGER` | - | `NOT NULL` | - | `balance_allocated_after >= 0` | - | Snapshot sau TX | `FR-AUD-001` |
| `reference_type` | Loại chứng từ liên quan | `VARCHAR(30)` | - | `NOT NULL` | - | `reference_type IN ('PURCHASE_ORDER', 'GOODS_RECEIPT_NOTE', 'OUTBOUND_ORDER', 'TRANSFER_ORDER', 'DISCREPANCY_REPORT')` | `idx_sml_ref` | Transaction kho | `FR-AUD-001` |
| `reference_id` | Khóa chính của chứng từ | `BIGINT` | - | `NOT NULL` | - | - | `idx_sml_ref` | Transaction kho | `FR-AUD-001` |
| `reference_number` | Mã nghiệp vụ chứng từ (PO/GRN/TRF) | `VARCHAR(50)` | - | `NOT NULL` | - | - | `idx_sml_ref_num` | Transaction kho | `FR-AUD-001` |
| `idempotency_key` | Khóa chống lặp nếu có | `VARCHAR(100)` | - | `NULL` | - | - | `idx_sml_idem` | Transaction kho | `FR-AUD-002`, `BR-CON-003` |
| `created_by` | Định danh người thực hiện | `BIGINT` | `FK -> users(id)` | `NOT NULL` | - | - | - | Auth context | `FR-AUD-001` |
| `created_at` | Thời điểm ghi nhận bất biến | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | `idx_sml_created_at` | Hệ thống | `FR-AUD-001` |

*Phạm vi kiểm toán:* Sổ cái `stock_movement_logs` là sổ cái kiểm toán bất biến chỉ ghi nhận biến động vật lý (`on_hand`) và cam kết giữ chỗ nội bộ (`allocated`) tại từng kho. Số dư `on_order` là cam kết ngoài kho, được đối soát độc lập thông qua việc tổng hợp trực tiếp từ các đơn mua hàng (`purchase_orders`) và lệnh điều chuyển (`transfer_orders`) còn hiệu lực theo quy tắc `BR-INV-004`.

---

### Nhóm 4: Nhập kho từ Nhà Cung Cấp (Inbound)

#### 9. Bảng `purchase_orders`
Đơn đặt mua hàng từ nhà cung cấp bên ngoài.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính đơn PO | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_purchase_orders` | Hệ thống | `FR-INB-001` |
| `po_number` | Mã số đơn đặt hàng | `VARCHAR(50)` | `UK` | `NOT NULL` | - | - | `uq_po_number` | Hệ thống sinh | `FR-INB-001` |
| `supplier_id` | Nhà cung cấp được đặt | `BIGINT` | `FK -> suppliers(id)` | `NOT NULL` | - | - | `fk_po_supplier` | Manager tạo | `FR-INB-001` |
| `destination_warehouse_id` | Kho hàng đích nhập về | `BIGINT` | `FK -> warehouses(id)` | `NOT NULL` | - | - | `fk_po_dest_wh` | Manager tạo | `FR-INB-001` |
| `status` | Trạng thái vòng đời PO | `VARCHAR(30)` | - | `NOT NULL` | `'DRAFT'` | `status IN ('DRAFT', 'CONFIRMED', 'PARTIALLY_RECEIVED', 'COMPLETED', 'CLOSED', 'CANCELLED')` | `idx_po_status` | Manager/Operator | `BR-INB-001`..`005` |
| `order_date` | Ngày lập đơn mua | `DATE` | - | `NOT NULL` | `CURRENT_DATE` | - | - | Manager tạo | `FR-INB-001` |
| `expected_delivery_date` | Ngày dự kiến giao hàng | `DATE` | - | `NULL` | - | `expected_delivery_date >= order_date` | - | Manager tạo | `FR-INB-001` |
| `created_by` | Người lập đơn | `BIGINT` | `FK -> users(id)` | `NOT NULL` | - | - | - | Auth context | `FR-INB-001` |
| `approved_by` | Người duyệt đơn | `BIGINT` | `FK -> users(id)` | `NULL` | - | - | - | Manager duyệt | `BR-INB-001` |
| `close_reason` | Lý do đóng đơn sớm | `VARCHAR(255)` | - | `NULL` | - | - | - | Manager đóng | `BR-INB-005` |
| `created_at` | Thời điểm tạo | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-INB-001` |
| `updated_at` | Thời điểm cập nhật | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-INB-001` |

#### 10. Bảng `purchase_order_items`
Chi tiết danh mục các SKU đặt mua trong đơn PO.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính dòng PO | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_po_items` | Hệ thống | `FR-INB-001` |
| `po_id` | Đơn PO cha | `BIGINT` | `FK -> purchase_orders(id)` | `NOT NULL` | - | - | `idx_poi_po_id` | Hệ thống | `FR-INB-001` |
| `sku_id` | Sản phẩm đặt mua | `BIGINT` | `FK -> products(id)` | `NOT NULL` | - | - | `fk_poi_sku` | Manager tạo | `FR-INB-001` |
| `ordered_qty` | Số lượng đặt mua ($Q_{\text{PO}}$) | `INTEGER` | - | `NOT NULL` | - | `ordered_qty > 0` | - | Manager tạo | `BR-INB-001` |
| `received_qty` | Số lượng lũy kế đã nhập | `INTEGER` | - | `NOT NULL` | `0` | `received_qty >= 0 AND received_qty <= ordered_qty` | - | GRN cập nhật | `BR-INB-003`, `004` |
| `unit_price` | Giá mua thỏa thuận | `NUMERIC(15,2)` | - | `NOT NULL` | `0` | `unit_price >= 0` | - | Manager tạo | `FR-INB-001` |
| `created_at` | Thời điểm tạo | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-INB-001` |
| `updated_at` | Thời điểm cập nhật | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-INB-001` |

*Ràng buộc duy nhất:* `uq_po_item_sku UNIQUE (po_id, sku_id)`.

#### 11. Bảng `goods_receipt_notes` (Phiếu Nhập kho GRN)
Phiếu chứng nhận kiểm đếm nhập kho vật lý từ nhà cung cấp (hỗ trợ nhập nhiều đợt cho 1 PO).

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính phiếu GRN | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_grn` | Hệ thống | `FR-INB-002` |
| `grn_number` | Mã số phiếu nhập kho | `VARCHAR(50)` | `UK` | `NOT NULL` | - | - | `uq_grn_number` | Hệ thống sinh | `FR-INB-002` |
| `po_id` | Đơn PO được nhận | `BIGINT` | `FK -> purchase_orders(id)` | `NOT NULL` | - | - | `idx_grn_po_id` | Operator tạo | `FR-INB-002` |
| `warehouse_id` | Kho thực hiện tiếp nhận | `BIGINT` | `FK -> warehouses(id)` | `NOT NULL` | - | - | `fk_grn_wh` | Operator tạo | `FR-INB-002` |
| `received_by` | Thủ kho kiểm đếm | `BIGINT` | `FK -> users(id)` | `NOT NULL` | - | - | - | Auth context | `FR-INB-002` |
| `receipt_date` | Ngày giờ thực nhận hàng | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Operator xác nhận | `FR-INB-002` |
| `status` | Trạng thái phiếu GRN | `VARCHAR(30)` | - | `NOT NULL` | `'COMPLETED'` | `status IN ('COMPLETED', 'CANCELLED')` | - | Operator | `BR-INB-002`, `003` |
| `notes` | Ghi chú tình trạng hàng nhận | `VARCHAR(255)` | - | `NULL` | - | - | - | Operator | `FR-INB-002` |
| `created_at` | Thời điểm tạo | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-INB-002` |

#### 12. Bảng `goods_receipt_items`
Chi tiết số lượng thực nhận của từng SKU trong từng phiếu GRN.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính dòng GRN | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_grn_items` | Hệ thống | `FR-INB-002` |
| `grn_id` | Phiếu GRN cha | `BIGINT` | `FK -> goods_receipt_notes(id)` | `NOT NULL` | - | - | `idx_gri_grn_id` | Hệ thống | `FR-INB-002` |
| `po_item_id` | Dòng mặt hàng PO tương ứng | `BIGINT` | `FK -> purchase_order_items(id)` | `NOT NULL` | - | - | `fk_gri_poi` | Operator | `BR-INB-003` |
| `sku_id` | Sản phẩm thực nhận | `BIGINT` | `FK -> products(id)` | `NOT NULL` | - | - | `fk_gri_sku` | Operator | `FR-INB-002` |
| `received_qty` | Số lượng kiểm đếm thực nhận đợt này | `INTEGER` | - | `NOT NULL` | - | `received_qty > 0` | - | Operator kiểm đếm | `BR-INB-002`, `003` |
| `created_at` | Thời điểm tạo | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-INB-002` |

---

### Nhóm 5: Bán hàng, Giữ chỗ và Xuất kho (Outbound)

#### 13. Bảng `outbound_orders`
Đơn xuất kho giao hàng cho khách hàng hoặc kênh phân phối.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính đơn xuất | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_outbound_orders` | Hệ thống | `FR-OUT-001` |
| `order_number` | Mã đơn hàng xuất | `VARCHAR(50)` | `UK` | `NOT NULL` | - | - | `uq_order_number` | Hệ thống sinh | `FR-OUT-001` |
| `customer_name` | Tên khách hàng / Đại lý | `VARCHAR(150)` | - | `NOT NULL` | - | - | - | Operator / API | `FR-OUT-001` |
| `delivery_address` | Địa chỉ giao hàng | `VARCHAR(255)` | - | `NOT NULL` | - | - | - | Operator / API | `FR-OUT-001` |
| `assigned_warehouse_id` | Kho được phân bổ xuất | `BIGINT` | `FK -> warehouses(id)` | `NULL` | - | - | `idx_ord_assigned_wh` | Allocation Engine | `FR-ORD-001`, `BR-ORD-002` |
| `status` | Trạng thái vòng đời đơn | `VARCHAR(30)` | - | `NOT NULL` | `'DRAFT'` | `status IN ('DRAFT', 'ALLOCATED', 'UNFULFILLABLE', 'DISPATCHED', 'CANCELLED')` | `idx_ord_status` | Engine / Operator | `BR-OUT-001`..`005` |
| `order_date` | Ngày tiếp nhận đơn hàng | `DATE` | - | `NOT NULL` | `CURRENT_DATE` | - | - | Hệ thống | `FR-OUT-001` |
| `created_by` | Người tạo đơn | `BIGINT` | `FK -> users(id)` | `NOT NULL` | - | - | - | Auth context | `FR-OUT-001` |
| `created_at` | Thời điểm tạo | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-OUT-001` |
| `updated_at` | Thời điểm cập nhật | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-OUT-001` |

#### 14. Bảng `outbound_order_items`
Chi tiết danh mục SKU và số lượng yêu cầu trong đơn xuất kho.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính dòng đơn xuất | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_outbound_items` | Hệ thống | `FR-OUT-001` |
| `order_id` | Đơn xuất kho cha | `BIGINT` | `FK -> outbound_orders(id)` | `NOT NULL` | - | - | `idx_ooi_order_id` | Hệ thống | `FR-OUT-001` |
| `sku_id` | Sản phẩm cần xuất | `BIGINT` | `FK -> products(id)` | `NOT NULL` | - | - | `fk_ooi_sku` | Người tạo đơn | `FR-OUT-001` |
| `requested_qty` | Số lượng yêu cầu xuất | `INTEGER` | - | `NOT NULL` | - | `requested_qty > 0` | - | Người tạo đơn | `FR-OUT-001` |
| `unit_price` | Đơn giá xuất | `NUMERIC(15,2)` | - | `NOT NULL` | `0` | `unit_price >= 0` | - | Người tạo đơn | `FR-OUT-001` |
| `created_at` | Thời điểm tạo | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-OUT-001` |

*Ràng buộc duy nhất:* `uq_outbound_item_sku UNIQUE (order_id, sku_id)`.

#### 15. Bảng `stock_reservations`
Quản lý các cam kết giữ chỗ tồn kho cho đơn xuất kho, ngăn chặn xuất vượt tồn khả dụng.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính bản ghi giữ chỗ | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_stock_reservations` | Hệ thống | `FR-OUT-001` |
| `order_id` | Đơn hàng xuất được giữ | `BIGINT` | `FK -> outbound_orders(id)` | `NOT NULL` | - | - | `idx_res_order_id` | Allocation Engine | `BR-OUT-002` |
| `sku_id` | Sản phẩm giữ chỗ | `BIGINT` | `FK -> products(id)` | `NOT NULL` | - | - | `fk_res_sku` | Allocation Engine | `BR-OUT-002` |
| `warehouse_id` | Kho hàng giữ chỗ | `BIGINT` | `FK -> warehouses(id)` | `NOT NULL` | - | - | `idx_res_wh_sku` | Allocation Engine | `BR-OUT-002` |
| `reserved_qty` | Số lượng giữ chỗ | `INTEGER` | - | `NOT NULL` | - | `reserved_qty > 0` | - | Allocation Engine | `BR-OUT-002` |
| `status` | Trạng thái giữ chỗ | `VARCHAR(30)` | - | `NOT NULL` | `'ACTIVE'` | `status IN ('ACTIVE', 'RELEASED', 'FULFILLED')` | `idx_res_status` | Engine / Dispatch | `BR-OUT-003`, `004` |
| `reserved_at` | Thời điểm bắt đầu giữ | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `BR-OUT-002` |
| `released_at` | Thời điểm giải phóng/xuất | `TIMESTAMPTZ` | - | `NULL` | - | - | - | Cancel / Dispatch | `BR-OUT-003`, `004` |

#### 16. Bảng `goods_dispatch_notes` (Phiếu Xuất kho GDN)
Ghi nhận xác nhận xuất kho vật lý của thủ kho, trừ tồn kho On-hand thực tế.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính phiếu GDN | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_gdn` | Hệ thống | `FR-OUT-003` |
| `gdn_number` | Mã số phiếu xuất kho | `VARCHAR(50)` | `UK` | `NOT NULL` | - | - | `uq_gdn_number` | Hệ thống sinh | `FR-OUT-003` |
| `order_id` | Đơn xuất kho tương ứng | `BIGINT` | `UK, FK -> outbound_orders(id)`| `NOT NULL` | - | - | `uq_gdn_order` | Operator tạo | `FR-OUT-003` |
| `warehouse_id` | Kho thực hiện xuất hàng | `BIGINT` | `FK -> warehouses(id)` | `NOT NULL` | - | - | `fk_gdn_wh` | Operator tạo | `FR-OUT-003` |
| `dispatched_by` | Thủ kho bấm xuất | `BIGINT` | `FK -> users(id)` | `NOT NULL` | - | - | - | Auth context | `FR-OUT-003` |
| `dispatch_date` | Thời điểm hàng rời kho | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Operator xác nhận | `BR-OUT-004` |
| `status` | Trạng thái phiếu xuất | `VARCHAR(30)` | - | `NOT NULL` | `'COMPLETED'` | `status IN ('COMPLETED')` | - | Hệ thống | `BR-OUT-004`, `005` |
| `created_at` | Thời điểm tạo | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-OUT-003` |

#### 17. Bảng `goods_dispatch_items`
Chi tiết số lượng thực xuất từng SKU theo phiếu GDN.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính dòng GDN | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_gdn_items` | Hệ thống | `FR-OUT-003` |
| `gdn_id` | Phiếu GDN cha | `BIGINT` | `FK -> goods_dispatch_notes(id)`| `NOT NULL` | - | - | `idx_gdi_gdn_id` | Hệ thống | `FR-OUT-003` |
| `sku_id` | Sản phẩm xuất kho | `BIGINT` | `FK -> products(id)` | `NOT NULL` | - | - | `fk_gdi_sku` | Operator | `FR-OUT-003` |
| `dispatched_qty` | Số lượng xuất vật lý | `INTEGER` | - | `NOT NULL` | - | `dispatched_qty > 0` | - | Operator xác nhận | `BR-OUT-004` |
| `created_at` | Thời điểm tạo | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-OUT-003` |

---

### Nhóm 6: Điều chuyển Kho Nội bộ và Sai lệch (Transfer)

#### 18. Bảng `transfer_orders`
Lệnh điều chuyển hàng hóa giữa hai kho nội bộ (Kho gửi và Kho nhận).

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính lệnh chuyển | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_transfer_orders` | Hệ thống | `FR-TRF-001` |
| `transfer_number` | Mã lệnh điều chuyển | `VARCHAR(50)` | `UK` | `NOT NULL` | - | - | `uq_transfer_number` | Hệ thống sinh | `FR-TRF-001` |
| `origin_warehouse_id` | Kho xuất chuyển (Kho gửi) | `BIGINT` | `FK -> warehouses(id)` | `NOT NULL` | - | - | `idx_trf_origin` | Planner tạo | `FR-TRF-001` |
| `destination_warehouse_id`| Kho tiếp nhận (Kho nhận) | `BIGINT` | `FK -> warehouses(id)` | `NOT NULL` | - | `destination_warehouse_id <> origin_warehouse_id` | `idx_trf_dest` | Planner tạo | `FR-TRF-001` |
| `status` | Trạng thái vòng đời lệnh | `VARCHAR(30)` | - | `NOT NULL` | `'DRAFT'` | `status IN ('DRAFT', 'REQUESTED', 'IN_TRANSIT', 'COMPLETED', 'COMPLETED_WITH_DISCREPANCY', 'CANCELLED')` | `idx_trf_status` | Operator/Engine | `BR-TRF-001`..`005` |
| `created_by` | Người lập lệnh | `BIGINT` | `FK -> users(id)` | `NOT NULL` | - | - | - | Auth context | `FR-TRF-001` |
| `dispatched_by` | Người bấm xuất kho gửi | `BIGINT` | `FK -> users(id)` | `NULL` | - | - | - | Thủ kho gửi | `BR-TRF-003` |
| `dispatched_at` | Thời điểm xuất khỏi kho gửi | `TIMESTAMPTZ` | - | `NULL` | - | - | - | Thủ kho gửi | `BR-TRF-003` |
| `received_by` | Người bấm nhận kho nhận | `BIGINT` | `FK -> users(id)` | `NULL` | - | - | - | Thủ kho nhận | `BR-TRF-004`, `005` |
| `received_at` | Thời điểm nhận tại kho nhận | `TIMESTAMPTZ` | - | `NULL` | - | - | - | Thủ kho nhận | `BR-TRF-004`, `005` |
| `created_at` | Thời điểm tạo | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-TRF-001` |
| `updated_at` | Thời điểm cập nhật | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-TRF-001` |

#### 19. Bảng `transfer_order_items`
Chi tiết danh mục SKU, số lượng yêu cầu chuyển và số lượng thực nhận.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính dòng chuyển | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_transfer_items` | Hệ thống | `FR-TRF-001` |
| `transfer_id` | Lệnh chuyển kho cha | `BIGINT` | `FK -> transfer_orders(id)` | `NOT NULL` | - | - | `idx_toi_trf_id` | Hệ thống | `FR-TRF-001` |
| `sku_id` | Sản phẩm điều chuyển | `BIGINT` | `FK -> products(id)` | `NOT NULL` | - | - | `fk_toi_sku` | Planner | `FR-TRF-001` |
| `transfer_qty` | Số lượng xuất đi ($Q_{\text{transfer}}$) | `INTEGER` | - | `NOT NULL` | - | `transfer_qty > 0` | - | Planner | `BR-TRF-001` |
| `received_qty` | Số lượng thực nhận ($Q_{\text{received}}$)| `INTEGER` | - | `NOT NULL` | `0` | `received_qty >= 0 AND received_qty <= transfer_qty` | - | Thủ kho nhận | `BR-TRF-004`, `005` |
| `created_at` | Thời điểm tạo | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-TRF-001` |
| `updated_at` | Thời điểm cập nhật | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-TRF-001` |

*Ràng buộc duy nhất:* `uq_transfer_item_sku UNIQUE (transfer_id, sku_id)`.

#### 21. Bảng `discrepancy_logs`
Ghi nhận chi tiết biên bản hao hụt, mất mát, sai lệch hàng hóa trong quá trình điều chuyển kho.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính biên bản | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_discrepancy` | Hệ thống | `FR-TRF-005` |
| `transfer_id` | Lệnh chuyển kho phát sinh | `BIGINT` | `FK -> transfer_orders(id)` | `NOT NULL` | - | - | `idx_disc_trf_id` | Hệ thống | `BR-TRF-005` |
| `transfer_item_id` | Dòng chi tiết lệnh chuyển | `BIGINT` | `FK -> transfer_order_items(id)` | `NOT NULL` | - | - | `idx_disc_item_id` | Hệ thống | `FR-TRF-005` |
| `sku_id` | Sản phẩm bị hao hụt | `BIGINT` | `FK -> products(id)` | `NOT NULL` | - | - | `fk_disc_sku` | Thủ kho nhận | `FR-TRF-005` |
| `discrepancy_qty` | Lượng thiếu/thất thoát ($\Delta Q_{\text{loss}}$) | `INTEGER` | - | `NOT NULL` | - | `discrepancy_qty > 0` | - | Hệ thống tính | `BR-TRF-005` |
| `is_total_loss` | Đánh dấu mất mát toàn bộ ($Q_{\text{received}} = 0$) | `BOOLEAN` | - | `NOT NULL` | `FALSE` | - | - | Thủ kho nhận | `BR-TRF-005` |
| `reason` | Nguyên nhân hao hụt | `VARCHAR(255)` | - | `NOT NULL` | - | - | - | Thủ kho nhận nhập | `FR-TRF-005` |
| `reported_by` | Thủ kho lập biên bản | `BIGINT` | `FK -> users(id)` | `NOT NULL` | - | - | - | Auth context | `FR-TRF-005` |
| `reported_at` | Thời điểm ghi nhận | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-TRF-005` |

---

### Nhóm 7: Dữ liệu và Đánh giá Mô hình AI (AI & Forecasting)

#### 21. Bảng `daily_demand_history`
Chuỗi thời gian nhu cầu thực tế theo ngày/SKU/kho, đã qua tiền xử lý điền giá trị 0 (zero-imputation).

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính bản ghi | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_demand_history` | Hệ thống | `FR-AI-001` |
| `demand_date` | Ngày tiêu thụ | `DATE` | - | `NOT NULL` | - | - | `idx_ddh_date` | Daily Batch Job | `BR-AI-001` |
| `sku_id` | Sản phẩm tiêu thụ | `BIGINT` | `FK -> products(id)` | `NOT NULL` | - | - | `idx_ddh_sku_wh` | Daily Batch Job | `BR-AI-001` |
| `warehouse_id` | Kho phát sinh tiêu thụ | `BIGINT` | `FK -> warehouses(id)` | `NOT NULL` | - | - | `idx_ddh_sku_wh` | Daily Batch Job | `BR-AI-001` |
| `actual_demand` | Lượng tiêu thụ thực tế ($y_t$) | `INTEGER` | - | `NOT NULL` | `0` | `actual_demand >= 0` | - | Tổng hợp xuất kho | `BR-AI-001` |
| `is_zero_filled` | Cờ đánh dấu ngày tự điền số 0 | `BOOLEAN` | - | `NOT NULL` | `FALSE` | - | - | Pipeline tiền xử lý | `BR-AI-001` |
| `created_at` | Thời điểm ghi nhận | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-AI-001` |

*Ràng buộc duy nhất:* `uq_demand_date_sku_wh UNIQUE (demand_date, sku_id, warehouse_id)`.

#### 22. Bảng `ai_model_runs`
Theo dõi các phiên huấn luyện, siêu tham số, phân chia tập dữ liệu và kết quả đánh giá mô hình AI.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính phiên chạy | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_ai_model_runs` | Hệ thống | `FR-AI-003` |
| `run_code` | Mã phiên chạy duy nhất | `VARCHAR(50)` | `UK` | `NOT NULL` | - | - | `uq_amr_code` | AI Service | `FR-AI-003` |
| `model_type` | Loại thuật toán | `VARCHAR(30)` | - | `NOT NULL` | - | `model_type IN ('LIGHTGBM', 'RANDOM_FOREST', 'ROLLING_AVG_7D', 'NAIVE_WEEKLY')` | `idx_amr_type` | AI Service | `BR-AI-004` |
| `model_version` | Phiên bản mã nguồn mô hình | `VARCHAR(30)` | - | `NOT NULL` | - | - | - | AI Service | `FR-AI-003` |
| `train_start_date` | Ngày bắt đầu tập Train | `DATE` | - | `NOT NULL` | - | - | - | AI Service | `BR-AI-002` |
| `train_end_date` | Ngày kết thúc tập Train | `DATE` | - | `NOT NULL` | - | `train_end_date >= train_start_date` | - | AI Service | `BR-AI-002` |
| `val_start_date` | Ngày bắt đầu tập Validation | `DATE` | - | `NULL` | - | `val_start_date > train_end_date` | - | AI Service | `BR-AI-002` |
| `val_end_date` | Ngày kết thúc tập Validation | `DATE` | - | `NULL` | - | `val_end_date >= val_start_date` | - | AI Service | `BR-AI-002` |
| `test_start_date` | Ngày bắt đầu tập Test | `DATE` | - | `NOT NULL` | - | - | - | AI Service | `BR-AI-002` |
| `test_end_date` | Ngày kết thúc tập Test | `DATE` | - | `NOT NULL` | - | `test_end_date >= test_start_date` | - | AI Service | `BR-AI-002` |
| `random_seed` | Seed tái lập thực nghiệm | `INTEGER` | - | `NOT NULL` | `42` | - | - | AI Service | `GEMINI.md` |
| `hyperparameters` | Siêu tham số mô hình | `TEXT` | - | `NULL` | - | - | - | AI Service (JSON) | `FR-AI-003` |
| `mae` | Sai số MAE trên tập Test | `NUMERIC(8,4)` | - | `NOT NULL` | - | `mae >= 0` | - | Đánh giá mô hình | `BR-AI-003` |
| `rmse` | Sai số RMSE trên tập Test ($\sigma_e$) | `NUMERIC(8,4)` | - | `NOT NULL` | - | `rmse >= 0` | - | Đánh giá mô hình | `BR-AI-003`, `BR-REC-002` |
| `wape` | Tỷ lệ sai số WAPE trên tập Test | `NUMERIC(8,4)` | - | `NULL` | - | `wape >= 0 OR wape IS NULL` | - | Đánh giá mô hình | `BR-AI-003` |
| `baseline_wape` | WAPE của mô hình Baseline đối sánh | `NUMERIC(8,4)` | - | `NULL` | - | `baseline_wape >= 0 OR baseline_wape IS NULL` | - | Đánh giá đối chứng | `BR-AI-004` |
| `is_accepted` | Đạt chuẩn để đưa vào suy luận | `BOOLEAN` | - | `NOT NULL` | `FALSE` | - | `idx_amr_accepted` | Gatekeeper logic | `BR-AI-004` |
| `created_at` | Thời điểm huấn luyện | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | AI Service | `FR-AI-003` |

*(Ghi chú: Trường `wape` và `baseline_wape` nhận giá trị `NULL` khi tổng nhu cầu thực tế trên tập kiểm thử bằng 0 và sai số tuyệt đối lớn hơn 0; khi đó hệ thống Gatekeeper tự động fallback so sánh bằng `mae`).*

#### 23. Bảng `forecast_values`
Lưu trữ các giá trị dự báo nhu cầu tương lai đa bước do mô hình AI sinh ra.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính giá trị dự báo | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_forecast_values` | Hệ thống | `FR-AI-004` |
| `model_run_id` | Phiên chạy AI sinh dự báo | `BIGINT` | `FK -> ai_model_runs(id)` | `NOT NULL` | - | - | `idx_fv_run_id` | AI Service | `FR-AI-004` |
| `sku_id` | Sản phẩm được dự báo | `BIGINT` | `FK -> products(id)` | `NOT NULL` | - | - | `idx_fv_sku_wh_date` | AI Service | `FR-AI-004` |
| `warehouse_id` | Kho được dự báo | `BIGINT` | `FK -> warehouses(id)` | `NOT NULL` | - | - | `idx_fv_sku_wh_date` | AI Service | `FR-AI-004` |
| `forecast_date` | Ngày tương lai được dự báo | `DATE` | - | `NOT NULL` | - | - | `idx_fv_sku_wh_date` | AI Service | `FR-AI-004` |
| `horizon_step` | Bước dự báo tương lai ($k$) | `INTEGER` | - | `NOT NULL` | - | `horizon_step >= 1 AND horizon_step <= 14` | - | AI Service | `FR-AI-004` |
| `predicted_demand`| Nhu cầu dự báo ($\hat{d}_{t+k}$) | `NUMERIC(10,2)` | - | `NOT NULL` | - | `predicted_demand >= 0` | - | AI Service | `FR-AI-004`, `BR-REC-001` |
| `created_at` | Thời điểm sinh dự báo | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | AI Service | `FR-AI-004` |

*Ràng buộc duy nhất:* `uq_forecast_point UNIQUE (model_run_id, sku_id, warehouse_id, forecast_date)`.

---

### Nhóm 8: Động cơ Bổ sung Tồn kho (Replenishment Engine)

#### 24. Bảng `replenishment_recommendations`
Lưu trữ các khuyến nghị đặt hàng hoặc điều chuyển được động cơ tính toán tự động sinh ra khi $\text{IP} \le \text{ROP}$.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính khuyến nghị | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_replenishment` | Hệ thống | `FR-REC-002` |
| `recommendation_code`| Mã khuyến nghị duy nhất | `VARCHAR(50)` | `UK` | `NOT NULL` | - | - | `uq_rec_code` | Hệ thống sinh | `FR-REC-002` |
| `sku_id` | Sản phẩm cần bổ sung | `BIGINT` | `FK -> products(id)` | `NOT NULL` | - | - | `fk_rec_sku` | Replenishment Job | `BR-REC-004` |
| `warehouse_id` | Kho cần bổ sung | `BIGINT` | `FK -> warehouses(id)` | `NOT NULL` | - | - | `fk_rec_wh` | Replenishment Job | `BR-REC-004` |
| `current_on_hand` | Tồn vật lý lúc quét | `INTEGER` | - | `NOT NULL` | - | - | - | Snapshot lúc quét | `FR-REC-002` |
| `current_allocated` | Tồn giữ chỗ lúc quét | `INTEGER` | - | `NOT NULL` | - | - | - | Snapshot lúc quét | `FR-REC-002` |
| `current_available` | Tồn khả dụng lúc quét | `INTEGER` | - | `NOT NULL` | - | - | - | Snapshot lúc quét | `FR-REC-002` |
| `current_on_order` | Hàng đang về lúc quét | `INTEGER` | - | `NOT NULL` | - | - | - | Snapshot lúc quét | `BR-INV-002` |
| `current_ip` | Vị thế tồn kho ($\text{IP}$) | `INTEGER` | - | `NOT NULL` | - | - | - | Snapshot lúc quét | `BR-REC-004` |
| `forecast_lead_demand`| Nhu cầu dự báo Lead time ($\hat{D}_{\text{lead}}$) | `NUMERIC(10,2)`| - | `NOT NULL` | - | - | - | Replenishment Job | `BR-REC-001` |
| `safety_stock` | Tồn an toàn áp dụng ($\text{SS}$) | `INTEGER` | - | `NOT NULL` | - | `safety_stock >= 0` | - | Replenishment Job | `BR-REC-002` |
| `reorder_point` | Điểm đặt hàng lại ($\text{ROP}$) | `INTEGER` | - | `NOT NULL` | - | `reorder_point >= safety_stock` | - | Replenishment Job | `BR-REC-003` |
| `target_stock_level`| Mức tồn mục tiêu ($S$) | `INTEGER` | - | `NOT NULL` | - | - | - | Replenishment Job | `BR-REC-004` |
| `suggested_qty` | Lượng đề xuất ($Q_{\text{suggest}}$) | `INTEGER` | - | `NOT NULL` | - | `suggested_qty > 0` | - | Replenishment Job | `BR-REC-004` |
| `recommended_action`| Loại hành động đề xuất | `VARCHAR(30)` | - | `NOT NULL` | - | `recommended_action IN ('PURCHASE_ORDER', 'TRANSFER_CDC')` | - | Replenishment Job | `BR-ORD-003` |
| `source_warehouse_id`| Kho nguồn chuyển (nếu là chuyển) | `BIGINT` | `FK -> warehouses(id)` | `NULL` | - | - | - | Replenishment Job | `BR-ORD-003` |
| `source_supplier_id` | Nhà cung cấp (nếu là PO) | `BIGINT` | `FK -> suppliers(id)` | `NULL` | - | - | - | Replenishment Job | `FR-REC-002` |
| `status` | Trạng thái phê duyệt | `VARCHAR(30)` | - | `NOT NULL` | `'PENDING_APPROVAL'` | `status IN ('PENDING_APPROVAL', 'APPROVED', 'REJECTED', 'CONVERTED_TO_PO', 'CONVERTED_TO_TRF')` | `idx_rec_status` | Planner duyệt | `FR-REC-003` |
| `model_run_id` | Phiên AI cung cấp số dự báo | `BIGINT` | `FK -> ai_model_runs(id)` | `NULL` | - | - | - | Hệ thống | `BR-REC-002` |
| `is_fallback_used` | Cờ ghi nhận dùng tham số tĩnh | `BOOLEAN` | - | `NOT NULL` | `FALSE` | - | - | Hệ thống | `BR-REC-005` |
| `approved_by` | Người phê duyệt | `BIGINT` | `FK -> users(id)` | `NULL` | - | - | - | Auth context | `FR-REC-003` |
| `rejection_reason` | Lý do từ chối | `VARCHAR(255)` | - | `NULL` | - | - | - | Planner nhập | `FR-REC-003` |
| `created_at` | Thời điểm sinh khuyến nghị | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | `idx_rec_created` | Hệ thống | `FR-REC-002` |
| `updated_at` | Thời điểm cập nhật cuối | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-REC-003` |

---

### Nhóm 9: Toàn vẹn Kỹ thuật và Chống Lặp Giao dịch (Idempotency)

#### 25. Bảng `idempotency_records`
Bảo vệ toàn vẹn giao dịch API chống trùng lặp dữ liệu khi mạng chập chờn.

| Tên cột | Ý nghĩa nghiệp vụ | Kiểu dữ liệu | PK/FK/UK | Nullable | Mặc định | Ràng buộc CHECK | Chỉ mục | Nguồn cập nhật | FR / BR liên quan |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| `id` | Khóa chính bản ghi | `BIGINT` | `PK` | `NOT NULL` | `IDENTITY` | - | `pk_idempotency` | Hệ thống | `FR-AUD-002` |
| `idempotency_key` | Khóa duy nhất do client gửi lên | `VARCHAR(100)` | `UK` | `NOT NULL` | - | `LENGTH(idempotency_key) >= 10` | `uq_idem_key` | Client Request | `FR-AUD-002`, `BR-CON-003` |
| `request_path` | Đường dẫn URI của API | `VARCHAR(255)` | - | `NOT NULL` | - | - | - | Client Request | `BR-CON-003` |
| `request_hash` | Hash SHA-256 của Request Body | `VARCHAR(64)` | - | `NOT NULL` | - | - | - | Filter/Interceptor | `BR-CON-003` |
| `status` | Trạng thái xử lý của request | `VARCHAR(30)` | - | `NOT NULL` | `'PROCESSING'` | `status IN ('PROCESSING', 'COMPLETED', 'FAILED')` | `idx_idem_status` | Giao dịch kho | `BR-CON-003` |
| `response_status_code`| Mã trạng thái HTTP trả về | `INTEGER` | - | `NULL` | - | - | - | Sau khi TX commit | `BR-CON-003` |
| `response_body` | Nội dung JSON kết quả đã lưu | `TEXT` | - | `NULL` | - | - | - | Sau khi TX commit | `BR-CON-003` |
| `created_at` | Thời điểm nhận request | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP` | - | - | Hệ thống | `FR-AUD-002` |
| `expires_at` | Thời điểm hết hạn khóa (TTL 24h) | `TIMESTAMPTZ` | - | `NOT NULL` | `CURRENT_TIMESTAMP + INTERVAL '24 hours'` | - | `idx_idem_expires` | Hệ thống | `FR-AUD-002` |

---

## Phần D — Mô hình Tồn kho và Tính toán Số dư

### D.1. Phân loại Trường Lưu trữ (Stored) vs Trường Tính toán (Computed)
Trong bảng `inventory_balances`:

```
┌────────────────────────────────────────────────────────────────────────┐
│               TRƯỜNG LƯU TRỮ VẬT LÝ (Physical Stored Columns)          │
│   • on_hand: Số lượng hàng thực tế đang nằm trên kệ kho.              │
│   • allocated: Số lượng hàng đã cam kết cho đơn xuất/chuyển.           │
│   • on_order: Số lượng hàng đã đặt mua từ NCC hoặc đang chuyển về.     │
└────────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌────────────────────────────────────────────────────────────────────────┐
│            TRƯỜNG TÍNH TOÁN TỰ ĐỘNG (Generated / Computed Values)      │
│   • available = on_hand - allocated (Cột GENERATED STORED trong DB)   │
│   • IP = available + on_order = (on_hand - allocated) + on_order       │
│     (Tính toán tức thời qua View / Repository Query khi quét ROP)      │
└────────────────────────────────────────────────────────────────────────┘
```

1. **`available` (Tồn khả dụng):**
   - Được định nghĩa là cột `GENERATED ALWAYS AS (on_hand - allocated) STORED` ngay trong bảng `inventory_balances`.
   - Lợi ích: Cơ sở dữ liệu tự động duy trì giá trị chính xác tuyệt đối mà không cần ứng dụng phải gán thủ công; cho phép đánh chỉ mục B-Tree `idx_inv_available` trực tiếp để tăng tốc các truy vấn kiểm tra điều kiện xuất kho `available >= Q`.
2. **`IP` (Inventory Position - Vị thế tồn kho):**
   - Không cần cột vật lý lưu trữ vì biến động liên tục theo `on_order`. Giá trị $\text{IP}$ được tính toán động tại tầng truy vấn SQL:
     ```sql
     SELECT (available + on_order) AS inventory_position FROM inventory_balances ...
     ```
3. **Bản chất của `In-transit` và Giải thích vì sao KHÔNG cộng thêm vào IP:**
   - Trong quá trình điều chuyển kho, khi lệnh chuyển được phê duyệt (`REQUESTED`), kho nhận $W_B$ **đã được cộng số lượng điều chuyển vào `on_order` ngay từ đầu** (`BR-TRF-001`).
   - Khi kho gửi $W_A$ xuất hàng đi (`IN_TRANSIT`), hàng hóa rời khỏi $W_A$, nhưng tại $W_B$, số lượng này vẫn đang nằm nguyên trong `on_order` của $W_B$.
   - **KẾT LUẬN QUAN TRỌNG:** Vì $\text{On-order}_B$ đã bao gồm toàn bộ lượng hàng đang điều chuyển về, công thức $\text{IP}_B = \text{Available}_B + \text{On-order}_B$ đã tự động tính trọn vẹn số lượng hàng đang đi đường. **Nếu cộng thêm $\text{In-transit}$ một lần nữa vào IP, số lượng hàng điều chuyển sẽ bị tính trùng lặp hai lần ($+2Q$)**, làm $\text{IP}$ cao ảo và khiến hệ thống không kích hoạt cảnh báo thiếu hàng khi thực tế đang cần bổ sung.
4. **Cơ chế Kiểm toán và Đối soát Số dư `On-order`:**
   - Khác với `on_hand` và `allocated` được kiểm toán biến động trực tiếp qua sổ cái `stock_movement_logs`, số dư `on_order` là chỉ số cam kết ngoài kho. Vết kiểm toán của `on_order` được bảo đảm bằng cách đối soát độc lập với tổng số lượng còn chờ giao từ các đơn PO (`CONFIRMED`, `PARTIALLY_RECEIVED`) và các lệnh chuyển Transfer (`REQUESTED`, `IN_TRANSIT`) đang hoạt động theo quy tắc `BR-INV-004`.

### D.2. Cơ chế Ràng buộc Bất biến và Kiểm soát Tương tranh
1. **Ràng buộc cấp CSDL (Database Invariant):**
   ```sql
   ALTER TABLE inventory_balances ADD CONSTRAINT chk_inventory_invariant 
   CHECK (on_hand >= allocated AND on_hand >= 0 AND allocated >= 0 AND on_order >= 0);
   ```
   Ràng buộc này đảm bảo ở mọi thời điểm, không có bất kỳ dòng dữ liệu nào có `allocated > on_hand`, đồng nghĩa với việc `available` không bao giờ có thể bị âm ($\text{Available} \ge 0$).
2. **Cơ chế Khóa bi quan (Pessimistic Locking):**
   Khi có hai giao dịch đồng thời cố gắng giữ chỗ đơn hàng, tầng Spring Boot Repository thực thi:
   ```java
   @Lock(LockModeType.PESSIMISTIC_WRITE)
   @QueryHints({@QueryHint(name = "jakarta.persistence.lock.timeout", value = "3000")})
   @Query("SELECT b FROM InventoryBalance b WHERE b.sku.id = :skuId AND b.warehouse.id = :whId")
   Optional<InventoryBalance> findByIdForUpdate(@Param("skuId") Long skuId, @Param("whId") Long whId);
   ```
   Cơ chế này ép PostgreSQL khóa dòng bản ghi bằng câu lệnh `SELECT ... FOR UPDATE`, buộc giao dịch thứ hai phải chờ giao dịch thứ nhất hoàn tất và đọc lại số dư mới nhất trước khi kiểm tra điều kiện xuất.
3. **Quy tắc Sắp xếp Khóa tăng dần giảm thiểu rủi ro Deadlock:**
   Khi một đơn hàng hoặc giao dịch cần khóa số dư cho nhiều SKU cùng lúc tại một kho, tầng Application bắt buộc phải sắp xếp danh sách `sku_id` theo thứ tự tăng dần (`ORDER BY sku_id ASC`) trước khi thực thi tuần tự các câu lệnh `SELECT ... FOR UPDATE`.
   *(Ghi chú kỹ thuật: Đây là biện pháp thiết kế kiến trúc nhằm giảm thiểu nguy cơ xảy ra khóa chéo (Deadlock) giữa các luồng đồng thời; biện pháp này là nguyên lý giải thuật, chưa qua kiểm chứng thực nghiệm trên môi trường có tải thực tế).*

---

## Phần E — Chuỗi Giao dịch và Tính Khép kín Truy vết (Audit Traceability)

Mọi thao tác thay đổi số dư kho đều phải tuân thủ tính khép kín 100%, không cho phép bất kỳ giao dịch nào sửa đổi bảng `inventory_balances` mà không ghi lại bản ghi tương ứng trong `stock_movement_logs`.

```
[Purchase Order] ────────> [Goods Receipt Note] ────> [Update inventory_balances]
  (PO Items)                 (GRN Items)                     │
                                                             ▼
                                                    [stock_movement_logs]
                                                    (INBOUND_RECEIPT)

[Outbound Order] ────────> [Stock Reservation]  ────> [Update inventory_balances]
  (Order Items)              (Active Reservation)            │
                                                             ▼
                                                    [stock_movement_logs]
                                                    (OUTBOUND_RESERVE)

[Goods Dispatch] ────────> [Fulfill Reservation]───> [Update inventory_balances]
  (GDN Items)                                                │
                                                             ▼
                                                    [stock_movement_logs]
                                                    (OUTBOUND_DISPATCH)

[Transfer Order] ────────> [Dispatch Kho A]     ────> [Update balances Kho A]
  (Transfer Items)             │                             │
                               │                             ▼
                               │                    [stock_movement_logs]
                               │                    (TRANSFER_DISPATCH)
                               ▼
                           [Receive Kho B]      ────> [Update balances Kho B]
                               │                             │
                               ├── Đủ                        ▼
                               │                    [stock_movement_logs]
                               │                    (TRANSFER_RECEIPT)
                               └── Thiếu                     │
                                       │                     ▼
                                       └────────────> [discrepancy_logs]
                                                             │
                                                             ▼
                                                    [stock_movement_logs]
                                                    (DISCREPANCY_LOSS)
```

### Nguyên tắc Khép kín Giao dịch:
1. **Transaction Boundary:** Toàn bộ việc cập nhật bảng nghiệp vụ (`grn`, `gdn`, `transfer`), cập nhật số dư `inventory_balances` và ghi bản ghi vào `stock_movement_logs` phải nằm trong cùng một cơ sở dữ liệu transaction (`@Transactional(rollbackFor = Exception.class)`).
2. **Tính đơn điệu của Audit Log:** Bảng `stock_movement_logs` chỉ chấp nhận lệnh `INSERT`. Mọi quyền `UPDATE` và `DELETE` trên bảng này đều bị thu hồi ở cấp phân quyền cơ sở dữ liệu để chống giả mạo số liệu kiểm toán.
3. **Đối soát số dư toán học:**
   $$\text{on\_hand}(T) = \text{on\_hand}(T_0) + \sum_{t=T_0}^T \Delta \text{on\_hand}_{\text{log}}(t)$$
   Hệ thống có thể chạy một Job kiểm tra định kỳ tính toán lại tổng đại số của log để đối soát với trường `on_hand` hiện hành.

---

## Phần F — Cấu trúc Dữ liệu và Pipeline AI Dự báo Nhu cầu

### F.1. Quy cách Dữ liệu Chuỗi thời gian Lịch sử (`daily_demand_history`)
- **Granularity (Mức độ chi tiết):** 1 bản ghi duy nhất cho mỗi bộ 3 `(demand_date, sku_id, warehouse_id)`.
- **Zero-Imputation (Điền số 0):** Định kỳ 00:05 mỗi ngày, một batch job quét toàn bộ đơn xuất đã xuất kho trong ngày hôm trước để tính tổng `actual_demand`. Đối với những SKU có kinh doanh tại kho nhưng không phát sinh đơn xuất nào trong ngày, job tự động chèn bản ghi với `actual_demand = 0` và `is_zero_filled = TRUE`. Việc này bảo đảm chuỗi thời gian liên tục $100\%$, không có ngày khuyết.

### F.2. Quản lý Thực nghiệm và Chống Rò rỉ Dữ liệu Tương lai (`ai_model_runs`)
- Mô hình AI tuân thủ nguyên tắc **Temporal Split** nghiêm ngặt:
  - Tập huấn luyện (Train): Từ `train_start_date` đến `train_end_date`.
  - Tập kiểm thử (Test): Từ `test_start_date` đến `test_end_date` với điều kiện bất biến: `test_start_date > train_end_date`.
- **Tái lập thực nghiệm (Reproducibility):** Bảng `ai_model_runs` lưu trữ đầy đủ `random_seed`, chuỗi JSON `hyperparameters` và các mốc thời gian phân chia dữ liệu.
- **Tiêu chí nghiệm thu (Gatekeeper Rule - BR-AI-004):**
  Cột `is_accepted` chỉ nhận giá trị `TRUE` khi:
  $$\text{wape} < \text{baseline\_wape}$$
  Chỉ những phiên chạy có `is_accepted = TRUE` mới được phép ghi dữ liệu vào bảng `forecast_values` phục vụ tính toán bổ sung hàng.

### F.3. Cấu trúc Dự báo Đa bước (`forecast_values`)
- Lưu trữ dự báo nhu cầu cho từng ngày trong khoảng thời gian $H$ ngày tới ($H = 7 \text{ đến } 14$ ngày, tương ứng `horizon_step` từ 1 đến 14).
- Động cơ bổ sung hàng (`replenishment_recommendations`) đọc trực tiếp dữ liệu từ bảng này trong phạm vi Lead time $L$ ngày để tính tổng nhu cầu dự báo:
  $$\hat{D}_{\text{lead}} = \sum_{k=1}^L \text{predicted\_demand}_{t+k}$$

---

## Phần G — Ma trận Ràng buộc Toàn vẹn và Chỉ mục Hiệu năng

### G.1. Phân loại Ràng buộc Cơ sở dữ liệu vs Logic Ứng dụng

| Nhóm nghiệp vụ | Ràng buộc cấp Database (Constraint) | Ràng buộc cấp Transaction / Application |
| :--- | :--- | :--- |
| **Bảo vệ số dư không âm** | `CHECK (on_hand >= allocated AND on_hand >= 0 AND allocated >= 0 AND on_order >= 0)` | Kiểm tra `available >= Q` trước khi thực hiện câu lệnh trừ. |
| **Không trùng SKU / Kho** | `UNIQUE (sku_code)`, `UNIQUE (warehouse_code)`, `UNIQUE (sku_id, warehouse_id)` | Kiểm tra trạng thái `is_active` trước khi cho phép tạo giao dịch. |
| **Không xuất vượt giữ chỗ** | Ràng buộc khóa ngoại `gdn.order_id` trỏ tới `outbound_orders.id` | Khóa bi quan `SELECT ... FOR UPDATE` trên dòng giữ chỗ `stock_reservations`. |
| **Không nhận vượt PO** | `CHECK (received_qty >= 0 AND received_qty <= ordered_qty)` trên dòng PO | Tính toán số lượng còn lại $Q_{\text{remaining}} = Q_{\text{PO}} - Q_{\text{received}}$ trước khi tạo GRN. |
| **Chống giao dịch lặp** | `UNIQUE (idempotency_key)` trên bảng `idempotency_records` | Interceptor đọc trạng thái `PROCESSING` để trả về HTTP 409 Conflict. |
| **Toàn vẹn điều chuyển** | `CHECK (destination_warehouse_id <> origin_warehouse_id)` | Đóng dứt điểm lệnh chuyển, xóa sạch In-transit khi nhận thiếu. |

### G.2. Danh mục Chỉ mục Hiệu năng (Indexes) Quan trọng

1. **Chỉ mục tìm kiếm số dư tồn kho:**
   - `idx_inv_sku_wh ON inventory_balances(sku_id, warehouse_id)`: Tối ưu hóa truy vấn khóa bi quan đọc số dư tức thời.
   - `idx_inv_available ON inventory_balances(warehouse_id, sku_id, available)`: Tối ưu hóa thuật toán quét kho đủ hàng khi phân bổ đơn (`BR-ORD-002`).
2. **Chỉ mục truy vết nhật ký kiểm toán:**
   - `idx_sml_sku_wh_time ON stock_movement_logs(sku_id, warehouse_id, created_at DESC)`: Tăng tốc tra cứu lịch sử thẻ kho.
   - `idx_sml_ref ON stock_movement_logs(reference_type, reference_id)`: Truy vấn toàn bộ biến động gắn với một chứng từ cụ thể.
3. **Chỉ mục chuỗi thời gian AI:**
   - `idx_ddh_query ON daily_demand_history(sku_id, warehouse_id, demand_date ASC)`: Tối ưu hóa việc trích xuất chuỗi thời gian cho Python AI Service.
   - `idx_fv_lookup ON forecast_values(sku_id, warehouse_id, forecast_date)`: Tối ưu hóa việc đọc dự báo phục vụ tính ROP.
4. **Chỉ mục trạng thái giao dịch:**
   - `idx_po_active_status ON purchase_orders(status) WHERE status IN ('CONFIRMED', 'PARTIALLY_RECEIVED')`: Partial Index giúp tìm nhanh các đơn PO đang chờ nhận hàng.
   - `idx_trf_in_transit ON transfer_orders(status) WHERE status = 'IN_TRANSIT'`: Partial Index quản lý các lô hàng đang đi đường.

---

## Phần H — Bảng Quyết định Còn Mở (Pending & Architectural Decisions)

Bảng tổng hợp các vấn đề thiết kế dữ liệu và nghiệp vụ cần thống nhất, kèm trạng thái và phương án đề xuất:

| STT | Vấn đề kỹ thuật / Nghiệp vụ | Trạng thái | Phương án đề xuất trong Từ điển dữ liệu | Cơ sở lựa chọn & Đánh giá tác động |
| :---: | :--- | :---: | :--- | :--- |
| **1** | **Mức độ quản lý vị trí kho: Tổng kho (Warehouse-level) vs Vị trí giá kệ (Bin/Location)** | `CONFIRMED` | Quản lý số dư ở cấp **Tổng kho (`warehouse_id`)**; chưa thêm bảng `bins` / `locations`. | Phù hợp phạm vi Core MVP trong 6 tháng theo `docs/01_problem_scope.md`. Vị trí giá kệ thuộc phạm vi mở rộng Nice-to-have. |
| **2** | **Chiến lược Khóa chính: BIGINT Identity vs UUID** | `PROPOSED` | Sử dụng **`BIGINT GENERATED ALWAYS AS IDENTITY`** làm PK nội bộ; dùng `VARCHAR(50) UNIQUE` làm mã nghiệp vụ cho người dùng. | Tối ưu hóa bộ nhớ đệm và hiệu năng index B-Tree của PostgreSQL. Client chỉ tiếp xúc với mã nghiệp vụ (PO-xxx, ORD-xxx). |
| **3** | **Chính sách xóa dữ liệu (Data Deletion Policy)** | `CONFIRMED` | **Soft Delete (`is_active`)** cho Master Data; **Cấm xóa tuyệt đối (`NO DELETE`)** cho toàn bộ bảng giao dịch và nhật ký kho. | Bảo vệ tính toàn vẹn khóa ngoại và đáp ứng yêu cầu kiểm toán bất biến theo `GEMINI.md`. |
| **4** | **Cơ chế lưu trữ số dư: Bảng hiện hành vs Snapshot định kỳ** | `PROPOSED` | Duy trì **Bảng số dư hiện hành (`inventory_balances`)** cập nhật theo thời gian thực + Bảng sổ cái biến động (`stock_movement_logs`). | Không cần bảng snapshot hằng ngày trong Core MVP vì có thể tái hiện số dư quá khứ bằng cách quét ngược từ nhật ký `stock_movement_logs`. |
| **5** | **Công thức Safety Stock khi dự báo AI biến thiên theo ngày** | `PROPOSED - HEURISTIC` | Sử dụng công thức xấp xỉ thực hành: $\text{SS} = Z \times \text{RMSE} \times \sqrt{L}$ với $\text{RMSE} = \sigma_e$ của mô hình trên tập Test. | Đề xuất xấp xỉ cho Core MVP; không coi là tối ưu toàn cục do chưa có kiểm chứng thực nghiệm trên dữ liệu vận hành thực tế. |
| **6** | **Cơ chế đóng đơn PO khi nhà cung cấp giao thiếu (Force Close)** | `CONFIRMED` | Quản lý kho chủ động thao tác nút "Đóng đơn sớm" (`FORCE_CLOSE`) để chuyển PO sang `CLOSED` và trừ nốt `on_order`. | Thống nhất cơ chế thủ công có ghi lý do, không làm cơ chế auto-close timeout trong Core MVP. |
| **7** | **Quy trình tiếp nhận khi mất mát 100% hàng điều chuyển** | `CONFIRMED` | Kho nhận thực hiện xác nhận nhận với $Q_{\text{received}} = 0$, `is_total_loss = TRUE`, hệ thống tự động xóa sạch `In-transit` và ghi 100% số lượng vào `discrepancy_logs`. | Thống nhất quy trình tiếp nhận và ghi nhận mất toàn bộ để đóng dứt điểm lệnh chuyển. |
| **8** | **Cơ chế lưu trữ tham số tái lập dữ liệu giả lập (Synthetic Data)** | `PROPOSED` | Bảng `ai_model_runs` lưu trường `random_seed INT` và trường `dataset_version VARCHAR(50)`. | Đáp ứng chuẩn mực trung thực của `GEMINI.md`: dữ liệu giả lập phải có seed và phiên bản để tái lập chính xác mọi kết quả thử nghiệm. |

---

## Phần I — Ma trận Truy vết Thực thể (Entity Traceability Matrix)

Bảng đối soát toàn bộ 26 bảng thực thể với Yêu cầu chức năng (FR tại doc 02), Quy tắc nghiệp vụ (BR tại doc 03) và 8 Ca kiểm thử nghiệm thu chính thức (tại doc 01):

| Tên Bảng (Table Name) | Nhóm chức năng | Yêu cầu FR liên quan | Quy tắc BR liên quan | Ca kiểm thử nghiệm thu (`docs/01`) | Ghi chú vai trò thực thể |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `users` | Quản trị | `FR-AUTH-001` | - | Ca kiểm thử 8 (E2E Workflow) | Master tài khoản |
| `user_roles` | Phân quyền | `FR-AUTH-002` | - | Ca kiểm thử 8 (E2E Workflow) | Phân quyền RBAC 4 vai trò |
| `user_activity_logs` | Bảo mật hệ thống | `FR-AUTH-003` | - | Ca kiểm thử 8 (E2E Workflow) | Nhật ký đăng nhập & phân quyền |
| `warehouses` | Danh mục kho | `FR-MST-001`, `FR-ORD-003` | `BR-ORD-002` | Ca kiểm thử 8 (E2E Workflow) | Danh mục kho, thứ tự ưu tiên |
| `suppliers` | Danh mục NCC | `FR-MST-003` | `BR-INB-001` | Ca kiểm thử 4 (PO nhiều đợt) | Nhà cung cấp và Lead time mặc định |
| `products` | Danh mục SKU | `FR-MST-002` | `BR-INV-001` | Ca kiểm thử 1 (Ngăn tồn âm) | Thông tin sản phẩm cơ sở |
| `sku_warehouse_configs`| Cấu hình tồn | `FR-MST-004` | `BR-REC-004`, `005` | Ca kiểm thử 6 (Replenishment) | Cấu hình $MOQ, S, \text{SS}_{\text{static}}, \text{ROP}_{\text{static}}$ |
| `inventory_balances` | Số dư kho | `FR-INV-001`, `002`, `003` | `BR-INV-001`, `002`, `003` | **Ca kiểm thử 1, 2, 3, 6, 8** | Thực thể cốt lõi số dư kho |
| `stock_movement_logs` | Nhật ký kho | `FR-AUD-001`, `003` | `BR-INV-004` | **Ca kiểm thử 1, 3, 4, 8** | Sổ cái kiểm toán bất biến |
| `purchase_orders` | Đơn mua hàng | `FR-INB-001`, `003` | `BR-INB-001`, `004`, `005` | **Ca kiểm thử 4 (PO nhiều đợt)** | Quản lý cam kết mua hàng |
| `purchase_order_items` | Chi tiết PO | `FR-INB-001`, `003` | `BR-INB-001`, `003` | **Ca kiểm thử 4 (PO nhiều đợt)** | Số lượng đặt và lũy kế đã nhận |
| `goods_receipt_notes` | Phiếu nhập GRN| `FR-INB-002`, `003` | `BR-INB-002`, `003` | **Ca kiểm thử 4, 8** | Chứng từ nhập kho vật lý |
| `goods_receipt_items` | Chi tiết GRN | `FR-INB-002`, `003` | `BR-INB-002`, `003` | **Ca kiểm thử 4, 8** | Số lượng thực nhận từng đợt |
| `outbound_orders` | Đơn xuất kho | `FR-OUT-001`, `FR-ORD-001`| `BR-OUT-001`, `BR-ORD-001` | **Ca kiểm thử 1, 2, 8** | Đơn xuất hàng bán |
| `outbound_order_items`| Chi tiết đơn | `FR-OUT-001` | `BR-OUT-001` | **Ca kiểm thử 1, 2, 8** | Mặt hàng và số lượng yêu cầu |
| `stock_reservations` | Giữ chỗ tồn | `FR-OUT-001`, `002` | `BR-OUT-002`, `003` | **Ca kiểm thử 1, 2, 8** | Khóa giữ chỗ tồn khả dụng |
| `goods_dispatch_notes`| Phiếu xuất GDN| `FR-OUT-003` | `BR-OUT-004`, `005` | **Ca kiểm thử 8 (E2E Workflow)** | Chứng từ xuất kho vật lý |
| `goods_dispatch_items`| Chi tiết GDN | `FR-OUT-003` | `BR-OUT-004` | **Ca kiểm thử 8 (E2E Workflow)** | Số lượng thực xuất vật lý |
| `transfer_orders` | Lệnh chuyển | `FR-TRF-001`..`004` | `BR-TRF-001`..`004` | **Ca kiểm thử 3 (Điều chuyển kho)**| Quản lý vòng đời điều chuyển |
| `transfer_order_items`| Chi tiết chuyển| `FR-TRF-001`..`004` | `BR-TRF-001`..`004` | **Ca kiểm thử 3 (Điều chuyển kho)**| Số lượng chuyển và thực nhận |
| `discrepancy_logs` | Sai lệch chuyển| `FR-TRF-005` | `BR-TRF-005` | **Ca kiểm thử 3 (Điều chuyển kho)**| Biên bản hao hụt trên đường |
| `daily_demand_history`| Dữ liệu AI | `FR-AI-001` | `BR-AI-001` | **Ca kiểm thử 7 (Đánh giá AI)** | Chuỗi thời gian chuẩn hóa |
| `ai_model_runs` | Huấn luyện AI | `FR-AI-002`, `003` | `BR-AI-002`, `003`, `004` | **Ca kiểm thử 7 (Đánh giá AI)** | Quản lý phiên chạy & Gatekeeper |
| `forecast_values` | Dự báo AI | `FR-AI-004` | `BR-REC-001` | **Ca kiểm thử 7 (Đánh giá AI)** | Kết quả dự báo đa bước |
| `replenishment_recommendations` | Bổ sung hàng | `FR-REC-001`..`005` | `BR-REC-001`..`005` | **Ca kiểm thử 6 (Replenishment)**| Khuyến nghị đặt hàng/chuyển kho |
| `idempotency_records` | Chống lặp | `FR-AUD-002` | `BR-CON-003` | **Ca kiểm thử 5 (Idempotency)** | Hỗ trợ kỹ thuật chống lặp API |

---

## Phần J — Kết luận và Hướng dẫn Kỹ thuật Tiếp theo

1. **Tính sẵn sàng của Tài liệu:**
   - Bản đặc tả Từ điển dữ liệu này cung cấp đầy đủ thông tin chi tiết về kiểu dữ liệu PostgreSQL, khóa chính, khóa ngoại, ràng buộc CHECK, chỉ mục và nguồn cập nhật dữ liệu cho toàn bộ 26 bảng.
   - Tài liệu đã giải quyết triệt để vấn đề logic: không cộng trùng `In-transit` vào `IP`, đảm bảo `available` không âm thông qua Generated Column và CHECK constraint, khép kín 100% vết kiểm toán thông qua `stock_movement_logs`, và phân định độc lập cơ chế đối soát cam kết `On-order`.
2. **Các bước kỹ thuật tiếp theo:**
   - Tiến hành vẽ Sơ đồ quan hệ thực thể (ERD - Entity Relationship Diagram) cho 26 bảng dữ liệu.
   - Soạn thảo kịch bản Database Migration Scripts (Flyway/Liquibase) chuẩn bị cho việc triển khai mã nguồn backend Spring Boot.
