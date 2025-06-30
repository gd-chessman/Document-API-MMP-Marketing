# Tài Liệu API Referral Clicks

## Tổng Quan
Tài liệu này cung cấp chi tiết về các API quản lý thống kê click giới thiệu trong hệ thống MMP Marketing.

## Các Endpoint API

### 1. Theo Dõi Click Giới Thiệu
- **Endpoint**: `POST http://localhost:8000/api/v1/referral-clicks`
- **Mô tả**: Ghi lại một click vào link giới thiệu
- **Xác thực**: Không yêu cầu
- **Dữ liệu gửi lên**:
  ```json
  {
    "referral_code": "string"  // Mã giới thiệu (chính xác 8 ký tự)
  }
  ```
- **Phản hồi**:
  ```json
  {
    "success": "boolean",
    "message": "string"
  }
  ```
- **Phản hồi thành công**:
  ```json
  {
    "success": true,
    "message": "Click tracked successfully"
  }
  ```
- **Phản hồi lỗi**:
  ```json
  {
    "success": false,
    "message": "Invalid referral code"
  }
  ```
- **Lưu ý**: 
  - Không yêu cầu xác thực, có thể gọi từ frontend
  - Tự động tạo thống kê nếu chưa tồn tại
  - Cập nhật tất cả các counter (total, today, week, month)
  - Cập nhật thời gian click cuối cùng

## Cấu Trúc Dữ Liệu

### ReferralClick Entity
```typescript
{
  id: number;                     // ID của click stats
  wallet_id: number;              // ID của ví (unique)
  wallet: Wallet;                 // Thông tin ví (relation)
  total_clicks: number;           // Tổng số click
  clicks_today: number;           // Số click hôm nay
  clicks_this_week: number;       // Số click tuần này
  clicks_this_month: number;      // Số click tháng này
  last_click_at: Date;            // Thời gian click cuối cùng
  created_at: Date;               // Thời gian tạo
  updated_at: Date;               // Thời gian cập nhật
}
```

## Cách Hoạt Động

### Quy Trình Theo Dõi Click
1. **Nhận referral code**: API nhận referral code từ frontend
2. **Tìm ví**: Tìm ví tương ứng với referral code
3. **Kiểm tra tồn tại**: Kiểm tra xem đã có thống kê click chưa
4. **Tạo mới hoặc cập nhật**: Tạo mới nếu chưa có, cập nhật nếu đã có
5. **Tăng counter**: Tăng tất cả các counter liên quan
6. **Cập nhật thời gian**: Cập nhật thời gian click cuối cùng

### Các Counter Được Cập Nhật
- **total_clicks**: Tổng số click từ khi bắt đầu
- **clicks_today**: Số click trong ngày hôm nay
- **clicks_this_week**: Số click trong tuần này
- **clicks_this_month**: Số click trong tháng này

## Validation

### Referral Code Validation
- **Độ dài**: Chính xác 8 ký tự
- **Kiểu dữ liệu**: String
- **Bắt buộc**: Có
- **Format**: Không có ký tự đặc biệt

### Ví Dụ Referral Code Hợp Lệ
```json
{
  "referral_code": "ABC12345"
}
```

### Ví Dụ Referral Code Không Hợp Lệ
```json
{
  "referral_code": "ABC123"  // Quá ngắn
}
```

## Xử Lý Lỗi

### Lỗi Referral Code Không Tồn Tại
- **HTTP Status**: 404 Not Found
- **Message**: "Invalid referral code"
- **Nguyên nhân**: Referral code không tồn tại trong hệ thống

### Lỗi Validation
- **HTTP Status**: 400 Bad Request
- **Message**: Validation error message
- **Nguyên nhân**: Referral code không đúng format

## Tích Hợp Với Frontend

### Sử Dụng Trong Link Giới Thiệu
```javascript
// Ví dụ sử dụng trong React
const handleReferralClick = async (referralCode) => {
  try {
    const response = await fetch('/api/v1/referral-clicks', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        referral_code: referralCode
      })
    });
    
    const result = await response.json();
    if (result.success) {
      console.log('Click tracked successfully');
    }
  } catch (error) {
    console.error('Error tracking click:', error);
  }
};
```

### Sử Dụng Trong URL Tracking
```javascript
// Ví dụ tracking từ URL parameter
const urlParams = new URLSearchParams(window.location.search);
const referralCode = urlParams.get('ref');

if (referralCode) {
  handleReferralClick(referralCode);
}
```

## Lưu Ý Chung
- API không yêu cầu xác thực để dễ dàng tích hợp với frontend
- Tự động tạo thống kê cho ví mới khi có click đầu tiên
- Có thể sử dụng để phân tích hiệu quả của các chiến dịch giới thiệu
- Dữ liệu được lưu trữ để phân tích xu hướng theo thời gian
- Có thể tích hợp với hệ thống analytics khác
- Hỗ trợ tracking từ nhiều nguồn khác nhau (web, mobile, social media) 