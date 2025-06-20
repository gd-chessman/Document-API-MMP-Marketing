# Tài Liệu API Ví

## Tổng Quan
Tài liệu này cung cấp chi tiết về các API quản lý ví trong hệ thống.

## Các Endpoint API

### 1. Lấy Thông Tin Ví Của Tôi
- **Endpoint**: `GET http://localhost:8000/api/v1/wallets/me`
- **Mô tả**: Lấy thông tin ví của người dùng đang đăng nhập
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**: Không có
- **Phản hồi**:
  ```json
  {
    "id": "number",
    "user_id": "string",
    "sol_address": "string",      // Địa chỉ ví Solana
    "balance_sol": "number",      // Số dư SOL (độ chính xác 8 số thập phân)
    "balance_usdt": "number",     // Số dư USDT (độ chính xác 8 số thập phân)
    "balance_usdc": "number",     // Số dư USDC (độ chính xác 8 số thập phân)
    "balance_mmp": "number",      // Số dư token MMP (độ chính xác 8 số thập phân)
    "balance_mpb": "number",      // Số dư token MPB (độ chính xác 8 số thập phân)
    "referral_code": "string",    // Mã giới thiệu (có thể null)
    "wallet_type": "string",      // Loại ví: "normal" hoặc "bj"
    "referred_by": "string",      // Mã giới thiệu của người giới thiệu (có thể null)
    "created_at": "Date",         // Thời gian tạo ví
    "user": {
      "id": "string",
      "full_name": "string",      // Tên đầy đủ (có thể null)
      "email": "string",          // Email (có thể null)
      "is_verified_email": "boolean",
      "is_verified_gg_auth": "boolean",
      "telegram_id": "string",    // ID Telegram (có thể null)
      "created_at": "Date"
    }
  }
  ```
- **Lưu ý**: 
  - Trường `private_key` sẽ bị loại bỏ khỏi phản hồi (được đánh dấu @Exclude)
  - Thông tin ví được lấy từ JWT token trong cookie
  - Tất cả các số dư đều có độ chính xác 8 số thập phân
  - Bao gồm thông tin người dùng liên quan

## Cấu Trúc Dữ Liệu

### Wallet Entity
```typescript
{
  id: number;                     // ID của ví
  user_id: string;                // ID của người dùng sở hữu ví
  user: User;                     // Thông tin người dùng (relation)
  sol_address: string;            // Địa chỉ ví Solana
  private_key: string;            // Private key của ví (được ẩn trong phản hồi)
  balance_sol: number;            // Số dư SOL (độ chính xác 8 số thập phân)
  balance_usdt: number;           // Số dư USDT (độ chính xác 8 số thập phân)
  balance_usdc: number;           // Số dư USDC (độ chính xác 8 số thập phân)
  balance_mmp: number;            // Số dư token MMP (độ chính xác 8 số thập phân)
  balance_mpb: number;            // Số dư token MPB (độ chính xác 8 số thập phân)
  referral_code: string;          // Mã giới thiệu (có thể null)
  wallet_type: string;            // Loại ví: "normal" hoặc "bj"
  referred_by: string;            // Mã giới thiệu của người giới thiệu (có thể null)
  created_at: Date;               // Thời gian tạo ví
}
```

### User Entity (Relation)
```typescript
{
  id: string;                     // ID của người dùng
  full_name: string;              // Tên đầy đủ (có thể null)
  email: string;                  // Email (có thể null)
  is_verified_email: boolean;     // Trạng thái xác thực email
  is_verified_gg_auth: boolean;   // Trạng thái xác thực Google Authenticator
  telegram_id: string;            // ID Telegram (có thể null)
  created_at: Date;               // Thời gian tạo tài khoản
}
```

## Loại Ví
- **normal**: Ví thường (tỷ lệ thưởng giới thiệu 5%)
- **bj**: Ví BJ (tỷ lệ thưởng giới thiệu 10%)

## Lưu Ý Chung
- Tất cả các API đều yêu cầu xác thực thông qua JwtGuestGuard
- Private key của ví luôn được ẩn trong phản hồi API
- Tất cả các số dư đều được lưu với độ chính xác 8 số thập phân (precision: 20, scale: 8)
- Giá trị mặc định của các số dư là 0
- Ví được tự động tạo khi người dùng đăng ký tài khoản mới
- Mỗi ví có một mã giới thiệu duy nhất
- Hệ thống hỗ trợ giới thiệu qua mã giới thiệu 