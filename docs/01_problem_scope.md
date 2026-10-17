# TÀI LIỆU XÁC ĐỊNH BÀI TOÁN VÀ PHẠM VI DỰ ÁN (PROBLEM STATEMENT & SCOPE)

**Đề tài:** Xây dựng nền tảng quản lý kho và phân phối hàng hóa thông minh ứng dụng AI dự báo nhu cầu và hỗ trợ tối ưu tồn kho  
**Loại tài liệu:** Đặc tả bài toán, ranh giới phạm vi nghiệp vụ và tiêu chí nghiệm thu đồ án tốt nghiệp  
**Trạng thái tài liệu:** Bản hoàn thiện phạm vi dự án (Đã phân tầng Core MVP / Advanced MVP / Nice-to-have, chuẩn hóa quy tắc giao dịch đa kho và pipeline AI)  
**Môi trường xác lập:** Môi trường doanh nghiệp giả lập; phát triển dựa trên mô hình nghiệp vụ chuẩn hóa và dữ liệu giả lập (synthetic data).

---

## 1. Đặt vấn đề và Mục tiêu dự án

### 1.1. Bối cảnh và Vấn đề thực tế
Trong hoạt động chuỗi cung ứng và phân phối hàng hóa của các doanh nghiệp thương mại, bán lẻ quy mô vừa và nhỏ (SMEs):
- **Cân đối tồn kho gặp nhiều rủi ro:** Tồn kho quá mức (overstock) gây ứ đọng vốn lưu động và tăng chi phí lưu kho. Ngược lại, thiếu hàng (stockout) dẫn đến gián đoạn kinh doanh, mất doanh thu và giảm mức độ hài lòng của khách hàng.
- **Quyết định bổ sung hàng mang tính cảm tính:** Kế hoạch mua hàng và nhập hàng chủ yếu dựa vào kinh nghiệm cá nhân hoặc các mức tồn tối thiểu cố định, không phản ánh kịp thời xu hướng tiêu thụ, biến động mùa vụ và độ trễ giao hàng.
- **Rời rạc trong phân phối đa kho:** Doanh nghiệp có mạng lưới nhiều kho (kho trung tâm và các kho vệ tinh/chi nhánh) thường gặp khó khăn khi chọn kho xuất hàng cho đơn và điều phối hàng luân chuyển giữa các kho, dẫn đến tình trạng kho thừa vẫn thừa, kho thiếu vẫn thiếu.
- **Ranh giới bài toán:** Đồ án **không** nhằm mục đích xây dựng một phần mềm ghi chép sổ sách nhập/xuất đơn giản (CRUD cơ bản), mà tập trung vào **chiều sâu nghiệp vụ quản trị kho đa điểm, kiểm soát trạng thái số dư chặt chẽ (ACID, chống tồn âm, lưu vết kiểm toán) kết hợp ứng dụng AI dự báo chuỗi thời gian để hỗ trợ ra quyết định tồn kho có cơ sở khoa học**.

### 1.2. Mục tiêu tổng quát
Xây dựng nền tảng phần mềm quản lý kho và điều phối phân phối hàng hóa đa điểm có chiều sâu nghiệp vụ, tích hợp dịch vụ AI dự báo nhu cầu tiêu thụ theo chuỗi thời gian để cung cấp dữ liệu định lượng đầu vào cho các công thức bổ sung tồn kho xác định, hỗ trợ người quản lý đưa ra quyết định nhập và chuyển hàng hợp lý trên môi trường doanh nghiệp giả lập.

### 1.3. Mục tiêu cụ thể
1. **Quản lý kho và giao dịch đa trạng thái vững chắc:** Xây dựng hệ thống backend (định hướng Java 21 / Spring Boot, PostgreSQL) quản lý 5 trạng thái tồn kho (`On-hand`, `Allocated`, `Available`, `On-order`, `In-transit`), bảo đảm tính toàn vẹn dữ liệu, chống tồn âm tuyệt đối, xử lý tranh chấp đồng thời và lưu vết kiểm toán (audit log) đầy đủ.
2. **Pipeline AI dự báo nhu cầu khoa học:** Xây dựng dịch vụ AI độc lập (định hướng Python) có khả năng sinh dữ liệu giả lập có quy luật, xử lý chuỗi thời gian, huấn luyện mô hình dự báo nhu cầu ngắn hạn theo từng SKU/kho, đánh giá trên tập kiểm thử chia theo thời gian (temporal split) và so sánh công bằng với các mô hình baseline.
3. **Động cơ hỗ trợ quyết định tồn kho định lượng:** Kết hợp kết quả dự báo của AI với các tham số tồn kho định lượng (Lead time, Safety Stock, Reorder Point, Inventory Position) để tự động sinh khuyến nghị bổ sung hàng ($Q_{\text{suggest}}$) và cảnh báo nguy cơ thiếu hàng theo cơ chế có sự phê duyệt của con người (Human-in-the-loop).
4. **Điều phối phân phối đa kho khả thi:** Cài đặt quy tắc phân bổ đơn hàng xác định cho Core MVP (Single-warehouse fulfillment theo độ ưu tiên cấu hình trước) và quy trình điều chuyển nội bộ giữa các kho có xử lý đầy đủ các trường hợp hàng đi đường, nhận đủ, nhận thiếu và ghi nhận hao hụt.
5. **Giao diện web trực quan hóa và kiểm soát:** Xây dựng giao diện web cho các luồng tác nghiệp kho chính, theo dõi tồn kho theo thời gian thực và hiển thị bảng điều khiển trực quan hóa dự báo/khuyến nghị bổ sung.

---

## 2. Đối tượng sử dụng và Nhu cầu nghiệp vụ

| Đối tượng (Actor) | Vai trò chính | Nhu cầu chức năng trên hệ thống |
| :--- | :--- | :--- |
| **Quản lý kho (Warehouse Manager)** | Giám sát toàn diện hoạt động của các kho hàng trong hệ thống. | - Theo dõi tổng quan số dư tồn kho theo 5 trạng thái (`On-hand`, `Allocated`, `Available`, `On-order`, `In-transit`).<br>- Phê duyệt các yêu cầu nhập, xuất và lệnh điều chuyển giữa các kho.<br>- Tiếp nhận cảnh báo tồn kho chạm ngưỡng đặt hàng lại ($ROP$). |
| **Nhân viên kho (Warehouse Operator)** | Thực hiện các thao tác xuất, nhập thực tế tại kho vật lý. | - Tạo và xác nhận phiếu nhập kho (Goods Receipt Note - GRN), ghi nhận nhập hàng đủ hoặc một phần.<br>- Xác nhận xuất kho vật lý theo phiếu xuất (Goods Dispatch Note - GDN).<br>- Xác nhận xuất đi và nhận đến trong luồng điều chuyển nội bộ.<br>- Ghi nhận số lượng thực nhận và biên bản chênh lệch khi nhận thiếu hàng. |
| **Chuyên viên kế hoạch tồn kho (Inventory Planner)** | Theo dõi cung - cầu, cân đối tồn kho và lập kế hoạch cung ứng. | - Xem biểu đồ lịch sử tiêu thụ và kết quả dự báo nhu cầu của AI.<br>- Xem danh sách khuyến nghị bổ sung hàng (gợi ý số lượng đặt/chuyển kèm lý do tính toán).<br>- Tạo lệnh điều chuyển cân đối tồn kho từ kho trung tâm về kho vệ tinh.<br>- Cấu hình các tham số tồn kho: Lead time, mức tồn an toàn mong muốn. |
| **Quản trị viên hệ thống (System Administrator)** | Quản trị kỹ thuật nền tảng và danh mục hệ thống. | - Quản lý tài khoản, phân quyền vai trò (Role-Based Access Control - RBAC).<br>- Quản lý danh mục cơ sở (Sản phẩm, Kho hàng, Đối tác).<br>- Giám sát trạng thái kết nối giữa Backend và AI Service. |

