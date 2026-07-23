# Tổng quan phân hệ Nghỉ phép và Chấm công (Leave & Attendance)

---

> [!NOTE]
> **Phạm vi tham khảo:** Tài liệu này chỉ sử dụng nguồn chính thức của SAP, gồm SAP SuccessFactors, SAP Employee Central, SAP Employee Central Payroll, SAP Fieldglass, SAP Help Portal và các giải pháp SAP liên quan. Thuật ngữ tiếng Anh được giữ trong ngoặc khi cần thiết để hỗ trợ BA/PO đối chiếu với tài liệu cấu hình và triển khai của SAP.


## Mục lục

```text
Tổng quan phân hệ Nghỉ phép và Chấm công (Leave & Attendance)
├── 1. Bối cảnh nghiệp vụ (Domain Context)
│   ├── 1.1. Vị trí trong HRIS
│   ├── 1.2. Vai trò trong vận hành doanh nghiệp
│   └── 1.3. Mối liên hệ trong hệ sinh thái hệ thống
├── 2. Khái niệm nghiệp vụ cốt lõi (Core Business Concepts)
│   ├── 2.1. Bản ghi chấm công (Attendance Record)
│   ├── 2.2. Ca làm việc (Shift)
│   ├── 2.3. Làm thêm giờ (Overtime – OT)
│   ├── 2.4. Loại nghỉ phép (Leave Types)
│   ├── 2.5. Kỳ chốt dữ liệu (Cut-off Period)
│   └── 2.6. Luồng phê duyệt (Approval Workflow)
├── 3. Quy trình đầu-cuối điển hình (Typical End-to-End Process)
├── 4. So sánh chính sách (Policy) theo quy mô doanh nghiệp
├── 5. Các điểm đau phổ biến (Common Pain Points)
├── 6. Quy tắc nghiệp vụ trọng yếu (Key Business Rules)
│   ├── 6.1. Quy tắc khoảng dung sai (Grace Period Rule)
│   ├── 6.2. Quy tắc tính làm thêm giờ (OT Calculation Rule)
│   ├── 6.3. Quy tắc tích lũy và chuyển phép (Leave Accrual & Carry Forward Rule)
│   └── 6.4. Quy tắc chốt kỳ và điều chỉnh (Cut-off & Adjustment Rule)
├── 7. Góc nhìn dữ liệu và tích hợp (Data & Integration Perspective)
│   ├── 7.1. Dữ liệu cốt lõi trong miền nghiệp vụ (domain)
│   ├── 7.2. Logic quan hệ dữ liệu (Data Relationship Logic)
│   ├── 7.3. Luồng chấm công → tính lương (Attendance → Payroll Flow)
│   ├── 7.4. Rủi ro khuếch đại (Error Amplification Effect)
│   └── 7.5. Lưu ý cho BA/PO về dữ liệu và tích hợp
├── 8. Bản đồ phỏng vấn bên liên quan (Stakeholder Interview Mapping)
├── 9. Bảng thuật ngữ chuyên ngành
└── 10. Ghi chú nghiên cứu và nguồn SAP chính thức
```

---

## 1. Bối cảnh nghiệp vụ (Domain Context)

### 1.1. Vị trí trong HRIS
Leave & Attendance Management là một phân hệ cốt lõi trong HRIS (Human Resource Information System) – hệ thống quản lý vòng đời nhân sự trong doanh nghiệp.

Trong cấu trúc HRIS, module này thường nằm trong:
* **Core HR** – nơi quản lý hồ sơ nhân viên, hợp đồng, cấp bậc.
* **lực lượng lao động (workforce) Management** – quản lý nguồn lực lao động theo thời gian thực.
* **Employee Lifecycle Management** – theo dõi toàn bộ hành trình làm việc của nhân viên.

> [!NOTE]
> Nếu Core HR trả lời câu hỏi “Nhân viên là ai?”, thì Leave & Attendance trả lời câu hỏi “Nhân viên đã làm việc bao nhiêu và như thế nào?”

#### Vai trò kiến trúc hệ thống
Leave & Attendance đóng vai trò:
* Là tầng dữ liệu vận hành (Operational Data Layer)
* Là nguồn dữ liệu đầu vào bắt buộc cho Payroll
* Là điểm kiểm soát tuân thủ giờ làm việc

Nó chuyển hóa: **Thời gian → Dữ liệu → Quy tắc kinh doanh → Chi phí lương**

Về mặt hệ thống, đây là module có mức độ phụ thuộc cao (high integration dependency), vì:
1. Nhận dữ liệu từ thiết bị chấm công
2. Áp dụng rule nội bộ
3. Đẩy dữ liệu sang Payroll
4. Lưu trữ phục vụ kiểm toán


#### Tham chiếu giải pháp SAP

