# Tài Liệu API User Stakes

## Tổng Quan
Tài liệu này cung cấp chi tiết về các API quản lý staking (khóa token) trong hệ thống MMP Marketing.

## Các Endpoint API

### 1. Stake Bằng Ví Server (S2S)
- **Endpoint**: `POST http://localhost:8000/api/v1/user-stakes/stake-by-wallet`
- **Mô tả**: Stake token MMP bằng private key lưu trên server
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**:
  ```json
  {
    "staking_plan_id": "number",  // ID của gói staking
    "amount_staked": "number",    // Số lượng token MMP muốn stake
    "lock_months": "number"       // Số tháng khóa
  }
  ```
- **Phản hồi**:
  ```json
  {
    "id": "number",
    "wallet_id": "number",
    "staking_plan_id": "number",
    "stake_id": "number",
    "stake_account_pda": "string",
    "staking_tx_signature": "string",
    "unstaking_tx_signature": "string",
    "amount_staked": "number",
    "amount_claimed": "number",
    "start_date": "Date",
    "end_date": "Date",
    "status": "string",
    "created_at": "Date",
    "updated_at": "Date"
  }
  ```
- **Lưu ý**: 
  - Sử dụng private key lưu trên server để ký giao dịch
  - Tự động tạo stake account PDA
  - Giao dịch được thực hiện trên blockchain Solana
  - Token MMP được chuyển vào vault của smart contract

### 2. Unstake Bằng Ví Server (S2S)
- **Endpoint**: `POST http://localhost:8000/api/v1/user-stakes/unstake-by-wallet/:id`
- **Mô tả**: Unstake token MMP bằng private key lưu trên server
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Path Parameters**:
  - `id`: ID của user stake cần unstake
- **Dữ liệu gửi lên**: Không có
- **Phản hồi**: Tương tự như stake, với `unstaking_tx_signature` được cập nhật
- **Lưu ý**: 
  - Chỉ có thể unstake sau khi hết thời gian khóa
  - Token MMP và phần thưởng được trả về ví người dùng

### 3. Chuẩn Bị Giao Dịch Stake (Client-side)
- **Endpoint**: `POST http://localhost:8000/api/v1/user-stakes/prepare-stake-transaction`
- **Mô tả**: Tạo giao dịch stake chưa ký để client ký
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**:
  ```json
  {
    "amount_staked": "number",    // Số lượng token MMP muốn stake
    "lock_months": "number"       // Số tháng khóa
  }
  ```
- **Phản hồi**:
  ```json
  {
    "transaction": "string"       // Giao dịch đã serialize dạng base64
  }
  ```
- **Lưu ý**: 
  - Giao dịch cần được ký bởi ví client (Phantom, Solflare, etc.)
  - Sử dụng cho luồng client-side signing

### 4. Thực Thi Giao Dịch Stake (Client-side)
- **Endpoint**: `POST http://localhost:8000/api/v1/user-stakes/execute-stake-transaction`
- **Mô tả**: Thực thi giao dịch stake đã được client ký
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**:
  ```json
  {
    "signedTransaction": "string",  // Giao dịch đã ký dạng base64
    "staking_plan_id": "number"     // ID của gói staking
  }
  ```
- **Phản hồi**: Tương tự như stake S2S
- **Lưu ý**: 
  - Xác minh chữ ký và nội dung giao dịch
  - Tạo user stake record sau khi xác minh thành công

### 5. Chuẩn Bị Giao Dịch Unstake (Client-side)
- **Endpoint**: `POST http://localhost:8000/api/v1/user-stakes/prepare-unstake-transaction/:id`
- **Mô tả**: Tạo giao dịch unstake chưa ký để client ký
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Path Parameters**:
  - `id`: ID của user stake cần unstake
- **Dữ liệu gửi lên**: Không có
- **Phản hồi**:
  ```json
  {
    "transaction": "string"       // Giao dịch đã serialize dạng base64
  }
  ```
