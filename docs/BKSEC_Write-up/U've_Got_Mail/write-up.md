# U've got mail

![You've Got Mail challenge description](imgs/challenge_description.png)

Đầu tiên đề cho 1 CRM với hàm ý đừng RCE (remote code execution) 

---

## Quá trình làm

### Khảo sát website & Phát hiện IDOR

![NexaCRM dashboard showing admin_anhht](imgs/crm_dashboard_admin_username.png)

Đăng nhập vào ta thấy được 1 vài chức năng cơ bản như `/dashboard`, `/customers`, ...

Để ý là t thấy được luôn username của admin ở đây

sau khi lượn 1 vòng trang web thì ta thấy được 2 khá hay.

![My Profile edit page](imgs/my_profile_page.png)

chỗ hồ sơ này cho ta thay đổi thông tin của mình

![Burp Suite update staff profile request](imgs/burp_update_staff_profile.png)

mò vào burp suite cho ta thấy được 2 request, 1 là `GET /api/user/staff1`, 2 là `POST /api/user/staff1/update` (để update thông tin)

nhớ rằng ta vừa biết được username của admin là `admin_anhht`

![Burp Suite GET admin info](imgs/burp_get_admin_details.png)

và ta thực sự GET được thông tin của Admin.

![Burp Suite attempt to elevate role to admin](imgs/burp_update_role_failed.png)

thử update role của mình thành admin xem được không, kết quả là không.

![Burp Suite 403 Forbidden updating admin profile](imgs/burp_update_admin_password_forbidden.png)

maybe đổi mật khẩu admin? nope. tra gg thì ra đây là lỗi IDOR.

intent của tác giả có vẻ cũng lộ rõ ở đây là chỉ muốn m đọc th, còn sửa thì đi chỗ khác. vì thế nên mình biến chỗ khác và tìm thấy:

---

### Khai thác Twig Template Engine (SSTI)

![Email template editor with Twig syntax](imgs/email_template_editor.png)

trông rất quen mắt

tra gg ta biết được đây là twig, là 1 template engine, cho phép gán thẳng phép xử lý logic vào trong văn bản (giống ERB và ruby bên portswigger)

tuy nhiên nó đã được sandbox (không chạy được các lệnh shell), nên cần slop AI để gen list payload 

sau khi slop và test xong thì mình tìm được 

![Twig SSTI dumping context keys](imgs/twig_dump_context_keys.png)

cũng như 

![Twig SSTI dumping user attributes](imgs/twig_dump_user_attributes.png)

có vẻ họ cho xem context về dev đã nạp thông tin gì users và company, customer.

đầu tiên coi hết tất cả thông tin về admin

![Twig SSTI dumping admin info with empty reset token](imgs/twig_dump_admin_info_empty_token.png)

chỗ này có 2 hướng để giải: 1 là dùng hashcat để giải mã password đã bị bcrypt thành hash, 2 là dùng `reset_token` (khả năng rất cao là token để reset password)

và hashcat password là 1 rabbit hole. mất rất nhiều thời gian tuy nhiên không thể giải mã được

---

### Reset Password Admin & Lấy Flag

trong lúc đó thử sử dụng `reset_token`. ở trên hình ghi rỗng vì mình chưa thử reset password. logout ra để reset password 

![Forgot password request with admin email](imgs/forgot_password_submit_admin_email.png)

ở đây ta điền email của admin vào (ta đã có email admin từ đoạn gửi request `GET` tới `/api/user/admin_anhht`)

sau đó đăng nhập tài khoản staff và quay lại kiểm tra

![Twig SSTI dumping generated admin reset token](imgs/twig_dump_admin_reset_token.png)

-> đã có token reset, h chỉ còn việc lsao để dùng cái token đó

![Failed guess of reset password URL](imgs/forgot_password_token_failed_url.png)

ban đầu mình có đoán mò mà không thành công, nên phải nhờ lão sư AI

![Reset password page with valid token](imgs/reset_password_page_token.png)

điền mật khẩu mới cho admin 

![Admin dashboard displaying BKSEC flag](imgs/admin_dashboard_flag.png)

và vào trang admin lấy flag :Đ

---

Wrote by: h1inn9