| Giải pháp/tài liệu SAP | Phạm vi tham khảo |
| :--- | :--- |
| [SAP SuccessFactors Time Tracking](https://www.sap.com/products/hcm/employee-time-tracking-software.html) | Ghi nhận thời gian, xác thực dữ liệu, phê duyệt bảng chấm công và tích hợp với tính lương. |
| [SAP SuccessFactors Time Management – SAP Help Portal](https://help.sap.com/docs/successfactors-employee-central/using-time-management-in-sap-successfactors/what-is-sap-successfactors-time-management) | Nghỉ phép, số dư thời gian, bảng chấm công và tác vụ tự phục vụ của nhân viên/quản lý. |
| [SAP SuccessFactors Employee Central Payroll](https://www.sap.com/products/hcm/employee-central-payroll.html) | Mối liên hệ giữa dữ liệu thời gian, kỳ chốt và tính lương. |

---

### 1.2. Vai trò trong vận hành doanh nghiệp
Leave & Attendance không chỉ là bài toán “ghi nhận giờ vào – ra”. Nó là cơ chế kiểm soát chi phí lao động và duy trì công bằng nội bộ.

#### Ảnh hưởng đến độ chính xác quỹ lương (Payroll Accuracy)
Payroll không tự tính giờ làm – nó dựa vào Attendance. Nếu Attendance sai:
* Tính thiếu giờ → trả thiếu lương → khiếu nại
* Tính thừa giờ → tăng chi phí không kiểm soát
* Chỉ cần sai lệch nhỏ (ví dụ 15 phút/người/ngày) khi nhân lên toàn công ty có thể tạo sai số tài chính đáng kể mỗi tháng.

#### Kiểm soát chi phí OT và vắng mặt
Overtime là một trong những khoản chi phí biến động lớn nhất. Attendance giúp:
* Phát hiện OT bất thường
* Theo dõi absenteeism (tỷ lệ vắng mặt)
* Phân tích xu hướng tăng ca theo phòng ban

Đây là công cụ hỗ trợ CFO (Chief Financial Officer) và HR trong quản trị chi phí.

#### Duy trì công bằng nội bộ
Nếu hai nhân viên làm cùng số giờ nhưng hệ thống tính khác nhau:
* → Tạo cảm giác bất công
* → Ảnh hưởng tinh thần làm việc
* → Giảm mức độ gắn kết

Minh bạch attendance là nền tảng của niềm tin tổ chức.

#### Tuân thủ luật lao động
Module này đảm bảo:
* Không vượt quá số giờ OT tối đa theo luật
* Đảm bảo số ngày nghỉ phép tối thiểu
* Tính đúng lương ngày lễ, ngày nghỉ

Nếu sai sót:
* Có thể bị xử phạt hành chính
* Bị thanh tra lao động
* Tổn hại uy tín doanh nghiệp

#### Hỗ trợ phân tích và hoạch định nhân sự
Dữ liệu Attendance cho phép:
* Phân tích tỷ lệ đi muộn
* Phân tích OT theo mùa vụ
* Xác định phòng ban thiếu nhân sự
* Hoạch định hoạch định lực lượng lao động (workforce planning)

Ở cấp chiến lược, đây là nguồn dữ liệu cho quyết định mở rộng hoặc tái cấu trúc nhân sự.

---

### 1.3. Mối liên hệ trong hệ sinh thái hệ thống

| miền nghiệp vụ (domain) liên quan | Mối quan hệ nghiệp vụ | Rủi ro nếu sai |
| :--- | :--- | :--- |
| **Timekeeping** | Cung cấp dữ liệu thô (clock-in/out) | Sai timestamp → sai tính công |
| **Payroll** | Nhận dữ liệu tổng hợp (worked days, OT, leave) | Sai lương, sai báo cáo tài chính |
| **Compliance** | Kiểm soát giờ làm và nghỉ phép | Vi phạm luật lao động |
| **hiệu suất (performance)** | Dữ liệu chuyên cần ảnh hưởng đánh giá | Đánh giá thiếu công bằng |
| **kế toán (accounting)** | Ghi nhận chi phí nhân sự | Sai lệch chi phí doanh nghiệp |

> [!TIP]
> **Nhận định:**
> Khi làm việc với module này, BA cần nhận thức rằng:
> * Đây là miền nghiệp vụ (domain) có độ nhạy cảm cao
> * Stakeholder thường có cách hiểu rule khác nhau
> * Policy nội bộ có thể không được văn bản hóa đầy đủ
> * Quy định luật có thể thay đổi theo thời gian

---

## 2. Khái niệm nghiệp vụ cốt lõi (Core Business Concepts)

### 2.1. Bản ghi chấm công (Attendance Record)
Attendance Record là bằng chứng pháp lý và nghiệp vụ ghi nhận sự hiện diện thực tế của nhân viên trong một khoảng thời gian cụ thể.

#### Thành phần dữ liệu cơ bản
Một Attendance Record thường bao gồm:
* Employee ID
* Ngày làm việc
* Timestamp check-in
* Timestamp check-out
* Tổng giờ làm
* Trạng thái (Late / Early Leave / Absent / Present)

#### Điểm quan trọng về nghiệp vụ
Attendance Record không có ý nghĩa độc lập. Ví dụ:
* Check-in 08:10 có phải đi muộn không?
* Check-out 17:30 có được tính OT không?

Câu trả lời phụ thuộc vào:
* Shift được gán
* Grace period
* Policy nội bộ

Vì vậy, Attendance Record là dữ liệu thô cần được “diễn giải” thông qua bộ máy quy tắc (rule engine).

#### Rủi ro phổ biến
* Sai timestamp do thiết bị
* Nhân viên quên chấm công
* Sai múi giờ (multi-location)
* Double check-in/out

Nếu không có cơ chế ngoại lệ (exception) handling tốt → payroll sai hàng loạt.

---

### 2.2. Ca làm việc (Shift)
Shift là khung tham chiếu để hệ thống hiểu cách diễn giải Attendance. Nó định nghĩa tiêu chuẩn làm việc cho một nhóm nhân viên hoặc vị trí.

#### Thành phần cấu hình
Một shift thường bao gồm:
* Giờ bắt đầu – kết thúc
* Thời gian nghỉ giữa ca
* Grace period (khoảng thời gian được phép trễ)
* Cách tính OT
* Phụ cấp ca (đêm, nguy hiểm, lễ)

#### Vai trò quản trị
Shift giúp:
* Chuẩn hóa giờ làm
* Kiểm soát OT
* Phân bổ nhân sự theo thời gian
* Tối ưu năng suất

Trong doanh nghiệp sản xuất hoặc bán lẻ, shift là yếu tố quyết định cấu trúc vận hành.

#### Complexity Drivers
Complexity Drivers là các yếu tố làm gia tăng độ phức tạp của một hệ thống, sản phẩm hoặc dự án — khiến việc phân tích, thiết kế, triển khai và vận hành trở nên khó kiểm soát hơn.

Độ phức tạp tăng mạnh khi:
* Nhân viên xoay ca (rotating shift)
* Một người có nhiều shift trong cùng kỳ lương
* Có multi-location
* Có remote/hybrid policy

> [!WARNING]
> Sai ánh xạ (mapping) shift → sai toàn bộ tính công.

---

### 2.3. Làm thêm giờ (Overtime – OT)
Overtime là thời gian làm việc vượt quá giờ tiêu chuẩn theo: Luật lao động, Hợp đồng lao động, Nội quy công ty. OT luôn gắn với hệ số trả lương cao hơn giờ bình thường.

#### Từ góc độ quản trị
OT là:
* Khoản chi phí biến động lớn
* Dễ bị lạm dụng
* Yếu tố nhạy cảm trong thanh tra lao động

#### Các yếu tố BA cần làm rõ khi elicitation
* **Điều kiện phát sinh OT:** Có cần phê duyệt trước không?; Có tự động tính nếu check-out muộn không?
* **Cách tính:** Theo giờ, ngày hay theo tuần?; Làm tròn theo block 15 phút / 30 phút?
* **Hệ số thay đổi theo:** Ngày thường, Cuối tuần, Ngày lễ, Ca đêm
* **Giới hạn pháp lý:** Tối đa giờ/ngày/tháng/năm

Nếu không kiểm soát tốt → rủi ro tài chính và pháp lý cao.

---

### 2.4. Loại nghỉ phép (Leave Types)
Leave là quyền lợi hoặc nghĩa vụ nghỉ làm được quy định bởi: Luật lao động, Chính sách công ty, Thỏa ước lao động. Mỗi loại leave có logic riêng về:
* Điều kiện sử dụng
* Cách tích lũy
* Ảnh hưởng lương
* Ảnh hưởng bảo hiểm

#### Phân loại phổ biến
| Loại | Logic nghiệp vụ chính |
| :--- | :--- |
| **Annual Leave** | Tích lũy theo thời gian làm việc |
| **Sick Leave** | Có thể yêu cầu chứng từ |
| **nghỉ không lương (unpaid leave)** | Trừ lương trực tiếp |
| **Maternity** | Liên quan BHXH |
| **Compensatory** | Sinh ra từ OT |

#### Các yếu tố phức tạp
* Leave tích lũy/trích trước (accrual) (nghỉ tích lũy) theo tháng hay năm
* Seniority (thâm niên làm việc) có ảnh hưởng không
* Carry forward (Chuyển số ngày phép chưa sử dụng của năm nay sang năm sau) tối đa bao nhiêu ngày
* Có auto-expire (Ngày phép chuyển sang có tự động hết hạn không) không
* Leave theo cấp bậc khác nhau

> [!IMPORTANT]
> Sai logic leave có thể: Vi phạm luật, Sai nghĩa vụ tài chính, Dẫn đến kiện tụng.

---

### 2.5. Kỳ chốt dữ liệu (Cut-off Period)
chốt dữ liệu (cut-off) là thời điểm hệ thống đóng băng dữ liệu attendance để chuyển sang Payroll.

Sau thời điểm này:
* Không được chỉnh sửa tự do
* Mọi thay đổi xử lý ở kỳ sau

#### Vì sao đây là điểm nhạy cảm nhất?
* Payroll phụ thuộc hoàn toàn vào dữ liệu đã khóa
* quản lý (manager) thường duyệt trễ
* HR cần thời gian đối soát (reconciliation) (đối soát)
* Sai sót sau chốt dữ liệu (cut-off) gây hiệu ứng dây chuyền

#### BA cần làm rõ
* Ngày chốt dữ liệu (cut-off) cố định hay thay đổi?
* Có cho phép chỉnh sửa sau chốt dữ liệu (cut-off) không?
* Ai có quyền ghi đè đặc quyền (override)?
* Có ghi kiểm toán (audit) log khi chỉnh sửa không?

---

### 2.6. Luồng phê duyệt (Approval Workflow)
luồng phê duyệt (workflow) thể hiện cách doanh nghiệp kiểm soát:
* Leave request
* OT request
* Attendance adjustment

Nó phản ánh cấu trúc quyền hạn và mức độ kiểm soát rủi ro.

#### Biến số thiết kế quan trọng
* Số cấp phê duyệt
* Điều kiện auto-approve
* SLA xử lý
* Escalation khi quá hạn / Kích hoạt cơ chế leo thang khi quá thời hạn cam kết
* kiểm toán (audit) log và lịch sử thay đổi

#### Trade-off trong thiết kế
* **luồng phê duyệt (workflow) quá đơn giản:**
  * → Dễ lạm dụng OT
  * → Rủi ro gian lận
* **luồng phê duyệt (workflow) quá phức tạp:**
  * → Chậm xử lý
  * → Trễ payroll
  * → Tăng chi phí vận hành

BA cần cân bằng giữa: **Kiểm soát rủi ro – Tốc độ xử lý – Trải nghiệm người dùng**.

---

## 3. Quy trình đầu-cuối điển hình (Typical End-to-End Process)

1. Nhân viên chấm công
2. Hệ thống tạo Attendance Record
3. ánh xạ (mapping) với Shift
4. Tính trễ/sớm/OT
5. Phát sinh ngoại lệ
6. Nhân viên gửi request
7. quản lý (manager) phê duyệt
8. HR tổng hợp công
9. Khóa chốt dữ liệu (cut-off)
10. Export sang Payroll
11. kiểm toán (audit) & lưu trữ

```mermaid
flowchart TD
    A[Employee Check-in/out] --> B[Attendance Record Created]
    B --> C[Map to Shift Rules]
    C --> D[Auto Calculate Late/Early/OT]
    D --> E{ngoại lệ (exception)?}
    E -- Yes --> F[Submit Adjustment / Leave / OT]
    F --> G[Manager Approval]
    G --> H[HR Review]
    E -- No --> I[Attendance Aggregation]
    H --> I
    I --> J[Cut-off Lock]
    J --> K[Export to Payroll]
    K --> L[Salary Calculation]
    L --> M[Audit & Archive]
```

---

## 4. So sánh chính sách (Policy) theo quy mô doanh nghiệp
Sự khác biệt về quy mô tổ chức không chỉ làm thay đổi số lượng nhân sự, mà làm thay đổi mức độ phức tạp của cấu hình, kiểm soát rủi ro và yêu cầu tích hợp hệ thống trong Leave & Attendance.

### Bảng so sánh mở rộng

| Yếu tố | Khởi nghiệp (Startup) | Doanh nghiệp vừa và nhỏ (SME) | Doanh nghiệp lớn (Enterprise) |
| :--- | :--- | :--- | :--- |
| **Shift Structure** | 1–2 ca cố định | Phân theo phòng ban | Rotating, multi-site, multi-country |
| **phê duyệt (approval) Model** | 1 cấp (Line Manager) | 2 cấp (Manager + HR) | Multi-level, matrix phê duyệt (approval) |
| **OT Control** | Linh hoạt, ít kiểm soát | Giới hạn nội bộ | Theo luật + kiểm toán (audit) định kỳ |
| **Leave Policy** | Tính theo năm | Tích lũy theo tháng | Theo seniority, cấp bậc, quốc gia |
| **chốt dữ liệu (cut-off) & Payroll Sync** | Thủ công | Export file định kỳ | Real-time tích hợp (integration) |
| **ngoại lệ (exception) Handling** | Xử lý thủ công | Có form điều chỉnh | luồng phê duyệt (workflow) + kiểm toán (audit) log |
| **Compliance Pressure** | Thấp | Trung bình | Rất cao (luật, kiểm toán, nội bộ) |
| **báo cáo (reporting) & phân tích (analytics)** | Excel | Dashboard cơ bản | BI, forecasting, hoạch định lực lượng lao động (workforce planning) |

### Phân tích theo từng cấp độ

| Yếu tố | Khởi nghiệp (Startup) – Tối giản để vận hành nhanh | Doanh nghiệp vừa và nhỏ (SME) – Bắt đầu kiểm soát và chuẩn hóa | Doanh nghiệp lớn (Enterprise) – Phức tạp và rủi ro cao |
| :--- | :--- | :--- | :--- |
| **Đặc điểm** | * Tổ chức phẳng<br>* Nhân sự ít<br>* Văn hóa linh hoạt | * Nhiều phòng ban<br>* Có HR chuyên trách<br>* Bắt đầu chịu áp lực chi phí | * Nhiều chi nhánh<br>* Có thể multi-country<br>* Bị kiểm toán (audit) thường xuyên |
| **Policy thường** | * 1–2 shift cố định (ví dụ 9–18h)<br>* Leave tính theo năm<br>* OT linh hoạt, ít phê duyệt<br>* Payroll xử lý bán thủ công | * Shift theo bộ phận<br>* Leave tích lũy/trích trước (accrual) theo tháng<br>* OT cần phê duyệt 1–2 cấp<br>* Xuất file cho payroll | * Xoay ca phức tạp<br>* phê duyệt (approval) nhiều cấp (theo cấu trúc matrix)<br>* OT theo luật từng quốc gia<br>* Leave theo seniority và legal requirement<br>* API thời gian thực (real-time) với Payroll & BI |
| **Rủi ro chính** | * Thiếu minh bạch khi quy mô tăng<br>* Phụ thuộc vào con người thay vì hệ thống<br>* Không chuẩn hóa dữ liệu → khó mở rộng | * Dữ liệu không đồng nhất giữa phòng ban<br>* Trễ duyệt ảnh hưởng chốt dữ liệu (cut-off)<br>* Phụ thuộc Excel để đối soát (reconcile) | * Sai rule có thể ảnh hưởng hàng nghìn nhân viên<br>* Không đồng bộ hệ thống → sai payroll diện rộng<br>* Rủi ro pháp lý và phạt hành chính |
| **BA Considerations** | Giai đoạn này cần thiết kế đơn giản nhưng phải có khả năng scale, tránh mã hóa cứng (hard-code) rule quá cứng. | * Cần chuẩn hóa quy tắc nghiệp vụ (business rule)<br>* Cần quy định rõ SLA phê duyệt<br>* Cần cơ chế ngoại lệ (exception) management | * Rule engine phải linh hoạt và configurable<br>* Phải có kiểm toán (audit) trail đầy đủ<br>* Phải phân quyền cực kỳ chặt chẽ<br>* Không thể thiết kế “mã hóa cứng (hard-code)d logic” |

### Xu hướng tăng độ phức tạp theo quy mô
Khi doanh nghiệp lớn dần:
1. Số biến số tăng theo cấp số nhân: **(Shift × Department × Location × Policy × Seniority)**
2. Tính phụ thuộc hệ thống tăng mạnh: **Attendance → Leave → OT → chốt dữ liệu (cut-off) → Payroll → Finance**
3. Chi phí sai sót tăng theo cấp số nhân:
   * Startup: sai 5 người
   * Enterprise: sai 5.000 người

### Phân tích từ góc nhìn thiết kế hệ thống

| Yếu tố | Ảnh hưởng tới thiết kế |
| :--- | :--- |
| **Quy mô nhỏ** | Có thể dùng rule cố định |
| **Quy mô trung bình** | Cần rule theo nhóm |
| **Quy mô lớn** | Cần bộ máy quy tắc (rule engine) + parameterization |
| **Multi-country** | Cần layer cấu hình theo pháp lý địa phương |
| **kiểm toán (audit) bắt buộc** | Cần immutable log |

---

## 5. Các điểm đau phổ biến (Common Pain Points)

| Điểm đau (Pain Point) | Biểu hiện thực tế | Nguyên nhân gốc rễ | Tác động kinh doanh | Lưu ý cho BA/PO |
| :--- | :--- | :--- | :--- | :--- |
| **Sai lệch dữ liệu Attendance** | Thiếu giờ, sai OT, trừ lương nhầm | ánh xạ (mapping) shift sai, rule cấu hình sai, thiết bị chấm công lỗi, chỉnh sửa thủ công | Khiếu nại lương, tăng workload HR, giảm niềm tin | Xác định source of truth, kiểm tra bộ máy quy tắc (rule engine), thiết kế cơ chế đối soát (reconciliation) trước chốt dữ liệu (cut-off) |
| **Phụ thuộc Excel thủ công** | Xuất file → chỉnh sửa ngoài hệ thống → nhập lại payroll | Hệ thống thiếu linh hoạt, không cover hết policy, thiếu tích hợp (integration) | Không kiểm toán (audit) được, rủi ro gian lận, sai sót lặp lại | Phân tích quy trình “shadow process”, đưa rule vào hệ thống thay vì xử lý offline |
| **Thiếu minh bạch với nhân viên** | Nhân viên không biết còn bao nhiêu leave, không hiểu cách tính OT | Không có self-service, thiếu dashboard, rule không được giải thích | Mất niềm tin, tăng ticket nội bộ | Thiết kế màn hình chi tiết cấu thành (breakdown), hiển thị logic tính toán, tăng tính explainability |
| **OT vượt giới hạn pháp lý** | Vượt trần giờ OT năm, không phân biệt OT ngày lễ | Không cấu hình giới hạn, không có cảnh báo thời gian thực (real-time) | Rủi ro phạt hành chính, chi phí lao động tăng | Thiết kế rule kiểm soát trần OT, cảnh báo trước khi duyệt, báo cáo theo phòng ban |
| **luồng phê duyệt (workflow) phê duyệt chậm** | Leave/OT pending qua chốt dữ liệu (cut-off) | Nhiều cấp duyệt, không SLA, không escalation | Trễ payroll, xử lý kỳ sau phức tạp | Phân tích SLA, đề xuất auto-escalation, cân bằng giữa kiểm soát & tốc độ |
| **Không đồng bộ dữ liệu với Payroll** | Payroll nhận dữ liệu sai hoặc thiếu | Export thủ công, thiếu API, chốt dữ liệu (cut-off) không rõ | Sai lương diện rộng | Thiết kế tích hợp (integration) rõ ràng, xác định thời điểm lock dữ liệu |
| **Thiếu kiểm toán (audit) trail** | Không biết ai chỉnh sửa dữ liệu | Không lưu log, chỉnh sửa trực tiếp | Không đáp ứng kiểm toán nội bộ | Thiết kế immutable log, phân quyền chặt chẽ |

---

## 6. Quy tắc nghiệp vụ trọng yếu (Key Business Rules)
Business Rules là tầng quyết định cách hệ thống “diễn giải” dữ liệu attendance thành nghĩa vụ tài chính. Đây là phần dễ sai nhất vì:
* Có nhiều biến số cấu hình
* Phụ thuộc luật lao động
* Thường thay đổi theo chính sách nội bộ
* Tác động trực tiếp đến payroll

### Bảng tổng hợp quy tắc nghiệp vụ (Business Rules)

| Nhóm quy tắc (Rule) | Câu hỏi nghiệp vụ trọng tâm | Biến số cấu hình | Rủi ro nếu sai |
| :--- | :--- | :--- | :--- |
| **Grace Period Rule** | Cho phép trễ bao nhiêu phút? Có áp dụng cho tất cả shift không? | Số phút, theo vị trí, theo ca | Sai trạng thái Late → ảnh hưởng đánh giá & lương |
| **OT tính toán (calculation) Rule** | OT tính theo ngày hay theo tuần? Làm tròn thế nào? | Block làm tròn, hệ số theo loại ngày | Tính thừa/thiếu OT → sai chi phí |
| **OT điều kiện áp dụng (eligibility) Rule** | OT có cần phê duyệt trước không? | phê duyệt (approval) required, auto-trigger | Lạm dụng OT hoặc bỏ sót OT hợp lệ |
| **Leave tích lũy/trích trước (accrual) Rule** | Tích lũy theo tháng, năm hay theo ngày làm thực tế? | Tốc độ tích lũy/trích trước (accrual), probation inclusion | Sai số dư leave → tranh chấp |
| **Carry Forward Rule** | Tối đa chuyển bao nhiêu ngày sang năm sau? Có expire không? | Số ngày tối đa, thời hạn sử dụng | Tăng nghĩa vụ tài chính ngoài dự kiến |
| **Leave khoản khấu trừ (deduction) Rule** | Trừ leave theo ngày hay theo giờ? | Minimum unit (0.5 ngày, 1 giờ) | Sai phép khi nghỉ nửa ngày |
| **chốt dữ liệu (cut-off) Rule** | Thời điểm khóa dữ liệu là khi nào? | Ngày cố định / theo kỳ lương | Payroll nhận dữ liệu chưa hoàn chỉnh |
| **Post chốt dữ liệu (cut-off) Adjustment Rule** | Ai được chỉnh sửa sau chốt dữ liệu (cut-off)? Có cần phê duyệt (approval) đặc biệt? | vai trò (role)-based permission, kiểm toán (audit) log | Gian lận hoặc mất kiểm soát dữ liệu |

### 6.1. Quy tắc khoảng dung sai (Grace Period Rule)
Grace period là cơ chế cân bằng giữa kỷ luật và thực tế vận hành.

* **Biến số:**
  * Số phút cho phép trễ
  * Có áp dụng cho check-out sớm không?
  * Có reset mỗi ngày không?
  * Có phân biệt theo cấp bậc?
* **Trade-off:**
  * Grace quá rộng → giảm tính kỷ luật
  * Grace quá chặt → tăng khiếu nại

### 6.2. Quy tắc tính làm thêm giờ (OT Calculation Rule)
* **Các mô hình phổ biến:**
  * Tính theo ngày (Daily OT)
  * Tính theo tuần (Weekly threshold)
  * Kết hợp cả hai
* **Các biến số cấu hình:**
  * Làm tròn theo 15 / 30 phút
  * Tính sau khi trừ break hay không
  * Phân biệt OT ngày thường / cuối tuần / lễ

### 6.3. Quy tắc tích lũy và chuyển phép (Leave Accrual & Carry Forward Rule)

#### Các mô hình tích lũy/trích trước (accrual)
| Mô hình | Đặc điểm |
| :--- | :--- |
| **Theo năm** | Đơn giản, cấp toàn bộ đầu năm |
| **Theo tháng** | Tích lũy dần, giảm rủi ro nghỉ hết sớm |
| **Theo ngày thực tế làm việc** | Phức tạp nhưng chính xác |

#### Carry Forward
* Có giới hạn số ngày tối đa không?
* Có expire sau bao lâu?
* Có được cash-out không?

#### Tác động tài chính
Leave tồn đọng = nghĩa vụ phải trả (liability). Enterprise thường rất kiểm soát phần này.

### 6.4. Quy tắc chốt kỳ và điều chỉnh (Cut-off & Adjustment Rule)

#### chốt dữ liệu (cut-off) Logic
* Khóa toàn bộ kỳ công vào ngày X
* Payroll lấy ảnh chụp dữ liệu (snapshot) dữ liệu tại thời điểm đó

#### Sau chốt dữ liệu (cut-off)
Cần làm rõ:
* Có cho chỉnh sửa không?
* Nếu có → ai được chỉnh sửa?
* Có cần phê duyệt (approval) đặc biệt không?
* Có log lịch sử thay đổi không?

> [!WARNING]
> **Rủi ro nếu không kiểm soát:** Thay đổi ngầm trước khi kiểm toán (audit), Gian lận OT, Sai lệch payroll kỳ sau.

#### Phân tích theo mức độ nhạy cảm
| Mức độ nhạy cảm | Rule |
| :--- | :--- |
| **Rất cao** | OT tính toán (calculation), chốt dữ liệu (cut-off) Rule |
| **Cao** | Leave tích lũy/trích trước (accrual), Carry Forward |
| **Trung bình** | Grace Period |
| **Phụ thuộc cấu trúc tổ chức** | phê duyệt (approval) & Adjustment Rule |

---

## 7. Góc nhìn dữ liệu và tích hợp (Data & Integration Perspective)

### 7.1. Dữ liệu cốt lõi trong miền nghiệp vụ (domain)

| Đối tượng dữ liệu (Data Object) | Vai trò nghiệp vụ | Phụ thuộc vào | Rủi ro nếu sai |
| :--- | :--- | :--- | :--- |
| **Employee ID** | Định danh nhân sự | Core HR | ánh xạ (mapping) sai người → sai lương |
| **Period (Payroll Cycle)** | Xác định kỳ tính công | Payroll calendar | Sai kỳ → sai tính lương |
| **Shift ID** | Chuẩn hóa giờ làm | Shift configuration | Sai rule Late/OT |
| **Clock-in/out** | Dữ liệu thô | Timekeeping device | Sai giờ làm thực tế |
| **Attendance Status** | Kết quả áp rule | Shift + Grace + Policy | Sai đánh giá chuyên cần |
| **Leave Balance** | Quyền lợi còn lại | tích lũy/trích trước (accrual) rule | Sai nghĩa vụ tài chính |
| **OT Hours** | Chi phí tăng thêm | OT rule + phê duyệt (approval) | Sai quỹ lương |
| **phê duyệt (approval) Status** | Kiểm soát hợp lệ | luồng phê duyệt (workflow) | Payroll tính dữ liệu chưa duyệt |
| **kiểm toán (audit) Log** | Truy vết thay đổi | System sự kiện (event) | Không đáp ứng kiểm toán |

### 7.2. Logic quan hệ dữ liệu (Data Relationship Logic)
Một số mối quan hệ quan trọng:
* `1 Employee` → `N Attendance Records`
* `1 Shift` → `N Employees`
* `Attendance Record` → phát sinh `OT` hoặc `Leave khoản khấu trừ (deduction)`
* `Leave Balance` = `tích lũy/trích trước (accrual)` – `Used` – `Expired`
* `Payroll ảnh chụp dữ liệu (snapshot)` = `Attendance Aggregation` tại thời điểm `chốt dữ liệu (cut-off)`

### 7.3. Luồng chấm công → tính lương (Attendance → Payroll Flow)

```mermaid
graph TD
    subgraph RawLayer[Raw Data Layer]
        A[Clock-in/out]
        B[Shift Mapping]
        C[Leave Request]
        D[OT Request]
    end

    subgraph AggLayer[Aggregation Layer]
        E[Calculate Total Hours]
        F[Apply Grace Period]
        G[Classify Attendance Status]
        H[Sum OT & Leave Deductions]
    end

    subgraph PayLayer[Payroll Calculation Layer]
        I[Apply Basic Salary]
        J[Apply OT Coefficients]
        K[Apply Allowances & Deductions]
    end

    subgraph kế toán (accounting)[Accounting Layer]
        L[Record Personnel Costs]
        M[Allocate to Cost Centers]
        N[Generate Journal Entries]
    end

    RawLayer --> AggLayer
    AggLayer -->|Attendance Summary ảnh chụp dữ liệu (snapshot)| PayLayer
    PayLayer -->|Finalized Payroll| kế toán (accounting)
```

### 7.4. Rủi ro khuếch đại (Error Amplification Effect)

**Hiệu ứng khuếch đại:** Sai lệch nhỏ (Ví dụ: Sai 30 phút OT) 
* → Sai hệ số 150% 
* → Sai lương 
* → Sai thuế TNCN 
* → Sai bảo hiểm 
* → Sai chi phí phòng ban 
* → Sai báo cáo tài chính.

### 7.5. Lưu ý cho BA/PO về dữ liệu và tích hợp

* **Về Data Integrity:**
  * Nguồn dữ liệu nào là authoritative?
  * Có cơ chế validation trước khi gửi payroll không?
  * Có đối soát (reconciliation) report không?
* **Về chốt dữ liệu (cut-off) & ảnh chụp dữ liệu (snapshot):**
  * Payroll lấy dữ liệu theo ảnh chụp dữ liệu (snapshot) hay thời gian thực (real-time)?
  * Nếu attendance thay đổi sau chốt dữ liệu (cut-off) thì xử lý thế nào?
* **Về kiểm toán (audit):**
  * Có log thay đổi attendance không?
  * Có log thay đổi rule không?
  * Có lưu phiên bản (version) policy theo thời gian không?
* **Về tích hợp (integration) Strategy:**
  * Export file hay API?
  * Đồng bộ theo batch hay hướng sự kiện (event-driven)?
  * Có cơ chế thử lại (retry) khi lỗi không?

---

## 8. Bản đồ phỏng vấn bên liên quan (Stakeholder Interview Mapping)

| Nhóm mục tiêu | Bên liên quan chính | Tập trung vào | Câu hỏi ví dụ |
| :--- | :--- | :--- | :--- |
| **Rule & Policy Clarity** | HR | Leave policy, OT rule, tích lũy/trích trước (accrual), chốt dữ liệu (cut-off) | Leave tích lũy theo tháng hay năm? OT tính theo ngày hay tuần? Có cho phép chỉnh sửa sau chốt dữ liệu (cut-off) không? |
| **Operational Efficiency** | HR, quản lý (manager) | Thời gian xử lý, luồng phê duyệt (workflow) phê duyệt | Tổng hợp công mất bao lâu? Có khó khi duyệt OT/Leave không? luồng phê duyệt (workflow) có bị nghẽn ở cấp nào? |
| **lực lượng lao động (workforce) Control** | quản lý (manager) | Kiểm soát OT, hiệu suất đội nhóm | Có cần cảnh báo khi nhân viên vượt trần OT không? Cần báo cáo gì để quản lý team? |
| **Financial Accuracy** | Payroll | Data bắt buộc, đối soát (reconciliation), payroll timing | Dữ liệu tối thiểu để tính lương là gì? Khi sai dữ liệu thì xử lý thế nào? Lấy dữ liệu theo ảnh chụp dữ liệu (snapshot) hay thời gian thực (real-time)? |
| **System Architecture** | IT | tích hợp (integration), API, multi-location, quyền truy cập (access) control | Máy chấm công có API không? Có multi-location không? Có cần phân quyền chi tiết theo vai trò (role) không? |
| **Compliance & Risk** | HR, Payroll, IT | kiểm toán (audit) log, legal OT limit, lưu trữ dữ liệu | Có từng bị thanh tra lao động chưa? kiểm toán (audit) log cần lưu bao lâu? Có giới hạn OT theo luật không? |

## 9. Bảng thuật ngữ chuyên ngành

| Thuật ngữ (viết tắt) | Dịch | Mô tả |
| :--- | :--- | :--- |
| **HRIS** | Hệ thống thông tin nhân sự | Hệ thống quản lý dữ liệu và quy trình nhân sự cốt lõi. |
| **Bản ghi chấm công (Attendance Record)** | Bản ghi hiện diện | Dữ liệu ghi nhận giờ vào, giờ ra, tổng giờ và trạng thái chuyên cần. |
| **Ca làm việc (Shift)** | Khung giờ làm việc | Lịch chuẩn dùng để diễn giải dữ liệu chấm công. |
| **OT** | Làm thêm giờ | Thời gian làm vượt giờ tiêu chuẩn và có thể phát sinh hệ số trả lương. |
| **Nghỉ phép (Leave)** | Thời gian vắng mặt hợp lệ | Khoảng thời gian nhân viên không làm việc theo quyền lợi hoặc nghĩa vụ. |
| **Tích lũy phép (Leave Accrual)** | Cộng dồn quyền nghỉ | Cơ chế phát sinh số dư phép theo thời gian hoặc điều kiện. |
| **Chuyển phép (Carry Forward)** | Chuyển số dư sang kỳ sau | Cho phép chuyển một phần phép chưa dùng sang năm hoặc kỳ tiếp theo. |
| **Khoảng dung sai (Grace Period)** | Khoảng cho phép sai lệch | Số phút được phép đi muộn hoặc về sớm trước khi bị ghi nhận ngoại lệ. |
| **Kỳ chốt dữ liệu (Cut-off Period)** | Thời điểm khóa dữ liệu | Mốc đóng băng dữ liệu thời gian trước khi chuyển sang tính lương. |
| **Bảng chấm công (Timesheet)** | Bảng tổng hợp thời gian | Tập hợp giờ làm, nghỉ, làm thêm và điều chỉnh trong một kỳ. |
| **Điều chỉnh chấm công (Attendance Adjustment)** | Sửa dữ liệu thời gian | Yêu cầu bổ sung hoặc sửa bản ghi chấm công có kiểm soát. |
| **Luồng phê duyệt (Workflow)** | Chuỗi xử lý phê duyệt | Cơ chế chuyển yêu cầu qua các vai trò theo điều kiện và cấp duyệt. |
| **FTE** | Tương đương nhân sự toàn thời gian | Đơn vị quy đổi khối lượng lao động về mức toàn thời gian. |
| **SoR** | Hệ thống dữ liệu gốc | Hệ thống được công nhận là nguồn dữ liệu chính thức cho một đối tượng. |

---

## 10. Ghi chú nghiên cứu và nguồn SAP chính thức

### 10.1. Nguyên tắc nghiên cứu

* Chỉ sử dụng tài liệu và trang sản phẩm chính thức thuộc hệ sinh thái SAP.
* Nội dung được chuẩn hóa theo miền nghiệp vụ để BA/PO có thể dùng cho khám phá sản phẩm, phân rã quy trình, mô hình miền và quản lý tồn đọng sản phẩm.
* Tên tính năng cụ thể có thể thay đổi theo phiên bản phát hành và cấu hình của từng khách hàng SAP SuccessFactors.
* Quy tắc pháp lý theo quốc gia vẫn cần được xác minh riêng theo ngày hiệu lực trước khi chuyển thành yêu cầu chính thức.

### 10.2. Nguồn tham khảo

| Giải pháp/tài liệu SAP | Phạm vi sử dụng trong nghiên cứu |
| :--- | :--- |
| [SAP SuccessFactors Time Tracking](https://www.sap.com/products/hcm/employee-time-tracking-software.html) | Ghi nhận thời gian, xác thực dữ liệu, phê duyệt bảng chấm công và tích hợp với tính lương. |
| [SAP SuccessFactors Time Management – SAP Help Portal](https://help.sap.com/docs/successfactors-employee-central/using-time-management-in-sap-successfactors/what-is-sap-successfactors-time-management) | Nghỉ phép, số dư thời gian, bảng chấm công và tác vụ tự phục vụ của nhân viên/quản lý. |
| [SAP SuccessFactors Employee Central Payroll](https://www.sap.com/products/hcm/employee-central-payroll.html) | Mối liên hệ giữa dữ liệu thời gian, kỳ chốt và tính lương. |

