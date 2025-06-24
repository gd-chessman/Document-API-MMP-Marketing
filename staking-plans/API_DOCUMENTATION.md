# Tài Liệu API Staking Plans

## Tổng Quan
Tài liệu này cung cấp chi tiết về các API quản lý gói staking (staking plans) trong hệ thống MMP Marketing.

## Các Endpoint API

### 1. Lấy Danh Sách Gói Staking
- **Endpoint**: `GET http://localhost:8000/api/v1/staking-plans`
- **Mô tả**: Lấy danh sách tất cả các gói staking đang hoạt động
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**: Không có
- **Phản hồi**:
  ```json
  [
    {
      "id": "number",
      "name": "string",
      "interest_rate": "number",
      "period_days": "number",
      "is_active": "boolean",
      "created_at": "Date"
    }
  ]
  ```
- **Lưu ý**: 
  - Chỉ trả về các gói staking có `is_active: true`
  - Sắp xếp theo thứ tự tạo
  - Sử dụng để hiển thị danh sách gói staking cho người dùng chọn

## Cấu Trúc Dữ Liệu

### StakingPlan Entity
```typescript
{
  id: number;                     // ID của gói staking
  name: string;                   // Tên gói staking
  interest_rate: number;          // Lãi suất hàng năm (ví dụ: 0.12 = 12%)
  period_days: number;            // Thời gian khóa tính bằng ngày
  is_active: boolean;             // Trạng thái hoạt động của gói
  created_at: Date;               // Thời gian tạo gói
}
```

## Ví Dụ Gói Staking

### Gói Staking Ngắn Hạn
```json
{
  "id": 1,
  "name": "Staking 30 ngày",
  "interest_rate": 0.08,
  "period_days": 30,
  "is_active": true,
  "created_at": "2024-01-01T00:00:00.000Z"
}
```

### Gói Staking Trung Hạn
```json
{
  "id": 2,
  "name": "Staking 90 ngày",
  "interest_rate": 0.12,
  "period_days": 90,
  "is_active": true,
  "created_at": "2024-01-01T00:00:00.000Z"
}
```

### Gói Staking Dài Hạn
```json
{
  "id": 3,
  "name": "Staking 180 ngày",
  "interest_rate": 0.18,
  "period_days": 180,
  "is_active": true,
  "created_at": "2024-01-01T00:00:00.000Z"
}
```

## Tính Toán Lợi Nhuận

### Công Thức Tính Lợi Nhuận
```
Lợi nhuận = Số lượng token stake × Lãi suất × (Số ngày khóa / 365)
```

### Ví Dụ Tính Toán
- **Gói**: Staking 90 ngày với lãi suất 12%
- **Số lượng stake**: 1000 MMP
- **Lợi nhuận**: 1000 × 0.12 × (90/365) = 29.59 MMP

## Lưu Ý Chung
- Tất cả các API đều yêu cầu xác thực thông qua JwtGuestGuard
- Chỉ hiển thị các gói staking đang hoạt động (`is_active: true`)
- Lãi suất được tính theo năm (APY - Annual Percentage Yield)
- Thời gian khóa được tính bằng ngày
- Gói staking được sử dụng trong API user-stakes để xác định thời gian khóa
- Có thể có nhiều gói staking với thời gian khóa và lãi suất khác nhau
- Gói staking được tạo bởi admin và không thể thay đổi bởi người dùng 