---
title: "Blog 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# KIẾN TRÚC HYBRID MULTI-TENANT CHO DỊCH VỤ STATEFUL TRÊN AWS

Hôm nay mình muốn chia sẻ một case study khá hay từ AWS Architecture Blog, nói về bài toán kiến trúc multi-tenant cho các dịch vụ stateful (dịch vụ có lưu trạng thái trong bộ nhớ) — cụ thể là hệ thống ad-serving của Amazon Ads, xử lý hàng triệu request/giây.

Điều mình thấy thú vị là bài viết này không đi theo hướng "multi-tenant thuần túy dùng account riêng" hay "multi-tenant share hết tài nguyên", mà đề xuất một mô hình lai (hybrid) cân bằng giữa cô lập (isolation) và hiệu quả vận hành.

## Bài toán ban đầu: Kiến trúc "cellular"

Trước đây, đội ngũ Amazon Ads dùng mô hình mỗi tenant (khách hàng) một AWS account riêng, kèm theo ALB và ECS cluster riêng. Cách này cô lập rất tốt, nhưng phát sinh hàng loạt vấn đề vận hành:

* **Vấn đề scale:** chỉ 18 client trên 4 Region mà đã cần tới 181 target riêng biệt — mỗi client là một bộ account, VPC, load balancer, IAM role, kết nối downstream service.
* **Vấn đề hiệu suất:** server rảnh rỗi hơn 98% thời gian, CPU trung bình chỉ 3%, RAM chỉ 19%. Tức là trả tiền cho hạ tầng khổng lồ nhưng phần lớn thời gian không dùng tới.
* **Vấn đề onboarding:** thêm một client mới mất khoảng 52 ngày — riêng việc tạo AWS account, dựng VPC/network, cấu hình IAM, tích hợp downstream đã ngốn gần 2 tháng.
* **Vấn đề scale-out:** mỗi lần traffic tăng hoặc có client mới, giải pháp duy nhất là dựng nguyên một "cell" mới rồi di dời — không thể chạy đồng thời nhiều sự kiện lớn (tier-1) cùng lúc.
* **Vấn đề "noisy neighbor":** dù đã cố cô lập, các tenant chia sẻ hạ tầng vẫn ảnh hưởng lẫn nhau về hiệu năng.

## Tại sao phải cần compute riêng cho mỗi tenant?

Đây là điểm mấu chốt: hệ thống ad-serving là stateful — nó load và giữ dữ liệu của từng tenant trong bộ nhớ (in-memory) thay vì query database mỗi request, để tối ưu tốc độ. Nhưng chính vì vậy, nếu hai tenant share chung một cluster, dữ liệu in-memory của họ sẽ cạnh tranh chung một vùng heap. Một tenant có dataset lớn có thể gây out-of-memory và ảnh hưởng luôn tenant bên cạnh.

Vì vậy bài toán đặt ra là: làm sao vừa giữ được cô lập ở cấp cluster, vừa giảm được overhead vận hành như mô hình cellular cũ.

## Giải pháp: Kiến trúc phân tầng 3 lớp (Tier – Cell – Infra Group)

Điểm sáng của thiết kế này là tách bạch "hạ tầng dùng chung" ra khỏi "hạ tầng riêng cho tenant":

* **Tier:** lớp phân loại tenant theo đặc điểm traffic (High TPS, Standard TPS, Low TPS).
* **Cell:** là ranh giới một AWS account — đơn vị scale-out theo chiều ngang ở cấp account.
* **Infra group:** đơn vị hạ tầng độc lập bên trong mỗi cell — gồm 1 VPC, 1 ALB, nhiều ECS cluster (mỗi tenant một cluster riêng), IAM role và hệ thống giám sát.

Thiết kế 3 lớp này giải quyết đúng 2 loại giới hạn (quota) khác nhau của AWS: giới hạn target group trên một ALB, và giới hạn ENI/VPC endpoint trên một account. Khi chạm giới hạn nào, thêm đơn vị tương ứng (infra group hoặc cell) mà không cần đập đi xây lại toàn bộ.

![Kiến trúc hybrid multi-tenant cho dịch vụ stateful](/images/3-Blog/blog2.png)

### Các kỹ thuật chính được áp dụng

* **Route 53 Weighted Routing:** một endpoint DNS cố định cho cả tier, phân phối traffic tới nhiều ALB ở nhiều account khác nhau. Muốn scale thêm chỉ cần thêm một weighted record mới — tenant không cần đổi DNS.
* **ALB listener rules theo tenant:** mỗi ALB route request tới đúng ECS cluster của tenant dựa trên path hoặc header, với giới hạn khoảng 50 tenant/infra group (do quota 100 target group và 5 target group/listener rule).
* **ECS cluster riêng cho từng tenant:** đảm bảo in-memory state của tenant này không đụng tenant khác, đồng thời quota 5.000 task/service chỉ áp dụng riêng cho tenant đó.
* **AWS PrivateLink dùng chung:** thay vì mỗi tenant tự thiết lập kết nối riêng tới downstream service, các VPC endpoint được dựng sẵn ở cấp tier — tenant mới tự động thừa hưởng kết nối mà không cần cấu hình lại, giúp giảm 80% bước cấu hình network.

## Kết quả đo được

* Thời gian onboarding tenant: từ 52 ngày xuống còn 7 ngày (giảm 86%)
* Số bước setup hạ tầng/tenant: giảm 80%
* Công sức kỹ sư bỏ ra mỗi lần onboarding: giảm 80%
* Thời gian release tính năng: từ 2-3 ngày xuống còn 1 ngày
* Một account có thể phục vụ tới 100 tenant mà vẫn giữ cô lập ở cấp cluster

## Bài học mình rút ra

Điều mình tâm đắc nhất ở case study này là quyết định thiết kế cốt lõi: tách việc thiết lập dependency ra khỏi quá trình onboarding tenant. Việc pre-wire sẵn PrivateLink, IAM role, cache endpoint ngay từ lúc tạo tier — chứ không phải lúc thêm tenant — mới là thứ biến việc onboarding từ "một dự án hạ tầng kéo dài nhiều tuần" thành "một thao tác đổi config".

Case study này cũng cho thấy:

* Multi-tenant không nhất thiết phải chọn giữa "cô lập tuyệt đối" (mỗi tenant một account) và "share tất cả" — có thể thiết kế lai để lấy điểm cân bằng phù hợp.
* Với hệ thống stateful (giữ dữ liệu trong RAM), cô lập ở cấp compute/cluster quan trọng hơn nhiều so với cô lập ở cấp account.
* Một kiến trúc phân tầng tốt cho bạn nhiều "đòn bẩy" scale độc lập, thay vì chỉ có một cách duy nhất để mở rộng.
* Đầu tư vào pre-provisioning và automation ngay từ đầu giúp tiết kiệm cực lớn về sau, dù ban đầu tốn công thiết kế hơn.

**Nguồn tham khảo:** [AWS Architecture Blog - Building hybrid multi-tenant architecture for stateful services on AWS](https://aws.amazon.com/blogs/architecture/building-hybrid-multi-tenant-architecture-for-stateful-services-on-aws/)

**Facebook:** [AWS Study Group FCJ](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2201873483910945/)