---

## 3. Quy tắc Quản trị Tồn kho & Giao dịch Đa kho (Nghiệp vụ Cốt lõi)

### 3.1. Định nghĩa và Phân biệt chính xác 5 chỉ số tồn kho
Hệ thống quản lý số dư tồn kho cho từng cặp `(SKU, Warehouse)` thông qua 5 chỉ số có mối quan hệ logic chặt chẽ:

1. **Tồn thực tế (On-hand):** Tổng số lượng vật lý thực tế đang nằm trong kho tại thời điểm hiện tại.
2. **Tồn đã giữ chỗ / phân bổ (Allocated / Reserved):** Số lượng hàng trong kho đã được cam kết cho các đơn hàng xuất hoặc lệnh chuyển kho đang chờ xuất, nhưng chưa đưa ra khỏi kho vật lý.
3. **Tồn khả dụng (Available):** Số lượng hàng tự do, sẵn sàng để tiếp tục cam kết bán hoặc điều chuyển:
   $$\text{Available} = \text{On-hand} - \text{Allocated}$$
   *Ràng buộc bất biến hệ thống:* Luôn bảo đảm $\text{Available} \ge 0$ và $\text{On-hand} \ge \text{Allocated} \ge 0$. Mọi thao tác làm $\text{Available} < 0$ đều bị hệ thống chặn lại lập tức.
4. **Hàng đang về (On-order):** Tổng số lượng hàng đã được đặt mua hoặc đang chuyển về kho này (bao gồm các đơn mua từ nhà cung cấp đang chờ giao và hàng $\text{In-transit}$ từ kho khác đang chuyển tới).
5. **Hàng đang đi đường (In-transit):** Số lượng hàng đã thực xuất khỏi kho gửi nhưng chưa hoàn tất nhập vào kho nhận trong một lệnh điều chuyển nội bộ.
6. **Vị thế tồn kho (Inventory Position - IP):** Tổng lượng hàng sẵn có và hàng đang trên đường về dùng để ra quyết định bổ sung tồn kho:
   $$\text{IP} = \text{Available} + \text{On-order} = (\text{On-hand} - \text{Allocated}) + \text{On-order}$$

> **NGUYÊN TẮC BẢO VỆ TỒN KHO CỐT TỬ:**  
> Hàng đang vận chuyển ($\text{In-transit}$ hoặc $\text{On-order}$) **TUYỆT ĐỐI KHÔNG** được cộng vào tồn khả dụng ($\text{Available}$) hoặc tồn thực tế ($\text{On-hand}$) của kho nhận trước khi kho nhận thực tế kiểm đếm và bấm xác nhận nhập kho.

---

### 3.2. Thời điểm thay đổi số dư chi tiết trong từng loại giao dịch

#### A. Giao dịch Nhập kho từ Nhà cung cấp (Inbound / Goods Receipt)
- **Khi tạo đơn mua hàng (Purchase Order - PO):**
  - Tăng hàng đang về: $\text{On-order} \mathrel{+}= Q_{\text{PO}}$.
  - Tồn $\text{On-hand}$ và $\text{Available}$ chưa thay đổi.
- **Khi xác nhận nhập kho thực tế (GRN Completed):** 
  - *Trường hợp nhận đủ ($Q_{\text{actual}} = Q_{\text{PO}}$):*
    + Tăng tồn thực tế: $\text{On-hand} \mathrel{+}= Q_{\text{actual}}$.
    + Giảm hàng đang về: $\text{On-order} \mathrel{-}= Q_{\text{PO}}$.
    + Tồn khả dụng $\text{Available}$ tự động tăng theo công thức ($\text{Available} = \text{On-hand} - \text{Allocated}$).
    + Đơn PO chuyển trạng thái "Đã hoàn thành" (Completed).
  - *Trường hợp nhận một phần (Partial Receipt - $Q_{\text{actual}} < Q_{\text{PO}}$):*
    + Tăng tồn thực tế theo đúng số thực nhận: $\text{On-hand} \mathrel{+}= Q_{\text{actual}}$.
    + Giảm hàng đang về đúng bằng số thực nhận: $\text{On-order} \mathrel{-}= Q_{\text{actual}}$.
    + Tồn khả dụng $\text{Available}$ tăng theo $Q_{\text{actual}}$.
    + Phần còn lại ($Q_{\text{remaining}} = Q_{\text{PO}} - Q_{\text{actual}}$) tiếp tục được giữ trong $\text{On-order}$ và đơn PO chuyển sang trạng thái "Nhận một phần" (Partially Received) để chờ các đợt giao tiếp theo hoặc đóng đơn thủ công.

#### B. Giao dịch Xuất kho (Outbound / Goods Dispatch)
- **Khi tiếp nhận và phân bổ đơn xuất (Order Reservation):** 
  - Kiểm tra điều kiện: $\text{Available} \ge Q_{\text{order}}$.
  - Nếu đủ, tăng giữ chỗ: $\text{Allocated} \mathrel{+}= Q_{\text{order}} \implies \text{Available}$ giảm $Q_{\text{order}}$.
  - Tồn thực tế $\text{On-hand}$ chưa thay đổi.
- **Khi hủy giữ chỗ / hủy đơn xuất trước khi xuất vật lý (Reservation Cancellation):**
  - Giải phóng giữ chỗ: $\text{Allocated} \mathrel{-}= Q_{\text{order}} \implies \text{Available}$ tăng $Q_{\text{order}}$.
  - Tồn thực tế $\text{On-hand}$ không đổi.
- **Khi xác nhận xuất kho vật lý (GDN Completed):** Hàng thực tế rời khỏi kho:
  - Giảm tồn thực tế: $\text{On-hand} \mathrel{-}= Q_{\text{order}}$.
  - Giải phóng phần giữ chỗ: $\text{Allocated} \mathrel{-}= Q_{\text{order}}$.
  - Tồn khả dụng $\text{Available}$ không đổi (vì cả $\text{On-hand}$ và $\text{Allocated}$ cùng giảm một lượng bằng nhau).

#### C. Giao dịch Điều chuyển kho nội bộ (Inter-warehouse Transfer)
Quá trình chuyển hàng từ Kho gửi ($W_A$) sang Kho nhận ($W_B$) trải qua 3 mốc thời gian:

```
[Mốc 1: Lập lệnh chuyển] ──> [Mốc 2: Xuất kho gửi] ───────> [Mốc 3: Nhập kho nhận]
Kho A: Tăng Allocated        Kho A: Giảm On-hand & Allocated Kho B: Tăng On-hand
       Giảm Available        Hệ thống: Tăng In-transit              Giảm On-order
Kho B: Tăng On-order         Kho B: On-hand CHƯA ĐỔI         Hệ thống: Giảm In-transit
```

