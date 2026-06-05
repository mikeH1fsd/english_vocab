# English Vocabulary Hub - Cấu trúc & Tài liệu Dự án

File này được tạo ra để AI (và cả bạn) có thể nắm nhanh toàn bộ cấu trúc dự án, chức năng của từng file và các hàm JavaScript quan trọng. **Xin vui lòng không xóa file này! Mỗi lần AI xử lý một yêu cầu mới, AI có thể đọc file này để lấy Context (ngữ cảnh) nhằm tránh xóa nhầm các đoạn code quan trọng.**

## 1. Cấu trúc Thư mục & File

- `index.html`: Trang chủ (Home) của dự án. Hiển thị danh sách các "Book" (quyển) từ vựng từ 1 đến 6 và TOEIC. Giao diện dạng card trực quan.
- `story1.html` -> `story6.html` & `story_TOEIC.html`: Các trang chi tiết cho mỗi quyển. Mỗi trang chứa nhiều chủ đề (Story). Đây là nơi hiển thị nội dung câu chuyện tiếng Anh với các từ vựng được highlight.
- `vocabulary1.csv` -> `vocabulary6.csv` & `vocabulary_TOEIC.csv`: Dữ liệu thô gốc dạng CSV chứa danh sách từ vựng, nghĩa tiếng Việt, và tên file âm thanh tương ứng.
- `vocab_chunks/`: Thư mục tạm dùng để chứa các file JSON chia nhỏ dữ liệu từ vựng cho AI xử lý song song (Enrichment). 
- `media/`: Thư mục chứa các tệp âm thanh (mp3) dùng để phát âm thanh của từ vựng.
- `*.py` (vd: `split_vocab.py`, `recover_json.py`, `merge_vocab.py`...): Các tệp mã nguồn Python hỗ trợ AI làm nhiệm vụ tự động hóa, chia nhỏ dữ liệu, gọi API song song, ghép dữ liệu và bơm dữ liệu vào các file HTML.

## 2. Cấu trúc File HTML Chi Tiết (`story1.html`...)

Mỗi file `storyX.html` thường có cấu trúc chính bao gồm 3 phần:
1. **Phần Giao Diện (UI):** Các thẻ `<button>` chuyển chủ đề, thẻ `<div>` chứa nội dung truyện (content), thẻ hiển thị thanh Check-list từ vựng ở cuối mỗi câu chuyện. 
2. **Phần Popup:** Thẻ `<div id="meaning-display"></div>` dùng làm Popup nổi lên khi người dùng click vào một từ vựng, và thẻ `<audio id="audio-player"></audio>` dùng để phát âm thanh.
3. **Phần Script (Javascript):** Rất dễ bị ghi đè/xóa nhầm nếu không cẩn thận. Bao gồm các thẻ `<script>` sau:

### Các Script và Function Quan Trọng trong HTML

* **Dữ liệu từ vựng (`const allVocab`)**: 
  - Thường nằm ở đầu phần Script.
  - Chứa đối tượng JSON lớn bao gồm Nghĩa (meaning), Âm thanh (audio), và mảng Định nghĩa đa nghĩa (definitions - bao gồm từ loại (pos), ngữ cảnh (context), ví dụ (example)).
  
* **Function `window.showStory(num)`**: 
  - Xử lý việc chuyển đổi giữa các câu chuyện (Story 1, 2, 3, 4...). 
  - Nếu lỡ tay xóa hàm này, các nút bấm chuyển chủ đề trên cùng sẽ bị liệt.

* **Function `toggleDarkMode()`**: 
  - Bật/tắt chế độ ban đêm. Đồng thời lưu trữ trạng thái vào `localStorage` của trình duyệt. 
  - Nếu xóa hàm này, nút 🌙 / ☀️ sẽ bị liệt.

* **Xử lý Bôi Đen & Dịch (Translate Selected Text)**: 
  - Một khối IIFE (Immediately Invoked Function Expression) tự động gán sự kiện `mouseup` / `touchend`. Khi người dùng bôi đen một đoạn văn bản > 2 ký tự, popup `🌐 Dịch` sẽ hiện lên, bấm vào sẽ nhảy sang Google Translate.

* **Event Listener cho Click Từ Vựng (`document.addEventListener('click', ...)`)**: 
  - Bắt sự kiện khi người dùng nhấn vào các thẻ có class `.vocab-word`.
  - Tự động lấy từ vựng, tìm trong `allVocab`, đánh dấu "Đã xem" (chuyển chữ sang màu cam nhạt/xanh), phát âm thanh, và cập nhật nội dung Popup (Nghĩa, Từ loại, Bối cảnh, Ví dụ) rồi hiển thị lên. 
  - Có cả bộ đếm thời gian (Timeout) tự động tắt Popup sau một thời gian (ví dụ 12 giây), và logic bắt sự kiện click ra ngoài để đóng Popup.

## 3. Quy trình Bơm Dữ Liệu (Dành cho AI)

Tiến độ: **Đã bơm xong dữ liệu Đa Nghĩa cho Book 1, Book 2 và Book 3.**
*Lưu ý riêng từ Book 3 trở đi: Đã nâng cấp AI thành `vocab_enricher_v2` với Prompt ép buộc tự tìm thêm nghĩa thực tế, không bị phụ thuộc vào bản dịch gốc!*

Khi cần cập nhật thêm dữ liệu từ vựng (ví dụ làm tiếp Book 4):
1. **Lấy dữ liệu:** Đọc `vocabulary4.csv`.
2. **Xử lý:** Chia thành các chunk (VD: `chunk1.json` -> `chunk6.json`) vào thư mục `vocab_chunks/`.
3. **Gọi AI song song:** Chạy 6 subagents để phân tích từng từ vựng và tự động viết `definitions` đa nghĩa (pos, context, example).
4. **Ghép và Bơm:** Dùng script `apply_fix_book3.py` (tương tự như book 2) để đọc `story3.html`. **TUYỆT ĐỐI** dùng `.find` để thay thế từ khối `const posMap = {` đến `</body>`, có nhét thêm `</script>` ở cuối đoạn tiêm để không làm sập Javascript, và thay thế khối `const allVocab = {...};`. KHÔNG ĐƯỢC ghi đè lố xuống dưới làm mất `showStory` hay `toggleDarkMode`.

---
*Lưu ý cho AI: Hãy đọc file này bất kỳ lúc nào để nắm được ngữ cảnh hoặc để thiết kế Regex an toàn trước khi chỉnh sửa các file HTML.*
