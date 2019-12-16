# OSRM Routing Engine

Một trong những bài toán cơ bản và quan trọng mà OSRM giải quyết đó là tìm đường đi tốt nhất giữa các điểm trên bản đồ.

Trong OSRM có thể chia bài toán trên thành 2 dạng chính:
- **Direction**: tìm đường đi ngắn nhất giữa hai tọa độ cho trước. OSRM cung cấp **Route Service** để giải quyết vấn đề này.
- **Distance Matrix**: tính toán thời gian và khoảng cách ngắn nhất giữa các cặp tọa độ cho trước. OSRM cung cấp **Table Service** để giải quyết vấn đề này.

Thuật toán chủ yếu được sử dụng ở đây là *tìm đường đi ngắn nhất* (shortest paths) với dữ liệu đầu vào là *road networks*. Các thuật toán này liên quan chặt chẽ đến lý thuyết đồ thị (graph theory).

Các thuật toán này thường được cài đặt bên trong một **routing engine** mà OSRM là một ví dụ. Các phương thức để tìm đường đi tốt nhất được gọi chung là **routing planning**

Một **routing engine** tốt cần đảm bảo các yếu tố sau:
- Routing engine cần kết hợp tất cả các chi tiết của mạng lưới đường bộ, sử dụng các mô hình đã được đơn giản hóa là không đủ. Ví dụ nhiều mô hình loại bỏ các chi tiết như *turn costs* (thời gian quay đầu hoặc rẽ), *restrictions* (đường giới hạn tốc độ, đường cấm). Nhiều thuật toán sẽ có performance khá tốt khi loại bỏ các yếu tố trên, tuy nhiên performance sẽ giảm đi khá nhiều nếu tính toán thêm các yếu tố đó.
- Ngoài hàm mất mát (*cost function* hay *metric*) cơ bản như *thời gian di chuyển* (travel times), routing engine cần xử lý được các loại metric khác như: *khoảng cách ngắn nhất* (shortest distance), *loại hình di chuyển* (transporation type, ví dụ như walking, biking), tránh các U-turns, tránh đường cao tốc, các giới hạn về chiều cao hoặc chiều rộng (height and weight restrictions).
- Routing engine cần có khả năng xử lý nhiều metric đồng thời, và các loại metric mới cần được tích hợp một cách nhanh chóng. Nếu làm được việc này, routing engine có thể làm việc với các metric như thông tin giao thông theo thời gian thực (real-time traffic information).
- Routing engine không chỉ hỗ trợ việc tìm đường ngắn nhất từ một điểm đến một điểm khác (point-to-point shortest paths) mà còn có thể support các tính năng như đưa ra các tuyến đường thay thế (alternative routes).
- Routing engine cần có performance khá tốt và có thể xử lý các routing query gần như thời gian thực, từ đó không gây ra hiện tượng nghẽn cổ chai cho ứng dụng bản đồ.

Các thuật toán routing của OSRM được xây dựng dựa trên một phương pháp có tên **Customizable Route Planning - CRP**, được phát triển bởi các researcher tại Microsoft.

