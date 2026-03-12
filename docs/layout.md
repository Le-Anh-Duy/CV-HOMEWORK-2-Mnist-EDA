
# BÁO CÁO PHÂN TÍCH DỮ LIỆU KHÁM PHÁ (EDA) ĐA MIỀN

**Chủ đề: Hành trình từ Ký hiệu Hình thức đến Cấu trúc Giải phẫu Y khoa (Digit, Fashion & Pneumonia MNIST)**

### I. Định nghĩa Bài toán & Ngữ cảnh (Problem Formulation & Clinical Context)

* **Mục tiêu:** Thiết lập lăng kính đánh giá cho từng tập dữ liệu, chuyển từ góc nhìn "nhận diện vật thể" sang "tầm soát bệnh lý".
* **Hành động:** * Định nghĩa Digit/Fashion: Bài toán Multi-class Classification (10 lớp), độ chính xác (Accuracy) là mục tiêu hàng đầu.
* Định nghĩa PneumoniaMNIST: Bài toán Binary Classification (Bình thường vs. Viêm phổi).
* *Đánh giá rủi ro y khoa:* Sai lầm loại 2 (False Negative - Bỏ sót bệnh nhân viêm phổi) nguy hiểm hơn rất nhiều so với Sai lầm loại 1 (False Positive).


* **Insight kỳ vọng:** Người đọc hiểu rằng EDA cho Pneumonia sẽ không cố gắng tìm ra sự tách biệt hoàn hảo, mà tìm kiếm những dấu hiệu mờ nhạt nhất để không bỏ lọt ca bệnh.

### II. Khảo sát Tổng quan & Phân phối Nhãn (Dataset Overview & Label Distribution)

* **Mục tiêu:** Đánh giá tính toàn vẹn và mức độ cân bằng của dữ liệu để chọn Metric đánh giá phù hợp.
* **Hành động:** Vẽ Bar chart đếm số lượng ảnh.
* **Insight kỳ vọng:** * Digit/Fashion: Cân bằng tuyệt đối (VD: 6000 ảnh/class).
* Pneumonia: Sẽ thấy sự mất cân bằng (thường số ca Viêm phổi nhiều hơn ca Bình thường trong tập học gốc, hoặc ngược lại tùy bản chia). Từ đó kết luận bắt buộc phải dùng F1-Score, Recall hoặc `class_weights` thay vì Accuracy.



### III. Phân tích Đặc trưng Điểm ảnh & Vùng Tín hiệu (Pixel-Level & Signal Region Analysis)

* **Mục tiêu:** Trả lời câu hỏi *"Sự khác biệt giữa các class nằm ở độ sáng tổng thể hay nằm ở một vùng không gian cụ thể?"*
* **Hành động:**
* 1. Vẽ Histogram phân phối giá trị pixel (0-255) và chồng các class lên nhau.


* 2. Tính và trực quan hóa **Ảnh trung bình (Mean Image)** của từng class.


* 3. Phép tính "Bóng ma": Tính `Mean(Viêm phổi) - Mean(Bình thường)` để tạo ra Bản đồ nhiệt chênh lệch (Difference Heatmap).




* **Insight kỳ vọng:**
* *Digit:* Bimodal (2 đỉnh 0 và 255 rõ rệt). Difference Heatmap hiện rõ hình dáng từng con số.
* *Pneumonia:* Histogram của Bình thường và Viêm phổi đè lên nhau gần như 100%. Tuy nhiên, Difference Heatmap sẽ phát sáng mờ mờ ở hai bên thùy phổi (chứng tỏ vùng này chứa vệt mờ dịch tiết do viêm), trong khi vùng giữa (tim, cột sống) sẽ tối đen vì không thay đổi.



### IV. Phân tích Không gian và Hình thái (Spatial & Morphological Analysis)

