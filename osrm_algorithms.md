# OSRM Routing Engine

Một trong những bài toán cơ bản và quan trọng mà OSRM giải quyết đó là tìm đường đi tốt nhất giữa các điểm trên bản đồ.

Trong OSRM có thể chia bài toán trên thành 2 dạng chính:
- **Direction**: tìm đường đi ngắn nhất giữa hai tọa độ cho trước. OSRM cung cấp **Route Service** để giải quyết vấn đề này.
- **Distance Matrix**: tính toán thời gian và khoảng cách ngắn nhất giữa các cặp tọa độ cho trước. OSRM cung cấp **Table Service** để giải quyết vấn đề nay.

Thuật toán chủ yếu được sử dụng ở đây là *tìm đường đi ngắn nhất* (shortest paths) với dữ liệu đầu vào là *road networks*. Các thuật toán này liên quan chặt chẽ đến lý thuyết đồ thị (graph theory).

Các thuật toán này thường được cài đặt bên trong một **routing engine** mà OSRM là một ví dụ. Các phương thức để tìm đường đi tốt nhất được gọi chung là **routing planning**

Một **routing engine** tốt cần đảm bảo các yếu tố sau:
- a



























