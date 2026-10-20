# BN-0 — HỒ SƠ LỊCH SỬ QUYẾT ĐỊNH THIẾT KẾ (BƯỚC 0–72)

**Dự án:** Bayesian Network dự đoán sự hiện diện PFAS trong chemical formulation  
**Phạm vi:** BN-Chemical; BN-Material để giai đoạn sau  
**Cơ sở:** 72 quyết định trong cuộc thảo luận thiết kế, cùng hai bản tổng hợp `BN-0_DAG_v1.0.md` và `BN-0_Node_Dictionary_v1.0.md`  
**Ngày biên soạn:** 15/10/2026  
**Trạng thái:** Hồ sơ truy vết quyết định. **DAG v1.0 hiện là bản nháp để thẩm định khoa học, chưa Structural Freeze.**

> **Cách đọc:** Mỗi bước ghi rõ **vấn đề**, **phương án chốt**, **nội dung thiết kế** và **lưu ý khi cải tiến**. Đây là hồ sơ lịch sử quyết định, **không phải** tuyên bố rằng mọi giả thuyết nhân quả đã được thực nghiệm xác minh. Một số phần được diễn đạt lại cho nhất quán thuật ngữ mà không đổi lựa chọn của người thiết kế.

## I. Bối cảnh và nguyên tắc bất biến

### Bước 0 — Khởi tạo BN-0 (bối cảnh tái dựng, không phải quyết định A/B)

- **Bài toán:** Ước lượng xác suất một **chemical formulation thực sự có ít nhất một chất thuộc định nghĩa PFAS OECD 2021**, kể cả khi công thức không khai báo đầy đủ hoặc kết quả kiểm nghiệm chưa bao phủ hết các chất.
- **Đầu ra mong muốn:** `P(PFAS_Presence = Yes | Evidence)` kèm khả năng giải thích nguồn PFAS và những giới hạn của bằng chứng.
- **Giới hạn BN-0:** Chốt mục tiêu, phạm vi, sơ đồ nhân quả DAG, định nghĩa các biến, các giả định và tiêu chí hoàn tất; **chưa xây CPT, chưa huấn luyện xác suất, chưa xây ứng dụng**.
- **Giới hạn vật liệu:** Chỉ làm rõ cầu nối về mặt khái niệm đến BN-Material; chưa thiết kế BN-Material trong BN-0.
- **Lưu ý lịch sử:** Không có bản ghi cho thấy “Bước 0 chọn A/B”. Đây là phần bối cảnh được tổng hợp từ toàn chuỗi và tài liệu chốt phạm vi; **không được tính thành quyết định thứ 73**.

### Những bất biến xuyên suốt

1. **Target không phải compliance:** PFAS Presence không phải vượt ngưỡng quy định, cũng không tương đương phát hiện bởi một test panel.
2. **OECD 2021:** Phân loại chất dựa trên cấu trúc phù hợp; CAS, tên thương mại, tổng fluorine hoặc tuyên bố thương mại không tự quyết định danh tính PFAS.
3. **Bốn con đường nguồn PFAS:** intentional input, impurity, transformation products, cross-contamination. Đây là bốn cách diễn đạt nguồn vào ở cấp chemical formulation, không phải bốn biến luôn độc lập.
4. **Trạng thái thật ≠ quan sát:** Hồ sơ, SDS, claim và đo lường chỉ là bằng chứng có giới hạn.
5. **Thiếu thông tin ≠ Không có PFAS:** `Unknown` là thông tin chưa quan sát, không thêm giá trị thứ ba cho biến hóa học nhị phân.
6. **Các lựa chọn đơn giản hóa có chủ đích:** Bước **56A, 57C, 59A, 60A, 62A** được giữ nguyên; không lén thay bằng phương án B.
7. **Kết thúc thiết kế:** Bước 71B đặt tiêu chí đóng băng; Bước 72B dừng mở rộng; việc xác minh khoa học cấu trúc còn phải hoàn tất.

## II. Nhật ký chi tiết 72 quyết định

### Nhóm 1 — Bản chất bài toán, các mạng liên kết và cơ chế nguồn (Bước 1–15)

#### Bước 1 — Target Definition — **A**
- **Vấn đề:** PFAS Presence nên phản ánh sự thật hóa học hay kết quả xét nghiệm/giới hạn pháp lý?
- **Đã chốt:** `PFAS_Presence ∈ {Yes, No}`; `Yes` khi **ít nhất một PFAS theo OECD 2021 thực sự hiện diện**, không phụ thuộc nồng độ hoặc ngưỡng compliance. Dữ liệu chưa đủ dẫn đến xác suất hậu nghiệm không chắc chắn, không thêm trạng thái `Unknown`.
- **Tác động:** Đặt nút mục tiêu hóa học thực, tách khỏi observed result và legal screening.
- **Cải tiến sau này:** Xây module compliance riêng; không tái định nghĩa target BN-0 thành `Above_Limit`.

#### Bước 2 — Two Linked Networks — **B**
- **Vấn đề:** Dùng một BN chung cho chemical formulation và vật liệu thành phẩm hay tách hai cấp?
- **Đã chốt:** Hai mạng liên kết: **BN-Chemical** cho chemical formulation và **BN-Material** cho thành phẩm.
- **Tác động:** Cho phép mô hình nguyên nhân phù hợp từng cấp; không coi xác suất ở công thức là xác suất ở vật liệu.
- **Cải tiến:** Thiết kế BN-Material về sau, không lấn phạm vi BN-0.

#### Bước 3 — Chemical–Material Linkage — **B**
- **Vấn đề:** Chuyển duy nhất xác suất PFAS từ hóa chất sang vật liệu hay biểu diễn quá trình sử dụng?
- **Đã chốt:** Mối liên kết cần xét **chemical state, application, transfer, retention, transformation, removal** thay vì truyền một scalar probability đơn giản.
- **Tác động:** Tránh suy luận “hóa chất có PFAS = sản phẩm cuối chắc chắn có PFAS”.
- **Cải tiến:** Giữ thông số chuyển giao ở mức khái niệm cho tới BN-Material.

#### Bước 4 — Chemical-Class Intermediate States — **B**
- **Vấn đề:** Chỉ dùng một trạng thái PFAS tổng hay cho phép các nhóm PFAS trung gian?
- **Đã chốt:** Giữ target nhị phân, nhưng có thể biểu diễn **các lớp/nhóm PFAS** ở cấp trung gian khi có ý nghĩa cơ chế; các nhóm có thể cùng tồn tại.
- **Tác động:** Hỗ trợ tính dị biệt hóa học và nguồn dữ liệu khác nhau.
- **Cải tiến:** Không đồng nhất nhóm chất với bốn cơ chế nguồn; tránh cộng xác suất các nhóm khi chúng giao nhau.