1. **Mốc 1 - Lập lệnh điều chuyển (Transfer Request Created):**
   - Kiểm tra: $\text{Available}_{A} \ge Q_{\text{transfer}}$.
   - Kho gửi $A$: Tăng $\text{Allocated}_{A} \mathrel{+}= Q_{\text{transfer}} \implies \text{Available}_{A}$ giảm $Q_{\text{transfer}}$.
   - Kho nhận $B$: Ghi nhận hàng đang về: $\text{On-order}_{B} \mathrel{+}= Q_{\text{transfer}}$.
   - *Tồn thực tế $\text{On-hand}$ tại cả hai kho chưa thay đổi.*
   - *Hủy lệnh tại mốc này:* Nếu lệnh chuyển bị hủy khi chưa xuất kho gửi, hệ thống giải phóng $\text{Allocated}_{A} \mathrel{-}= Q_{\text{transfer}}$ ($\text{Available}_{A}$ phục hồi) và trừ $\text{On-order}_{B} \mathrel{-}= Q_{\text{transfer}}$.
2. **Mốc 2 - Xuất kho gửi (Dispatched):**
   - Hàng thực tế rời kho $A$: $\text{On-hand}_{A} \mathrel{-}= Q_{\text{transfer}}$ và $\text{Allocated}_{A} \mathrel{-}= Q_{\text{transfer}}$.
   - Ghi nhận hàng đi đường: $\text{In-transit} \mathrel{+}= Q_{\text{transfer}}$.
   - *Kho nhận $B$: $\text{On-hand}_{B}$ và $\text{Available}_{B}$ vẫn giữ nguyên chưa tăng.*
   - *Lưu ý:* Khi đã Dispatched, không thể bấm "Hủy lệnh" đơn giản. Mọi xử lý phải qua quy trình hoàn hàng hoặc biên bản sự cố.
3. **Mốc 3 - Nhập kho nhận (Received):**
   - **Trường hợp nhận đủ ($Q_{\text{received}} = Q_{\text{transfer}}$):**
     + Kho $B$ tăng tồn thực tế: $\text{On-hand}_{B} \mathrel{+}= Q_{\text{received}}$.
     + Kho $B$ giảm hàng đang về: $\text{On-order}_{B} \mathrel{-}= Q_{\text{transfer}}$.
     + Hệ thống xóa trạng thái đi đường: $\text{In-transit} \mathrel{-}= Q_{\text{transfer}}$.
   - **Trường hợp nhận thiếu / hư hỏng / hao hụt ($Q_{\text{received}} < Q_{\text{transfer}}$):**
     + Kho $B$ chỉ tăng tồn thực tế đúng số lượng thực nhận: $\text{On-hand}_{B} \mathrel{+}= Q_{\text{received}}$.
     + Xóa toàn bộ số lượng lệnh khỏi $\text{On-order}_{B}$ và $\text{In-transit}$ ($\mathrel{-}= Q_{\text{transfer}}$), đóng lệnh chuyển hoàn toàn, không để treo số dư đi đường.
     + Ghi nhận phần chênh lệch thiếu ($\Delta Q = Q_{\text{transfer}} - Q_{\text{received}}$) vào Sổ theo dõi sai lệch/hao hụt (`Discrepancy Log`) kèm mã tham chiếu và lý do đối soát.

---

### 3.3. Các kịch bản biên và kiểm soát giao dịch nâng cao

1. **Xử lý tranh chấp xuất kho đồng thời (Concurrency Control):**
   - Áp dụng cơ chế khóa dữ liệu tại tầng cơ sở dữ liệu (`Pessimistic Locking` với `SELECT ... FOR UPDATE` hoặc `Optimistic Locking` với trường `@Version`) trên bản ghi số dư tồn kho của SKU tại kho tương ứng.
   - Khi có nhiều yêu cầu xuất/giữ chỗ đồng thời, các giao dịch phải được tuần tự hóa (serialized) để kiểm tra $\text{Available}$. Tuyệt đối không để xảy ra hiện tượng "race condition" làm tồn khả dụng bị âm.
2. **Xử lý hủy giao dịch (Transaction Cancellation):**
   - Lệnh xuất hoặc lệnh chuyển kho chỉ được phép hủy khi ở trạng thái **Chờ xử lý / Đã giữ chỗ** (chưa xuất kho vật lý).
   - Khi hủy hợp lệ, hệ thống hoàn trả lại tồn khả dụng: $\text{Allocated} \mathrel{-}= Q_{\text{cancel}} \implies \text{Available} \mathrel{+}= Q_{\text{cancel}}$, đồng thời giảm $\text{On-order}$ tương ứng ở kho nhận nếu là lệnh điều chuyển.
   - Khi hàng đã xuất kho (đang $\text{In-transit}$ hoặc đã giao hàng), nút hủy bị vô hiệu hóa; mọi điều chỉnh phải thực hiện qua quy trình nhập trả hoặc lệnh điều chuyển ngược lại.
3. **Chống ghi nhận giao dịch lặp (Idempotency Control):**
   - Mọi request API xác nhận giao dịch (nhập, xuất, chuyển) bắt buộc gửi kèm một `idempotency_key` duy nhất.
   - Tầng cơ sở dữ liệu cài đặt Unique Constraint trên mã khóa này kết hợp kiểm tra trạng thái máy trạng thái (State Machine). Nếu nhận request trùng key, hệ thống trả về kết quả của lần xử lý trước đó mà không thực hiện cộng/trừ số dư lần thứ hai.

---

## 4. Cơ chế AI Dự báo và Động cơ Hỗ trợ Quyết định Tồn kho

### 4.1. Bản chất dữ liệu và Quy trình kết hợp AI
AI trong hệ thống đóng vai trò cung cấp dữ liệu định lượng, không phải là "hộp đen" tự động ra quyết định. Quy trình 4 bước kết hợp chặt chẽ:

```
[1. Dữ liệu xuất kho giả lập]
    (SKU · Kho · Thời gian · Số lượng tiêu thụ)
                 │
                 ▼
[2. Mô hình dự báo nhu cầu]
    (Baseline & Mô hình ứng viên · Đánh giá chia theo thời gian)
                 │  Nhu cầu dự báo kỳ vọng (D_hat)
                 ▼
[3. Động cơ quy tắc tồn kho]
    (Lead time · Safety Stock · ROP · Inventory Position)
                 │  Số lượng đề xuất bổ sung (Q_suggest) & Lý do
                 ▼
[4. Khuyến nghị nhập hoặc điều chuyển]
    (Hiển thị trực quan trên Web UI để con người xem xét & phê duyệt)
```

---