MLD (Multilevel Dijkstra) mà OSRM sử dụng sẽ chỉ là một bước trong CRP. Cụ thể, CRP sẽ gồm các bước chính sau:
- Metric-independent preprocessing: bước này có nhiệm vụ tiếp nhận cấu trúc liên kết đồ thị gốc (original graph topology) và tạo các các dạng dữ liệu phụ trợ. Cụ thể đồ thị gốc sẽ được chia thành các tế bào (cells) nhỏ hơn và sử dụng nhiều lớp overlay khác nhau. Quá trình tiền xử lý này khá chậm và mất khá nhiều thời gian, tuy nhiên nó không được chạy thường xuyên. Kết quả của quá trình này là chuyển đồi đồ thị gốc thành dạng đồ thị đơn giản hơn với chỉ các dữ liệu cần thiết. Trong phần giới thiệu về OSRM, quá trình này được thực hiện sử dụng hai command là `osrm-extract` và `osrm-partition` với nhiệm vụ chuyển đổi dữ liệu từ OpenStreetMap thành các dạng đồ thị đơn giản hơn.
- Metric customization: được thực hiện cho từng metric sử dụng đồ thị được tạo ra từ bước 1. Nhiệm vụ của bước này là xây dựng một overlay graph cho từng metric. Dữ liệu cho từng metric sẽ được lưu trữ riêng biệt nhằm phục vụ cho việc hỗ trợ nhiều metric khác nhau tại một thời điểm cho cùng một mạng lưới đường bộ. Trong phần giới thiệu về OSRM, chúng ta sử dụng command `osrm-customize` để thực hiện bước này.
- Queries: sử dụng Multilevel Dijkstra để tính toán các cung đường ngắn nhất cho từng metric. 

Để đơn giản hóa, chúng ta sẽ hạn chế các công thức toán học trong các phần sau.

Bài toán tìm đương ngắn nhất có thể phát biểu như sau:
- Input: Một đồ thị dùng để biểu diễn mạng lưới đường `G = (V, A)`. Một hàm mất mát `l(v, w)` cho từng arc (cạnh của đồ thị) `arc (v, W)`. Hiểu đơn giản, các đỉnh (vertices) của đồ thị biểu thị các điểm giao nhau trên bản đồ. Trong khi đó, các arcs (cạnh đồ thị hay còn gọi là edge) biểu diễn các cung đường. Các arcs sẽ có trọng số (hay còn gọi là cost) ví dụ như thời gian di chuyển, độ dài hay tình trạng giao thông.
- Một đường đi (path) là một tập hợp các vertices trong đồ thị `(v1, v2, ... , vn)`, trong đó `(v-i, v-i+1)` là một arc trong đồ thị.
- Problem: cho trước hai điểm trên bản đồ: `s` (source), `t` (target), chúng ta cần tìm khoảng cách ngắn nhất giữa hai điểm đó - `shortest dist(s,t)`

Một thuật toán khá cơ bản để giải quyết vấn đề trên là **Dijkstra's**. Tốc độ của thuật toán này khá tốt và phụ thuộc vào việc sử dụng các cấu trúc dữ liệu để lưu trữ vertex tiếp theo mà thuật toán cần quét. Ngoài việc quét các vertext theo chiều tăng dần của khoảng cách (forward scanning), chúng ta có thể tăng tốc thuật toán này sử dụng *bidirectional scanning*. Thuật toán sẽ chạy 2 dạng scan đồng thời, forward scanning từ s đến t và backward scanning từ t đến s. Thuật toán sẽ dừng khi hai quá trình scan gặp nhau.

Tuy nhiên, khi dữ liệu bản đồ lớn, thuật toán Dijkstra's vẫn cần mất tới một vài giây để hoàn thành quá trình tìm đường đi ngắn nhất. Đối với ứng dụng bản đồ, khoảng thời gian đó là không thể áp dụng được. Đó là lý do tại sao, CRP sử dụng bước tiền xử lý để chuyển đổi dạng đồ thị gốc thành *multilevel overlay graphs*.

![](./overlay_graph.png)

Multilevel overlay graph được tạo trong quá trình *graph separators* của CRP sử dụng phương pháp *partition-based overlay graphs*. Hiểu một các đơn giản overlay graphs được xây dựng từ graph gốc bằng cách tạo ra các **clique** (hay các đồ thị con đầy đủ, hay tập hợp các đỉnh của đồ thị con đó). Mục đích của overlay graph là tạo ra các lớp đồ thị đơn giản hơn (có chung đường biên - boundary với đồ thị gốc), sử dụng các shortcuts mà vẫn đảm bảo khoảng cách giữa các đỉnh của đồ thị. Việc tìm đường đi ngắn nhất sẽ được thực hiện trên các lớp overlay đó nhằm tăng tốc độ xử lý của quá trình searching.