#### Bước 5 — Four PFAS Source Mechanisms — **B**
- **Vấn đề:** Có cần phân biệt cơ chế đưa PFAS vào formulation?
- **Đã chốt:** **Intentional ingredient**, **impurity**, **transformation product**, **cross-contamination**. Một formulation có thể chịu nhiều cơ chế cùng lúc.
- **Tác động:** Bốn trạng thái nguồn là đầu vào chính để tổng hợp target.
- **Cải tiến:** `Unknown` là thiếu thông tin, không là nguồn thứ năm; PFAS tái chế, lưu trữ... phải ánh xạ đúng đường nguồn/thời điểm.

#### Bước 6 — Application → Function/Technology — **B**
- **Vấn đề:** Ứng dụng sản phẩm có trực tiếp quyết định PFAS?
- **Đã chốt:** Ứng dụng ảnh hưởng đến **yêu cầu tính năng → công nghệ hóa học → cơ chế PFAS**, không là điểm rủi ro nhân quả trực tiếp. Bản đồ sử dụng ECHA có thể giúp chuẩn hóa taxonomy.
- **Tác động:** DAG cần các biến trung gian về chức năng/công nghệ.
- **Cải tiến:** Không dùng nhãn “water repellent” như bằng chứng xác nhận PFAS.

#### Bước 7 — True vs Declared Composition — **B**
- **Vấn đề:** SDS/full disclosure phản ánh bản chất thành phần đến đâu?
- **Đã chốt:** Tách **true composition** khỏi **declared composition**; biểu diễn quy trình khai báo và giới hạn công bố.
- **Tác động:** SDS là nút quan sát hoặc tài liệu bằng chứng, không đồng nhất với hóa học thật.
- **Cải tiến:** “Không có PFAS trên SDS” không bằng “không có PFAS”.

#### Bước 8 — Laboratory-Aware Measurement Model — **B**
- **Vấn đề:** Kết quả lab có nên được hiểu là quan sát chịu ảnh hưởng của phương pháp?
- **Đã chốt:** Trạng thái/nồng độ thật dẫn đến **analytical observations**, điều chỉnh bởi panel, method, matrix, LOD/LOQ, độ không đảm bảo, hiệu năng lab.
- **Tác động:** Đảm bảo hướng nhân quả true chemical → test result.
- **Cải tiến:** Năng lực lab dựa trên phạm vi, PT/ILC, QC..., không gán hiệu năng theo quốc gia. Bước 56 về sau quyết định **không** đưa QC chi tiết vào DAG.

#### Bước 9 — Sample Selection Process — **B**
- **Vấn đề:** Có dùng tỷ lệ dương tính từ khảo sát để suy ra tỷ lệ PFAS thị trường?
- **Đã chốt:** Biểu diễn **sampling frame, sampling design, targeted selection, previous screening** và lý do chọn mẫu.
- **Tác động:** Tránh selection bias khi dùng dữ liệu giám sát.
- **Cải tiến:** Tỷ lệ positive trong khảo sát có chọn lọc không tự là prevalence thị trường.

#### Bước 10 — Supplier as Indirect Context — **B**
- **Vấn đề:** Tên nhà cung cấp có trực tiếp gây ra PFAS?
- **Đã chốt:** Supplier tác động qua **sourcing, technology, manufacturing controls, disclosure process**; không tạo cạnh mặc định `Supplier → PFAS`.
- **Tác động:** Phân biệt proxy thương mại và cơ chế nhân quả.
- **Cải tiến:** Không gán điểm PFAS rủi ro mặc định theo tên công ty.

#### Bước 11 — Production Period Context — **B**
- **Vấn đề:** Yếu tố thời gian có vai trò gì?
- **Đã chốt:** Thời kỳ sản xuất có thể ảnh hưởng công nghệ, nguyên liệu và kiểm soát; ngày sản xuất, lấy mẫu và thử nghiệm được tách riêng.
- **Tác động:** Hạn chế dùng dữ liệu lịch sử sai ngữ cảnh.
- **Cải tiến:** Năm lịch không tự gây PFAS, mà gắn với các thay đổi thực tế.

#### Bước 12 — Unknown as Unobserved — **B**
- **Vấn đề:** Xử lý thành phần chưa rõ bằng cách nào?
- **Đã chốt:** Giữ các trạng thái thật trong mô hình và **marginalize** phần không quan sát; xét disclosure/missingness khi cần. `Unknown` không là trạng thái hóa học.
- **Tác động:** Posterior phản ánh bất định thay vì phán đoán nhị phân cưỡng ép.
- **Cải tiến:** Được cụ thể hóa ở Bước 52.

#### Bước 13 — Expert Priors with Uncertainty — **B**
- **Vấn đề:** Dùng expert judgment và dữ liệu thực nghiệm như thế nào?
- **Đã chốt:** Tri thức chuyên gia giúp chọn DAG và prior có bất định; dữ liệu thực nghiệm cập nhật posterior, không tự bịa giá trị CPT.
- **Tác động:** Phân tách cấu trúc khoa học và hiệu chỉnh tham số.
- **Cải tiến:** Chưa có số liệu phù hợp thì để TBD, không điền xác suất cho đủ bảng.

#### Bước 14 — PFAS Class Dependence — **B**
- **Vấn đề:** Các lớp PFAS có độc lập không?
- **Đã chốt:** Có thể cùng phụ thuộc công nghệ, nguyên liệu và quá trình, do đó **không mặc định độc lập**; các nhóm PFAS có thể đồng hiện diện.
- **Tác động:** Tránh double counting khi gộp nhiều nhóm chất.
- **Cải tiến:** Kiểm tra chung nguyên nhân và tương quan bằng chứng.

#### Bước 15 — Finish BN-Chemical First — **B**
- **Vấn đề:** Triển khai song song chi tiết Chemical và Material?
- **Đã chốt:** Hoàn thiện BN-Chemical trước; BN-Material chỉ lưu giao diện khái niệm.
- **Tác động:** Ràng buộc phạm vi nghiên cứu BN-0.
- **Cải tiến:** Không mở phần di chuyển PFAS qua textile trong DAG hóa chất.

### Nhóm 2 — Thành phần, quy trình, batch và lịch sử (Bước 16–26)

#### Bước 16 — Component-Aware Formulation — **B**
- **Vấn đề:** Formulation có nhiều thành phần với số lượng biến thiên.
- **Đã chốt:** Phân biệt cấp **Formulation** và **Component**; có thể mô hình nhiều ingredient trong một formulation.
- **Tác động:** Cho phép xác định thành phần được dùng thực tế và class PFAS của thành phần.
- **Cải tiến:** Quyết định 62A giữ polymer/UVCB **ở cấp substance**, không tách constituent nội bộ.

