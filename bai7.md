# Bài 7: Thiết kế Quy trình Che mờ (Data Masking) Khách VIP

## 1. Mục tiêu
* Làm chủ quy trình Data Masking (Che mờ PII) để bảo vệ dữ liệu cá nhân khách hàng.
* Định nghĩa yêu cầu che mờ và thiết kế luồng khôi phục dữ liệu (De-masking).

---

## Phần A: Bảng che mờ dữ liệu (Masking Table)

Dựa trên nội dung email khiếu nại, ta bóc tách toàn bộ dữ liệu định danh cá nhân (PII) và các thông tin giao dịch nhạy cảm để gắn nhãn thay thế (Placeholders) mà vẫn giữ nguyên ngữ cảnh cho AI hiểu:

| Thông tin PII thật | Nhãn che mờ (Placeholder) | Loại dữ liệu |
| :--- | :--- | :--- |
| Phạm Minh C | `[TEN_KHACH_HANG]` | Họ và tên cá nhân |
| 0988.111.222 | `[SO_DIEN_THOAI]` | Số điện thoại liên hệ |
| minhc@gmail.com | `[EMAIL_KHACH_HANG]` | Địa chỉ thư điện tử |
| Căn hộ 502 Chung cư Sunrise, Q7 | `[DIA_CHI_NHA]` | Địa chỉ cư trú chi tiết |
| Ngày 20/09 | `[NGAY_MUA_HANG]` | Thời gian giao dịch cụ thể |
| MartX District 7 | `[TEN_CHI_NHANH]` | Địa điểm cửa hàng giao dịch |
| 4.500.000 VNĐ | `[GIA_TRI_DON_HANG]` | Giá trị tài chính đơn hàng |

> **Lưu ý:** Các thông tin ngữ cảnh như *"khách hàng VIP"*, *"2kg thịt bò Wagyu"*, *"giao chậm 3 tiếng"*, *"làm hỏng tiệc gia đình"*, *"bồi thường 100%"* được giữ lại để AI nắm rõ mức độ nghiêm trọng và soạn thư xin lỗi sát thực tế.

---

## Phần B: Văn bản đã che mờ (Masked Input)

Văn bản khiếu nại sau khi thay thế 100% thông tin nhạy cảm bằng các nhãn che mờ để sẵn sàng gửi sang AI công cộng:

> "Tôi là `[TEN_KHACH_HANG]` (SĐT: `[SO_DIEN_THOAI]`, Email: `[EMAIL_KHACH_HANG]`, ở `[DIA_CHI_NHA]`). Ngày `[NGAY_MUA_HANG]` tôi mua 2kg thịt bò Wagyu giá `[GIA_TRI_DON_HANG]` tại `[TEN_CHI_NHANH]` nhưng bị giao chậm 3 tiếng làm hỏng tiệc gia đình. Tôi yêu cầu bồi thường 100%!"

---

## Phần C: Câu lệnh gửi AI (Prompt)

Câu lệnh hướng dẫn AI soạn thảo thư phản hồi chuyên nghiệp dựa trên dữ liệu đã che mờ, đồng thời ép AI giữ nguyên các thẻ Placeholder để dễ dàng khôi phục ở bước cuối:

```text
Bạn là Chuyên viên Chăm sóc Khách hàng Cấp cao tại hệ thống siêu thị MartX. Hãy viết một email phản hồi xin lỗi và giải quyết khiếu nại cho một khách hàng VIP dựa trên nội dung khiếu nại đã được ẩn danh dưới đây:

Nội dung khiếu nại:
"Tôi là [TEN_KHACH_HANG] (SĐT: [SO_DIEN_THOAI], Email: [EMAIL_KHACH_HANG], ở [DIA_CHI_NHA]). Ngày [NGAY_MUA_HANG] tôi mua 2kg thịt bò Wagyu giá [GIA_TRI_DON_HANG] tại [TEN_CHI_NHANH] nhưng bị giao chậm 3 tiếng làm hỏng tiệc gia đình. Tôi yêu cầu bồi thường 100%!"

Yêu cầu bắt buộc:
1. Giọng văn chân thành, cầu thị, thể hiện sự trân trọng đặc biệt đối với khách hàng VIP và nhận trách nhiệm hoàn toàn về sự cố giao hàng chậm trễ làm ảnh hưởng đến buổi tiệc gia đình.
2. Đưa ra phương án giải quyết thỏa đáng: Chấp thuận mức bồi thường 100% giá trị đơn hàng ([GIA_TRI_DON_HANG]) và tặng kèm voucher ưu đãi cho lần mua sắm tiếp theo.
3. Tuyệt đối giữ nguyên định dạng các nhãn trong ngoặc vuông (như [TEN_KHACH_HANG], [NGAY_MUA_HANG], [TEN_CHI_NHANH], [GIA_TRI_DON_HANG], [SO_DIEN_THOAI], [DIA_CHI_NHA]) ở những vị trí phù hợp trong thư, không tự ý bịa đặt hay thay đổi tên nhãn.
```

---

## Phần D: Văn bản khôi phục cuối cùng (De-masked Output)

Sau khi nhận kết quả phản hồi từ AI, nhân viên CSKH thực hiện bước **De-masking** (thay ngược thông tin thật vào các thẻ Placeholder) để tạo thành bức thư hoàn chỉnh gửi cho khách hàng:

> **Tiêu đề:** Thư xin lỗi chân thành và Phương án bồi thường đơn hàng ngày 20/09 – Hệ thống siêu thị MartX
>
> **Kính gửi Quý khách hàng VIP Phạm Minh C,**
>
> Lời đầu tiên, Bộ phận Chăm sóc Khách hàng hệ thống siêu thị MartX xin gửi lời xin lỗi chân thành và sâu sắc nhất tới Quý khách vì sự cố giao hàng chậm trễ 3 tiếng đối với đơn hàng 2kg thịt bò Wagyu vào **ngày 20/09** tại chi nhánh **MartX District 7**.
>
> Chúng tôi hoàn toàn thấu hiểu sự thất vọng của Quý khách khi sự cố này đã làm ảnh hưởng trực tiếp đến buổi tiệc quan trọng của gia đình. Với tư cách là khách hàng VIP luôn tin tưởng và đồng hành cùng MartX, trải nghiệm của Quý khách là ưu tiên hàng đầu của chúng tôi, và MartX xin nhận hoàn toàn trách nhiệm về sơ suất lần này.
>
> Để khắc phục sự cố một cách thỏa đáng nhất, MartX xin phép thực hiện phương án giải quyết như sau:
> * **Bồi thường 100% giá trị đơn hàng:** Hoàn trả toàn bộ số tiền **4.500.000 VNĐ** theo đúng yêu cầu của Quý khách.
> * **Quà tặng tri ân:** Gửi tặng Quý khách một phiếu mua hàng ưu đãi trị giá 500.000 VNĐ như một lời xin lỗi chân thành cho lần trải nghiệm tiếp theo.
>
> Chuyên viên phụ trách khách hàng VIP của chúng tôi sẽ liên hệ trực tiếp với Quý khách qua số điện thoại **0988.111.222** trong vòng 30 phút tới để xác nhận hình thức hoàn tiền và hỗ trợ gửi quà tận nơi tại địa chỉ **Căn hộ 502 Chung cư Sunrise, Q7**.
>
> Một lần nữa, MartX thành thật xin lỗi Quý khách **Phạm Minh C** và rất mong tiếp tục nhận được sự thông cảm, tin yêu từ Quý khách.
>
> Trân trọng,  
> **Bộ phận Chăm sóc Khách hàng Cấp cao – MartX**
