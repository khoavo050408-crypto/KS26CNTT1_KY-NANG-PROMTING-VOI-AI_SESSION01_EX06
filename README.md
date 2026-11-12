
---

## PHẦN 1: GIẢI THÍCH HÌNH TƯỢNG "CHIẾC CỐC NƯỚC" (CONTEXT WINDOW & TRUNCATION)

Để hiểu vì sao AI lại quên thông tin khách hàng ngay trong cùng một cuộc trò chuyện, hãy hình dung bộ nhớ tạm thời (Context Window) của AI giống hệt như **một chiếc cốc nước có dung tích cố định** (ví dụ: chứa tối đa 32.000 giọt nước, tương đương 32.000 Tokens):

### 1. Bản chất của "Chiếc cốc nước" (Context Window)
* **Cơ chế đọc lại toàn bộ:** AI không thực sự "nhớ" theo cách con người ghi chép vào sổ tay. Mỗi lần bạn gửi tin nhắn mới, AI sẽ đọc lại toàn bộ nội dung từ đầu cốc đến đáy cốc để hiểu ngữ cảnh trước khi trả lời.
* **Trạng thái ở Lượt 1:** Thông tin *"Anh Trần Văn B - đơn #MTX-9988"* được rót vào đầu tiên và nằm ở đáy cốc. Lúc này cốc còn trống tới 95%, nên AI đọc rất rõ và trả lời chính xác.

### 2. Hiện tượng "Tràn cốc" và Cắt gọt dữ liệu cũ (Truncation)
* **Tích lũy Token:** Qua 40 lượt hỏi đáp dài dòng về rau củ, cả câu hỏi của khách lẫn câu trả lời chi tiết của AI đều biến thành những "giọt nước" (Tokens) rót liên tục vào cốc. Đến khoảng lượt 41, chiếc cốc đã **đầy 100% (chạm ngưỡng 32.000 Tokens)**.
* **Cơ chế Truncation:** Khi khách hàng nhắn tiếp ở Lượt 42, lượng nước mới đổ vào buộc chiếc cốc phải đẩy phần nước cũ nhất ở dưới đáy tràn ra ngoài để lấy chỗ chứa. Hiện tượng hệ thống tự động xóa bỏ các dữ liệu cũ nhất ở đầu phiên chat gọi là **Truncation (Cắt gọt ngữ cảnh)**.
* **Hậu quả thực tế:** Dữ liệu ở Lượt 1 (*anh B, mã #MTX-9988*) bị đẩy văng hoàn toàn khỏi cốc. AI không còn nhìn thấy thông tin đó nữa nên đành hỏi lại, thậm chí tự suy đoán nhầm tên khách hàng thành *"chị Mai"* dựa trên các đoạn hội thoại mẫu có sẵn.

---

## PHẦN 2: PHƯƠNG ÁN QUẢN LÝ "RAM BỘ NHỚ" ĐỂ KHÔNG BỊ TRÀN CỐC

Để luồng tư vấn CSKH luôn chính xác và không bao giờ rơi vào tình trạng mất trí nhớ giữa chừng, MartX cần kết hợp đồng thời hai nhóm giải pháp sau:

### 1. Phương án về Thao tác Người dùng (Dành cho nhân viên CSKH)
* **Kỹ thuật "Tóm tắt & Mở phiên mới" (Summarize & Reset):** Khi cuộc trò chuyện kéo dài khoảng 15–20 lượt hoặc chuyển sang chủ đề mới, nhân viên yêu cầu AI: *"Hãy tóm tắt ngắn gọn thông tin khách hàng, mã đơn và vấn đề chính trong 3 gạch đầu dòng"*. Sau đó, mở một **đoạn chat mới (New Chat)** và dán bản tóm tắt đó vào làm tin nhắn đầu tiên để làm sạch "chiếc cốc".
* **Thiết lập "Ghim thông tin cốt lõi" (Context Pinning):** Thay vì chỉ nhắc tên và mã đơn ở lượt đầu, khi hỏi các câu mang tính quyết định ở cuối phiên (như chính sách hoàn tiền), nhân viên nên chủ động nhắc kèm định danh vào câu lệnh (Ví dụ: *"Kiểm tra điều kiện hoàn tiền cho khách Trần Văn B - đơn #MTX-9988"*).
* **Yêu cầu AI trả lời súc tích:** Tránh để AI tự do viết các đoạn giải thích dài lê thê về chính sách hay mô tả sản phẩm nếu khách không hỏi sâu, giúp tiết kiệm đáng kể lượng Token tiêu thụ trên mỗi lượt.

### 2. Phương án về Thiết kế Hệ thống (Dành cho đội ngũ Kỹ thuật / Vận hành)
* **Tách biệt Bộ nhớ tĩnh (System Prompt / Memory Injection):** Thiết kế luồng chatbot sao cho thông tin định danh khách hàng (Tên, SĐT, Mã đơn hàng lấy từ CRM) luôn được **ghim cứng ở phần System Prompt** (luôn được gửi kèm cố định ở đầu mỗi lượt gọi API) thay vì trôi theo lịch sử chat thông thường. Nhờ đó, dù lịch sử hội thoại có bị cắt gọt (Truncation), thông tin khách hàng vẫn bất biến.
* **Áp dụng cơ chế Cửa sổ trượt thông minh (Sliding Window + Summarization Memory):** Lập trình hệ thống tự động kích hoạt một tác vụ chạy ngầm mỗi khi dung lượng hội thoại chạm mức **70% Context Window**. Hệ thống sẽ tự động nén 20 lượt chat cũ thành một đoạn tóm tắt ngắn khoảng 200 Tokens để giải phóng 80% dung lượng cốc cho các lượt tư vấn tiếp theo.