#### Bước 17 — Structure-Based PFAS Identity — **B**
- **Vấn đề:** Phân loại chất qua fluorine/CAS hay cấu trúc OECD?
- **Đã chốt:** Phân biệt có nguyên tố fluorine với đáp ứng định nghĩa OECD 2021; nếu cấu trúc đủ rõ, phân loại là **deterministic**. Polymer, UVCB, chất chưa rõ cấu trúc phải được xử lý thận trọng.
- **Tác động:** Không yêu cầu BN đoán điều đã được xác định từ cấu trúc.
- **Cải tiến:** Cụ thể hóa ở Bước 61 và giới hạn bởi 62A.

#### Bước 18 — Mechanism-Aware Synthesis — **B**
- **Vấn đề:** Tạp chất hoặc sản phẩm chuyển hóa từ tổng hợp có nguồn nào?
- **Đã chốt:** Xem xét starting materials, reaction pathway, reaction conditions, purification và khả năng sinh/tồn dư PFAS.
- **Tác động:** Các quá trình tổng hợp tạo mối liên kết hợp lý đến impurity và transformation.
- **Cải tiến:** Chỉ giữ nút trung gian có ý nghĩa theo Bước 70B.

#### Bước 19 — Upstream Synthesis vs Formulation Manufacturing — **B**
- **Vấn đề:** Gộp tổng hợp hóa chất với phối trộn formulation?
- **Đã chốt:** Tách **upstream substance synthesis** và **downstream formulation manufacturing** (mixing, dispersing, diluting, possible reaction, contamination).
- **Tác động:** Tránh gán sai cơ chế tạp chất của nhà cung cấp cho dây chuyền phối trộn.
- **Cải tiến:** Phải giữ đúng đơn vị nhà máy/lot ở mỗi công đoạn.

#### Bước 20 — Mechanism-Aware Cross-Contamination — **B**
- **Vấn đề:** Nhiễm chéo nên là điểm số đơn hay cơ chế sản xuất?
- **Đã chốt:** Xét **line history, equipment sharing, cleaning effectiveness** và nguồn nguyên liệu bị nhiễm. Tạp chất trong nguyên liệu được quy về impurity để tránh đếm lại ở cross-contamination.
- **Tác động:** Biến riêng `Manufacturing_Cross_Contamination`.
- **Cải tiến:** Cleaning là biến kiểm soát, không là cơ chế PFAS độc lập.

#### Bước 21 — Product–Batch Hierarchy — **B**
- **Vấn đề:** Mọi lô của cùng sản phẩm có chung PFAS state?
- **Đã chốt:** Tách cấp **Product** và **Batch**, mỗi batch có trạng thái và bằng chứng riêng.
- **Tác động:** Không tự mang kết quả kiểm nghiệm lô này sang lô khác.
- **Cải tiến:** Bước 24 bổ sung formulation version ở giữa.

#### Bước 22 — Within-Batch Heterogeneity — **B**
- **Vấn đề:** Một mẫu có luôn đại diện toàn bộ batch?
- **Đã chốt:** Phân biệt batch state, heterogeneity và sample state.
- **Tác động:** Phép thử của mẫu cụ thể không đồng nhất tuyệt đối với cả batch.
- **Cải tiến:** Không coi heterogeneity là kết quả nhiễm trong phòng lab; Bước 55 tách yếu tố đó.

#### Bước 23 — Joint Evidence, Correlated Measurements — **B**
- **Vấn đề:** Gộp nhiều mẫu/lab như các bằng chứng độc lập?
- **Đã chốt:** Cập nhật Bayesian dựa trên nhiều quan sát đồng thời, có tính đến **shared error sources**, mẫu chung và phụ thuộc giữa dữ liệu.
- **Tác động:** Tránh overconfidence khi nhiều báo cáo cùng một nguồn.
- **Cải tiến:** Không nhân likelihood độc lập nếu thiếu căn cứ.

#### Bước 24 — Product → Formulation Version → Batch — **B**
- **Vấn đề:** Sản phẩm giữ thương hiệu nhưng thay đổi công thức?
- **Đã chốt:** Phân cấp **Product → Formulation Version → Production Batch**.
- **Tác động:** Giới hạn tính hợp lệ của bằng chứng theo phiên bản và lô.
- **Cải tiến:** Không đồng nhất thay đổi supplier/lot với việc buộc đổi version trong mọi trường hợp.

#### Bước 25 — PFAS-Free Claims as Bounded Evidence — **B**
- **Vấn đề:** Certification/claim có chứng minh hoàn toàn không có PFAS?
- **Đã chốt:** Claim/chứng nhận là bằng chứng có **scope, time validity, version/batch**; không là bảo đảm tuyệt đối hoặc nguyên nhân hóa học.
- **Tác động:** Liên kết với declaration process, tránh double counting cùng dữ liệu kiểm nghiệm gốc.
- **Cải tiến:** Không nhập “PFAS-free” như hard constraint No.

#### Bước 26 — Hierarchical Historical Evidence — **B**
- **Vấn đề:** Kết quả các batch cũ có hỗ trợ batch mới không?
- **Đã chốt:** Batch cùng formulation version có thể chia sẻ tham số phân cấp và học từ lịch sử, đồng thời cho phép thay đổi qua revision.
- **Tác động:** Thông tin lịch sử hỗ trợ prior nhưng không là phép đo trực tiếp lô hiện tại.
- **Cải tiến:** Soát dataset shift và change events trước khi chuyển giao.

### Nhóm 3 — Ngành hàng, bối cảnh sản xuất, vòng đời và tiền chất (Bước 27–34)

#### Bước 27 — Product Category vs Function/Technology — **B**
- **Vấn đề:** Danh mục sản phẩm có thể đồng nhất với chức năng hoặc công nghệ?
- **Đã chốt:** `Chemical_Product_Category` riêng với `Application`, `Chemical_Function`, `Chemical_Technology`.
- **Tác động:** Giảm suy diễn PFAS từ nhãn ngành hàng.
- **Cải tiến:** Mỗi biến phải có định nghĩa và vai trò nhân quả riêng.

#### Bước 28 — Category–Subcategory Hierarchy — **B**
- **Vấn đề:** Tổ chức taxonomy danh mục hóa chất thế nào?
- **Đã chốt:** Có phân cấp **Category → Subcategory**, nhưng phân cấp taxonomy không tự động là cạnh nhân quả trong DAG.
- **Tác động:** Cho phép phân loại sản phẩm mà không suy diễn nguyên nhân trực tiếp.
- **Cải tiến:** Quản lý hierarchy metadata nếu không làm thay đổi xác suất có điều kiện.

