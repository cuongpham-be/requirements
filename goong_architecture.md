# Goong System Architecture

## Goong API system architecture

![](./goong_api_arc.png)

Kiến trúc hệ thống của Goong API gồm các thành phần chính sau:
- Reverse Proxy: sử dụng IIS, NGINX và Apache, có nhiệm vụ tiếp nhận các request truyền đến và chuyển tiếp các request đó đến *Goong Core API*. Thành phần này sẽ làm tăng tính bảo mật do thông tin các service đứng sau sẽ không được public ra bên ngoài. Ngoài ra các tác vụ như authentication, request caching, request logging, request compressing. Ngoài ra reverse proxy sẽ giúp việc scale hệ thống dễ dàng hơn thông qua việc sử dụng Load Balancing.
- Goong Core API: xây dựng trên nền tảng .NET, sử dụng Kestrel webserver cho việc tiếp nhận và xử lý các request từ reverse proxy.
- Routing service: thực hiện các tác vụ như tìm đường ngắn nhất giữa 2 cặp toạ độ.
- Searching service: thực hiện các tác vụ như geocoding, reverse geocoding, autocomple search, lấy chi tiết của một place nào đó.

Goong REST API reference: https://docs.goong.io/rest/guide

## Goong system components

![](./goong_components.png)

Cấu trúc hệ thống của Goong được chia thành 3 tầng chính sau:
- Tầng dữ liệu (Data layer)
- Tầng nghiệp vụ (Business layer)
- Tầng giao diện người dùng (UI layer)

### Tầng dữ liệu
Tầng này có nhiệm vụ xây dựng kho cơ sở dữ liệu bản đồ cho Goong. Goong thu thập dữ liệu bản đồ qua khá nhiều nguồn khác nhau. Trong đó có hai nguồn dữ liệu chính sau:
- Các điểm POIs
- Dữ liệu đường phố

Các điểm POIs (ngân hàng, trạm y tế, trạm xăng, trạm sửa xe,...) sẽ được thu thập bằng các phương pháp sau:
- Thực hiện các bài khảo sát sử dụng các thiết bị có chức năng GPS như smart phone. Trong đó bài khảo sát sẽ yêu cầu người sử dụng cung cấp các thông tin liên quan đến một tọa độ lat/lon nào đó. Phương pháp này đòi hỏi sự ủng hộ và đóng góp từ phía người dùng. Một số công cụ survey điển hình: [Survey123 for ArcGIS](https://survey123.arcgis.com)
- Tạo crawlers và crawl dữ liệu từ các dịch vụ bản đồ khác như: [Coccoc Map](https://map.coccoc.com), [Foursquare](https://foursquare.com)
- Thu thập từ các nguồn khác như GOV, từ mobile app của Goong, Autocad,...

Các dữ liệu đường phố sẽ được thu thập bằng các phương pháp sau:
- Thực hiện các bài khảo sát tương tự như POIs. Dữ liệu của các bài khảo sát có thể ở dạng hình ảnh hoặc video. Dữ liệu đó sau đó có thể được dùng làm input cho các thuật toán machine learning để giải quyết các bài toán như: xác định đèn báo hiệu giao thông, xác định đường cấm đường một chiều thông qua các biển báo giao thông,...
- Sử dụng các dữ liệu hình ảnh từ vệ tinh (từ Google, OpenStreetMap, Microsoft) để làm giàu và bổ sung dữ liệu cho Goong.

Dữ liệu từ các bước thu thập trên sẽ là các shapefiles và các thông tin metadata. Dữ liệu này sẽ được dùng để:
- Lưu trữ trong hệ cơ sở dữ liệu quan hệ PostgreSQL phụ vụ cho việc searching các dữ liệu liên quan đến địa điểm đường phố, các điểm POIs,...
- Dùng như dữ liệu đầu vào cho các phần mềm như ArcGIS. ArcGIS là một bộ phần mềm ứng dụng bao gồm ArcMap, ArcCatalog, ArcToolbox, ModelBuilder, ArcScene và ArcGlobe dùng để giải quyết các bài toán ứng dụng GIS như thành lập bản đồ, phân tích địa lý, chỉnh sửa và biên tập dữ liệu bản đồ,...

### Tầng nghiệp vụ
Tầng này cung cấp các giao diện ứng dụng (application interface) để các bên thứ 3 có thể sử dụng Goong để tích hợp vào ứng dụng của họ.

Tầng này gồm 3 phần chính sau:
- Goong REST API: cung cấp một số API endpoints để thực hiện các việc liên quan đến searching và routing. Người dùng sẽ phải trả phí để sử dụng các API này. Reference: https://docs.goong.io/rest/guide
- Map SDK cho các môi trường như web, mobile application (Android and iOS). [Goong JavaScript SDK](https://docs.goong.io/js/guide), [Goong Android SDK](https://docs.goong.io/android/guide), [Goong iOS SDK](https://docs.goong.io/ios/guide).
- Map View: cho phép thao các với các thành phần trên bản đồ, như zoom, pan, street view,...

### Tần giao diện người dùng
Tâng này chủ yếu là hệ sinh thái mà Goong xây dựng trên nền của tầng nghiệp vụ, bao gồm
- Goong Live Map - https://maps.goong.io
- Goong Android và iOS apps

## Goong and OSM

Việc thu thập dữ liệu:
- Giống nhau: OSM và Goong đều sử dụng phương pháp khảo sát để thu thập dữ liệu bản đồ về.
- Khác nhau: OSM cung cấp các công cụ cho phép người dùng submit những thay đổi về bản đồ lên OpenStreetMap, trong khi đó Goong không có công cụ này. Dữ liệu POIs từ Goong được thu thập chủ động thông qua các nguồn đáng tin cậy hơn, trong khi dữ liệu OSM là khá ngheo nàn và không có độ chính xác cao. Goong có support việc hiển thị thông tin traffic,trong khí đó OSM không support nguồn dữ liêu này.

Lưu trữ dữ liệu:
- Giống nhau: OSM và Goong đều sử dụng PostgreSQL là công cụ để lưu trữ dữ liệu bản đồ.
- Khác nhau: OSM ngoài dữ liệu bản đồ và các map tile cần phải xử lý việc export dữ liệu bản đồ ra các định dạng như .osm, .pbf, lưu trữ và update dữ liệu đó, cũng như cho phép người dùng download và sử dụng các nguồn dữ liệu bản đồ đó.


SDK:
- Khác nhau: OSM và Goong đều cung cấp các REST API. Tuy nhiên các API của OSM tập trung chủ yếu vào việc thay đổi dữ liệu bản đồ. Trong khi đó Goong API phụ vụ chủ yếu các công việc như searching và routing, OSM phải dựa vào các tool bên thứ ba để làm các việc đó như OSRM (routing) và Nominatim (searching). OSM không cung cấp các SDK cho môi trường mobile.

Mục đích sử dụng
- OSM: cung cấp nguồn dữ liệu bản đồ mã nguồn mở, mọi người đều có thể đóng góp
- Goong: là phần mềm doanh nghiệp, tập trung vào giải quyết các bài toán cụ thể như dẫn đường, tìm kiếm các điểm POIs,...

Map layers:
- Goong có 2 layer chính là transport layer và Satellite layer
- OSM ngoài 2 layer trên có thể hiển thị thêm các layer khác sử dụng các bên thứ 3.

Map rendering:
- Goong sử dụng Mapbox GL và một số tool như Leaflet để tạo interactive map
- OSM sử dụng các tool như Mapnik Leaflet để render map.



















