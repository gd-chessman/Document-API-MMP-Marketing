# Tài Liệu API Swap Order

## Tổng Quan
Tài liệu này cung cấp chi tiết về các API quản lý giao dịch swap token trong hệ thống.

## Các Endpoint API

### 1. Tạo Swap Order
- **Endpoint**: `POST http://localhost:8000/api/v1/swap-orders`
- **Mô tả**: Tạo một giao dịch swap token mới
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**:
  ```json
  {
    "input_token": "string",    // Loại token đầu vào (SOL, USDT, USDC)
    "output_token": "string",   // Loại token đầu ra (MMP, MPB)
    "input_amount": "number"    // Số lượng token đầu vào
  }
  ```
- **Phản hồi**:
  ```json
  {
    "id": "number",
    "wallet_id": "number",
    "input_token": "string",
    "output_token": "string",
    "input_amount": "number",
    "swap_rate": "number",
    "status": "string",
    "mmp_received": "number",    // Số lượng MMP nhận được (nếu output là MMP)
    "mpb_received": "number",    // Số lượng MPB nhận được (nếu output là MPB)
    "tx_hash_send": "string",    // Hash giao dịch gửi token
    "tx_hash_ref": "string",     // Hash giao dịch nhận token
    "created_at": "Date"
  }
  ```
- **Lưu ý**: 
  - Tự động tạo Associated Token Account (ATA) nếu chưa tồn tại
  - Tự động chuyển token từ ví người dùng đến ví đích
  - Tự động gửi token MMP/MPB đến ví người dùng
  - Yêu cầu số dư đủ để thực hiện giao dịch và trả phí

### 2. Lấy Danh Sách Swap Order Của Tôi
- **Endpoint**: `GET http://localhost:8000/api/v1/swap-orders`
- **Mô tả**: Lấy danh sách các giao dịch swap đã hoàn thành của người dùng
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**: Không có
- **Phản hồi**:
  ```json
  [
    {
      "id": "number",
      "wallet_id": "number",
      "input_token": "string",
      "output_token": "string",
      "input_amount": "number",
      "swap_rate": "number",
      "status": "string",
      "mmp_received": "number",
      "mpb_received": "number",
      "tx_hash_send": "string",
      "tx_hash_ref": "string",
      "created_at": "Date"
    }
  ]
  ```
- **Lưu ý**: 
  - Chỉ trả về các giao dịch có trạng thái COMPLETED
  - Sắp xếp theo thời gian tạo giảm dần

### 3. Khởi Tạo Swap Với Web3 Wallet
- **Endpoint**: `POST http://localhost:8000/api/v1/swap-orders/web3-wallet`
- **Mô tả**: Khởi tạo giao dịch swap sử dụng ví Web3 (Phantom, Solflare, etc.)
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**:
  ```json
  {
    "publicKey": "string",      // Public key của ví Web3
    "inputToken": "string",     // Loại token đầu vào (SOL, USDT, USDC)
    "outputToken": "string",    // Loại token đầu ra (MMP, MPB)
    "inputAmount": "number"     // Số lượng token đầu vào
  }
  ```
- **Phản hồi**:
  ```json
  {
    "orderId": "number",        // ID của swap order
    "serializedTx": "string"    // Transaction đã serialize để ký
  }
  ```
- **Lưu ý**: 
  - Transaction cần được ký bởi ví Web3
  - Tự động tạo ATA nếu chưa tồn tại
  - Kiểm tra số dư trước khi tạo transaction

### 4. Hoàn Thành Swap Với Web3 Wallet
- **Endpoint**: `POST http://localhost:8000/api/v1/swap-orders/web3-wallet/complete`
- **Mô tả**: Hoàn thành giao dịch swap sau khi đã ký transaction
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**:
  ```json
  {
    "orderId": "number",        // ID của swap order
    "signature": "string"       // Chữ ký của transaction
  }
  ```
- **Phản hồi**:
  ```json
  {
    "id": "number",
    "wallet_id": "number",
    "input_token": "string",
    "output_token": "string",
    "input_amount": "number",
    "swap_rate": "number",
    "status": "string",
    "mmp_received": "number",
    "mpb_received": "number",
    "tx_hash_send": "string",
    "tx_hash_ref": "string",
    "created_at": "Date"
  }
  ```
- **Lưu ý**: 
  - Xác minh chữ ký và nội dung transaction
  - Tự động gửi token MMP/MPB sau khi xác minh thành công
  - Cập nhật trạng thái order thành COMPLETED

### 5. Lấy Giá SOL
- **Endpoint**: `GET http://localhost:8000/api/v1/swap-orders/sol-price`
- **Mô tả**: Lấy giá hiện tại của SOL theo USD
- **Xác thực**: Không yêu cầu
- **Dữ liệu gửi lên**: Không có
- **Phản hồi**:
  ```json
  {
    "price_usd": "number"  // Giá SOL theo USD
  }
  ```
- **Lưu ý**: 
  - Cache giá trong 15 giây
  - Lấy giá từ Jupiter API
  - Sử dụng giá cache nếu API không khả dụng

## Các Loại Token Hỗ Trợ

### Token Đầu Vào
- SOL: `So11111111111111111111111111111111111111112`
- USDT: `Es9vMFrzaCERmJfrF4H2FYD4KCoNkY11McCe8BenwNYB`
- USDC: `EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v`

### Token Đầu Ra
- MMP: Token MMP01 (1 MMP01 = 0.001 USD)
- MPB: Token MPB (1 MPB = 0.001 USD)

## Trạng Thái Swap Order
- PENDING: Đang chờ xử lý
- COMPLETED: Đã hoàn thành
- FAILED: Thất bại

## Lưu Ý Chung
- Tất cả các API đều yêu cầu xác thực thông qua JwtGuestGuard (trừ API lấy giá SOL)
- Số dư token được lưu với độ chính xác 8 số thập phân
- Tự động tạo Associated Token Account (ATA) nếu chưa tồn tại
- Yêu cầu số dư đủ để thực hiện giao dịch và trả phí
- Hỗ trợ cả ví thông thường và ví Web3
- Tự động gửi token MMP/MPB sau khi xác minh giao dịch thành công 