#### Bước 29 — Properties as Selective Evidence — **B**
- **Vấn đề:** pH, độ nhớt, tính chất hóa lý có trực tiếp gây PFAS?
- **Đã chốt:** Thuộc tính hóa lý chỉ sử dụng khi có cơ chế hoặc giá trị bằng chứng, phân biệt **true property** với **reported/measured property**.
- **Tác động:** Không tạo cạnh tùy tiện `pH → PFAS`.
- **Cải tiến:** Chỉ đưa property vào DAG nếu có hỗ trợ khoa học và ảnh hưởng suy luận đáng kể.

#### Bước 30 — Manufacturing Site as Context — **B**
- **Vấn đề:** Supplier và nhà máy/dây chuyền thực sự có phải một?
- **Đã chốt:** Phân biệt `Supplier`, `Manufacturing_Site`, `Production_Line`; xét nguồn nguyên liệu, lịch sử thiết bị và cleaning.
- **Tác động:** Có thể đánh giá gia công bên thứ ba, các site khác nhau của cùng nhà cung cấp.
- **Cải tiến:** Không suy luận theo quốc gia hoặc khu vực nếu không có cơ chế và dữ liệu.

#### Bước 31 — Hierarchical Raw Material Traceability — **B**
- **Vấn đề:** Cùng CAS nhưng nguồn lô nguyên liệu khác nhau có thể khác tạp chất?
- **Đã chốt:** `Raw_Material_Identity`, `Raw_Material_Supplier`, `Raw_Material_Lot` là các cấp truy vết.
- **Tác động:** Tạp chất PFAS có thể phụ thuộc nguồn sản xuất/lot cụ thể.
- **Cải tiến:** Mã nhà cung cấp và mã lot là định danh; cơ chế thật là synthesis, purification, contamination.

#### Bước 32 — Lifecycle-Aware Chemical State — **B**
- **Vấn đề:** PFAS có thể được đưa vào formulation sau khi xuất xưởng?
- **Đã chốt:** Theo dõi `Post_Manufacturing_Handling`, `Storage_Container_History`, `Storage_Conditions`, `Post_Manufacturing_Contamination`.
- **Tác động:** Phân biệt trạng thái ở cuối sản xuất với trạng thái sau đóng gói/lưu trữ.
- **Cải tiến:** Không mặc định mọi bao bì hoặc điều kiện lưu trữ tạo PFAS.

#### Bước 33 — Time-Indexed PFAS State — **B**
- **Vấn đề:** Xác suất PFAS Presence áp dụng ở thời điểm nào?
- **Đã chốt:** `PFAS_State_At_Production`, `PFAS_State_At_Sampling`, `PFAS_State_At_Use`.
- **Tác động:** Mọi kết quả phải gắn với giai đoạn; trạng thái thay đổi bởi những cơ chế vật lý/hóa học có căn cứ.
- **Cải tiến:** Pha loãng đơn thuần không làm `Presence = Yes` trở thành `No` về mặt khái niệm.

#### Bước 34 — Precursor–Transformation Network — **B**
- **Vấn đề:** Chất tiền thân và sản phẩm chuyển hóa có nên phân biệt?
- **Đã chốt:** `Precursor_Identity`, `Transformation_Pathway`, `Transformation_Conditions`, `Transformation_Product_State`.
- **Tác động:** PFAS precursor đã thuộc OECD 2021 được tính là PFAS ngay trước chuyển hóa; precursor không thuộc PFAS chỉ tạo PFAS nếu chuyển hóa thực sự xảy ra.
- **Cải tiến:** Khả năng phản ứng ≠ phản ứng chắc chắn; không mặc định mọi pathway khả thi đều đã xảy ra.

### Nhóm 4 — Độ tin cậy bằng chứng, ràng buộc nhân quả và cơ chế kiểm soát (Bước 35–43)

#### Bước 35 — Evidence-Aware Bayesian Architecture — **B**
- **Vấn đề:** Nhiều báo cáo có thể trích cùng một dữ liệu gốc.
- **Đã chốt:** Quản lý `Evidence_Provenance`, `Evidence_Type`, `Evidence_Reliability`.
- **Tác động:** Không đếm một nghiên cứu gốc thành nhiều quan sát chỉ vì được OECD/ECHA/tổng quan trích lại.
- **Cải tiến:** Provenance có thể là metadata; không mặc định biến mọi trích dẫn thành DAG node.

#### Bước 36 — Mechanism-Specific Negative Evidence — **B**
- **Vấn đề:** Bằng chứng “không có PFAS” có nhiều bản chất.
- **Đã chốt:** Phân biệt `No_Intentional_PFAS_Use_Evidence`, `No_PFAS_Ingredient_Evidence`, `No_Contamination_Evidence`, `Analytical_Non_Detection`.
- **Tác động:** Mỗi bằng chứng phủ định chỉ cập nhật cơ chế mà nó đánh giá được.
- **Cải tiến:** Phân biệt **absence of evidence** và **evidence of absence**.

#### Bước 37 — Hybrid Deterministic–Probabilistic Network — **B**
- **Vấn đề:** Có dùng CPT để suy luận lại các định nghĩa hóa học đã xác định?
- **Đã chốt:** Quy tắc cấu trúc OECD là deterministic khi cấu trúc rõ; sự hiện diện chất trong formulation và cơ chế chưa quan sát là probabilistic.
- **Tác động:** Tách “PFOA là PFAS” khỏi “PFOA có thật trong batch này”.
- **Cải tiến:** Khai báo PFOA chưa xác minh không đương nhiên là chắc chắn có PFOA.

#### Bước 38 — Evidence-Constrained Mechanisms — **B**
- **Vấn đề:** Có cố định cơ chế No khi có bằng chứng phủ định?
- **Đã chốt:** Chỉ áp dụng ràng buộc xác định khi đã loại trừ thật sự trong phạm vi rõ ràng; còn lại giữ bất định.
- **Tác động:** Tránh ép posterior về 0 do tài liệu hạn chế.
- **Cải tiến:** Giá trị gần 0 không tự là bằng 0; nếu xuất hiện mâu thuẫn phải xem lại hard constraint.

#### Bước 39 — Probabilistic Evidence Reconciliation — **B**
- **Vấn đề:** SDS, claim và hai kết quả lab mâu thuẫn nhau.
- **Đã chốt:** Đối chiếu `Evidence_Conflict`, `Evidence_Scope`, `Evidence_Reliability`, `Shared_Evidence_Source` và kết hợp trong BN khi có cơ sở.
- **Tác động:** Không xây bảng ưu tiên cứng “lab luôn hơn SDS”.
- **Cải tiến:** Đầu tiên kiểm tra khác batch, version, analyte, ngày, LOD/LOQ; không phải mọi khác biệt là mâu thuẫn thật.