### 4.2. Chiến lược Dữ liệu tổng hợp (Synthetic Data Generator)
Do chưa có dữ liệu thực tế từ doanh nghiệp, toàn bộ dữ liệu tiêu thụ lịch sử sẽ được sinh bằng script có kiểm soát:
- **Cấu trúc bản ghi:** Chuỗi thời gian theo định dạng `(sku_id, warehouse_id, date, demand_quantity)`.
- **Đơn vị thời gian (Granularity):** Theo ngày (Daily).
- **Quy luật sinh dữ liệu:** Kết hợp có chủ đích các thành phần:
  $$D_t = \text{BaseLevel} + \text{Trend}(t) + \text{WeeklySeasonality}(t) + \text{AnnualSeasonality}(t) + \epsilon_t$$
  Bao gồm các kịch bản thực tế: nhu cầu ngắt quãng có nhiều ngày bằng 0 (intermittent demand), ngày có đột biến tiêu thụ (promotional spike), và nhiễu ngẫu nhiên Gaussian $\epsilon_t$.
- **Khả năng tái lập:** Cố định tham số ngẫu nhiên (`random_seed`) để đảm bảo dữ liệu sinh ra hoàn toàn tái lập được khi chạy lại kiểm thử.
- **Ranh giới trung thực:** Toàn bộ kết quả thử nghiệm được gắn nhãn rõ là **thực nghiệm trên môi trường giả lập**, không tuyên bố là hiệu quả trên dữ liệu sản xuất của doanh nghiệp thực tế.

---

### 4.3. Pipeline Huấn luyện, Đánh giá và So sánh Baseline

1. **Phân chia tập dữ liệu theo thời gian (Temporal Split):**
   - Tuyệt đối không xáo trộn ngẫu nhiên (no random shuffle) để ngăn chặn rò rỉ thông tin tương lai về quá khứ (Data Leakage).
   - Chia tập dữ liệu tuần tự theo trục thời gian: Tập Huấn luyện (Train set: 70–80% thời gian đầu) $\rightarrow$ Tập Đánh giá (Validation set: 10–15% tiếp theo) $\rightarrow$ Tập Kiểm thử (Test set: 10–15% thời gian cuối cùng).
2. **Khoảng dự báo (Forecast Horizon):** Dự báo trước từ 7 đến 14 ngày tới (theo ngày), phù hợp với chu kỳ bổ sung hàng của doanh nghiệp vừa và nhỏ.
3. **So sánh Baseline bắt buộc:**
   - Không mặc định bất kỳ mô hình học máy nào (như LightGBM, Random Forest, hay ARIMA) chắc chắn là tốt nhất trước khi có kết quả thực nghiệm.
   - Bắt buộc so sánh mô hình ứng viên với ít nhất 01 mô hình baseline truyền thống trên cùng tập kiểm thử và cùng khoảng dự báo:
     + Mô hình Ngây thơ (Naive Forecast: lấy giá trị cùng kỳ chu kỳ trước).
     + Mô hình Trung bình trượt (Rolling Moving Average 7 ngày / 14 ngày).
4. **Tách biệt chỉ số Đánh giá Dự báo vs chỉ số Vận hành Tồn kho:**
   - **Đánh giá sai số dự báo (Forecast Error Metrics):**
     + $\text{MAE}$ (Mean Absolute Error) và $\text{RMSE}$ (Root Mean Squared Error).
     + $\text{WAPE}$ (Weighted Absolute Percentage Error):
       $$\text{WAPE} = \frac{\sum |y_t - \hat{y}_t|}{\sum y_t}$$
       *(Sử dụng WAPE thay cho MAPE để tránh lỗi chia cho 0 khi nhu cầu bằng 0).*
   - **Đánh giá hiệu quả vận hành tồn kho (Inventory Operational Metrics):** Đánh giá qua bài toán mô phỏng (Simulation) chính sách tồn kho trên tập test:
     + Tỷ lệ hết hàng (Stockout Rate): Phần trăm ngày hoặc đơn bị thiếu hàng.
     + Tỷ lệ đáp ứng đơn hàng (Service Level / Fill Rate).
     + Mức tồn kho bình quân (Average Inventory Holding): Đo lường lượng vốn ứ đọng.

---

### 4.4. Công thức Động cơ Bổ sung Tồn kho Định lượng

Để khắc phục lỗi đề xuất đặt thêm hàng khi đã có hàng đang trên đường về, hệ thống chuẩn hóa 4 công thức:

1. **Vị thế tồn kho (Inventory Position - IP):** Tổng lượng hàng sẵn có và hàng đang trên đường về dùng để ra quyết định:
   $$\text{IP} = \text{Available} + \text{On-order} = (\text{On-hand} - \text{Allocated}) + \text{On-order}$$
2. **Điểm đặt hàng lại (Reorder Point - ROP):** Ngưỡng tồn kho kích hoạt việc xem xét bổ sung:
   $$\text{ROP} = \hat{D}_{\text{lead}} + \text{SS}$$
   - $\hat{D}_{\text{lead}}$: Nhu cầu tiêu thụ dự báo trong thời gian giao hàng (Lead time demand). Với dự báo theo ngày, $\hat{D}_{\text{lead}} = \sum_{k=1}^{L} \hat{d}_{t+k}$ (tổng các giá trị dự báo ngày từ ngày $t+1$ đến $t+L$), hoặc xấp xỉ bằng $(\bar{d} \times L)$ nếu lấy nhu cầu trung bình ngày $\bar{d}$ nhân với Lead time.
   - $L$: Thời gian bổ sung hàng (Lead time, tính bằng ngày). Trong Core MVP giả định là hằng số đã biết cho từng nhà cung cấp / tuyến điều chuyển.
   - $\text{SS}$: Mức tồn kho an toàn (Safety Stock).
3. **Tồn kho an toàn (Safety Stock - SS):**
   $$\text{SS} = Z \times \sigma_{D} \times \sqrt{L}$$
   - $Z$: Hệ số dịch vụ theo mức độ phục vụ mong muốn (ví dụ: $Z = 1.65$ cho mức phục vụ 95%).
   - $\sigma_{D}$: Độ lệch chuẩn sai số nhu cầu (ước lượng từ độ lệch chuẩn nhu cầu lịch sử hoặc sai số dự báo của mô hình).
4. **Lượng hàng đề xuất bổ sung ($Q_{\text{suggest}}$ - Replenishment Quantity):**
   - **Điều kiện kích hoạt:** Chỉ sinh đề xuất khi vị thế tồn kho chạm hoặc thấp hơn ROP:
     $$\text{IP} \le \text{ROP}$$
   - **Số lượng đề xuất:** Tính theo mức tồn kho mục tiêu tối đa $S$ (Order-up-to level):
     $$Q_{\text{suggest}} = \max(0, S - \text{IP})$$
     *(Nếu áp dụng số lượng đặt hàng tối thiểu MOQ: $Q_{\text{suggest}} = \max(\text{MOQ}, S - \text{IP})$).*

### 4.5. Cơ chế xử lý khi dữ liệu chưa đủ (Cold-start / Fallback Policy)
Đối với các SKU mới thêm vào hệ thống chưa đủ dữ liệu chuỗi thời gian lịch sử (dưới 30 ngày ghi nhận tiêu thụ) để mô hình AI huấn luyện hoặc suy luận tin cậy:
- Hệ thống kích hoạt cơ chế **Dự phòng quy tắc tĩnh (Fallback to Static Policy)**: Sử dụng các tham số $ROP$ và $SS$ cấu hình thủ công trong danh mục sản phẩm thay vì chờ mô hình AI.
- Hệ thống hiển thị nhãn cảnh báo trên UI: *"Đề xuất tính theo tham số tĩnh do SKU chưa đủ lịch sử dữ liệu cho AI"*.

