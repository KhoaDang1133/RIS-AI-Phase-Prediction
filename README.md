# RIS-AI Phase Prediction — Mô tả hệ thống

Tài liệu này mô tả tổng quan mô hình hệ thống, quy trình tạo Dataset, kiến trúc mạng FC-MultiHead và cách đánh giá khả năng dự đoán pha RIS trong repository này.

## Tổng quan

Đề tài xây dựng một hệ thống ứng dụng AI để dự đoán cấu hình pha cho bề mặt phản xạ thông minh RIS trong mô hình truyền thông vô tuyến BS–RIS–UE, gồm:

- Mô phỏng các kênh truyền `H_BR`, `H_RU` và `H_BU` bằng MATLAB.
- Xét ảnh hưởng của mô hình kênh TDL, SNR, Doppler Shift, AWGN và path loss.
- Sử dụng thuật toán Coordinate Descent để tìm cấu hình pha RIS làm nhãn.
- Xây dựng Dataset từ CSI, vị trí UE, khoảng cách, path loss và nhiều Snapshot.
- Huấn luyện mạng FC-MultiHead để dự đoán đồng thời pha của 16 phần tử RIS.
- Đánh giá mô hình theo độ chính xác, sai số pha và thời gian xử lý.

Mục tiêu chính là giảm thời gian lựa chọn cấu hình pha RIS so với quá trình tối ưu lặp bằng Coordinate Descent, đồng thời duy trì khả năng dự đoán phù hợp trong nhiều điều kiện kênh truyền.

## Mô hình hệ thống

Hệ thống mô phỏng gồm:

- Trạm gốc BS có `4 anten`.
- Bề mặt RIS có `16 phần tử phản xạ`.
- Thiết bị người dùng UE có `1 anten`.
- Mỗi phần tử RIS có `8 trạng thái pha rời rạc`.

Kênh truyền hiệu dụng tại UE được xác định bởi:

`H_eff = H_BU + H_RU × Θ × H_BR`

Trong đó:

- `H_BR`: kênh truyền từ BS đến RIS.
- `H_RU`: kênh truyền từ RIS đến UE.
- `H_BU`: kênh truyền trực tiếp từ BS đến UE.
- `Θ`: ma trận pha phản xạ của RIS.
- `H_eff`: kênh truyền hiệu dụng tại UE.

Chỉ số được sử dụng để đánh giá chất lượng cấu hình pha là:

`Metric = ||H_eff||²_F`

Giá trị này là bình phương Frobenius norm của kênh truyền hiệu dụng. Metric càng lớn thì cấu hình pha đang xét càng phù hợp với mục tiêu tối ưu của hệ thống.

## Thành phần chính

- `src/dataset_generation/`: chứa code mô phỏng kênh truyền và tạo Dataset.
- `src/training/`: chứa code xây dựng và huấn luyện mạng FC-MultiHead.
- `src/evaluation/`: chứa code đánh giá mô hình và so sánh thời gian xử lý.
- `src/functions/`: chứa các hàm MATLAB hỗ trợ.
- `data/`: chứa Dataset mẫu hoặc hướng dẫn tạo lại Dataset.
- `models/`: chứa mô hình đã huấn luyện và thông số chuẩn hóa.
- `results/`: chứa hình ảnh, biểu đồ và bảng kết quả.
- `docs/`: chứa sơ đồ hệ thống, lưu đồ và tài liệu liên quan.
- `demo/`: chứa chương trình chạy thử mô hình trên dữ liệu mẫu.

## Thông số mô phỏng

- Số anten BS: `4`.
- Số phần tử RIS: `16`.
- Số anten UE: `1`.
- Số trạng thái pha: `8`.
- Số đặc trưng đầu vào: `551`.
- Số Output Head: `16`.
- Mô hình kênh: `TDL-A`, `TDL-B`, `TDL-C`, `TDL-D`, `TDL-E`.
- SNR khảo sát: `-20`, `-10`, `0`, `10`, `20 dB`.
- Doppler Shift khảo sát: `30`, `60`, `120 Hz`.
- Tỷ lệ Train/Validation/Test: `80% / 10% / 10%`.
- Thuật toán tạo nhãn: `Coordinate Descent`.
- Mô hình AI chính: `FC-MultiHead`.
- Môi trường thực thi: `MATLAB`, hỗ trợ chạy bằng CPU.

## Luồng hoạt động tổng quát

1. Thiết lập thông số mô phỏng và vị trí của BS, RIS.
2. Tạo ngẫu nhiên vị trí UE cho từng mẫu dữ liệu.
3. Mô phỏng các kênh `H_BR`, `H_RU`, `H_BU` theo mô hình TDL.
4. Xét ảnh hưởng của path loss, SNR, AWGN và Doppler Shift.
5. Lấy nhiều trạng thái kênh truyền theo Snapshot để tạo đặc trưng đầu vào.
6. Sử dụng Coordinate Descent để tìm pha RIS phù hợp làm nhãn.
7. Chuẩn hóa và chia Dataset thành Train, Validation và Test.
8. Huấn luyện mạng FC-MultiHead.
9. Đánh giá mô hình trên tập Test và trong các điều kiện kênh khác nhau.
10. So sánh thời gian suy luận của AI với thời gian tối ưu bằng Coordinate Descent.