#### Bước 40 — Control Effectiveness as Causal Mechanism — **B**
- **Vấn đề:** Có chính sách kiểm soát PFAS có nghĩa nguy cơ đã giảm?
- **Đã chốt:** Tách `Control_Measures_Implemented`, `Control_Effectiveness`, `Control_Verification_Evidence`.
- **Tác động:** Hiệu quả thực tế của biện pháp tác động đến các cơ chế nguồn tương ứng, bằng chứng xác minh là quan sát.
- **Cải tiến:** Không dùng chứng nhận quản lý chung như hệ số giảm PFAS mặc định.

#### Bước 41 — Mechanism-Specific Control Effectiveness — **B**
- **Vấn đề:** Một chỉ số kiểm soát tổng thể có làm mất khác biệt?
- **Đã chốt:** Tách `Ingredient_Control_Effectiveness`, `Synthesis_Control_Effectiveness`, `Cross_Contamination_Control_Effectiveness`, `Post_Manufacturing_Control_Effectiveness`.
- **Tác động:** Kiểm soát nguyên liệu không đồng nghĩa vệ sinh thiết bị hiệu quả; `Cleaning_Effectiveness` thuộc nhóm nhiễm chéo.
- **Cải tiến:** Tránh một điểm QC chung tác động tất cả cơ chế.

#### Bước 42 — Recycled Feedstock Legacy Model — **B**
- **Vấn đề:** PFAS từ lịch sử sử dụng của nguyên liệu tái chế?
- **Đã chốt:** `Feedstock_Origin`, `Previous_Use_History`, `Recycling_Process`, `Legacy_PFAS_Residues`.
- **Tác động:** Legacy residues là con đường đóng góp impurity từ nguyên liệu, không là cơ chế nguồn thứ năm.
- **Cải tiến:** Recycled ≠ PFAS positive; cần đánh giá dòng vật liệu, xử lý và bằng chứng.

#### Bước 43 — Raw Material Change Events — **B**
- **Vấn đề:** Giữ cùng formulation version nhưng thay supplier/lot?
- **Đã chốt:** `Raw_Material_Change_Event`, `Change_Impact_Assessment`, `Affected_Production_Batches`, `Post_Change_Verification`.
- **Tác động:** Bằng chứng từ lô trước không tự áp dụng lô sau thay đổi nguồn.
- **Cải tiến:** Impact assessment là hồ sơ đánh giá, không phải cơ chế tạo PFAS.

### Nhóm 5 — Chuyển giao bằng chứng, giới hạn mô hình, giải thích và quyết định (Bước 44–52)

#### Bước 44 — Conditional Evidence Transferability — **B**
- **Vấn đề:** Dữ liệu một formulation có hỗ trợ formulation khác?
- **Đã chốt:** Chia sẻ thông tin qua technology, function, raw material profile, manufacturing process và PFAS source mechanisms chung.
- **Tác động:** Học tham số chung có điều kiện; test A không là test B.
- **Cải tiến:** Không đặt cạnh `Observed Test A → True State B` nếu không có cơ chế hợp lệ.

#### Bước 45 — Context-Aware Dataset Shift — **B**
- **Vấn đề:** Dữ liệu cũ có còn phù hợp công nghệ hiện tại?
- **Đã chốt:** `Source_Data_Context`, `Target_Formulation_Context`, `Distribution_Shift`, `Transportability_Assumption`.
- **Tác động:** Kiểm tra sự thay đổi tỷ lệ công nghệ và thay đổi bản thân cơ chế trước khi chuyển mô hình.
- **Cải tiến:** Dataset shift là khái niệm phân bố, không mặc nhiên là nguyên nhân hóa học trong DAG.

#### Bước 46 — Explicit Applicability Domain — **B**
- **Vấn đề:** BN có nên trả xác suất như nhau cho công nghệ chưa được nghiên cứu?
- **Đã chốt:** `Applicability_Domain`, `Domain_Compatibility`, `Knowledge_Gap`, `Prediction_Validity`.
- **Tác động:** Phân biệt “xác suất PFAS theo mô hình” với “mô hình có phù hợp đối tượng này không”.
- **Cải tiến:** Một posterior thấp có thể đi kèm validity thấp.

#### Bước 47 — Explicit Residual Causal Uncertainty — **B**
- **Vấn đề:** Một phát hiện PFAS không giải thích được bởi DAG hiện tại.
- **Đã chốt:** Quản lý `Unmodelled_Causal_Factors`, `Mechanism_Completeness`, `Unexplained_PFAS_Evidence`.
- **Tác động:** Không tạo nguồn PFAS thứ năm; biểu diễn khoảng trống ở mô tả bốn cơ chế hoặc điều kiện của chúng.
- **Cải tiến:** Xem xét sai số đo và thông tin thiếu trước khi kết luận có cơ chế mới.

#### Bước 48 — Parameter vs Structural Uncertainty — **B**
- **Vấn đề:** Chưa biết xác suất của cạnh khác với chưa chắc cạnh tồn tại.
- **Đã chốt:** `Parameter_Uncertainty`, `Structural_Uncertainty`, `Structural_Hypotheses`, `Model_Evidence`.
- **Tác động:** Lưu giả thuyết DAG thay thế và tình trạng khoa học; không cần dựng nhiều BN ngay.
- **Cải tiến:** Không gán certainty cho cạnh chỉ vì chuyên gia đề xuất hợp lý.

#### Bước 49 — Causally Explainable BN — **B**
- **Vấn đề:** Ngoài posterior tổng còn muốn biết đường nguồn nào có thể tạo PFAS.
- **Đã chốt:** Có thể suy luận posterior riêng của bốn cơ chế cùng bằng chứng liên quan.
- **Tác động:** Hỗ trợ lý giải và hướng điều tra tiếp.
- **Cải tiến:** Posterior cơ chế ≠ tỷ lệ đóng góp nồng độ hoặc nguyên nhân đã chứng minh.

#### Bước 50 — Expected Value of Information — **B**
- **Vấn đề:** Nên thu thập bằng chứng nào tiếp theo khi thông tin/chi phí hạn chế?
- **Đã chốt:** `Candidate_Evidence`, `Expected_Information_Gain`, `Evidence_Acquisition_Cost`, `Evidence_Priority`.
- **Tác động:** Chuẩn bị khả năng ưu tiên thu thập bằng chứng.
- **Cải tiến:** EIG là giảm bất định; EVI thực sự đòi hỏi quyết định và hàm lợi ích. **Không triển khai thuật toán EVI trong BN-0**.