---

## 5. Quy tắc Phân bổ Đơn hàng và Điều chuyển Đa kho trong MVP

Để bảo đảm tính khả thi và có thể kiểm thử độc lập trong 6 tháng, hệ thống tránh đưa các thuật toán tối ưu tuyến tính phức tạp vào Core MVP khi chưa có đủ dữ liệu chi phí vận tải thực tế:

### 5.1. Quy tắc Phân bổ Đơn hàng Core MVP (Order Allocation Rule)
1. **Nguyên tắc trọn vẹn (Single-Warehouse Fulfillment):** Trong Core MVP, mỗi đơn hàng được xuất trọn vẹn từ **01 kho duy nhất**. Hệ thống **chưa tự động tách đơn thành nhiều kiện xuất từ nhiều kho**.
2. **Thuật toán chọn kho xuất:**
   - Bước 1: Lọc danh sách các kho có $\text{Available}_{w, i} \ge Q_{\text{order}, i}$ với toàn bộ các mặt hàng $i$ trong đơn.
   - Bước 2: Nếu có nhiều kho thỏa mãn, chọn kho đầu tiên theo **Thứ tự ưu tiên kho được cấu hình sẵn** (Configured Warehouse Priority list: $W_1 > W_2 > ...$).
   - Bước 3: **Xử lý khi không kho nào đủ hàng:** Hệ thống từ chối phân bổ tự động, chuyển trạng thái đơn hàng thành **"Không thể tự động phân bổ" (Unfulfillable / Pending Manual Review)** và sinh **Cảnh báo thiếu hàng (Stockout Alert)** hiển thị trên màn hình điều phối để nhân viên xử lý thủ công (chọn kho xuất thiếu hoặc tạo lệnh điều chuyển bổ sung trước).
3. **Cơ chế tách đơn đa kho:** Được định vị rõ là tính năng của giai đoạn **Advanced MVP**, chỉ thực hiện khi luồng đơn kho đã hoàn toàn ổn định.

### 5.2. Quy tắc Điều chuyển Cân đối Tồn kho (Stock Balancing Transfer Rule)
- Trong Core MVP, lệnh điều chuyển được tạo thủ công hoặc bán tự động theo quy tắc ngưỡng:
  - Khi một kho vệ tinh ($W_{\text{RDC}}$) có vị thế tồn kho $\text{IP} \le \text{ROP}$, hệ thống gợi ý chuyển hàng từ Kho trung tâm ($W_{\text{CDC}}$) nếu $W_{\text{CDC}}$ thỏa mãn điều kiện tồn an toàn:
    $$\text{Available}_{\text{CDC}} > \text{ROP}_{\text{CDC}} + Q_{\text{transfer}}$$
- Việc giải bài toán tối ưu luân chuyển mạng lưới đa điểm (Multi-echelon inventory optimization) bằng quy hoạch tuyến tính được xếp vào nhóm Advanced MVP / Nice-to-have.

---

## 6. Phân chia Phạm vi Dự án (Kế hoạch 6 tháng cho 1 sinh viên)

Khối lượng công việc được phân tầng thành 3 cấp độ rõ ràng nhằm tránh rủi ro vỡ tiến độ nhưng vẫn giữ trọn vẹn chiều sâu kỹ thuật:

```
┌─────────────────────────────────────────────────────────────────┐
│                     NICE-TO-HAVE (Mở rộng)                      │
│   Vị trí Bin/Kệ · Lô hàng/Hạn sử dụng FEFO · Quét mã Barcode    │
├─────────────────────────────────────────────────────────────────┤
│                     ADVANCED MVP (Giai đoạn 2)                  │
│   Tách đơn đa kho · Tối ưu điều chuyển CDC-RDC · Mô phỏng tồn   │
├─────────────────────────────────────────────────────────────────┤
│                     CORE MVP (Bắt buộc hoàn thành)              │
│   Quản lý 5 trạng thái tồn · Giao dịch ACID · Audit Log         │
│   Pipeline AI chuỗi thời gian & Baseline · Động cơ ROP/SS/Q     │
│   Phân bổ Single-Warehouse · Điều chuyển có In-transit · Web UI │
└─────────────────────────────────────────────────────────────────┘
```

### 6.1. Core MVP (Bắt buộc phải hoàn thành và kiểm thử được)
Tập trung xây dựng hệ thống nền tảng vững chắc, có chiều sâu về mặt giao dịch và khoa học về mặt AI:
1. **Danh mục cơ bản:** Quản lý sản phẩm (SKU, tên, đơn vị tính, nhóm hàng), đối tác (nhà cung cấp, khách hàng/điểm nhận), danh mục kho ở **cấp Tổng kho (Warehouse level)**.
2. **Quản lý tồn kho 5 trạng thái:** Quản lý số dư `On-hand`, `Allocated`, `Available`, `On-order`, `In-transit` theo từng `(SKU, Warehouse)`; khóa giao dịch chống tồn âm tuyệt đối.
3. **Giao dịch nhập / xuất kho cốt lõi:** Phiếu nhập kho (GRN) cập nhật On-hand và trừ On-order (hỗ trợ nhập đủ hoặc một phần); phiếu xuất kho (GDN) kiểm tra Available, giữ chỗ Allocated và trừ On-hand khi xuất.
4. **Điều chuyển kho nội bộ có In-transit:** Lập lệnh $\rightarrow$ Xuất kho gửi $\rightarrow$ Nhập kho nhận; xử lý nhận đủ, nhận thiếu và ghi nhận hao hụt.
5. **Lịch sử giao dịch & Kiểm toán:** Ghi nhận Stock Movement Audit Log cho 100% biến động; chống ghi nhận giao dịch lặp qua `idempotency_key`.
6. **Pipeline Dữ liệu & AI Dự báo:** Script sinh synthetic data có seed; dịch vụ Python huấn luyện mô hình dự báo ngắn hạn theo ngày; so sánh với Rolling Moving Average baseline bằng WAPE/MAE; API suy luận dự báo.
7. **Động cơ Hỗ trợ quyết định tồn kho:** Tính toán ROP, SS, IP; sinh cảnh báo và gợi ý số lượng $Q_{\text{suggest}}$ khi $\text{IP} \le \text{ROP}$; có cơ chế fallback khi thiếu dữ liệu (Human-in-the-loop).
8. **Phân bổ đơn hàng Core MVP:** Tự động gán kho xuất trọn gói cho đơn hàng theo thứ tự ưu tiên cấu hình sẵn (không tách đơn; báo thiếu hàng khi không kho nào đủ).
9. **Giao diện Web tác nghiệp:** Một ứng dụng web duy nhất cho các luồng: Xem tồn kho 5 trạng thái theo thời gian thực, Thao tác nhập/xuất/chuyển, Xem biểu đồ dự báo & Phê duyệt đề xuất.