## Tạo Dataset

Đầu vào của mô hình được xây dựng từ nhiều nhóm đặc trưng:

- Thông tin CSI của các liên kết BS–RIS, RIS–UE và BS–UE.
- Phần thực và phần ảo của các hệ số kênh.
- Nhiều trạng thái kênh truyền theo Snapshot.
- Vị trí của UE.
- Khoảng cách giữa các thành phần trong hệ thống.
- Giá trị path loss của các liên kết.
- Các thông số liên quan đến điều kiện mô phỏng.

Sau khi tạo đủ số mẫu, dữ liệu được chuẩn hóa, xáo trộn và chia thành:

- `80%` cho tập Train.
- `10%` cho tập Validation.
- `10%` cho tập Test.

## Tạo nhãn bằng Coordinate Descent

Coordinate Descent lần lượt tối ưu pha của từng phần tử RIS trong khi giữ nguyên pha của các phần tử còn lại.

Tại mỗi phần tử RIS:

1. Thử lần lượt `8` trạng thái pha.
2. Tạo ma trận phản xạ `Θ`.
3. Tính lại kênh truyền hiệu dụng `H_eff`.
4. Tính `||H_eff||²_F`.
5. Giữ lại trạng thái pha tạo ra Metric lớn nhất.
6. Chuyển sang phần tử RIS tiếp theo.

Quá trình được lặp lại qua nhiều vòng quét. Cấu hình pha cuối cùng được sử dụng làm nhãn đầu ra để huấn luyện mô hình AI.

## Kiến trúc mạng FC-MultiHead

Mạng FC-MultiHead gồm một Shared Trunk và 16 Output Head.

### Shared Trunk

- Input: `551` đặc trưng.
- Fully Connected: `512`.
- Batch Normalization.
- ReLU.
- Dropout: `0.30`.
- Fully Connected: `384`.
- Batch Normalization.
- ReLU.
- Dropout: `0.30`.
- Fully Connected: `256`.
- Batch Normalization.
- ReLU.

Shared Trunk có nhiệm vụ trích xuất biểu diễn đặc trưng chung từ dữ liệu đầu vào.

### Output Head

Mỗi Output Head tương ứng với một phần tử RIS và gồm:

- Fully Connected: `64`.
- ReLU.
- Fully Connected: `8 logits`.

Mỗi Head dự đoán một trong 8 trạng thái pha cho một phần tử RIS. Tổng đầu ra của mô hình gồm:

`16 Head × 8 logits = 128 logits`

## Nhãn mềm và hàm mất mát

Mô hình có thể sử dụng Circular Soft Label để phản ánh tính tuần hoàn của pha:

- Trạng thái pha đúng nhận trọng số `0.80`.
- Hai trạng thái pha lân cận nhận trọng số `0.10`.
- Các trạng thái còn lại nhận trọng số `0`.

Cách biểu diễn này giúp mô hình nhận biết rằng hai trạng thái pha lân cận có sai lệch nhỏ hơn các trạng thái nằm xa trên vòng pha.

Hàm mất mát chính được sử dụng là Cross-Entropy, tính trên đầu ra của 16 Head.

## Chuẩn hóa dữ liệu

Dữ liệu đầu vào được chuẩn hóa theo từng đặc trưng bằng công thức:

`X_norm = (X - μ) / σ`

Trong đó:

- `μ`: giá trị trung bình của đặc trưng trên tập Train.
- `σ`: độ lệch chuẩn của đặc trưng trên tập Train.
- `X_norm`: dữ liệu sau chuẩn hóa.

Các giá trị `μ` và `σ` chỉ được tính từ tập Train, sau đó được sử dụng chung cho tập Validation và Test.

## Huấn luyện mô hình

Quá trình huấn luyện sử dụng:

- Optimizer: `Adam`.
- Mini-batch.
- Validation trong quá trình Train.
- Lưu mô hình có kết quả Validation tốt nhất.
- Theo dõi Loss và các chỉ số dự đoán pha theo từng Epoch.

Mô hình được thiết kế để có thể huấn luyện bằng CPU trong trường hợp máy tính không có GPU rời.

## Chỉ số đánh giá

Các chỉ số chính gồm:

- `Loss`: hàm mất mát Cross-Entropy.
- `Element Accuracy`: tỷ lệ dự đoán đúng pha của từng phần tử RIS.
- `Accuracy ±1`: tỷ lệ dự đoán đúng hoặc lệch tối đa một trạng thái pha.
- `Codeword Accuracy`: tỷ lệ dự đoán đúng toàn bộ 16 phần tử RIS.
- `MSE Index`: sai số bình phương trung bình của chỉ số pha.
- `Circular MAE`: sai số tuyệt đối trung bình theo vòng pha.
- `Circular RMSE`: căn sai số bình phương trung bình theo vòng pha.

