# Tài Liệu API Xác Thực

## Tổng Quan
Tài liệu này cung cấp chi tiết về các API xác thực có sẵn trong hệ thống.

## Các Endpoint API

### 1. Đăng Nhập Bằng Telegram
- **Endpoint**: `POST http://localhost:8000/api/v1/auth/login-telegram`
- **Mô tả**: Xác thực người dùng bằng thông tin đăng nhập Telegram
- **Dữ liệu gửi lên**:
  ```json
  {
    "id": "string",    // ID của người dùng Telegram
    "code": "string"   // Mã xác thực từ Telegram
  }
  ```
- **Phản hồi**:
  ```json
  {
    "success": true,
    "message": "Login successful"
  }
  ```
- **Lưu ý**: 
  - Tạo cookie `access_token` với thời hạn từ biến môi trường `COOKIE_EXPIRES_IN`
  - Tự động tạo ví Solana nếu chưa có

### 2. Đăng Xuất
- **Endpoint**: `POST http://localhost:8000/api/v1/auth/logout`
- **Mô tả**: Đăng xuất người dùng hiện tại
- **Dữ liệu gửi lên**: Không có
- **Phản hồi**:
  ```json
  {
    "success": true,
    "message": "Logout successful"
  }
  ```
- **Lưu ý**: Xóa cookie `access_token`

### 3. Đăng Nhập Bằng Google
- **Endpoint**: `POST http://localhost:8000/api/v1/auth/login-email`
- **Mô tả**: Xác thực người dùng bằng thông tin đăng nhập Google
- **Dữ liệu gửi lên**:
  ```json
  {
    "code": "string"  // Mã xác thực từ Google
  }
  ```
- **Phản hồi**:
  ```json
  {
    "status": true,
    "message": "Account created and login successful" // hoặc "Login successful" nếu là tài khoản cũ
  }
  ```
- **Lưu ý**: 
  - Tạo cookie `access_token`
  - Tự động tạo ví Solana cho tài khoản mới
  - Lưu thông tin email và tên đầy đủ từ Google

### 4. Thêm Xác Thực Google
- **Endpoint**: `POST http://localhost:8000/api/v1/auth/add-gg-auth`
- **Mô tả**: Thêm xác thực Google vào tài khoản hiện có
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**: Không có
- **Phản hồi**:
  ```json
  {
    "status": true,
    "message": "Google Authenticator setup successfully",
    "qr_code_url": "string",  // URL mã QR để quét
    "secret_key": "string"    // Khóa bí mật để cấu hình Google Authenticator
  }
  ```
- **Lưu ý**: 
  - Tạo secret key mới cho Google Authenticator
  - Lưu secret key vào database nhưng chưa kích hoạt

### 5. Xác Minh Xác Thực Google
- **Endpoint**: `POST http://localhost:8000/api/v1/auth/verify-gg-auth`
- **Mô tả**: Xác minh mã xác thực Google
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**:
  ```json
  {
    "code": "string"  // Mã 6 số từ Google Authenticator
  }
  ```
- **Phản hồi**:
  ```json
  {
    "status": true,
    "message": "Google Authenticator verified successfully"
  }
  ```
- **Lưu ý**: Kích hoạt xác thực 2 yếu tố sau khi xác minh thành công

### 6. Xóa Xác Thực Google
- **Endpoint**: `DELETE http://localhost:8000/api/v1/auth/remove-gg-auth`
- **Mô tả**: Xóa xác thực Google khỏi tài khoản
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**: Không có
- **Phản hồi**:
  ```json
  {
    "status": true,
    "message": "Google Authenticator removed successfully"
  }
  ```
- **Lưu ý**: Xóa secret key và tắt xác thực 2 yếu tố

### 7. Thêm Liên kết Email vào tài khoản Telegram đã có
- **Endpoint**: `POST http://localhost:8000/api/v1/auth/add-link-email`
- **Mô tả**: Thêm xác thực email vào tài khoản
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**:
  ```json
  {
    "code": "string"  // Mã xác thực từ Google
  }
  ```
- **Phản hồi**:
  ```json
  {
    "status": true,
    "message": "Email authentication added successfully"
  }
  ```
- **Lưu ý**: Cập nhật email và trạng thái xác thực email từ Google

### 8. Gửi Mã Xác Minh vào Telegram để thực hiện liên kết với email
- **Endpoint**: `GET http://localhost:8000/api/v1/auth/send-verify-email`
- **Mô tả**: Gửi mã xác minh cho người dùng
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**: Không có
- **Phản hồi**:
  ```json
  {
    "status": true,
    "message": "Verification email sent successfully"
  }
  ```
- **Lưu ý**: Tạo và gửi mã xác thực qua Telegram

### 9. Xác Minh liên kết tài khoản
- **Endpoint**: `POST http://localhost:8000/api/v1/auth/verify-code-email`
- **Mô tả**: Xác minh mã xác thực email
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**:
  ```json
  {
    "code": "string"  // Mã xác thực email
  }
  ```
- **Phản hồi**:
  ```json
  {
    "status": true,
    "message": "Email verified successfully"
  }
  ```
- **Lưu ý**: Cập nhật trạng thái xác thực email sau khi xác minh thành công

## Các Guard Xác Thực
Hệ thống sử dụng hai loại JWT guard:
1. JwtGuestGuard - Dùng cho xác thực người dùng thông thường
2. JwtAdminGuard - Dùng cho xác thực người dùng quản trị

## Lưu Ý Chung
- Tất cả các endpoint yêu cầu xác thực đều sử dụng JwtGuestGuard
- Cookie `access_token` được sử dụng để xác thực người dùng
- Phản hồi lỗi tuân theo mã trạng thái HTTP tiêu chuẩn
- Các thông báo lỗi được trả về dưới dạng BadRequestException 