### 6.2. Advanced MVP (Chỉ thực hiện sau khi Core MVP đã ổn định)
Các chức năng nâng cao nhằm gia tăng chiều sâu bài toán phân phối và tồn kho:
1. **Cơ chế tách đơn hàng đa kho (Multi-warehouse Order Splitting):** Tự động tách đơn thành nhiều kiện hàng xuất từ nhiều kho khi không có kho nào đủ hàng trọn vẹn.
2. **Đánh giá mô phỏng chính sách tồn kho (Inventory Simulation):** Viết script mô phỏng vận hành trên tập test để đo lường Service Level và Stockout Rate thực tế khi áp dụng gợi ý của AI so với chính sách tồn kho tĩnh.
3. **Tự động sinh đề xuất lệnh điều chuyển CDC $\rightarrow$ RDC:** Động cơ quét định kỳ và gợi ý lệnh luân chuyển cân bằng giữa kho tổng và kho vệ tinh.
4. **Phân loại hàng hóa ABC/XYZ:** Tự động phân nhóm SKU để gán hệ số an toàn ($Z$) và chính sách kiểm tra phù hợp.

### 6.3. Nice-to-have (Chỉ làm nếu còn dư thời gian)
1. Quản lý vị trí chi tiết trong kho (Zone / Rack / Shelf / Bin).
2. Quản lý hạn sử dụng và số lô hàng (Lot/Batch & Expiry Date) theo nguyên tắc FEFO.
3. Thuật toán tối ưu hóa điều chuyển đa kho bằng quy hoạch tuyến tính (Linear Programming).
4. Tích hợp quét mã vạch (Barcode/QR code) qua camera trên web mobile.

### 6.4. Chức năng nằm ngoài phạm vi đồ án (Out-of-Scope)
1. **Logistics vận tải ngoài kho:** Tối ưu hóa tuyến đường xe (Vehicle Routing Problem - VRP), điều hướng GPS phương tiện thời gian thực, quản lý đội tài xế và xếp hàng vào thùng xe (3D Container Loading).
2. **Kế toán tài chính chuyên sâu:** Bảng cân đối kế toán, định giá tài sản cố định, hạch toán dòng tiền và chi phí thuế.
3. **Thương mại điện tử bán lẻ:** Sàn thương mại điện tử, giỏ hàng người tiêu dùng cuối, cổng thanh toán trực tuyến.
4. **Quản lý sản xuất phức tạp (MES):** Lập lịch máy móc gia công, định mức nguyên phụ liệu nhiều cấp.

---

## 7. Tiêu chí Nghiệm thu theo Ca kiểm thử Đo lường được (Acceptance Test Cases)

Hệ thống được nghiệm thu bằng các tình huống kiểm thử cụ thể có điều kiện đầu vào, hành động thực hiện, kết quả mong đợi và phương pháp kiểm chứng rõ ràng:

### Ca kiểm thử 1: Kiểm soát xuất kho khi không đủ tồn khả dụng (Ngăn chặn tồn âm)
- **Điều kiện đầu vào (Pre-conditions):** SKU `SP01` tại Kho A có $\text{On-hand} = 10$, $\text{Allocated} = 2 \implies \text{Available} = 8$.
- **Hành động thực hiện (Action):** Client gửi yêu cầu xuất đơn hàng với số lượng yêu cầu là 10 đơn vị.
- **Kết quả mong đợi (Expected Output):** Hệ thống từ chối giao dịch, trả về mã lỗi HTTP 400 (`INSUFFICIENT_STOCK`) kèm thông điệp rõ ràng (Hiện có: 8, Yêu cầu: 10). Số dư tồn kho giữ nguyên ($\text{On-hand} = 10, \text{Allocated} = 2, \text{Available} = 8$), không tạo phiếu xuất kho GDN.
- **Phương pháp kiểm chứng (Verification):** Kiểm tra HTTP Response status code và truy vấn trực tiếp bảng số dư tồn kho trong cơ sở dữ liệu PostgreSQL.

### Ca kiểm thử 2: Kiểm soát giao dịch xuất kho đồng thời (Concurrency Test)
- **Điều kiện đầu vào:** SKU `SP02` tại Kho B có $\text{Available} = 5$.
- **Hành động thực hiện:** Hai tiến trình gửi đồng thời 2 yêu cầu xuất hàng (mỗi yêu cầu lấy 4 đơn vị) tại cùng một thời điểm (chênh lệch dưới 10ms).
- **Kết quả mong đợi:** Đúng 01 yêu cầu thành công (HTTP 200, $\text{Available}$ giảm từ 5 về 1); yêu cầu còn lại bị từ chối do không đủ tồn kho (HTTP 400). Tuyệt đối không để cả 2 yêu cầu cùng thành công làm $\text{Available} = -3$.
- **Phương pháp kiểm chứng:** Chạy integration test script đa luồng (multi-threaded test) gửi 2 request song song và kiểm tra số dư cuối cùng trong DB.

### Ca kiểm thử 3: Điều chuyển kho có sai lệch nhận thiếu (Under-receiving)
- **Điều kiện đầu vào:** Kho gửi A tạo lệnh chuyển 10 đơn vị sang Kho nhận B. Hàng đã thực xuất khỏi kho A.
- **Hành động thực hiện:** Nhân viên Kho B kiểm đếm và gửi xác nhận chỉ thực nhận được 9 đơn vị (thiếu 1 đơn vị do hư hỏng).
- **Kết quả mong đợi:**
  - Kho A: $\text{On-hand}_A$ đã giảm 10. Kho B: $\text{On-hand}_B$ tăng đúng 9 đơn vị.
  - Hệ thống: Trừ toàn bộ 10 đơn vị khỏi $\text{In-transit}$ và $\text{On-order}_B$ (đóng lệnh chuyển).
  - Bản ghi chênh lệch: Hệ thống tự động ghi 01 bản ghi vào `Discrepancy Log` với số lượng thiếu = 1, mã tham chiếu và lý do.
- **Phương pháp kiểm chứng:** Kiểm tra số dư tại cả 2 kho, trạng thái lệnh chuyển (Completed with Discrepancy) và truy vấn bảng `discrepancy_log`.

### Ca kiểm thử 4: Nhập kho một phần từ Nhà cung cấp (Partial Inbound Receipt)
- **Điều kiện đầu vào:** Đơn mua PO có số lượng đặt là 100 đơn vị SKU `SP04` về Kho A ($\text{On-order} = 100$).
- **Hành động thực hiện:** Nhà cung cấp giao đợt 1 với số lượng 60 đơn vị. Nhân viên kho tạo phiếu GRN xác nhận nhận 60 đơn vị.
- **Kết quả mong đợi:**
  - $\text{On-hand}_A$ tăng đúng 60 đơn vị; $\text{Available}_A$ tăng tương ứng 60.
  - $\text{On-order}_A$ giảm từ 100 xuống còn 40 đơn vị.
  - Đơn PO chuyển trạng thái sang "Nhận một phần" (Partially Received), không bị đóng vội vã.
- **Phương pháp kiểm chứng:** Kiểm tra số dư tồn kho Kho A và trạng thái của PO trong DB.

### Ca kiểm thử 5: Chống ghi nhận giao dịch lặp (Idempotency Test)
- **Điều kiện đầu vào:** Client chuẩn bị gửi request xác nhận nhập kho kèm mã `idempotency_key = "GRN-20261016-001"`.
- **Hành động thực hiện:** Client gửi request lần 1 thành công; sau đó gửi lại chính xác request đó một lần nữa với cùng `idempotency_key`.
- **Kết quả mong đợi:** Request thứ hai trả về kết quả thành công của lần xử lý trước đó mà không làm tăng thêm số dư tồn kho; $\text{On-hand}$ chỉ tăng đúng một lần duy nhất.
- **Phương pháp kiểm chứng:** Kiểm tra mã trạng thái trả về của request 2 và kiểm tra số dư $\text{On-hand}$ không bị cộng 2 lần.