* **Mục tiêu:** Đánh giá độ sắc nét của viền vật thể và mức độ nhiễu loạn cấu trúc.
* **Hành động:** * 1. Hiển thị ngẫu nhiên lưới ảnh (Grid).
* 2. Áp dụng bộ lọc dò biên Canny Edge Detection.




* **Insight kỳ vọng:**
* *Fashion:* Viền áo quần hiện ra rõ ràng, trơn tru. Có thể dùng hình dáng viền để phân loại.
* *Pneumonia:* Canny Edge sẽ trả ra một mớ hỗn độn (nhiễu từ xương sườn, xương quai xanh, mạch máu). Chứng minh rằng: Bài toán y tế không thể giải quyết bằng các đặc trưng hình học đơn giản (Shape/Contour) mà phải phân tích kết cấu (Texture).



### V. Đánh giá Khả năng Phân tách & Điểm mù (Separability & Blind Spots)

* **Mục tiêu:** Tiên tri độ khó của quá trình huấn luyện AI bằng cách xem các class có dễ bị nhầm lẫn với nhau trong không gian pixel thô (raw pixel) hay không.
* **Hành động:**
* 1. Chạy thuật toán t-SNE giảm 784 chiều xuống 2D và vẽ Scatter plot.


* 2. Đo chỉ số tương đồng cấu trúc SSIM giữa các class.




* **Insight kỳ vọng:**
* *Digit:* 10 cụm màu tách biệt nhau rõ rệt trên biểu đồ t-SNE.
* *Pneumonia:* Biểu đồ t-SNE chỉ là một "đám mây" duy nhất, điểm xanh (Bình thường) và điểm đỏ (Viêm phổi) trộn lẫn hoàn toàn. SSIM giữa hai class lên tới > 0.9. $\rightarrow$ Kết luận đắt giá: Bắt buộc phải dùng Deep Learning (CNN) để trích xuất đặc trưng bậc cao, các mô hình Machine Learning truyền thống (như SVM, KNN) sẽ thất bại thảm hại trên tập này.



### VI. Truy tìm Ngoại lai và Nhiễu Y khoa (Medical Artifacts & Outliers)

* **Mục tiêu:** Tìm ra các mẫu ảnh chụp hỏng, từ đó đề xuất quy trình làm sạch dữ liệu.
* **Hành động:** Lọc top 10 ảnh có Tổng độ sáng (Sum of Intensity) thấp nhất và cao nhất.
* **Insight kỳ vọng:** Phát hiện các tấm X-quang bị "cháy sáng" (trắng xóa) hoặc chụp thiếu tia (đen thui). Trong Digit, số mờ một chút AI vẫn đoán được. Nhưng trong Pneumonia, ảnh mờ sẽ lập tức tạo ra False Negative.

### VII. Tổng kết So sánh Đa miền & Đề xuất (Cross-Domain Summary & Augmentation Strategy)

* **Mục tiêu:** Chốt lại giá trị của bài báo cáo bằng một chiến lược xử lý dữ liệu khác biệt cho từng miền.
* **Hành động:** * Lập bảng so sánh (Digit: Ký hiệu / Phân tách tốt; Fashion: Vật thể / Phân tách khá; Pneumonia: Giải phẫu / Chồng chéo cao).
* Đề xuất Augmentation:
* *Digit:* Xoay nhẹ $\pm 10^\circ$. Không lật.
* *Fashion:* Xoay $\pm 15^\circ$, Lật ngang (Horizontal Flip) thoải mái.
* *Pneumonia:* **Rất cẩn thận!** Tuyệt đối KHÔNG LẬT NGANG (Horizontal Flip) vì bóng tim con người luôn nằm bên trái, lật ngang sẽ tạo ra ảnh giải phẫu sai thực tế (Dextrocardia - Tim chuột rút sang phải), làm hỏng mô hình y khoa. Chỉ nên dùng các phép tăng cường độ tương phản (Contrast/CLAHE) và dịch chuyển nhẹ.

