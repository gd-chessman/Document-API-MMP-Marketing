# Tài Liệu API Referral Rewards

## Tổng Quan
Tài liệu này cung cấp chi tiết về các API quản lý hệ thống giới thiệu và thưởng trong hệ thống MMP Marketing.

## Các Endpoint API

### 1. Lấy Danh Sách Phần Thưởng Giới Thiệu Của Tôi
- **Endpoint**: `GET http://localhost:8000/api/v1/referral-rewards`
- **Mô tả**: Lấy danh sách các phần thưởng giới thiệu đã được thanh toán
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Query Parameters**:
  - `page`: Số trang (mặc định: 1)
  - `limit`: Số lượng giao dịch mỗi trang (mặc định: 50)
- **Phản hồi**:
  ```json
  {
    "data": [
      {
        "id": "number",
        "reward_amount": "number",
        "reward_token": "string",
        "status": "string",
        "tx_hash": "string",
        "created_at": "Date",
        "referred_wallet": {
          "sol_address": "string"
        }
      }
    ],
    "total": "number",
    "page": "number",
    "limit": "number"
  }
  ```
- **Lưu ý**: 
  - Chỉ trả về các phần thưởng có trạng thái PAID
  - Bao gồm thông tin ví của người được giới thiệu
  - Sắp xếp theo thời gian tạo giảm dần

### 2. Lấy Danh Sách Phần Thưởng Theo Địa Chỉ Ví
- **Endpoint**: `GET http://localhost:8000/api/v1/referral-rewards/by-address/:walletAddress`
- **Mô tả**: Lấy danh sách các phần thưởng giới thiệu theo địa chỉ ví cụ thể
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Path Parameters**:
  - `walletAddress`: Địa chỉ ví Solana cần tra cứu
- **Query Parameters**:
  - `page`: Số trang (mặc định: 1)
  - `limit`: Số lượng giao dịch mỗi trang (mặc định: 50)
- **Phản hồi**:
  ```json
  {
    "data": [
      {
        "id": "number",
        "reward_amount": "number",
        "reward_token": "string",
        "status": "string",
        "tx_hash": "string",
        "created_at": "Date",
        "referred_wallet": {
          "sol_address": "string"
        }
      }
    ],
    "total": "number",
    "page": "number",
    "limit": "number"
  }
  ```
- **Lưu ý**: 
  - Chỉ trả về các phần thưởng có trạng thái PAID
  - Tra cứu theo địa chỉ ví Solana cụ thể
  - Sắp xếp theo thời gian tạo giảm dần

### 3. Lấy Thống Kê Giới Thiệu
- **Endpoint**: `GET http://localhost:8000/api/v1/referral-rewards/statistics`
- **Mô tả**: Lấy thống kê tổng quan về hệ thống giới thiệu
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**: Không có
- **Phản hồi**:
  ```json
  {
    "total_referrals": "number",
    "total_reward_sol": "number",
    "total_reward_mmp": "number",
    "total_reward_mpb": "number",
    "total_pending_reward_sol": "number",
    "total_pending_reward_mmp": "number",
    "total_wait_balance_reward_sol": "number",
    "total_wait_balance_reward_mmp": "number",
    "referred_wallets": [
      {
        "wallet_id": "number",
        "sol_address": "string",
        "created_at": "Date",
        "total_reward_sol": "number",
        "total_reward_mmp": "number",
        "total_reward_mpb": "number",
        "pending_reward_sol": "number",
        "pending_reward_mmp": "number",
        "wait_balance_reward_sol": "number",
        "wait_balance_reward_mmp": "number"
      }
    ]
  }
  ```
- **Lưu ý**: 
  - Chỉ tính các phần thưởng đã được thanh toán (PAID) cho các trường `total_reward_*`
  - Các trường `pending_reward_*` tính phần thưởng đang chờ xử lý (PENDING)
  - Các trường `wait_balance_reward_*` tính phần thưởng đang chờ balance (WAIT_BALANCE)
  - Tổng số người giới thiệu là số lượng ví unique đã được giới thiệu
  - Thống kê theo từng loại token thưởng
  - Bao gồm danh sách chi tiết các ví được giới thiệu và phần thưởng tương ứng

## Hệ Thống Giới Thiệu

### Cách Hoạt Động
1. **Đăng ký với mã giới thiệu**: Người dùng đăng ký tài khoản với `ref_code`
2. **Tạo mã giới thiệu**: Mỗi ví có một `referral_code` duy nhất
3. **Thực hiện swap**: Khi người được giới thiệu thực hiện swap
4. **Tự động thưởng**: Hệ thống tự động tạo và thanh toán phần thưởng

### Tỷ Lệ Thưởng
- **Ví thường**: 10% (0.10) của số lượng token nhận được
- **Ví BJ**: 20% (0.20) của số lượng token nhận được