Trong phần tiếp theo, các bước trong CRP sẽ được thảo luận. Tuy nhiên chúng ta sẽ không đi quá sau vào vấn đề implementation hoặc các vấn đề toán học bên dưới.

## Metric-independent preprocessing
Nhiệm vụ của bước này trong CRP:
- Thực hiện quá trình phân vùng trên nhiều level (multilevel partitioning) trên đồ thị gốc.
- Xây dựng cấu trúc liên kết (topology) của overlay graphs.
- Xây dựng các cấu trúc dữ liệu phục vụ cho quá trình *Metric customization*.

Sau khi đã phân vùng trên đồ thị gốc, CRP sẽ tạo các cấu trúc dữ liệu (không phụ thuộc vào các metric) cho overlay graph. Mục đích là việc xây dự cấu trúc liên kết của overlay graph chỉ được thực hiện một lần và có thể sử dụng lại cho nhiều loại metric khá nhau. Trong quá trình thực hiện các *queries*, CRP sẽ thực hiện việc chuyển đổi giữa overlay graph và original graph.

## Metric customization
Quá trình nay thực hiện việc đánh trọng số cho đồ thị được tạo ra từ bước 1 cho từng loại metric. Các trọng số này cần được tối ưu trong quá trình thực hiện các *queries*. Việc tính toán trọng số cần được thực hiện nhanh mỗi khi một metric nào đó cần được tối ưu. CRP sử dụng các phương pháp sau để thực hiện việc đó (chi tiết của từng phương pháp có thể tham khảo trong research paper):

- Improving Locality
- Pruning the Search graph
- Alternative Algorithms
- Multiple-source executions
- Parallelism
- Phantom Levels

OSRM sử dụng một số kĩ thuật trên khi cài đặt các thuật toán routing.

## Queries
Queries là bước cuối cung trong CRP mà quá trình tìm đương đi ngắn nhất sẽ được thực hiện. CRP sử dụng một phiên bản đã được custom của Dijkstra's - MLD hay Multilevel Dijkstra's. MLD sẽ được sử dụng cùng với phương pháp *bidirectional search*.

Ngoài việc tìm kiếm đường đi tốt nhất (best route), CRP cũng cho phép tìm kiếm các đường đi thay thế (alternative routes).

Trong các dịch vụ bản đồ hiện nay, việc xử lý thông tin traffic là khá phổ biến. Nếu muốn giải quyết bài toán tìm đường đi tốt nhất dựa trên tình trạng giao thông hiện tại, chúng ta cần thay đổi overlay graph mỗi khi có dữ liệu traffic mới. Một cách đơn giản là thực hiện lại bước *metric customization* cho traffic metric. Tuy nhiên trên thực tế chúng ta chỉ cần chạy lại quá trình *customization* cho các cell bị ảnh hưởng bởi dữ liệu traffic mới.

Quá trình *customization* trong CRP là khá nhanh, do vậy việc thay đổi trọng số đồ thị cho một metric nào đó là khả thi. Một điểm nữa là chúng ta chỉ cần tính toán dữ liệu traffic cho các cạnh đồ thị gần với điểm di chuyển hiện tại. Do dữ liệu traffic có thể đã thay đổi khi chúng ta di chuyển đi xa hơn điểm hiện tại. CRP xử lý việc này bằng cách sử dụng hai hàm mất mát: một hàm với thông tin traffic và một hàm không. CRP sẽ thực hiện việc chuyển đổi giữa hai hàm này sau một khoảng thời gian nhất định. Ngoài ra thông tin traffic có thể được dự đoán dựa trên các thông tin traffic trong quá khứ.

## Graph representation in OSRM
OSRM chuyển đổi dữ liệu từ OpenStreetMap thành một dạng đồ thị với tên gọi *edge-expanded graph*. Quá trình này tương đương với bước 1 và 2 trong CRP.

