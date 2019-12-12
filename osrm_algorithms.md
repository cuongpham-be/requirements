# OSRM Routing Engine

Một trong những bài toán cơ bản và quan trọng mà OSRM giải quyết đó là tìm đường đi tốt nhất giữa các điểm trên bản đồ.

Trong OSRM có thể chia bài toán trên thành 2 dạng chính:
- **Direction**: tìm đường đi ngắn nhất giữa hai tọa độ cho trước. OSRM cung cấp **Route Service** để giải quyết vấn đề này.
- **Distance Matrix**: tính toán thời gian và khoảng cách ngắn nhất giữa các cặp tọa độ cho trước. OSRM cung cấp **Table Service** để giải quyết vấn đề nay.

Thuật toán chủ yếu được sử dụng ở đây là *tìm đường đi ngắn nhất* (shortest paths) với dữ liệu đầu vào là *road networks*. Các thuật toán này liên quan chặt chẽ đến lý thuyết đồ thị (graph theory).

Các thuật toán này thường được cài đặt bên trong một **routing engine** mà OSRM là một ví dụ. Các phương thức để tìm đường đi tốt nhất được gọi chung là **routing planning**

Một **routing engine** tốt cần đảm bảo các yếu tố sau:
- Routing engine cần kết hợp tất cả các chi tiết của mạng lưới đường bộ, sử dụng các mô hình đã được đơn giản hóa là không đủ. Ví dụ nhiều mô hình loại bỏ các chi tiết như *turn costs* (thời gian quay đầu hoặc rẽ), *restrictions* (đường giới hạn tốc độ, đường cấm). Nhiều thuật toán sẽ có performance khá tốt khi loại bỏ các yếu tố trên, tuy nhiên performance sẽ giảm đi khá nhiều nếu tính toán thêm các yếu tố đó.
- Ngoài hàm mất mát (*cost function* hay *metric*) cơ bản như *thời gian di chuyển* (travel times), routing engine cần xử lý được các loại metric khác như: *khoảng cách ngắn nhất* (shortest distance), *loại hình di chuyển* (transporation type, ví dụ như walking, biking), tránh các U-turns, tránh đường cao tốc, các giới hạn về chiều cao hoặc chiều rộng (height and weight restrictions).
- Routing engine cần có khả năng xử lý nhiều metric đồng thời, và các loại metric mới cần được tích hợp một cách nhanh chóng. Nếu làm được việc này, routing engine có thể làm việc với các metric như thông tin giao thông theo thời gian thực (real-time traffic information).
- Routing engine không chỉ hỗ trợ việc tìm đường ngắn nhất từ một điểm đến một điểm khác (point-to-point shortest paths) mà còn có thể support các tính năng như đưa ra các tuyến đường thay thế (alternative routes).
- Routing engine cần có performance khá tốt và có thể xử lý các routing query gần như thời gian thực, từ đó không gây ra hiện tượng nghẽn cổ chai cho ứng dụng bản đồ.



