### Ca kiểm thử 6: Sinh cảnh báo và tính toán số lượng đề xuất bổ sung chính xác với IP
- **Điều kiện đầu vào:** SKU `SP03` tại Kho A có $\text{Available} = 5$, $\text{On-order} = 10 \implies \text{IP} = 15$. Tham số: Lead time $L = 3$ ngày, $SS = 10$, mức tồn tối đa $S = 50$. AI dự báo nhu cầu trung bình $\hat{D} = 5$ đơn vị/ngày $\implies \text{ROP} = (5 \times 3) + 10 = 25$.
- **Hành động thực hiện:** Trigger batch job hoặc gọi API quét và tính toán đề xuất bổ sung tồn kho.
- **Kết quả mong đợi:**
  - Hệ thống kích hoạt cảnh báo thiếu hàng do $\text{IP} (15) \le \text{ROP} (25)$.
  - Số lượng đề xuất bổ sung được tính đúng bằng: $Q_{\text{suggest}} = S - \text{IP} = 50 - 15 = 35$ đơn vị (không bị đề xuất thừa 10 đơn vị đang về nếu tính theo $S - \text{Available}$).
- **Phương pháp kiểm chứng:** Kiểm tra danh sách đề xuất hiển thị trên Web UI và dữ liệu trả về từ API `/replenishment-suggestions`.

### Ca kiểm thử 7: Đánh giá mô hình AI trên tập Test chia theo thời gian
- **Điều kiện đầu vào:** Tập dữ liệu chuỗi thời gian giả lập được chia theo trục thời gian (80% Train, 20% Test).
- **Hành động thực hiện:** Thực thi script đánh giá độc lập của dịch vụ AI (`python evaluate.py`).
- **Kết quả mong đợi:** Script chạy hoàn tất không lỗi, xuất bảng báo cáo so sánh chỉ số sai số $\text{WAPE}$ và $\text{MAE}$ giữa mô hình AI ứng viên và mô hình Rolling Moving Average trên cùng tập Test; xuất biểu đồ trực quan hóa.
- **Phương pháp kiểm chứng:** Đọc file log đánh giá và file báo cáo kết quả đánh giá thực tế.

### Ca kiểm thử 8: Luồng nghiệp vụ đầu-cuối Core MVP (End-to-End Workflow)
- **Điều kiện đầu vào:** Hệ sinh thái gồm 1 Kho trung tâm CDC và 1 Kho vệ tinh RDC.
- **Hành động thực hiện:** Thực thi chuỗi tác nghiệp liên hoàn:
  1. Nhập hàng vào CDC (PO $\rightarrow$ GRN $\rightarrow$ On-hand CDC tăng).
  2. Bán hàng xuất từ RDC $\rightarrow$ GDN hoàn tất $\rightarrow$ Tồn RDC giảm chạm ngưỡng ROP.
  3. Hệ thống kích hoạt cảnh báo thiếu hàng tại RDC và gợi ý điều chuyển từ CDC sang RDC.
  4. Người dùng bấm phê duyệt đề xuất $\rightarrow$ Lập lệnh chuyển $\rightarrow$ Xuất CDC (In-transit) $\rightarrow$ Nhập RDC (Available RDC phục hồi).
- **Kết quả mong đợi:** 100% các bước chuyển trạng thái thành công, số dư 5 trạng thái tại cả 2 kho phản ánh chính xác sau mỗi bước; phát sinh đầy đủ audit log cho từng giao dịch.
- **Phương pháp kiểm chứng:** Kiểm tra trạng thái giao dịch và số dư tồn kho qua từng màn hình tác nghiệp trên Web UI.

---

## 8. Kiến trúc Hệ thống Định hướng và Giả định Kỹ thuật

### 8.1. Định hướng Công nghệ và Phân tách Dịch vụ
Để bảo đảm tính khả thi cho 1 sinh viên trong 6 tháng mà vẫn giữ được tính chuẩn mực công nghệ:
- **Backend Service (Cốt lõi):** Định hướng sử dụng **Java 21 và Spring Boot (3.x)**. Đảm nhiệm toàn bộ logic nghiệp vụ kho, kiểm soát giao dịch ACID, phân quyền người dùng, lưu vết kiểm toán và quản lý dữ liệu danh mục.
- **Cơ sở dữ liệu:** Định hướng sử dụng **PostgreSQL**. Đảm bảo tính nhất quán dữ liệu, hỗ trợ khóa bi quan (`SELECT FOR UPDATE`), ràng buộc toàn vẹn và lưu trữ lịch sử giao dịch.
- **AI Service (Phân tích & Dự báo):** Định hướng sử dụng **Python (FastAPI hoặc Flask)**. Đảm nhiệm các tác vụ khoa học dữ liệu: sinh dữ liệu tổng hợp, huấn luyện mô hình dự báo chuỗi thời gian, tính toán sai số đánh giá và cung cấp API suy luận cho Backend.
- **Frontend Web:** Định hướng xây dựng ứng dụng web hiện đại (SPA) phục vụ người dùng tác nghiệp và theo dõi dashboard. *Công nghệ cụ thể (như React Vite hoặc Vue Vite) sẽ được chốt chính thức sau khi khảo sát và thiết kế kiến trúc, không tự ý khóa cứng sớm.*
- **Phương thức giao tiếp giữa Backend và AI Service:** Định hướng sử dụng **REST API (JSON qua HTTP)** cho các truy vấn suy luận trực tiếp; kết hợp cơ chế xử lý nền / Cron job cho các tác vụ huấn luyện lại định kỳ. *Sẽ được chốt sau khi thiết kế kiến trúc chi tiết.*

### 8.2. Giả định Kỹ thuật
1. **Dữ liệu thử nghiệm là Dữ liệu tổng hợp:** Mọi kết quả đánh giá mô hình và mô phỏng tồn kho đều dựa trên dữ liệu tổng hợp sinh bằng script có seed; không dùng để khẳng định hiệu quả tài chính thực tế cho doanh nghiệp.
2. **Lead time nhà cung cấp:** Trong phạm vi Core MVP, thời gian giao hàng $L$ của nhà cung cấp và thời gian chuyển hàng giữa các kho được giả định là hằng số đã biết trước (cấu hình trong danh mục).
3. **Môi trường vận hành:** Hệ thống triển khai cục bộ (Local development environment) phục vụ báo cáo và bảo vệ đồ án.

---

## 9. Rủi ro Kỹ thuật và Biện pháp Kiểm soát