OSRM sử dụng OSM tags để lấy các thông tin về tốc độ và tính toán thời gian di chuyển cho mỗi edge (arc). Ngoài ra OSRM còn tính toán thêm một số thông số *penalties* (tính bằng giây) cho các đoạn đường rẽ hoặc quay đầu.

Sau khi đã có một đồ thị với các trọng số đầy đủ. OSRM sẽ thực hiện thuật toán Dijkstra's để tìm đường đi tốt nhất trên đồ thị đó.

![](https://cloud.githubusercontent.com/assets/1892250/13710991/1b507fb6-e771-11e5-8710-8d5f16d805ce.gif)

Giả sử dữ liệu từ OSM có dạng như sau:

```
|   |   | d |
| a | b | c |
|   |   | e |
```

Ở đây chúng ta có 2 đường: abc và dce giao nhau tại điểm c

Đầu tiên chúng ta cần chia đồ thị gốc thành cách segment: ab,bc,cd,ce. Với mỗi segment, hai graph node sẽ được tạo cho từng direction:

```
ab, ba
bc, cb
cd, dc
ce, ec
```

Graph edge sẽ được tạo cho các hướng di chuyển có thể giữa các graph node trên:

```
dc-cd
dc-cb
dc-ce
ab-ba
ab-bc
ba-ab
bc-cd
bc-cb
bc-ce
cd-dc
cb-ba
cb-bc
ce-ec
ec-cd
ec-cb
ec-ce
```

Chú ý rằng node kết thúc của một edge sẽ là node bắt đầu của một edge kế tiếp, do chúng ta chỉ có thể di chuyển trong các edge liên tục.

Nhiều dạng u-turns không các edge được tạo ở trên là không khả thi, OSRM sẽ thực hiện việc xóa bỏ các u-turns đó. Ngoài ra OSRM cũng thực hiện việc loại bỏ các turns bị giới hạn nhưng vào đường một chiều hoặc đường cấm.

```text
ab-bc (from ab, continue on bc)
ba-ab (from ba, do a u-turn at a and return on ab)
bc-cd (from bc, turn left onto dc)
bc-ce (from bc, turn right onto ce)
cb-ba (from cb, continue along on ba)
cd-dc (from cd, do a u-turn at d and return on dc)
ce-ec (from ce, do a u turn at e and return on ec)
dc-cb (from dc, turn right onto cd)
dc-ce (from dc, continue on ce)
ec-cd (from ec, continue on cd)
ec-cb (from ec, turn left onto cb)
```

- Graph node: biểu thị hướng của một OSM segment (backward hoặc forward). Nếu segment là bidirectional (2 chiều) sẽ có 2 graph node, và 1 graph node cho oneway segment.

- Graph edge: dùng để liên kết các graph nodes, dùng kể biểu thị việc di chuyển theo một hướng nhất định từ một OSM segment này đên một OSM segment khác.

![text16557](https://cloud.githubusercontent.com/assets/1892250/19296198/1de51b4a-8ff7-11e6-8339-31eb6361338a.png)

> Đối với beMaps, OSRM sẽ được sử dụng như sau:
> - Dữ liệu đầu vào ở dạng **.pbf** từ OpenStreetMap
> - Bước **Metric-independent preprocessing** sẽ được thực hiện dùng tool `osrm-extract` của OSRM
> - Bước **Metric customization** sẽ được thực hiện bởi tool `osrm-customize` của OSRM
> - Bước **Queries** sẽ sử dụng **MLD** trên đồ thị được xây dựng từ các bước trước để thực hiện việc tìm đường đi ngắn nhất.
>
> OSRM algorithms sẽ được thực thi thông qua việc sử dụng OSRM API. Trong đó 2 API sẽ được sử dụng cho beMaps là: **Direction** và **Distance Matrix**.