## Kết quả chính

Trong điều kiện `TDL-A`, `SNR = 20 dB` và `Doppler Shift = 30 Hz`, mô hình FC-MultiHead đạt:

- Test Element Accuracy: `39.70%`.
- Test Accuracy ±1 pha: `80.34%`.
- Thời gian Coordinate Descent trung bình: `1.521242 ms`.
- Thời gian suy luận FC-MultiHead trung bình: `1.128201 ms`.

Kết quả cho thấy mô hình AI có thể rút ngắn thời gian lựa chọn cấu hình pha trong kịch bản thử nghiệm. Lợi thế về thời gian có thể rõ hơn khi cần xử lý đồng thời nhiều trạng thái kênh truyền.

## Ảnh hưởng của điều kiện kênh truyền

- Khi SNR giảm, nhiễu tăng và độ chính xác dự đoán pha giảm.
- Khi Doppler Shift tăng, kênh thay đổi nhanh hơn và bài toán dự đoán trở nên khó hơn.
- Các mô hình TDL khác nhau tạo ra đặc tính kênh và mức độ dự đoán khác nhau.
- FC-MultiHead tạo được sự cân bằng giữa hiệu suất dự đoán và chi phí tính toán.

## Yêu cầu phần mềm

- MATLAB.
- 5G Toolbox.
- Deep Learning Toolbox.
- Statistics and Machine Learning Toolbox.

## Cách chạy chương trình

### Tạo Dataset

Chạy script:

`src/dataset_generation/generate_RIS_dataset.m`

Script sẽ mô phỏng kênh truyền, tạo đặc trưng đầu vào và sinh nhãn bằng Coordinate Descent.

### Huấn luyện mô hình

Chạy script:

`src/training/train_FC_MultiHead.m`

Trước khi chạy, kiểm tra đường dẫn Dataset trong code:

`datasetPath = 'data/RIS_TDLA_Dataset_20dB_30Hz.mat';`

### Đánh giá mô hình

Chạy script:

`src/evaluation/evaluate_FC_MultiHead.m`

### So sánh thời gian xử lý

Chạy script:

`src/evaluation/compare_optimization_time.m`

## Cấu trúc repository

```text
RIS-AI-Phase-Prediction/
├── README.md
├── .gitignore
├── src/
│   ├── dataset_generation/
│   ├── training/
│   ├── evaluation/
│   └── functions/
├── data/
├── models/
├── results/
├── docs/
└── demo/
```

## Dữ liệu và mô hình

Dataset và model MATLAB có thể có dung lượng lớn. Repository nên ưu tiên lưu:

- Code tạo lại Dataset.
- Dataset mẫu có dung lượng nhỏ.
- Model demo nếu kích thước phù hợp.
- Các thông số chuẩn hóa `μ` và `σ`.
- Hình ảnh và bảng kết quả.

Các file `.mat` dung lượng lớn có thể được lưu bằng Git LFS hoặc cung cấp qua liên kết tải riêng.

## Gợi ý phát triển

- Tăng số lượng và độ đa dạng của Dataset.
- Xây dựng một mô hình chung cho nhiều mức SNR và Doppler Shift.
- Mở rộng số lượng phần tử RIS.
- Thử nghiệm CNN, Transformer hoặc Graph Neural Network.
- Sử dụng dữ liệu kênh truyền thực tế.
- Triển khai mô hình trên hệ thống nhúng hoặc phần cứng thời gian thực.
- Tối ưu kích thước mô hình và thời gian suy luận.

## Khắc phục sự cố nhanh

- Nếu MATLAB không nhận `nrTDLChannel`: kiểm tra việc cài đặt 5G Toolbox.
- Nếu không tìm thấy Dataset: kiểm tra lại biến `datasetPath`.
- Nếu không tìm thấy model: kiểm tra lại biến đường dẫn model trong script đánh giá.
- Nếu kích thước đầu vào không bằng `551`: kiểm tra quy trình trích xuất đặc trưng và thông số Snapshot.
- Nếu kết quả thay đổi giữa các lần chạy: kiểm tra giá trị seed của bộ sinh số ngẫu nhiên.
- Nếu hết bộ nhớ: giảm số mẫu, batch size hoặc chia quá trình tạo Dataset thành nhiều phần.

## Tác giả

**Khoa Đăng**

Đồ án tốt nghiệp: Ứng dụng trí tuệ nhân tạo trong dự đoán pha phản xạ RIS cho hệ thống truyền thông vô tuyến.

## Ghi chú

Repository được xây dựng phục vụ mục đích học tập và nghiên cứu. Kết quả có thể thay đổi tùy theo Dataset, thông số mô phỏng, phiên bản MATLAB và cấu hình phần cứng.