### Loại Token Thưởng
- **Token nội bộ (MMP/MPB)**: Thưởng bằng cùng loại token
- **SOL**: Thưởng bằng SOL
- **USDT/USDC**: Thưởng bằng SOL (quy đổi theo tỷ giá hiện tại)

## Trạng Thái Phần Thưởng
- **PENDING**: Đang chờ thanh toán
- **PAID**: Đã thanh toán thành công
- **FAILED**: Thanh toán thất bại
- **WAIT_BALANCE**: Đang chờ balance (chỉ áp dụng cho SOL rewards)

## Cơ Chế Thanh Toán Tự Động

### Quy Trình Thanh Toán
1. **Tạo phần thưởng**: Khi swap thành công, hệ thống tự động tạo referral reward
2. **Kiểm tra ngưỡng**: Chỉ thanh toán khi tổng MMP nhận được đạt ngưỡng 5,000
3. **Gửi token**: Sử dụng authority keypair để gửi token từ ví hệ thống
4. **Cập nhật trạng thái**: Chuyển trạng thái thành PAID sau khi gửi thành công

### Xử Lý Lỗi
- Nếu gửi token thất bại, trạng thái sẽ chuyển thành FAILED
- Đối với SOL rewards, nếu lỗi "insufficient funds for rent", trạng thái sẽ chuyển thành WAIT_BALANCE
- Hệ thống không tự động thử lại các giao dịch thất bại
- Cần can thiệp thủ công để xử lý các giao dịch FAILED

## Các Loại Token Hỗ Trợ

### Token Thưởng
- **SOL**: Token gốc của Solana
- **MMP**: Token MMP01 (token nội bộ)

### Token Đầu Vào (Từ Swap)
- **SOL**: Thưởng bằng SOL
- **USDT**: Thưởng bằng SOL (quy đổi)
- **USDC**: Thưởng bằng SOL (quy đổi)
- **MMP**: Thưởng bằng MMP
- **MPB**: Thưởng bằng MMP (với tỷ lệ giảm 50%)

## Cấu Trúc Dữ Liệu Thống Kê

### Referred Wallet Statistics
```typescript
{
  wallet_id: number;              // ID của ví được giới thiệu
  sol_address: string;            // Địa chỉ ví Solana
  created_at: Date;               // Thời gian tạo ví
  total_reward_sol: number;       // Tổng phần thưởng SOL đã thanh toán từ ví này
  total_reward_mmp: number;       // Tổng phần thưởng MMP đã thanh toán từ ví này
  total_reward_mpb: number;       // Tổng phần thưởng MPB đã thanh toán từ ví này (luôn = 0)
  pending_reward_sol: number;     // Tổng phần thưởng SOL đang chờ xử lý từ ví này
  pending_reward_mmp: number;     // Tổng phần thưởng MMP đang chờ xử lý từ ví này
  wait_balance_reward_sol: number; // Tổng phần thưởng SOL đang chờ balance từ ví này
  wait_balance_reward_mmp: number; // Tổng phần thưởng MMP đang chờ balance từ ví này
}
```

### Tổng Thống Kê
```typescript
{
  total_referrals: number;        // Tổng số người được giới thiệu
  total_reward_sol: number;       // Tổng phần thưởng SOL đã thanh toán
  total_reward_mmp: number;       // Tổng phần thưởng MMP đã thanh toán
  total_reward_mpb: number;       // Tổng phần thưởng MPB đã thanh toán (luôn = 0)
  total_pending_reward_sol: number; // Tổng phần thưởng SOL đang chờ xử lý
  total_pending_reward_mmp: number; // Tổng phần thưởng MMP đang chờ xử lý
  total_wait_balance_reward_sol: number; // Tổng phần thưởng SOL đang chờ balance
  total_wait_balance_reward_mmp: number; // Tổng phần thưởng MMP đang chờ balance
  referred_wallets: ReferredWalletDto[]; // Danh sách chi tiết các ví được giới thiệu
}
```

## Lưu Ý Chung
- Tất cả các API đều yêu cầu xác thực thông qua JwtGuestGuard
- Phần thưởng được tính dựa trên số lượng token mà người được giới thiệu nhận được
- Hệ thống tự động thanh toán khi tổng MMP nhận được đạt ngưỡng 5,000
- Chỉ tính phần thưởng cho các giao dịch swap thành công
- Mỗi swap order chỉ tạo một lần referral reward
- Phần thưởng được gửi trực tiếp từ ví authority của hệ thống
- Không tạo ATA cho người nhận, yêu cầu người nhận đã có ATA
- Sử dụng authority keypair để ký các giao dịch thưởng
- Thống kê bao gồm chi tiết từng ví được giới thiệu và phần thưởng tương ứng
- Endpoint by-address cho phép tra cứu phần thưởng của bất kỳ ví nào trong hệ thống
- Các trường pending và wait_balance giúp theo dõi tình trạng thưởng chưa được thanh toán 
