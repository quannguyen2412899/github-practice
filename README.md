# 🌳 SCAD-TSA: NÉN VÀ PHÁT HIỆN CHUỖI BẤT THƯỜNG BẰNG TRIE VÀ PHÂN TÍCH THỐNG KÊ

### 🎓 Thông tin đề tài
| Học phần | Cấu trúc dữ liệu và Giải thuật Mở rộng|
| :--- | :--- |
| **Sinh viên** | Nguyễn Anh Quân<br>Trần Tấn Phát |

### 🌐 Kho lưu trữ và Tài liệu
| Mô tả | Liên kết |
| :--- | :--- |
| **GitHub Repository** | [https://github.com/quannguyen2412899/SCAD-TSA](https://github.com/quannguyen2412899/SCAD-TSA) |
| Google Colab Notebook | *[Chưa có - Cần cập nhật sau]* |

---

### Tóm tắt hệ thống
- Hệ thống sử dụng cấu trúc dữ liệu **Trie** để nén dữ liệu và áp dụng các đặc trưng thống kê như **tần suất**, **độ dài** và **entropy cục bộ** tại mỗi nút để xác định các chuỗi có cấu trúc hoặc hành vi bất thường (**anomalies**).
- Phương pháp phát hiện bất thường sử dụng kỹ thuật **Phần vị (percentile)**, đảm bảo tính vững chắc trước sự lệch của phân phối tần suất chuỗi.

### 💡 Ý tưởng Cốt lõi và Lợi ích của Trie
Hệ thống sử dụng cấu trúc dữ liệu **Trie** với hai mục đích chính:
1.  **Nén Dữ liệu:** Chia sẻ prefix chung giữa các chuỗi, giúp tiết kiệm bộ nhớ và đạt được mức nén đáng kể trên các tập dữ liệu log/URL lớn.
2.  **Lưu trữ Thống kê Hiệu quả:** Mỗi nút (Node) trên Trie là một vị trí lý tưởng để lưu trữ các đặc trưng thống kê cục bộ (như **Tần suất**, **Độ dài**, **Entropy**) cần thiết cho việc phân tích nhánh.

### 🔬 Phương pháp Phát hiện Bất thường
Thay vì sử dụng các chỉ số thống kê truyền thống như Mean/Standard Deviation (dễ bị ảnh hưởng bởi dữ liệu lệch), hệ thống áp dụng phương pháp **Phần vị (Percentile)** để xác định ngưỡng một cách vững chắc.

Bất thường được xác định dựa trên ba tiêu chí chính:
| Tiêu chí | Đặc trưng | Ngưỡng Quyết định | Giải thích |
| :--- | :--- | :--- | :--- |
| **Bất thường Tần suất** | Tần suất | **Thấp hơn P5** | Chuỗi hiếm, xuất hiện không đủ thường xuyên để được coi là mẫu chuẩn. |
| **Bất thường Độ dài** | Tần suất độ dài | **Thấp hơn P5** | Chuỗi có độ dài hiếm, ít gặp trong phân phối tần suất độ dài chung. |
| **Bất thường Entropy** | Entropy Cục bộ | **Cao hơn P95** | Node có sự phân nhánh quá mức, chỉ ra sự đa dạng ký tự bất thường trong chuỗi. |

*Các giá trị P5 và P95 được tính dựa trên phân phối trọng số của toàn bộ dữ liệu, đảm bảo ngưỡng ổn định ngay cả khi dữ liệu đầu vào bị lệch mạnh.*

---
### 🏗️ Sơ đồ Pipeline Tổng thể

Hệ thống được thiết kế dưới dạng một chuỗi lệnh (pipeline) được điều phối bởi executable **`main_pipeline`**. Dữ liệu được xử lý tuần tự qua ba module chính:

1.  **Tiền xử lý (Preprocess):** Chuẩn hóa dữ liệu thô.
2.  **Phân tích (Analyze):** Xây dựng Trie, tính toán thống kê và phát hiện bất thường.
3.  **Trực quan hóa (Visualize):** Chuyển đổi kết quả JSON thành biểu đồ cây Trie dạng PNG/SVG.

### Luồng Dữ liệu giữa các Module

| Executable | Chức năng Chính | Đầu vào | Đầu ra Chính |
| :--- | :--- | :--- | :--- |
| **`main_pipeline`** | **Điều phối** các module, xử lý flags (`--visual-x`, `--regex=`). | `<input_file>`, `<output_dir>` | Là các đầu ra của 3 module còn lại |
| **`bin/preprocess`** | Chuẩn hóa, làm sạch chuỗi. | `<input_file>` | `cleaned_input_text.txt` |
| **`bin/analyze`** | Xây dựng Trie, tính toán Phần vị, đánh dấu bất thường, xuất báo cáo. | `cleaned_input_text.txt` | `overall_report.txt`<br>`all_entries.csv`<br>**{3 file CSV Anomaly}**<br>**{Các file JSON cho Visualize}** |
| **`bin/visualize`** | Wrapper C++ gọi script Python để render biểu đồ Graphviz. | `*.json` (từ Analyze) | `*.png` (Biểu đồ Trie) |

### Chi tiết các File Output Quan trọng

Module **`bin/analyze`** tạo ra các kết quả định lượng sau:
* `overall_report.txt`: Báo cáo tóm tắt chứa các **ngưỡng Phần vị (P5, P95)** được tính toán thực tế.
* `all_entries.csv`: Bảng thống kê đầy đủ (Count, Length, Entropy) của TẤT CẢ các chuỗi/tiền tố duy nhất.
* `frequency_anomalies.csv`: Danh sách các chuỗi bất thường về tần suất.
* `length_anomalies.csv`: Danh sách các chuỗi bất thường về độ dài.
* `entropy_anomalies.csv`: Danh sách các chuỗi bất thường về entropy.
* `*.json`: Các file chứa cấu trúc Trie đã được lọc/đánh dấu, dùng làm đầu vào cho module `visualize`.

---
### 🔧 Yêu cầu Hệ thống và Phụ thuộc

Để chạy toàn bộ pipeline, hệ thống của bạn cần đáp ứng các yêu cầu sau:

#### 1. Biên dịch C++ (Analysis & Pipeline)
* **Compiler:** GNU C++ Compiler (g++), hỗ trợ chuẩn C++17 trở lên.
* **Thư viện Bên ngoài:** **nlohmann/json**
    * Thư viện này được sử dụng trong module **`bin/analyze`** để xuất các file JSON cấu trúc Trie.
    * *Cách sử dụng:* File header `json.hpp` được đặt trong thư mục `include/` của dự án và được link vào lúc biên dịch.

#### 2. Môi trường Python (Visualization)
* **Python:** Python 3.9.
* **Thư viện Python:** **`graphviz`**
    ```bash
    pip install graphviz
    ```

#### 3. Công cụ Hệ thống (Visualization Engine)
**Graphviz Engine (dot):** Cần có công cụ dòng lệnh **`dot`** được cài đặt trên hệ thống của bạn. Công cụ này chịu trách nhiệm chuyển đổi file DOT (được sinh ra bởi script Python) thành hình ảnh PNG/SVG.

* **Trên Debian/Ubuntu:** `sudo apt-get install graphviz`
* **Trên macOS (Homebrew):** `brew install graphviz`
* **Hoặc tải trực tiếp từ website:** `https://graphviz.org/`
    
*Lưu ý:* Nếu chạy trên Google Colab, bạn có thể cần dùng lệnh `!apt install graphviz` trong notebook.

### 🏗️ Biên dịch (Dành cho Local và Colab)

```bash
# Bước 1: Tạo thư mục chứa các executable
mkdir -p bin

# Bước 2: Biên dịch các module C++
g++ -std=c++17 -I./include src/preprocess_module.cpp -o bin/preprocess
g++ -std=c++17 -I./include src/analyze_module.cpp -o bin/analyze
g++ -std=c++17 -I./include src/visualize_module.cpp -o bin/visualize
g++ -std=c++17 -I./include src/main_pipeline.cpp -o main_pipeline
```