- **Lưu ý**: 
  - Kiểm tra điều kiện unstake (thời gian khóa, trạng thái)
  - Giao dịch cần được ký bởi ví client

### 6. Thực Thi Giao Dịch Unstake (Client-side)
- **Endpoint**: `POST http://localhost:8000/api/v1/user-stakes/execute-unstake-transaction`
- **Mô tả**: Thực thi giao dịch unstake đã được client ký
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**:
  ```json
  {
    "signedTransaction": "string",  // Giao dịch đã ký dạng base64
    "user_stake_id": "number"       // ID của user stake
  }
  ```
- **Phản hồi**: Tương tự như unstake S2S
- **Lưu ý**: 
  - Xác minh chữ ký và nội dung giao dịch
  - Cập nhật trạng thái user stake sau khi xác minh thành công

### 7. Lấy Danh Sách Stake Của Tôi
- **Endpoint**: `GET http://localhost:8000/api/v1/user-stakes`
- **Mô tả**: Lấy danh sách tất cả các gói stake của người dùng
- **Xác thực**: Yêu cầu (JwtGuestGuard)
- **Dữ liệu gửi lên**: Không có
- **Phản hồi**:
  ```json
  [
    {
      "id": "number",
      "wallet_id": "number",
      "staking_plan_id": "number",
      "stake_id": "number",
      "stake_account_pda": "string",
      "staking_tx_signature": "string",
      "unstaking_tx_signature": "string",
      "amount_staked": "number",
      "amount_claimed": "number",
      "start_date": "Date",
      "end_date": "Date",
      "status": "string",
      "created_at": "Date",
      "updated_at": "Date"
    }
  ]
  ```
- **Lưu ý**: 
  - Trả về tất cả các gói stake của người dùng
  - Bao gồm cả stake đang hoạt động và đã hoàn thành

## Trạng Thái User Stake
- **ACTIVE**: Đang hoạt động (chưa hết thời gian khóa)
- **COMPLETED**: Đã hoàn thành (đã unstake)
- **CANCELLED**: Đã hủy

## Cấu Trúc Dữ Liệu

### UserStake Entity
```typescript
{
  id: number;                     // ID của user stake
  wallet_id: number;              // ID của ví
  staking_plan_id: number;        // ID của gói staking
  stake_id: number;               // ID stake trong smart contract
  stake_account_pda: string;      // Địa chỉ PDA của stake account
  staking_tx_signature: string;   // Hash giao dịch stake
  unstaking_tx_signature: string; // Hash giao dịch unstake (có thể null)
  amount_staked: number;          // Số lượng token đã stake
  amount_claimed: number;         // Số lượng token đã claim (có thể null)
  start_date: Date;               // Ngày bắt đầu stake
  end_date: Date;                 // Ngày kết thúc stake
  status: UserStakeStatus;        // Trạng thái stake
  created_at: Date;               // Thời gian tạo
  updated_at: Date;               // Thời gian cập nhật
}
```

## Smart Contract Integration

### Program ID
- Sử dụng `STAKING_PROGRAM_ID` từ biến môi trường
- Smart contract được triển khai trên Solana blockchain

### Token Mint
- Chỉ hỗ trợ token MMP
- Mint address: `MMP_TOKEN_MINT` từ biến môi trường

### PDA (Program Derived Address)
- `stake_state`: Quản lý trạng thái tổng thể của staking
- `user_stake`: Quản lý thông tin stake của từng user
- `vault`: Ví chứa token đã stake

## Lưu Ý Chung
- Tất cả các API đều yêu cầu xác thực thông qua JwtGuestGuard
- Hỗ trợ 2 luồng: Server-to-Server (S2S) và Client-side signing
- Số lượng token được lưu với độ chính xác 8 số thập phân
- Chỉ có thể unstake sau khi hết thời gian khóa
- Giao dịch được thực hiện trên blockchain Solana
- Sử dụng Borsh serialization cho instruction data
- Tự động tạo PDA cho mỗi stake
- Hỗ trợ nhiều gói staking khác nhau 