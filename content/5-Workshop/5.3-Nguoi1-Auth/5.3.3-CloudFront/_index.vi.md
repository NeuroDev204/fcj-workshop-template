---
title : "Amazon CloudFront & S3 Hosting"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3.3. </b> "
---

Để phân phối website React đến tay người dùng với tốc độ cao và độ trễ thấp nhất, tôi đã thiết lập **Amazon CloudFront** đóng vai trò là mạng lưới phân phối nội dung (CDN) toàn cầu. 

CloudFront trong kiến trúc của chúng ta có nhiệm vụ phân luồng (routing) dữ liệu từ hai nguồn (Origins) hoàn toàn khác nhau.

#### 1. Origin S3 (Frontend Static Files)
Website React (SPA) được build và lưu trữ trên một Amazon S3 Bucket. 
- Mọi request mặc định (`Default Cache Behavior: /*`) đều được CloudFront hướng về bucket S3 này.
- Để tăng cường bảo mật, tôi áp dụng **Origin Access Control (OAC)**. Nhờ đó, S3 Bucket có thể khóa hoàn toàn quyền truy cập công khai (Block all public access). Cách duy nhất để đọc file trên S3 là đi xuyên qua CloudFront.
- Do bản chất của Single Page Application (SPA), tôi cũng thiết lập trang báo lỗi mặc định trên CloudFront: Khi có lỗi `404 Not Found` (do user nhập trực tiếp đường dẫn con của React Router), CloudFront sẽ trả về mã `200 OK` kèm nội dung file `/index.html` để React tự điều hướng giao diện.

#### 2. Origin API Gateway (Backend APIs)
Mọi request có tiền tố `/api/*` sẽ không đi vào S3 mà được định tuyến đến **Amazon API Gateway** (đứng trước .NET Lambda Backend).
- Đối với đường dẫn này, tôi vô hiệu hóa tính năng caching (Cache Disabled) để luôn lấy dữ liệu thực tế mới nhất.
- Tôi đã cấu hình cho phép chuyển tiếp tất cả các HTTP Headers quan trọng (như `Authorization: Bearer ...`) từ người dùng sang Backend để đảm bảo xác thực danh tính chính xác.

*(Cấu hình CloudFront Origins và OAC cho S3 Bucket Policy))*
![CloudFront Origins Setup và S3 OAC Policy](/images/5-Workshop/5.3-Nguoi1-Auth/05-cloudfront-origins.png)
