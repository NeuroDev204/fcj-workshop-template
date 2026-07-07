---
title : "User Profile & Unit Tests"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.3.4. </b> "
---

Một phần không thể thiếu trong vai trò của tôi là lập trình (Development) Backend API và Frontend xử lý trực tiếp thông tin người dùng. Phần việc này nhằm đảm bảo ngay sau khi xác thực thành công trên AWS Cognito, người dùng có thể lưu trữ dữ liệu cá nhân của mình vào DynamoDB.

#### 1. Xây dựng UserProfilesController
Tôi đã viết các Endpoints quản trị hồ sơ cá nhân (`/api/userprofiles/me`) dựa trên nguyên lý bảo mật cao nhất:
- **Zero-trust input:** API không bao giờ tin tưởng Client gửi `UserId` lên. Thay vào đó, nó tự động trích xuất mã định danh từ Token (`ClaimTypes.NameIdentifier` hoặc `"sub"` của Cognito) để đảm bảo User chỉ có thể tương tác với đúng hồ sơ của bản thân.
- Xử lý mượt mà các tình huống tạo mới (POST) khi vừa đăng ký xong, hoặc cập nhật (PUT) khi user thay đổi thông số như Cân nặng, Mục tiêu tập luyện.

#### 2. Viết Unit Tests kiểm định tính đúng đắn
Trách nhiệm cuối cùng của tôi là đảm bảo đoạn code xử lý Auth & Profile này chạy chuẩn trong mọi kịch bản. Tôi đã áp dụng thư viện `xUnit` và `Moq` để viết toàn bộ Unit Test cho `UserProfilesController`.

Đặc biệt, tôi đã giả lập (Mock) một **ClaimsPrincipal** y hệt như cấu trúc JWT của Cognito để đưa vào bối cảnh test (Controller Context):
```csharp
private void SetupUserClaims(string sub, string email, string[]? groups = null)
{
    var claims = new List<Claim> { new("sub", sub), new("email", email) };
    if (groups != null)
    {
        foreach (var g in groups)
            claims.Add(new("cognito:groups", g));
    }

    var identity  = new ClaimsIdentity(claims, "TestAuth");
    var principal = new ClaimsPrincipal(identity);
    _controller.ControllerContext = new ControllerContext {
        HttpContext = new DefaultHttpContext { User = principal }
    };
}
```

Kết quả: Toàn bộ **15 Test Cases** thuộc Controller của tôi đều đạt **PASSED 100%**, bao trùm từ nhánh thành công (`200 OK`) cho đến các biên lỗi (`409 Conflict`, `404 Not Found`).

*(Bảng kết quả Unit Test xanh)*
![Unit Tests Results](/images/5-Workshop/5.3-Nguoi1-Auth/06-unit-tests-passed.png)