| Rủi ro kỹ thuật | Mức độ tác động | Biện pháp kiểm soát cụ thể |
| :--- | :---: | :--- |
| **Quá tải phạm vi (Scope Creep):** Cố gắng triển khai các tính năng tách đơn đa kho hoặc thuật toán tối ưu quá sớm làm chậm tiến độ Core MVP. | Cao | Khóa cứng danh sách chức năng Core MVP ở mục 6.1; chỉ chuyển sang Advanced MVP khi 8 tình huống kiểm thử đã pass 100%. |
| **Tranh chấp dữ liệu tồn kho (Race Condition):** Nhiều giao dịch xuất kho cùng tác động vào một SKU dẫn đến tồn âm. | Cao | Cài đặt khóa bi quan (`SELECT ... FOR UPDATE`) có timeout ngắn tại tầng Repository của Spring Data JPA. |
| **Dữ liệu giả lập không phản ánh đúng thách thức:** Dữ liệu quá "đẹp" hoặc quá ngẫu nhiên khiến mô hình AI không học được hoặc học vẹt. | Trung bình | Kiểm tra phân phối dữ liệu giả lập, trực quan hóa biểu đồ chuỗi thời gian trước khi đưa vào huấn luyện; tích hợp các kịch bản nhiễu và nhu cầu ngắt quãng có chủ đích. |
| **Độ trễ tích hợp giữa Spring Boot và AI Service:** Giao tiếp qua mạng cục bộ gặp lỗi kết nối hoặc timeout làm gián đoạn tác nghiệp kho. | Trung bình | Chuẩn hóa hợp đồng dữ liệu REST API (OpenAPI/Swagger) từ đầu; cho phép Backend hoạt động ở chế độ fallback tĩnh nếu AI Service chưa sẵn sàng. |

---

## 10. Danh mục các Quyết định: Phân định Nội dung Đã chốt và Đề xuất Chờ xác nhận

Nhằm bảo đảm tính minh bạch theo `GEMINI.md`, bảng dưới đây phân tách rành mạch giữa các nội dung đã thống nhất chốt trong tài liệu phạm vi này, các đề xuất kỹ thuật đang chờ người dùng phê duyệt, và các nội dung chưa thể hoàn thiện do thiếu thông tin:

### 10.1. Các quyết định đã chốt trong Phạm vi dự án
- [x] **Ranh giới bài toán:** Không thu hẹp thành CRUD đơn giản; giữ trọn vẹn chiều sâu quản trị đa kho và AI dự báo chuỗi thời gian.
- [x] **Ranh giới logistics ngoài kho:** Loại bỏ hoàn toàn bài toán VRP, định vị GPS xe, và xếp thùng xe khỏi đồ án.
- [x] **Nguyên tắc phân tầng phạm vi:** Tách biệt rõ 3 cấp độ Core MVP, Advanced MVP, Nice-to-have.
- [x] **Nguyên tắc bảo vệ số dư điều chuyển:** Hàng In-transit/On-order tuyệt đối không được cộng vào Available hay On-hand của kho nhận trước khi thực tế kiểm đếm và bấm nhận.
- [x] **Bản chất dữ liệu AI:** Sử dụng dữ liệu tổng hợp có seed; không ngụy tạo kết quả doanh nghiệp thực tế.
- [x] **Quy tắc phân bổ đơn hàng Core MVP:** Xuất trọn gói từ 1 kho theo thứ tự ưu tiên; nếu không kho nào đủ thì từ chối phân bổ tự động và báo thiếu hàng (không tách đơn trong Core MVP).

### 10.2. Các đề xuất kỹ thuật đang chờ Người dùng phê duyệt

| STT | Hạng mục kỹ thuật | Đề xuất mặc định của AI | Lý do đề xuất | Trạng thái |
| :---: | :--- | :--- | :--- | :---: |
| 1 | **Phạm vi quản lý kho Core MVP** | Quản lý ở cấp Tổng kho (Warehouse level), chưa chia vị trí kệ/bin. | Phù hợp tiến độ 6 tháng của 1 sinh viên; tập trung vào logic đa kho. | `[ ] Chờ người dùng xác nhận` |
| 2 | **Chu kỳ và Tầm dự báo AI** | Dự báo theo ngày (Daily), tầm dự báo 7–14 ngày. | Phù hợp với Lead time ngắn (2–5 ngày) của bán lẻ/SME; phản ánh biến động ngày. | `[ ] Chờ người dùng xác nhận` |
| 3 | **Phương thức giao tiếp Backend – AI** | REST API đồng bộ cho suy luận online; kết hợp Cron Job định kỳ chạy batch dự báo. | Dễ cài đặt, dễ debug kết nối giữa Spring Boot và FastAPI. | `[ ] Chờ người dùng xác nhận` |
| 4 | **Lựa chọn Framework Frontend** | Khuyến nghị React (Vite) hoặc Vue (Vite). | Phổ biến, hỗ trợ tốt thư viện biểu đồ trực quan hóa dự báo. | `[ ] Chờ người dùng xác nhận` |
| 5 | **Quy mô kịch bản doanh nghiệp giả lập** | Mô phỏng 2–3 kho hàng (1 CDC, 1–2 RDC) và khoảng 20–50 SKU đại diện. | Đủ phức tạp để chứng minh bài toán phân phối và dự báo mà không quá tải hiệu năng local. | `[ ] Chờ người dùng xác nhận` |

### 10.3. Các nội dung chưa thể hoàn thiện do thiếu thông tin (Cần nghiên cứu và chốt ở tài liệu tiếp theo)
1. **Thông số chi phí thực tế (Holding Cost & Ordering Cost):** Chưa thể đưa công thức Lô đặt hàng kinh tế (EOQ) chính xác vào Core MVP vì chưa có dữ liệu chi phí lưu kho và chi phí đặt hàng thực tế; sẽ được giả định định mức trong tài liệu quy tắc nghiệp vụ `docs/03_business_rules.md`.
2. **Hàm mục tiêu chi phí vận tải đa kho:** Chưa thể xác lập giải thuật tối ưu hóa điều chuyển mạng lưới đa điểm (Multi-echelon optimization) do thiếu ma trận chi phí vận chuyển thực tế giữa các cặp kho; nội dung này được chuyển sang giai đoạn Advanced MVP.
3. **Giới hạn dung tích vật lý của kho bãi (Physical Capacity Limits):** Tạm thời chưa quản lý giới hạn thể tích ($m^3$) và tải trọng ($kg$) tối đa của từng kho trong Core MVP; chỉ quản lý số lượng tồn từng SKU.

---

## 11. Lộ trình Tài liệu Kỹ thuật Tiếp theo

Tuân thủ quy trình phát triển chuẩn tắc trước khi lập trình:
1. **`docs/01_problem_scope.md` (Tài liệu hiện tại):** Xác định bài toán, ranh giới phạm vi Core MVP / Advanced MVP và tiêu chí nghiệm thu.
2. **`docs/02_business_requirements.md`:** Đặc tả chi tiết các tác nhân (actors), ca sử dụng (use cases) và luồng nghiệp vụ chuẩn (workflow diagrams).
3. **`docs/03_business_rules.md`:** Chốt chính thức các quy tắc tính toán tồn kho, điều chuyển, công thức bổ sung và tiêu chuẩn dự báo.
4. **`docs/04_data_dictionary.md`:** Thống nhất danh mục các thực thể, trường dữ liệu, kiểu dữ liệu và mối quan hệ.
5. **Thiết kế kiến trúc & Cơ sở dữ liệu:** Thiết kế ERD, kiến trúc dịch vụ (Spring Boot, Python, DB) sau khi các quy tắc cốt lõi đã hoàn toàn sáng tỏ.
