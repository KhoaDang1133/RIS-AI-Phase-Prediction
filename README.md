# Ứng dụng AI dự đoán pha RIS bằng mạng FC-MultiHead
1. Giới thiệu

Dự án xây dựng mô hình trí tuệ nhân tạo nhằm dự đoán cấu hình pha phản xạ của bề mặt phản xạ thông minh RIS trong hệ thống truyền thông vô tuyến BS–RIS–UE.

Mô hình FC-MultiHead được sử dụng để dự đoán trạng thái pha cho từng phần tử RIS. Mục tiêu của hệ thống là cải thiện chất lượng kênh truyền hiệu dụng và giảm thời gian lựa chọn cấu hình pha so với thuật toán Coordinate Descent.

2. Mô hình hệ thống

Hệ thống mô phỏng gồm:

Trạm gốc BS có 4 anten.
RIS gồm 16 phần tử phản xạ.
Thiết bị người dùng UE có 1 anten.
Mỗi phần tử RIS có 8 trạng thái pha rời rạc.

Kênh truyền hiệu dụng được tính theo công thức:

H_eff = H_BU + H_RU × Θ × H_BR

Trong đó:

H_BR là kênh truyền từ BS đến RIS.
H_RU là kênh truyền từ RIS đến UE.
H_BU là kênh truyền trực tiếp từ BS đến UE.
Θ là ma trận pha phản xạ của RIS.
H_eff là kênh truyền hiệu dụng tại UE.
3. Mục tiêu của đề tài

Đề tài được thực hiện với các mục tiêu chính:

Mô phỏng hệ thống truyền thông vô tuyến BS–RIS–UE bằng MATLAB.
Xây dựng Dataset từ các đặc trưng của kênh truyền.
Sử dụng Coordinate Descent để tạo nhãn pha RIS.
Huấn luyện mô hình FC-MultiHead để dự đoán pha cho từng phần tử RIS.
Đánh giá mô hình trong nhiều điều kiện SNR, Doppler Shift và TDL khác nhau.
So sánh thời gian dự đoán của AI với thời gian tối ưu bằng Coordinate Descent.
4. Thông số mô phỏng
Thông số	Giá trị
Số anten BS	4
Số phần tử RIS	16
Số anten UE	1
Số trạng thái pha	8
Số đặc trưng đầu vào	551
Số nhánh đầu ra	16
Mô hình kênh	TDL-A, TDL-B, TDL-C, TDL-D, TDL-E
SNR khảo sát	-20, -10, 0, 10, 20 dB
Doppler Shift	30, 60, 120 Hz
Thuật toán tạo nhãn	Coordinate Descent
Mô hình AI	FC-MultiHead
Tỷ lệ Train/Validation/Test	80% / 10% / 10%
5. Quy trình xây dựng Dataset

Quy trình tạo Dataset gồm các bước:

Thiết lập các thông số mô phỏng.
Khởi tạo vị trí của BS và RIS.
Tạo ngẫu nhiên vị trí của UE.
Mô phỏng các kênh H_BR, H_RU và H_BU.
Xét ảnh hưởng của path loss, SNR, AWGN, Doppler Shift và mô hình kênh TDL.
Lấy nhiều trạng thái kênh truyền theo Snapshot.
Trích xuất đặc trưng CSI, vị trí UE, khoảng cách và path loss.
Sử dụng Coordinate Descent để tìm cấu hình pha RIS làm nhãn.
Chuẩn hóa dữ liệu.
Chia Dataset thành các tập Train, Validation và Test.
6. Thuật toán Coordinate Descent

Coordinate Descent được sử dụng để tìm cấu hình pha phù hợp cho RIS.

Tại mỗi bước, thuật toán lần lượt thay đổi trạng thái pha của một phần tử RIS và đánh giá chỉ số:

Metric = ||H_eff||²_F

Trong đó, ||H_eff||²_F là bình phương Frobenius norm của kênh truyền hiệu dụng.

Trạng thái pha tạo ra giá trị Metric lớn nhất sẽ được chọn. Quá trình này được thực hiện lần lượt cho 16 phần tử RIS và lặp lại trong nhiều vòng quét.

Kết quả của Coordinate Descent được sử dụng làm nhãn đầu ra để huấn luyện mô hình AI.

7. Kiến trúc mạng FC-MultiHead

Mô hình gồm hai phần chính:

Shared Trunk dùng để trích xuất đặc trưng chung.
16 Output Head dùng để dự đoán pha cho 16 phần tử RIS.
Shared Trunk

Dữ liệu đầu vào gồm 551 đặc trưng và được đưa qua các lớp:

Input 551 → Fully Connected 512 → Batch Normalization → ReLU → Dropout → Fully Connected 384 → Batch Normalization → ReLU → Dropout → Fully Connected 256 → Batch Normalization → ReLU.

Output Head

Mỗi Head gồm:

Fully Connected 64 → ReLU → Fully Connected 8 logits.

Mỗi Head dự đoán một trong 8 trạng thái pha cho một phần tử RIS.

Tổng cộng mô hình có 16 Head, mỗi Head có 8 logits, tương ứng với 128 logits đầu ra.

8. Chuẩn hóa dữ liệu

