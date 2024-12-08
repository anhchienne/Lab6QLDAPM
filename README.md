# Lab6QLDAPM

# Project Title

A brief description of what this project does and who it's for

# RESTful API Documentation

## I. RESTful API

### 1. Lấy giá Funding Rate
**Method:** `GET`  
**Endpoint:** `/api/v1/funding-rate`  
**Params:** `symbol`

**Cơ chế:**  
Fetch API từ: `https://fapi.binance.com/fapi/v1/fundingInfo`. Khi người dùng gọi API đến server, server sẽ:
- Lấy thông tin `symbol` từ người dùng nhập vào.
- Truy xuất thông tin qua API từ bên thứ 3 (Binance).

**Xử lý:**
- **Trường hợp thiếu hoặc không hợp lệ:**
  - Thiếu trường `symbol`: Trả về **400 Bad Request** với thông báo lỗi phù hợp.
  - `symbol` không tồn tại: Trả về **400 Bad Request** với thông báo lỗi phù hợp.
- **Trường hợp hợp lệ:** Trả về **200 OK** và các thông tin liên quan đến `symbol`, bao gồm:
  - `fundingRate`: Giá trị funding rate của `symbol` nhập vào.
  - `fundingCountDown`: Thời gian đếm ngược để reset.
  - `eventTime`: Thời gian lúc gọi API.
  - `adjustedFundingRateCap`: Mức tối đa mà tỷ lệ tài trợ điều chỉnh có thể đạt được.
  - `adjustedFundingRateFloor`: Giới hạn tối thiểu (floor) cho tỷ lệ tài trợ điều chỉnh.
  - `fundingIntervalHours`: Khoảng thời gian giữa các lần thanh toán tỷ lệ tài trợ (thường là 8 giờ).

---

### 2. Lấy giá Kline
**Method:** `GET`  
**Endpoint:** `/api/vip1/kline`  
**Headers:** `Authorization`  
**Params:** `symbol`, `interval`

**Cơ chế:**  
Fetch API từ: `https://fapi.binance.com/fapi/v1/klines`. Khi người dùng gọi API đến server, server sẽ:
- Lấy thông tin `symbol`, `interval` từ người dùng nhập vào.
- Truy xuất thông tin qua API từ bên thứ 3 (Binance).

**Quy định truy cập:** API này yêu cầu tài khoản có **role từ VIP2 trở lên**. Các tài khoản có role thấp hơn sẽ không được truy cập.

**Xử lý:**
- **Trường hợp thiếu hoặc không hợp lệ:**
  - Thiếu `symbol`: Trả về **400 Bad Request**.
  - Thiếu `interval`: Trả về **400 Bad Request**.
  - `symbol` không tồn tại: Trả về **404 Not Found**.
  - `interval` sai format: Trả về **400 Bad Request**.
- **Trường hợp hợp lệ:** Trả về **200 OK** và thông tin Kline.

---

## WebSocket API

### 1. Lấy giá Funding Rate
**Endpoint:** `/funding-rate/websocket`  
**Query Params:** `symbol`

**Cơ chế:**  
Fetch từ: `wss://stream.binance.com/stream?streams=%s@markPrice@1s`

**Xử lý:**
- **TH1: Thiếu `symbol`**  
  Sau 5 giây, socket sẽ tự động đóng và trả về:
  - Mã lỗi: `1002`
  - Message: `Symbol error`

- **TH2: `symbol` không tồn tại**  
  Nếu không nhận được phản hồi từ Binance sau 5 giây, hệ thống sẽ:
  - Trả về mã lỗi: `1002`
  - Message: `Symbol error`
  - Đóng socket.

- **TH3: `symbol` tồn tại**  
  Nếu Binance trả về thông tin hợp lệ, server sẽ xử lý và gửi lại cho người dùng.

---

### 2. Lấy giá Kline
**Endpoint:** `/kline/websocket`  
**Headers:** `Authorization`  
**Query Params:** `symbol`

**Cơ chế:**  
Fetch từ: `wss://stream.binance.com/stream?streams=%s@kline_1s`

**Quy định truy cập:** Yêu cầu tài khoản từ **VIP2 trở lên**.

**Xử lý:**
- **TH1: Không có quyền truy cập**  
  Nếu role thấp hơn VIP2, socket sẽ tự động đóng sau 5 giây.

- **TH2: Thiếu `symbol`**  
  Socket sẽ tự động đóng sau 5 giây.

- **TH3: `symbol` không tồn tại**  
  Socket sẽ tự động đóng sau 5 giây.

- **TH4: `symbol` tồn tại**  
  Server trả về response sau 1 giây.

---

### 3. CoinMarketCap
**Endpoint:** `/market-stats`  
**Query Params:** `symbol`

**Cơ chế:**  
Fetch từ: `https://api.coingecko.com/api/v3/coins/%s`

**Xử lý:**
- **TH1: Thiếu `symbol`**  
  Server sẽ đóng socket và trả về:
  - Mã lỗi: `1000`
  - Message: `Symbol missing or invalid`

- **TH2: `symbol` không tồn tại**  
  Server sẽ đóng socket và trả về:
  - Mã lỗi: `1000`
  - Message: `Symbol missing or invalid`

- **TH3: `symbol` tồn tại**  
  Server trả về response cho người dùng và tự động cập nhật sau 15 phút.
