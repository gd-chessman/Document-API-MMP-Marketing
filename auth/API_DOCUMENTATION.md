# Tài Liệu API Xác Thực

## Tổng Quan
Tài liệu này cung cấp chi tiết về các API xác thực có sẵn trong hệ thống MMP Marketing.

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
  - Mã xác thực có thời hạn 5 phút

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
  - Mã QR có thể quét bằng ứng dụng Google Authenticator

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
- **Lưu ý**: 
  - Kích hoạt xác thực 2 yếu tố sau khi xác minh thành công
  - Cho phép sai số thời gian 30 giây

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
- **Lưu ý**: 
  - Cập nhật email và trạng thái xác thực email từ Google
  - Email phải chưa được sử dụng bởi tài khoản khác

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
- **Lưu ý**: 
  - Tạo và gửi mã xác thực qua Telegram
  - Mã xác thực có thời hạn 10 phút
  - Yêu cầu tài khoản đã có email và Telegram ID

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
- **Lưu ý**: 
  - Cập nhật trạng thái xác thực email sau khi xác minh thành công
  - Mã xác thực chỉ có thể sử dụng một lần

### 10. Đăng Nhập Bằng Phantom
- **Endpoint**: `POST http://localhost:8000/api/v1/auth/login-phantom`
- **Mô tả**: Xác thực người dùng bằng ví Phantom
- **Dữ liệu gửi lên**:
  ```json
  {
    "signature": "string",    // Chữ ký từ Phantom
    "public_key": "string",  // Public key của ví
    "message": "string"      // Message đã ký
  }
  ```
- **Phản hồi**:
  ```json
  {
    "status": true,
    "message": "Login successful"
  }
  ```
- **Lưu ý**:
  - Message phải có định dạng "Login to MMP Platform - {timestamp}"
  - Timestamp không được quá 2 phút
  - Tự động tạo ví nếu chưa tồn tại
  - Chỉ tạo JWT với wallet_id

## Các Guard Xác Thực
Hệ thống sử dụng hai loại JWT guard:
1. JwtGuestGuard - Dùng cho xác thực người dùng thông thường
2. JwtAdminGuard - Dùng cho xác thực người dùng quản trị

## Lưu Ý Chung
- Tất cả các endpoint yêu cầu xác thực đều sử dụng JwtGuestGuard
- Cookie `access_token` được sử dụng để xác thực người dùng
- Phản hồi lỗi tuân theo mã trạng thái HTTP tiêu chuẩn
- Các thông báo lỗi được trả về dưới dạng BadRequestException
- Trong môi trường production:
  - Cookie được set với `secure: true`
  - Cookie được set với `sameSite: 'none'`
  - Cookie có thời hạn từ biến môi trường `COOKIE_EXPIRES_IN` 