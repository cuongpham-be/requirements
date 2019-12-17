# OSRM API

## Request
Tất cả HTTP request của OSRM đều có chung cấu trúc sau:
```
GET: http://{server}/{service}/{version}/{profile}/{coordinates}[.{format}]?option=value&option=value
```
Trong đó:
|Tham số     | Ý nghĩa | Khoảng giá trị |Ví dụ |
|-|-|-|-|
|server|tên miền hoặc địa chỉ máy chủ | | |
|service|loại dịch vụ OSRM cung cấp | route / nearest / table / match / trip / tile|http://{server}/**route**/{version}/...|
|version|phiên bản api|v1 cho tất cả OSRM 5.x|.../{service}/**v1**/{profile}/...|
|profile|phương thức vận chuyển, giúp chuẩn bị dữ liệu bằng [osrm-extract](https://github.com/Project-OSRM/osrm-backend/tree/master/src/extractor)|car / bike / foot.  Nếu không muốn chọn phương thức, sử dụng 'driving'|.../{version}/**car**/{coordinates}...|
|coordinates|xâu kí tự mô tả các cặp tọa độ (kinh độ, vĩ độ) của các địa điểm thuộc lộ trình |0 < longtitute, latitute <= 180  Định dạng: {longitude},{latitude};{longitude},{latitude}[;{longitude},{latitude} ...] hoặc polyline({polyline}) hoặc polyline6({polyline6})  *polyline* là định dạng [Google's polyline](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) độ chính xác 5 chữ số thập phân và có thể được sinh bằng [npm polyline](https://www.npmjs.com/package/polyline)|Với longitude và latitude: .../car/13.388860,52.517037;13.397634,52.529407;13.428555,52.523219  Với polyline: .../driving/polyline(ofp_Ik_vpAilAyu@te@g`E)|
|format|định dạng response mong muốn|hiện tại chỉ hỗ trợ định dạng json. Mặc định: json|.../{coordinates}**.json**?option=value|
|option| tùy chọn yêu cầu, có thể sử dụng đồng thời nhiều tùy chọn | bearings, radiuses, approaches, exclude|..../{coordinates}[.{format}]**?exclude=motorway**|

Các tùy chọn:
|Tùy chọn| Ý nghĩa|Khoảng giá trị|Ví dụ|
|-|-|-|-|
|bearings|giới hạn tìm kiếm ở phạm vi địa lý dựa trên góc. Mốc là hướng Chính Bắc, xoay theo chiều kim đồng hồ| Định dạng: {{value},{range}};{{value},{range}}[;{{value},{range}} ...].  0 <= value, range <= 180|bearings=0,20 |
|radiuses| giới hạn kết quả trong bán kính tính bằng mét|Định dạng: {radius};{radius}[;{radius} ...]   radius >= 0. Mặc định: không giới hạn|radius=35|
|approaches|Giữ điểm tham chiếu vào lề đường| ĐỊnh dạng: {approach};{approach}[;{approach} ... approach = curd (lề đường). Mặc định: không hạn chế  | approachs=curd |
|exclude|Danh sách các phương thức vận chuyển không muốn có trong kết quả|Định dạng: {class}[,{class}]. class = motorway. Mặc định: không hạn chế| exclude=motorway|

### Chú ý
Định dạng tùy chọn:
```
    {option}={element};{element}[;{element} ... ]
```
Ngoại trừ *exclude*, số lượng *element* phải bằng chính xác số địa điểm (cặp tọa độ) trong lộ trình. Trường hợp không truyền giá trị để sử dụng giá trị mặc định của *element*, hãy để trống.
 Ví dụ: Địa điểm thứ 2 sử dụng giá trị mặc định cho *option* 
 ```
    {option}={element};;{element}
 ```
 
 ## Response
 
 Định dạng:
 ```
{
    "code": "{code}",
    "message": "{message}",
    "data_version": "{data_version}"
    .
    .
    .
}
 ```
 
 Trong đó:
 - data_version: Phiên bản dữ liệu, chứa nhãn thời gian của tệp OSM. Nếu *data_version* không được cài đặt trong *osrm-extract* hoặc tệp OSM không có *osmosis_replication_timestamp*
 - message: Mô tả dễ hiểu về code
 - code: Mọi response đều có thuộc tính code thông báo trạng thái. Code có thể là một trong các xâu kí tự dưới đây:
 

|Code|Ý nghĩa|
|-|-|
|Ok| Yêu cầu đã được xử lý như mong đợi|
|InvalidUrl| Url không hợp lệ|
|InvalidVersion| Phiên bản không tồn tại|
|InvalidOptions| Tùy chọn không hợp lệ|
|InvalidQuery|Tên truy vấn sai định dạng|
|InvalidValue|Truy vấn (gồm chuỗi tên truy vấn và giá trị) sai định dạng|
|NoSegment|Một tham số tọa độ không khớp với đoạn đường|
|TooBig|Kích thước request vượt mức cho phép|
|InvalidService| Tên dịch vụ không hợp lệ|

 ## Các loại API
 
 ### API số liệu khoảng cách (Distance Metric API)
 Tính thời lượng của tuyến đường nhanh nhất giữa tất cả các cặp tọa độ được cung cấp. Trả về thời lượng hoặc khoảng cách hoặc cả hai giữa các cặp tọa độ. Lưu ý rằng khoảng cách không phải là khoảng cách ngắn nhất giữa hai tọa độ, mà là khoảng cách của các tuyến nhanh nhất. Thời lượng tính bằng giây và khoảng cách tính bằng mét.

 #### Định dạng request

```
    /table/v1/{profile}/{coordinates}?{sources}=[{elem ...];&{destinations}=[{elem}...]&annotations={duration|distance|duration,distance}
```

 Ngoài các tùy chọn chung đã giới thiệu ở phần **Request**, chúng ta có thể sử dụng các tùy chọn sau:

|Tùy chọn| Ý nghĩa| Khoảng giá trị|Ví dụ|
|-|-|-|-|
|sources|sử dụng địa điểm với chỉ mục đã cho làm nguồn|all (mặc định) / {index};{index}[;{index} ...]. 0 <= index < #locations|.../{profile}/13.388860,52.517037;13.397634,52.529407;13.428555,52.523219?**sources=0**|
|destinations|sử dụng địa điểm với chỉ mục đã cho làm đích|all (mặc định) / {index};{index}[;{index} ...]. 0 <= index < #locations|.../{profile}/13.388860,52.517037;13.397634,52.529407;13.428555,52.523219?**destinations=1,2**|
|annotations|Trả về  thời lượng hoặc địa điểm hoặc cả thời lượng và địa điểm|duration (mặc định) / distance / hoặc duration,distance|annotations=duration,distance|
|fallback_speed|Nếu không tìm thấy tuyến đường giữa các cặp nguồn - đích, tính khoảng cách đường chim bay rồi chia cho tốc độ này để tính toán thời lượng|fallback_speed > 0|fallback_speed=35.7|
|fallback_coordinate|Khi sử dụng fallback_speed, sử dụng tọa độ do người dùng cung cấp hoặc vị trí lề đường (snap) để tính khoảng cách.| input (mặc định) / snnaped|fallback_coordinate=snnaped|
|scale_factor|Kết hợp với annotations=durations. Nhân duration với hệ số này |scale_factor > 0|scale_factor=3.14|

Số lượng của *sources* và *destinations* nhỏ hơn hoặc bằng số  địa điểm đầu vào

Ví dụ:

```
    sources=0;5;7&destinations=5;1;4;2;3;6
```

##### Ví dụ request

```
# Trả về ma trận thời lượng 3x3:
curl 'http://router.project-osrm.org/table/v1/driving/13.388860,52.517037;13.397634,52.529407;13.428555,52.523219'

# Trả về ma trận thời lượng 1x3
curl 'http://router.project-osrm.org/table/v1/driving/13.388860,52.517037;13.397634,52.529407;13.428555,52.523219?sources=0'

# 
Trả về ma trận thời lượng 3x2 không đối xứng với từ các vị trí được mã hóa đa tuyến (polyline) `qikdcB}~dpXkkHz`:
curl 'http://router.project-osrm.org/table/v1/driving/polyline(egs_Iq_aqAppHzbHulFzeMe`EuvKpnCglA)?sources=0;1;3&destinations=2;4'

# Trả về ma trận thời lượng 3x3:
curl 'http://router.project-osrm.org/table/v1/driving/13.388860,52.517037;13.397634,52.529407;13.428555,52.523219?annotations=duration'

# Trả về ma trận khoảng cách 3x3 cho CH:
curl 'http://router.project-osrm.org/table/v1/driving/13.388860,52.517037;13.397634,52.529407;13.428555,52.523219?annotations=distance'

# Trả về ma trận thời lượng 3x3 và ma trận khoảng cách 3x3 cho CH:
curl 'http://router.project-osrm.org/table/v1/driving/13.388860,52.517037;13.397634,52.529407;13.428555,52.523219?annotations=distance,duration'
```

#### Định dạng response

```
{
    "code" : "{code}",
    "durations":"{waypoints}",
    "distances":"{routes}"
    "sources": "{sources}", 
    "destinations": "{destinations}",
    "fallback_speed_cells":"{fallback_speed_cells}"
}
```

Trong đó:
- *code*: trạng thái response. Ngoài các trạng thái chung, code còn có giá trị "NoTable" mang ý nghĩa "Không tìm thấy tuyến đường" và giá trị "NotImplemented" mang ý nghĩa "Yêu cầu không đươc hỗ trợ"
- *duration*: mảng các mảng lưu trữ dạng ma trận. Thời lượng [i] [j] cho thời gian di chuyển từ điểm thứ i đến điểm thứ j. Các giá trị được đưa ra trong vài giây. Có thể là null nếu không tìm thấy tuyến giữa i và j
- *distances*: mảng của các mảng lưu trữ dạng ma trận. Khoảng cách [i] [j] cho khoảng cách di chuyển từ điểm thứ i đến điểm thứ j. Các giá trị được tính theo mét. Có thể là null nếu không tìm thấy tuyến giữa i và j. Lưu ý rằng tính toán bảng khoảng cách hiện chỉ được triển khai cho CH. Nếu annotations=distance hoặc annotations=duration, khoảng cách được yêu cầu khi chạy bộ định tuyến MLD, lỗi Không được thực hiện sẽ được trả về.
- *sources*: mảng các điểm tham chiếu (*Waypoint*) của tất cả *sources* theo thứ tự
- *destinations*: mảng các điểm tham chiếu (*Waypoint*) của tất cả *destinations* theo thứ tự
- *fallback_speed_cells*(tùy chọn): mảng các mảng chứa cặp i, j chỉ ra ô nào chứa giá trị ước tính dựa trên fallback_speed. Sẽ không có nếu fallback_speed không được sử dụng.

##### Ví dụ response 

```
{
  "sources": [
    {
      "location": [
        13.3888,
        52.517033
      ],
      "hint": "PAMAgEVJAoAUAAAAIAAAAAcAAAAAAAAArss0Qa7LNEHiVIRA4lSEQAoAAAAQAAAABAAAAAAAAADMAAAAAEzMAKlYIQM8TMwArVghAwEA3wps52D3",
      "name": "Friedrichstraße"
    },
    {
      "location": [
        13.397631,
        52.529432
      ],
      "hint": "WIQBgL6mAoAEAAAABgAAAAAAAAA7AAAAhU6PQHvHj0IAAAAAQbyYQgQAAAAGAAAAAAAAADsAAADMAAAAf27MABiJIQOCbswA_4ghAwAAXwVs52D3",
      "name": "Torstraße"
    },
    {
      "location": [
        13.428554,
        52.523239
      ],
      "hint": "7UcAgP___38fAAAAUQAAACYAAABTAAAAhSQKQrXq5kKRbiZCWJo_Qx8AAABRAAAAJgAAAFMAAADMAAAASufMAOdwIQNL58wA03AhAwMAvxBs52D3",
      "name": "Platz der Vereinten Nationen"
    }
  ],
  "durations": [
    [
      0,
      192.6,
      382.8
    ],
    [
      199,
      0,
      283.9
    ],
    [
      344.7,
      222.3,
      0
    ]
  ],
  "destinations": [
    {
      "location": [
        13.3888,
        52.517033
      ],
      "hint": "PAMAgEVJAoAUAAAAIAAAAAcAAAAAAAAArss0Qa7LNEHiVIRA4lSEQAoAAAAQAAAABAAAAAAAAADMAAAAAEzMAKlYIQM8TMwArVghAwEA3wps52D3",
      "name": "Friedrichstraße"
    },
    {
      "location": [
        13.397631,
        52.529432
      ],
      "hint": "WIQBgL6mAoAEAAAABgAAAAAAAAA7AAAAhU6PQHvHj0IAAAAAQbyYQgQAAAAGAAAAAAAAADsAAADMAAAAf27MABiJIQOCbswA_4ghAwAAXwVs52D3",
      "name": "Torstraße"
    },
    {
      "location": [
        13.428554,
        52.523239
      ],
      "hint": "7UcAgP___38fAAAAUQAAACYAAABTAAAAhSQKQrXq5kKRbiZCWJo_Qx8AAABRAAAAJgAAAFMAAADMAAAASufMAOdwIQNL58wA03AhAwMAvxBs52D3",
      "name": "Platz der Vereinten Nationen"
    }
  ],
  "code": "Ok",
  "distances": [
    [
      0,
      1886.89,
      3791.3
    ],
    [
      1824,
      0,
      2838.09
    ],
    [
      3275.36,
      2361.73,
      0
    ]
  ],
  "fallback_speed_cells": [
    [ 0, 1 ],
    [ 1, 0 ]
  ]
}
```

 ### API chi tiết địa điểm (Place Details API)
 ### API chỉ đường (Get Direction API)
 Tìm đường nhanh nhất giữa các tọa độ theo thứ tự yêu cầu.
 
 #### Định dạng request

 ```
 /route/v1/{profile}/{coordinates}?alternatives={true|false|number}&steps={true|false}&geometries={polyline|polyline6|geojson}&overview={full|simplified|false}&annotations={true|false}
```

Ngoài các tùy chọn chung đã giới thiệu ở phần **Request**, chúng ta có thể sử dụng các tùy chọn sau:

|Tùy chọn| Ý nghĩa| Khoảng giá trị|Ví dụ|
|-|-|-|-|
|alternatives|Nhận thêm các tuyến đường thay thế hay không hoặc nhận bao nhiêu tuyến đường thay thế. Dù vậy, kể cả khi được yêu cầu, số tuyến đường thay thế cũng không đảm bảo sẽ có trong kết quả|false (mặc định)/ true / hoặc số nguyên dương |alternatives=2 |
|steps|Trả về các bước cho lộ trình |false (mặc định)|steps=true|
|annotations|Trả về siêu dữ liệu (meta data) bổ sung cho mỗi tọa độ dọc tuyến đường|false (mặc định) / true / nodes / distance / duration / datasources / weight / speed| annotations=true|
|geometries|Định dạng hình học tuyến đường được trả về (ảnh hưởng tổng quan và mỗi bước)| polyline (mặc định) / polyline6 / geojson | geometries=geojson|
|overview|Thêm hình học tổng quan đầy đủ, đơn giản hóa theo mức thu phóng cao nhất có thể hiển thị hoặc không thêm bất cứ hình học nào| simplified (mặc định) / full / false|overview=false|
|continue_straight|Buộc tuyến đường tiếp tục đi thẳng tại các điểm tham chiếu (*waypoints*) ràng buộc ở đó ngay cả khi nó sẽ nhanh hơn. Giá trị mặc định phụ thuộc vào *profile*|default  (mặc định) / true / false|continue_straight=true|
|waypoints|Xử lý tọa độ đầu vào được chỉ định bởi các chỉ số đã cho làm điểm tham chiếu trong đối tượng *Match* được trả về. Mặc định coi tất cả các tọa độ đầu vào là điểm tham chiếu.|Định dạng: {index};{index};{index}...| waypoints=0;1|

##### Ví dụ request

```
# Truy vấn ở Berlin với ba tọa độ và không có hình học tổng quan nào được trả về:
curl 'http://router.project-osrm.org/route/v1/driving/13.388860,52.517037;13.397634,52.529407;13.428555,52.523219?overview=false'
```

#### Định dạng response

```
{
    "code" : "{code}",
    "waypoints":"{waypoints}",
    "routes":"{routes}"
}
```

Trong đó:
- *code* : trạng thái response. Ngoài các trạng thái chung, code còn có giá trị "NoRoute" mang ý nghĩa "Không tìm thấy tuyến đường"
- *waypoints*: mảng các điểm tham chiếu sắp xếp theo thứ tự
- *routes*: mảng các tuyến đường, sắp xếp theo thứ tự giảm dần của xếp hạng đề nghị

##### Ví dụ response 

```
{
    "routes": [
        {
            "legs": [
                {
                    "summary": "",
                    "weight": 707.6,
                    "duration": 497.9,
                    "steps": [],
                    "distance": 1998.6
                },
                {
                    "summary": "",
                    "weight": 714,
                    "duration": 525.9,
                    "steps": [],
                    "distance": 2838.7
                }
            ],
            "weight_name": "routability",
            "weight": 1421.6,
            "duration": 1023.8,
            "distance": 4837.299999999999
        }
    ],
    "waypoints": [
        {
            "hint": "6dftgJrX7YAsAAAA6gIAAAAAAAAAAAAASjFaQU1xpUEAAAAAAAAAACwAAADqAgAAAAAAAAAAAADapQAA_kvMAKlYIQM8TMwArVghAwAA7wqyvkTg",
            "distance": 4.231665624816857,
            "name": "Friedrichstraße",
            "location": [
                13.388798,
                52.517033
            ]
        },
        {
            "hint": "JDEigAOpH4oMAAAADAAAAAAAAAAJAQAAW7-PQOKcyEAAAAAApq6DQgwAAAAMAAAAAAAAAJEAAADapQAAf27MABiJIQOCbswA_4ghAwAAXwWyvkTg",
            "distance": 2.7893928415656375,
            "name": "Torstraße",
            "location": [
                13.397631,
                52.529432
            ]
        },
        {
            "hint": "ZcQfiv___38hAAAAyAAAACgAAABKAAAAsowKQkpQX0Lx6yZCvsQGQiEAAABkAAAAKAAAACUAAADapQAASufMAOdwIQNL58wA03AhAwMAvxCyvkTg",
            "distance": 2.2265954222656257,
            "name": "Platz der Vereinten Nationen",
            "location": [
                13.428554,
                52.523239
            ]
        }
    ],
    "code": "Ok"
}
```


## Tổng kết

### Ưu điểm

- Mã nguồn mở, miễn phí
- Tìm tuyến đường nhanh
- Sinh file nhị phân, tính di động cao
- Import linh hoạt dữ liêu OSM (có thể import custom data và [trafic data](https://github.com/Project-OSRM/osrm-backend/wiki/Traffic)) 
- Hỗ trợ tìm đường dựa trên phương tiện vận tải: ô tô, xe đạp, đi bộ
- Có tính toán thời gian rẽ, quay đầu

### Nhược điểm

- Tài liệu chưa nói chi tiết giai đoạn tiền xử lý của thuật toán tìm đường => cần nghiên cứu mã nguồn C++

### Ứng dụng cho beMap

- OSRM có hiệu năng rất tốt, beMap có thể  học gần như toàn bộ phần tìm đường.
- Ngoài việc cache sẵn kết quả của request để trả về client, có thể  sử dụng kết quả đó cho tuyến đường lớn hơn bao gồm các điểm của request đã cache. Tuy nhiên cần xác định trường hợp nào có thể dùng bằng một công thức đánh giá tuyến đường. 
- Nên kết hợp dịch vụ tính thời lượng, khoảng cách với dịch vụ tìm tuyến đường. Bởi Distance Metrics API của OSRM tính thời lượng, khoảng cách giữa một hoặc nhiều điểm với nhau (1 x 1 hoặc 1 x n hoặc n x 1 hoặc n x m) còn beMap chỉ cần tính giữa 2 điểm (1x1) nếu kết hợp với hướng (từ điểm xuất phát đến mép gần nhất của đích hoặc cổng vào của đích). Nên tách tìm đường và tính thời lượng, khoảng cách ra 2 api là không cần thiết