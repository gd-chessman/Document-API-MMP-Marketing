# Tài Liệu API Deposit & Withdraw

## Tổng Quan
Tài liệu này cung cấp chi tiết về các API quản lý giao dịch nạp và rút tiền trong hệ thống.

## Các Endpoint API

### 1. Tạo Giao Dịch Nạp/Rút Tiền
- **Endpoint**: `POST http://localhost:8000/api/v1/deposit-withdraws`
- **Mô tả**: Tạo một giao dịch nạp hoặc rút tiền mới
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**:
  ```json
  {
    "type": "string",        // Loại giao dịch: "deposit" hoặc "withdraw"
    "symbol": "string",      // Loại token: "SOL", "USDT", "USDC", "MMP", "MPB"
    "amount": "number",      // Số lượng token
    "to_address": "string"   // Địa chỉ ví đích (bắt buộc cho WITHDRAW)
  }
  ```
- **Phản hồi**:
  ```json
  {
    "id": "number",
    "wallet_id": "number",
    "from_address": "string",
    "to_address": "string",
    "type": "string",
    "amount": "number",
    "symbol": "string",
    "status": "string",
    "tx_hash": "string",
    "fee_usd": "number",
    "created_at": "Date"
  }
  ```
- **Lưu ý**: 
  - Tự động tạo Associated Token Account (ATA) nếu chưa tồn tại
  - Kiểm tra số dư trước khi thực hiện giao dịch
  - Phí giao dịch SOL: 0.000005 SOL
  - Phí rút tiền: $1 USD (cho SOL, USDT, USDC)
  - Token nội bộ (MMP, MPB) không tính phí rút

### 2. Lấy Lịch Sử Giao Dịch
- **Endpoint**: `GET http://localhost:8000/api/v1/deposit-withdraws`
- **Mô tả**: Lấy danh sách các giao dịch nạp/rút đã hoàn thành
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Query Parameters**:
  - `page`: Số trang (mặc định: 1)
  - `limit`: Số lượng giao dịch mỗi trang (mặc định: 50, tối đa: 50)
- **Phản hồi**:
  ```json
  {
    "data": [
      {
        "id": "number",
        "wallet_id": "number",
        "from_address": "string",
        "to_address": "string",
        "type": "string",
        "amount": "number",
        "symbol": "string",
        "status": "string",
        "tx_hash": "string",
        "fee_usd": "number",
        "created_at": "Date"
      }
    ],
    "total": "number",
    "page": "number",
    "limit": "number"
  }
  ```
- **Lưu ý**: 
  - Chỉ trả về các giao dịch có trạng thái COMPLETED
  - Bao gồm cả giao dịch gửi và nhận
  - Sắp xếp theo thời gian tạo giảm dần

## Các Loại Token Hỗ Trợ

### Token Được Hỗ Trợ
- **SOL**: `So11111111111111111111111111111111111111112`
- **USDT**: `Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB`
- **USDC**: `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`
- **MMP**: Token MMP01 (token nội bộ)
- **MPB**: Token MPB (token nội bộ)

## Loại Giao Dịch
- **DEPOSIT**: Nạp tiền vào ví
- **WITHDRAW**: Rút tiền từ ví

## Trạng Thái Giao Dịch
- **PENDING**: Đang chờ xử lý
- **COMPLETED**: Đã hoàn thành
- **FAILED**: Thất bại

## Phí Giao Dịch

### Phí Rút Tiền
- **SOL**: $1 USD (chuyển đổi theo giá SOL hiện tại)
- **USDT**: $1 USD
- **USDC**: $1 USD
- **MMP**: Không tính phí
- **MPB**: Không tính phí

### Phí Giao Dịch Blockchain
- **SOL**: 0.000005 SOL (phí giao dịch Solana)
- **SPL Token**: 0.000005 SOL (phí giao dịch Solana)
- **Tạo ATA**: 0.0025 SOL (nếu cần tạo Associated Token Account)

## Xử Lý Lỗi

### Lỗi Số Dư Không Đủ
- **SOL**: "Insufficient SOL balance. Required: {total} SOL (including fee), Available: {balance} SOL"
- **SPL Token**: "Insufficient {symbol} balance. Required: {total} {symbol} (including fee), Available: {balance} {symbol}"

### Lỗi Tạo ATA
- "ATA creation fee is 0.0025 SOL"

### Lỗi Địa Chỉ Ví
- "Invalid Solana wallet address"
- "Sender and receiver wallet addresses must be different"

## Lưu Ý Chung
- Tất cả các API đều yêu cầu xác thực thông qua JwtGuestGuard
- Số dư token được lưu với độ chính xác 8 số thập phân
- Tự động tạo Associated Token Account (ATA) nếu chưa tồn tại
- Yêu cầu số dư đủ để thực hiện giao dịch và trả phí
- Phí rút tiền được chuyển vào ví sàn
- Giao dịch được thực hiện trên blockchain Solana
- Hỗ trợ cả token gốc (SOL) và SPL token
- Token nội bộ (MMP, MPB) không tính phí rút tiền 