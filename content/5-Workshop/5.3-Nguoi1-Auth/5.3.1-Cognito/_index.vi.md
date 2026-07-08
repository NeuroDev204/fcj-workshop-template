---
title : "Amazon Cognito & Auth"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.3.1. </b> "
---

Trong dự án PeriodIQ, **Amazon Cognito** đóng vai trò là "người gác cổng" trung tâm xử lý toàn bộ quá trình định danh và xác thực người dùng.

#### Thiết lập User Pool trên AWS
Tôi đã tạo một Cognito User Pool với các cấu hình chính như sau:
- Yêu cầu xác thực Email thông qua mã OTP (One-Time Password).
- App Client được cấu hình cho ứng dụng Single Page Application (React) mà không sử dụng App Client Secret.
- Tạo sẵn 2 User Pool Groups:
  - **`Admin`**: Dành cho ban quản trị hệ thống.
  - **`Users`**: (Mặc định) Dành cho người dùng phổ thông.

*(Giao diện Cognito User Pool)*
![Cognito User Pool Setup](/images/5-Workshop/5.3-Nguoi1-Auth/01-cognito-user-pool.png)

*(Giao diện tạo Group Admin)*
![Cognito Groups](/images/5-Workshop/5.3-Nguoi1-Auth/02-cognito-groups.png)

---

#### Tích hợp vào Backend (.NET 10)

Để Backend hiểu và chấp nhận Token do Cognito cấp phát, tôi đã thiết lập **JwtBearer Authentication** trong file `Program.cs`. 

Đặc biệt, để hệ thống phân quyền của ASP.NET Core (`Role-Based Access Control`) có thể hoạt động hoàn hảo với Cognito Groups, tôi đã cấu hình `RoleClaimType` ánh xạ trực tiếp vào claim `"cognito:groups"` của AWS:

```csharp
builder.Services
    .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = cognitoAuthority;
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            ValidateIssuer           = true,
            ValidIssuer              = cognitoAuthority,
            ValidAudience            = cognitoAudience,
            ValidateAudience         = false, // Tắt để dùng chung IdToken & AccessToken
            ValidateLifetime         = true,
            RoleClaimType            = "cognito:groups" // <-- Mapping cực kỳ quan trọng
        };
    });
```

Nhờ cấu hình này, bất kỳ Controller nào cần bảo vệ quyền quản trị viên, tôi chỉ việc thêm cờ `[Authorize(Roles = "Admin")]`. Nếu user không có token hợp lệ hoặc không thuộc group `Admin`, ASP.NET Core sẽ lập tức chặn lại và trả về mã lỗi `403 Forbidden` trước khi request chạm tới các Layer bên dưới.

---

#### Xử lý tại Frontend (React + Vite)

Trên Frontend, tôi sử dụng thư viện `amazon-cognito-identity-js` để giao tiếp trực tiếp với AWS. Tôi đã viết file `authService.js` bao bọc (wrapper) toàn bộ các tác vụ: Đăng ký, Đăng nhập, Quên mật khẩu và lấy Refresh Token.

**Khắc phục lỗi Race Condition (Đồng bộ Session):**
Khi React ở chế độ Strict Mode tải lại trang, việc đọc thông tin Session và trích xuất nhóm quyền đôi khi bị rớt nhịp gây mất giao diện Admin. Tôi đã khắc phục bằng cách đọc trực tiếp claim `cognito:groups` từ Payload của Token ngay khi vừa lấy được Session:

```javascript
// Trích đoạn xử lý trong AuthContext.jsx
getCurrentSession()
  .then(async (result) => {
    if (result) {
      const attrs = await getUserAttributes();
      // Đọc trực tiếp từ payload của ID Token để đảm bảo đồng bộ 100%
      const payload = result.session.getIdToken().payload;
      const userGroups = payload['cognito:groups'] || [];
      
      setUser(attrs);
      setGroups(userGroups);
      setIsAuth(true);
    }
  });

// Hàm kiểm tra quyền gọn nhẹ, tái sử dụng khắp mọi component
const isAdmin = groups.includes('Admin');
```

Cuối cùng, mọi thao tác gọi API Backend đều được cấu hình qua một **Axios Interceptor** tự động đính kèm `Authorization: Bearer <Token>` vào Header, mang lại trải nghiệm trong suốt (seamless) cho người dùng cuối.