#### Bước 51 — Intervention-Aware Causal BN — **B**
- **Vấn đề:** Thay đổi dây chuyền có giúp giảm PFAS khác gì quan sát dây chuyền tốt?
- **Đã chốt:** `Observed_Condition`, `Proposed_Intervention`, `Affected_Causal_Mechanism`, `Post_Intervention_PFAS_State`.
- **Tác động:** Thiết kế DAG hướng tới phân biệt `P(Y|X=x)` với `P(Y|do(X=x))`.
- **Cải tiến:** Cần giả định nhân quả/identifiability trước khi suy ra hiệu quả can thiệp; không tính `do()` trong BN-0.

#### Bước 52 — Explicit Missingness Mechanism — **B**
- **Vấn đề:** Không cung cấp công thức do lý do gì, có thể phụ thuộc vào thành phần thật?
- **Đã chốt:** `Data_Availability`, `Missingness_Reason`, `Disclosure_Process`, `Missingness_Dependency`.
- **Tác động:** Không mặc định mọi thiếu dữ liệu là random.
- **Cải tiến:** Nhà cung cấp bảo mật công thức không mặc nhiên có PFAS; thiếu dữ liệu phụ thuộc trạng thái thật chỉ là giả thuyết khi chưa có bằng chứng.

### Nhóm 6 — Thử nghiệm, QC, định danh và mức chi tiết hóa học (Bước 53–63)

#### Bước 53 — Method-Specific Analytical Evidence — **B**
- **Vấn đề:** Total Fluorine, EOF/AOF, targeted PFAS và TOP assay có đo cùng đại lượng?
- **Đã chốt:** Phân biệt `Targeted_PFAS_Result`, `Total_Fluorine_Result`, `Extractable_Organic_Fluorine_Result`, `Adsorbable_Organic_Fluorine_Result`, `TOP_Assay_Result`.
- **Tác động:** Quan sát theo measurand và phương pháp; không đồng nhất tổng fluorine với PFAS.
- **Cải tiến:** Các phép screening có thể là bằng chứng gián tiếp nhưng không xác nhận danh tính PFAS.

#### Bước 54 — Analyte-Level Coverage Model — **B**
- **Vấn đề:** Panel 20 chất và panel 60 chất đều ND có cùng sức nặng?
- **Đã chốt:** `Analytical_Target_List`, `Analyte_Specific_LOD_LOQ`, `Method_Applicability`, `Uncovered_PFAS_Space`, `Coverage_Evidence`.
- **Tác động:** Scope từng analyte quan trọng hơn số chất đơn thuần.
- **Cải tiến:** Không mặc định panel lớn hơn luôn đủ loại trừ PFAS.

#### Bước 55 — Explicit Pre-Analytical Contamination — **B**
- **Vấn đề:** PFAS có thể được đưa vào khi lấy mẫu, đựng mẫu, vận chuyển, chuẩn bị mẫu.
- **Đã chốt:** `Sampling_Equipment`, `Sample_Container`, `Sample_Handling`, `Sample_Preparation`, `Pre_Analytical_Contamination`.
- **Tác động:** Tách PFAS thật trong sản phẩm tại thời điểm lấy mẫu khỏi PFAS mới xuất hiện trong mẫu đã xử lý.
- **Cải tiến:** Không nhập artifact tiền phân tích như bằng chứng chắc chắn sản phẩm ban đầu chứa PFAS.

#### Bước 56 — QC as External Validation — **A**
- **Vấn đề:** Có đưa method blank, recovery, precision, instrument QC thành mạng nút không?
- **Đã chốt:** **Không.** Phòng lab đánh giá QC bên ngoài BN; BN tiếp nhận kết quả và trạng thái chấp nhận `Valid/Invalid`.
- **Tác động:** Không tạo detailed QC subnetwork, dù vẫn giữ yếu tố scope, LOD/LOQ và measurement limitation.
- **Cải tiến:** Không được tự ghi lại phương án B là lựa chọn cuối. QC Valid không đồng nghĩa kết quả nhận diện toàn bộ PFAS.

#### Bước 57 — Presumed QC Validity for Published Results — **C (đề xuất của người thiết kế)**
- **Vấn đề:** Nghiên cứu công bố không phải lúc nào cho hồ sơ QC đầy đủ.
- **Đã chốt:** **Kết quả đã công bố chính thức được mặc định chấp nhận là QC-valid để đưa vào BN**, trừ khi chính nguồn chỉ rõ invalid, retracted hoặc lỗi nghiêm trọng. Không yêu cầu trạng thái `Not_Verified` trong DAG.
- **Tác động:** Đơn giản hóa tiếp Bước 56; giảm gánh nặng truy hồ sơ QC nội bộ từng phòng lab.
- **Cải tiến:** **Giả định vận hành** không phải bằng chứng phòng lab đã đạt toàn bộ QC; giữ provenance và thông tin sửa đổi/rút lại.

#### Bước 58 — Quantitative-Aware Analytical Evidence — **B**
- **Vấn đề:** Chỉ giữ Detected/ND hay giữ nồng độ và giới hạn?
- **Đã chốt:** `Analyte_Identity`, `Measured_Concentration`, `Measurement_Unit`, `Detection_Status`, `Reporting_Limit`.
- **Tác động:** Dữ liệu gốc định lượng vẫn truy vết được dù target là nhị phân.
- **Cải tiến:** Phân biệt LOD và LOQ trong metadata; không ép nồng độ thành target.

#### Bước 59 — Simplified Binary Conversion — **A**
- **Vấn đề:** Mô hình hóa riêng left-censored data dưới LOD/LOQ hay đơn giản hóa?
- **Đã chốt:** **Đơn giản hóa kết quả dưới LOD/LOQ thành `Not_Detected` cho BN-0**, không đưa `Censoring_Type`/`Latent_Analyte_Concentration` thành nút riêng. Lưu nguyên văn qualifier và giới hạn theo Bước 58.
- **Tác động:** Giảm độ phức tạp mô hình đo lường.
- **Cải tiến:** Đây là **quy ước xử lý bằng chứng**, không khẳng định dưới LOQ có nghĩa thật sự không được phát hiện, và không đặt nồng độ = 0. Khi nghiên cứu sau, cần soát nguy cơ mất thông tin vì “detected below LOQ” về mặt phân tích khác “ND”.

#### Bước 60 — Confirmed PFAS Identification Only — **A**
- **Vấn đề:** Có dùng tín hiệu suspect/non-target chưa định danh như PFAS evidence?
- **Đã chốt:** **Chỉ dùng bằng chứng định danh PFAS đã xác nhận** để cập nhật phần identity-specific; tín hiệu nghi ngờ/chưa định danh lưu tham khảo, không cập nhật trực tiếp posterior như xác nhận PFAS.
- **Tác động:** Không xây `Identification_Confidence` subnetwork.
- **Cải tiến:** Không xóa nhóm quan sát tổng TF/EOF/AOF của Bước 53; chúng là screening **không đặc hiệu** chứ không phải confirmed identity.