Dữ liệu đầu vào được chuẩn hóa theo từng đặc trưng bằng công thức:

X_norm = (X - μ) / σ

Trong đó:

μ là giá trị trung bình của đặc trưng trên tập Train.
σ là độ lệch chuẩn của đặc trưng trên tập Train.
X_norm là dữ liệu sau khi chuẩn hóa.

Việc chuẩn hóa giúp các đặc trưng có phạm vi giá trị phù hợp hơn, từ đó hỗ trợ quá trình huấn luyện ổn định và hội tụ tốt hơn.

9. Cấu trúc repository

Repository được tổ chức thành các thư mục chính như sau:

README.md: Giới thiệu tổng quan về dự án.
.gitignore: Loại bỏ các file tạm và file không cần thiết.
src/dataset_generation: Chứa code tạo Dataset.
src/training: Chứa code huấn luyện mô hình AI.
src/evaluation: Chứa code đánh giá và so sánh kết quả.
src/functions: Chứa các hàm MATLAB được sử dụng trong dự án.
demo: Chứa chương trình chạy thử mô hình.
data: Chứa Dataset mẫu.
models: Chứa mô hình AI đã huấn luyện.
results: Chứa hình ảnh, biểu đồ và bảng kết quả.
docs: Chứa tài liệu và các sơ đồ của đề tài.
10. Yêu cầu phần mềm

Để chạy dự án, máy tính cần cài đặt:

MATLAB.
5G Toolbox.
Deep Learning Toolbox.
Statistics and Machine Learning Toolbox.

Mô hình trong dự án được thiết kế để có thể huấn luyện và kiểm tra bằng CPU.

11. Hướng dẫn chạy chương trình
Bước 1: Tạo Dataset

Chạy file:

src/dataset_generation/generate_RIS_dataset.m

Chương trình sẽ mô phỏng các kênh truyền, xây dựng đặc trưng đầu vào và tạo nhãn pha RIS bằng Coordinate Descent.

Bước 2: Huấn luyện mô hình

Chạy file:

src/training/train_FC_MultiHead.m

Trong code, kiểm tra và thay đổi đường dẫn Dataset cho phù hợp với tên file thực tế của bạn.

Ví dụ:

datasetPath = 'data/RIS_TDLA_Dataset_20dB_30Hz.mat';

Bước 3: Đánh giá mô hình

Chạy file:

src/evaluation/evaluate_FC_MultiHead.m

Chương trình sẽ tính các chỉ số như:

Loss.
Element Accuracy.
Accuracy ±1 pha.
Codeword Accuracy.
Circular MAE.
Circular RMSE.
MSE chỉ số pha.
Bước 4: So sánh thời gian tối ưu

Chạy file:

src/evaluation/compare_optimization_time.m

Chương trình sẽ so sánh thời gian xử lý giữa Coordinate Descent và mô hình AI FC-MultiHead.

12. Kết quả chính

Trong điều kiện mô phỏng TDL-A, SNR 20 dB và Doppler Shift 30 Hz, mô hình FC-MultiHead đạt được:

Chỉ số	Kết quả
Test Element Accuracy	39.70%
Test Accuracy ±1 pha	80.34%
Thời gian Coordinate Descent	1.521242 ms
Thời gian FC-MultiHead	1.128201 ms

Kết quả cho thấy mô hình FC-MultiHead có khả năng dự đoán cấu hình pha RIS với thời gian thấp hơn Coordinate Descent trong kịch bản thử nghiệm.

Lợi thế về thời gian của AI có thể rõ hơn khi cần xử lý đồng thời nhiều trạng thái kênh truyền.

13. Ảnh hưởng của các điều kiện kênh truyền

Kết quả mô phỏng cho thấy:

Khi SNR giảm, độ chính xác dự đoán pha RIS giảm.
Khi Doppler Shift tăng, kênh truyền thay đổi nhanh hơn và việc dự đoán trở nên khó khăn hơn.
Các mô hình kênh TDL khác nhau tạo ra mức độ khó khác nhau đối với mô hình AI.
FC-MultiHead có sự cân bằng giữa hiệu suất dự đoán và chi phí tính toán.
14. Hướng phát triển

Trong tương lai, dự án có thể được phát triển theo các hướng:

Tăng số lượng và độ đa dạng của mẫu trong Dataset.
Mở rộng số lượng phần tử RIS.
Huấn luyện một mô hình chung cho nhiều mức SNR và Doppler Shift.
Thử nghiệm các kiến trúc CNN, Transformer hoặc Graph Neural Network.
Sử dụng dữ liệu kênh truyền thực tế.
Triển khai mô hình trên hệ thống nhúng hoặc phần cứng thời gian thực.
Tối ưu thêm thời gian suy luận và dung lượng mô hình.
15. Tác giả

Khoa Đăng

Đồ án tốt nghiệp: Ứng dụng trí tuệ nhân tạo trong dự đoán pha phản xạ RIS cho hệ thống truyền thông vô tuyến.

16. Ghi chú

Repository được xây dựng phục vụ mục đích học tập và nghiên cứu.

Kết quả có thể thay đổi tùy thuộc vào Dataset, thông số mô phỏng, phiên bản MATLAB và cấu hình phần cứng.
