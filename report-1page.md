# FIT4012 Lab 7 - Báo cáo 1 trang: SHA-256

## 1. Mục tiêu / Objective

Bài thực hành nhằm tìm hiểu thuật toán băm SHA-256 và ứng dụng của nó trong an toàn dữ liệu:

- Hiểu cấu trúc thuật toán: khởi tạo `H0_INITIAL`, padding Merkle–Damgård, mở rộng message schedule `W[0..63]` và 64 vòng nén với các hàm `Ch`, `Maj`, `Sigma0/1`, `sigma0/1`.
- Áp dụng SHA-256 vào ba bài toán thực tế: kiểm tra toàn vẹn file, băm mật khẩu, và băm mật khẩu có salt.
- Phân tích vì sao SHA-256 trần (không có salt, không làm chậm) chưa đủ an toàn để lưu mật khẩu trong hệ thống thật.

## 2. Cách làm / Approach

- Biên dịch các chương trình bằng `make all` (hoặc CMake), tạo ra 4 binary: `sha256`, `file_integrity`, `password_hash`, `salted_password_hash`.
- Chạy `./sha256 --self-test` để đối chiếu với 3 known answer test vectors (`""`, `"abc"`, `"hello FIT4012 SHA"`).
- Băm chuỗi tùy ý bằng `./sha256 --hash-string "..."` và băm file bằng `./sha256 --hash-file sample.txt`.
- Tạo file `sample.txt`, tính hash gốc, sau đó dùng `file_integrity` để xác minh; tiếp tục append nội dung để mô phỏng tamper và kiểm tra lại.
- `password_hash register/login` để minh họa luồng đăng ký – đăng nhập, đồng thời thử mật khẩu sai để có negative test.
- `salted_password_hash register` hai lần với **cùng một mật khẩu** rồi so sánh hai file `.hash` để chứng minh salt sinh hash khác nhau.
- Toàn bộ kết quả được ghi lại trong `logs/generated-sample-run.log` thông qua `bash scripts/run_sample.sh | tee logs/generated-sample-run.log`.

## 3. Kết quả / Result

Minh chứng chính trích từ `logs/generated-sample-run.log`:

- Hash của chuỗi `abc`:
  `ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad`
- Hash của chuỗi `hello FIT4012 SHA`:
  `e942ae0ecb44fe48e1490144162fc64e9c3c6c399bb4e2d686532195cdc3eae6`
- Hash của file mẫu `sample.txt` trước khi sửa:
  `5ee62dc925a9958dbd6732c570a23c7f65a8c11066e889b15068cfb4bf1a0bd9` → `[PASS] File integrity OK`
- Kết quả kiểm tra file sau khi sửa nội dung (append `tampered`):
  `[FAIL] File was changed or expected hash is incorrect`
  actual: `c232e5627e703ee5b311e0df8520b3d10dc8867b27636bb58fe58eb1fb9d6acb` → `[PASS] Tamper case detected`
- Kết quả đăng nhập với mật khẩu đúng (`fit4012-demo-password`): `[PASS] Login success`.
- Kết quả đăng nhập với mật khẩu sai (`wrong-password`): `[FAIL] Login failed: wrong password` → `[PASS] Wrong password rejected`.
- Hai bản ghi `salt:hash` của cùng một mật khẩu **khác nhau hoàn toàn**: `cmp -s test_password_salted_1.hash test_password_salted_2.hash` báo khác → `[PASS] Same password produced different salted records`.

## 4. Kết luận / Conclusion

- **SHA-256 phát hiện thay đổi dữ liệu** nhờ tính chất “avalanche”: chỉ cần thêm 1 byte vào `sample.txt` thì 100% các bit của hash đầu ra thay đổi, nên việc so sánh hash hiện tại với hash mong muốn đủ để kết luận file đã bị sửa.
- **Cần salt khi lưu hash mật khẩu** vì nếu không có salt, hai người dùng đặt cùng mật khẩu sẽ có cùng hash, và kẻ tấn công có thể dùng rainbow table tra ngược. Salt ngẫu nhiên (16 byte trong demo) khiến mỗi bản ghi là duy nhất, vô hiệu hóa rainbow table dù salt được lưu công khai cùng hash.
- **SHA-256 demo chưa đủ cho hệ thống xác thực thật** vì nó được thiết kế để nhanh – một GPU có thể thử hàng tỷ hash/giây, nên brute-force mật khẩu yếu rất rẻ. Hệ thống thật nên dùng các thuật toán password hashing chuyên dụng (Argon2id, bcrypt, scrypt) có chi phí tính toán/bộ nhớ điều chỉnh được để làm chậm tấn công.