#### Bước 61 — Structure-Centric Chemical Identity — **B**
- **Vấn đề:** CAS có đủ phân loại PFAS trong polymer, UVCB và mixture?
- **Đã chốt:** `Chemical_Identity`, `CAS_Registry_Number`, `Molecular_Structure`, `Substance_Type`, `Structure_Completeness`.
- **Tác động:** Cấu trúc phù hợp OECD 2021 là căn cứ phân loại; CAS là định danh bổ sung.
- **Cải tiến:** Thiếu SMILES không tự động là No.

#### Bước 62 — Substance-Level Classification — **A**
- **Vấn đề:** Polymer/UVCB có phân rã xuống constituent/oligomer/monomer trong DAG?
- **Đã chốt:** **Không.** Polymer/UVCB giữ **ở cấp substance**; không tạo mạng constituent-specific. Tạp chất PFAS tiếp tục xử lý bởi cơ chế `PFAS_Impurity` đã có.
- **Tác động:** Giảm mạnh độ phức tạp thành phần.
- **Cải tiến:** Chưa biết cấu trúc polymer/UVCB không được suy thành không có PFAS; không diễn giải lựa chọn A là bỏ qua impurity.

#### Bước 63 — Deterministic OR Aggregation — **B**
- **Vấn đề:** Gộp bốn đường nguồn thành PFAS Presence bằng CPT tùy ý hay luật logic?
- **Đã chốt:** `PFAS_Presence = OR(Intentional, Impurity, Transformation, Cross_Contamination)` **tại cấp/thời điểm được xác định**.
- **Tác động:** Chỉ cần ít nhất một nguồn thật sự hiện diện, target là Yes; nếu tất cả thật sự No thì No.
- **Cải tiến:** Các nguồn có thể phụ thuộc nhau; không cộng posterior các nguồn. Tại thời điểm sau sản xuất cần có transition hợp lý, không áp luật production thẳng lên mọi giai đoạn mà không xét biến cố.

### Nhóm 7 — Chuẩn hóa DAG, kết thúc phase và kiểm soát phạm vi (Bước 64–72)

#### Bước 64 — Scientifically Structured Node Dictionary — **B**
- **Vấn đề:** Có cần tài liệu định nghĩa nhất quán từng nút?
- **Đã chốt:** Mỗi nút phải có `Node_ID`, `Definition`, `Node_Type`, `Possible_States`, `Parent_Child_Relationships`, `Scientific_Rationale`, `Evidence_Source`, `Uncertainty_Status`.
- **Tác động:** Tạo nền kiểm toán khoa học cho DAG.
- **Cải tiến:** Nhiều thông tin là metadata, không phải nút DAG.

#### Bước 65 — Modular Causal DAG — **B**
- **Vấn đề:** Một sơ đồ khổng lồ hay phân mô-đun logic?
- **Đã chốt:** Bảy mô-đun **Chemical Identity, Formulation, Raw Material, Manufacturing, Lifecycle, Measurement, Evidence**.
- **Tác động:** Chia cấu trúc dễ thẩm định nhưng vẫn **một BN-Chemical thống nhất**.
- **Cải tiến:** Không biến bảy module thành bảy mạng độc lập vô điều kiện.

#### Bước 66 — Defined Causal Interfaces — **B**
- **Vấn đề:** Mô-đun giao tiếp bằng nút nào?
- **Đã chốt:** Các giao diện nhân quả chuẩn như `Raw_Material_PFAS_State`, `Manufacturing_Contamination_State`, `Formulation_PFAS_State`, `Lifecycle_PFAS_State`, `Sample_PFAS_State`.
- **Tác động:** Kiểm soát cạnh liên mô-đun; tái sử dụng nút canonical.
- **Cải tiến:** Mũi tên `measurement → true composition` không phải quan hệ nhân quả; cập nhật ngược xảy ra bằng inference.

#### Bước 67 — Scientific Causal Validation — **B**
- **Vấn đề:** DAG acyclic có đủ để xem là đúng?
- **Đã chốt:** Kiểm tra mỗi cạnh theo `Causal_Mechanism`, `Temporal_Consistency`, `Scientific_Evidence`, `Causal_Alternative` và cả kiểm tra kỹ thuật DAG.
- **Tác động:** Cạnh thiếu cơ sở chuyển thành giả thuyết, không mặc nhiên được chứng minh.
- **Cải tiến:** Cần review bằng nguồn và lập register từng cạnh trước Structural Freeze.

#### Bước 68 — Canonical Node and Causal Deduplication — **B**
- **Vấn đề:** Nhiều tên nút có đang đếm cùng sự kiện PFAS?
- **Đã chốt:** `Canonical_Node`, `Causal_Role`, `Redundancy_Check`.
- **Tác động:** Ví dụ `Legacy_PFAS_Residues → Raw_Material_PFAS_State` là nguyên nhân/giai đoạn của một dòng PFAS, không phải hai nguồn độc lập đưa thêm vào OR.
- **Cải tiến:** Không gộp các nút có vai trò nhân quả thật sự khác nhau chỉ vì tên gần giống.

#### Bước 69 — Mechanism-Appropriate State Spaces — **B**
- **Vấn đề:** Nên ép tất cả biến Yes/No?
- **Đã chốt:** Cho phép **Binary, Categorical, Ordinal, Continuous** khi có ý nghĩa cơ chế/bằng chứng; target vẫn nhị phân.
- **Tác động:** Giữ tính biểu đạt của chemical technology và quy trình, nhưng hạn chế CPT phình to.
- **Cải tiến:** `Unknown` không trở thành trạng thái hóa học; chỉ thêm mức phân loại thực sự có giá trị suy luận.

#### Bước 70 — Minimum Sufficient Causal Granularity — **B**
- **Vấn đề:** DAG mô tả mọi công đoạn hóa học hay mức tối thiểu đủ giải thích?
- **Đã chốt:** Chỉ giữ nút khi làm rõ cơ chế, thay đổi suy luận, có bằng chứng riêng, là biến can thiệp hoặc giúp tránh sai lệch/đếm trùng.
- **Tác động:** Giữ DAG khả thi cho thẩm định và CPT sau này.
- **Cải tiến:** “Tối thiểu” không được xóa biến confounder hoặc trung gian cần thiết cho mục tiêu nhân quả.

#### Bước 71 — Versioned Structural Freeze — **B**
- **Vấn đề:** BN-0 dừng khi nào, tránh thiết kế vô hạn?
- **Đã chốt:** Chỉ đóng băng khi đạt `Node_Dictionary_Complete`, `Causal_Edges_Reviewed`, `Module_Interfaces_Validated`, `Redundancy_Resolved`, `Uncertainty_Documented`.
- **Tác động:** Cho phép đặt phiên bản DAG và cập nhật theo lịch sử khoa học.
- **Cải tiến:** Chốt **tiêu chí**, không có nghĩa tiêu chí đã hoàn tất.

