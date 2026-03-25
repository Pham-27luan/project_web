
# Software Requirement Specification (SRS)

## Chức năng: Đăng nhập tài khoản (Login)

**Mã chức năng:** AUTH-LOGIN-01
**Trạng thái:** Draft / Review
**Người soạn thảo:** Phạm Đình Luân
**Vai trò:** Technical Writer / Developer

---

## 1. Mô tả chung (Overview)

Chức năng đăng nhập cho phép người dùng truy cập vào tài khoản của họ bằng cách nhập email và mật khẩu. Hệ thống sẽ xác thực thông tin và cấp quyền truy cập dựa trên JWT tokens.

---

## 2. Luồng nghiệp vụ (User Workflow)

| Bước | Hành động người dùng               | Phản hồi hệ thống                                                                                        |
| ---- | ---------------------------------- | -------------------------------------------------------------------------------------------------------- |
| 1    | Truy cập màn hình đăng nhập        | Hiển thị form gồm Email và Password.                                                                     |
| 2    | Nhập thông tin và nhấn "Đăng nhập" | Frontend gửi request POST /auth/login đến back-end.                                                      |
| 3    | Hệ thống kiểm tra dữ liệu đầu vào  | Validate định dạng email và độ dài mật khẩu bằng zod.                                                    |
| 4    | Hệ thống xác thực tài khoản        | Tìm user theo email trong MongoDB và so khớp mật khẩu bằng bcryptjs.                                     |
| 5    | Xác thực thành công                | Tạo AccessToken, RefreshToken, lưu refresh token kèm device_info, trả về thông tin đăng nhập thành công. |
| 6    | Xác thực thất bại                  | Trả lỗi 401 Unauthorized với thông báo chung cho email hoặc mật khẩu sai.                                |

---

## 3. Yêu cầu dữ liệu (Data Requirements)

### 3.1. Dữ liệu đầu vào (Input Fields)

- **Email:** string, bắt buộc, dùng định dạng email, được trim() và chuyển về chữ thường.
- **Password:** string, bắt buộc, tối thiểu 8 kí tự, tối đa 50 kí tự.

### 3.2. Dữ liệu đầu ra (Response Data)

Khi đăng nhập thành công, hệ thống trả về:

- **status:** success
- **message:** Đăng nhập thành công
- **data.id:** ID người dùng
- **data.AccessToken:** JWT access token
- **data.RefreshToken:** JWT refresh token

### 3.3. Dữ liệu lưu trữ / truy xuất

- **Collection users:** dùng để trả cứu email, password, role, is_verified.
- **Collection refreshTokens:** lưu user_id, jti, device_info, expires_at cho phiên đăng nhập.

---



## 4. Trường hợp ngoại lệ & xử lý lỗi (Edge Cases)

- **Trường hợp:** Email sai định dạng hoặc mật khẩu không đủ độ dài.
  - **Xử lý:** Trả 422 Unprocessable Entity với danh sách lỗi validate.

- **Trường hợp:** Email không tồn tại trong hệ thống.
  - **Xử lý:** Trả 401 Unauthorized với thông báo chung "Tài khoản hoặc mật khẩu không đúng".

- **Trường hợp:** Mật khẩu không khớp với tài khoản đã tìm thấy.
  - **Xử laking:** Trả 401 Unauthorized với cùng thống báo chung để tránh lộ thông tin.

- **Trường hợp:** Body JSON bị lỗi cú pháp.
  - **Xử lý:** Trả 400 Bad Request với thông báo dữ liệu không hợp lệ.

- **Trường hợp:** Lỗi hệ thống khi truy cập database hoặc sinh token.
  - **Xử lý:** Trả 500 Internal Server Error.

---

## 5. Giao diện (UI/UX)

- Form đăng nhập nên có tối thiểu 2 trường: Email, Password.
- Nút đăng nhập cần hiển thị trạng thái loading trong lúc chờ phản hồi từ server.
- Khi đăng nhập thất bại, nên hiển thị thông báo lỗi chung thay vì nêu rõ email hay mật khẩu sai.
- Frontend nên xử lý lưu token an toàn theo kiến trúc của hệ thống.