#### Bước 72 — Close Scope & Consolidate BN-0 — **B**
- **Vấn đề:** Sau 71 bước, tiếp tục đặt câu hỏi mới hay bắt tay tổng hợp?
- **Đã chốt:** **Không mở thêm quyết định thiết kế.** Chỉ (1) tổng hợp quyết định, (2) tạo DAG BN-Chemical thống nhất, (3) hoàn thiện Node Dictionary, (4) rà soát khoa học và đóng băng version.
- **Tác động:** Chặn overengineering; BN-0 có deliverable và điểm kết thúc rõ ràng.
- **Cải tiến:** Hai tài liệu DAG/Node Dictionary v1.0 hiện là **draft**; cần hoàn tất evidence-based edge validation, cycle/redundancy checks, sign-off trước khi coi là `FROZEN`; CPT thuộc BN-1.

## III. Bảng tra cứu quyết định theo chủ đề

| Chủ đề cần xem lại | Bước liên quan | Điểm cần giữ khi cải tiến |
|---|---|---|
| Định nghĩa target, OECD, danh tính cấu trúc | 1, 4, 17, 37, 60, 61, 62, 63 | Không nhầm PFAS Presence với TF, CAS, detected panel hoặc compliance |
| Phân cấp formulation và batch | 16, 21, 22, 24, 26, 31, 43 | Evidence gắn version, lot và thời điểm |
| Bốn nguồn PFAS | 5, 18–20, 31–34, 40–43, 63 | Không tạo nguồn thứ năm hoặc cộng nguồn trùng |
| Ứng dụng, danh mục, công nghệ và supplier | 6, 10, 11, 27–30 | Context không mặc định là PFAS risk score |
| Lab, test panel, QC và diễn giải ND | 8, 23, 53–60 | Đặc biệt giữ 56A/57C/59A/60A |
| Claims, missingness, provenance, conflict | 7, 9, 12, 25, 35, 36, 38, 39, 52 | Evidence có scope, không tự biến thành chemical truth |
| Chuyển giao dữ liệu và tính áp dụng | 14, 26, 44–48 | Tương đồng không phải phép đo trực tiếp; xem shift |
| Giải thích, EVI, can thiệp | 49, 50, 51 | Khả năng mở rộng, chưa triển khai BN-0 |
| Cấu trúc, dictionary, thẩm định và freeze | 13, 48, 64–72 | DAG v1.0 chưa là cấu trúc đã xác thực |
| BN-Material | 2, 3, 15, 33 | Defer; không nhảy sang chuyển giao vật liệu trong BN-0 |

## IV. Các lựa chọn ngoại lệ quan trọng — không được thay đổi khi tái dựng lịch sử

| Bước | Phương án thực tế | Kết quả | Điều không được suy diễn |
|---|---|---|---|
| 1 | **A** | PFAS Presence Yes/No theo OECD 2021 | Không biến thành regulatory pass/fail |
| 56 | **A** | QC được lab xác nhận ngoài BN | Không xây mạng QC chi tiết |
| 57 | **C** | Kết quả chính thức công bố được mặc định QC-valid khi không có thông tin bất lợi | Không khẳng định QC được thực nghiệm xác minh |
| 59 | **A** | Đơn giản hóa dữ liệu dưới LOD/LOQ thành ND trong lớp BN | Không xóa qualifier gốc, không đặt nồng độ 0 |
| 60 | **A** | Chỉ confirmed identity cho bằng chứng định danh PFAS | Không cấm TF/EOF/AOF như screening gián tiếp |
| 62 | **A** | Polymer/UVCB ở cấp substance | Không tự suy Unknown = No, không loại bỏ impurity |
| 71 | **B** | Áp dụng tiêu chí structural freeze có phiên bản | Không tuyên bố đã vượt tiêu chí |
| 72 | **B** | Khóa phạm vi, không thêm câu hỏi thiết kế | Không tự động chuyển sang BN-1 khi review chưa xong |

## V. Liên kết hai tài liệu tổng hợp BN-0

1. **`BN-0_DAG_v1.0.md`** — DAG khái niệm, bảy mô-đun, các bất biến, giao diện và checklist thẩm định.
2. **`BN-0_Node_Dictionary_v1.0.md`** — Danh sách biến canonical, metadata, quan hệ và bản đồ chống trùng lặp.
3. **Tài liệu hiện tại** — Lịch sử *tại sao* lựa chọn đã được chốt, gồm cả các lựa chọn A/C nhằm tránh bị ghi đè khi cải tiến.

**Cách dùng khi nâng phiên bản:** (i) xác định vấn đề và bước lịch sử liên quan; (ii) mở DAG/Dictionary để xem trạng thái hiện tại; (iii) ghi rõ giả định và bằng chứng mới; (iv) lập change log mô tả thay đổi, tác động lên cấu trúc, khả năng so sánh với phiên bản trước; (v) không sửa ngược lịch sử quyết định gốc. Một nâng cấp có thể thay quyết định của phiên bản mới, nhưng không được viết lại rằng phiên bản BN-0 đã chọn khác đi.

## VI. Hạng mục còn mở trước khi đóng băng cấu trúc (không phải quyết định thiết kế mới)

- [ ] Gắn ID canonical cho mọi nút đưa vào DAG cuối cùng; metadata không đứng giả làm nguyên nhân PFAS.
- [ ] Kiểm từng cạnh về mechanism, temporal order, evidence và causal alternatives.
- [ ] Kiểm DAG không chu trình và không có đường đếm trùng nguồn PFAS.
- [ ] Kiểm phân biệt `formulation at sampling` và `collected/handled sample`.
- [ ] Kiểm khai báo, certification và analytical observation không đảo hướng nhân quả.
- [ ] Kiểm treatment thực tế của TF/EOF/AOF/TOP, panel scope, LOD/LOQ và ND tương thích với các lựa chọn đã chốt.
- [ ] Lưu rõ các cạnh còn ở dạng structural hypothesis và giới hạn applicability.
- [ ] Ký xác nhận `DAG v1.0` + `Node Dictionary v1.0` là `FROZEN` sau scientific review.

---

**Kết luận lịch sử:** BN-0 có 72 **quyết định thiết kế** (Bước 1–72), cộng Bước 0 là bối cảnh khởi tạo được tái dựng. Đích của BN-0 là **DAG và Node Dictionary có thể thẩm định**, không phải một BN đã ước lượng CPT. Khi review hoàn tất và cấu trúc được freeze, mới chuyển BN-1.
