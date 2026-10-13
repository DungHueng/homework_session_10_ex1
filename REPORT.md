# BÁO CÁO KỸ THUẬT: PHÂN TÍCH KIẾN TRÚC CONTAINER VS VIRTUAL MACHINE

## 1. Tổng quan

Trong quá trình phát triển và triển khai phần mềm, doanh nghiệp cần một môi trường vận hành ổn định, dễ mở rộng và tối ưu chi phí hạ tầng. Virtual Machine (VM) và Container là hai công nghệ quan trọng giúp giải quyết nhu cầu này.

**Virtual Machine (VM)** sử dụng công nghệ ảo hóa phần cứng thông qua Hypervisor để chạy nhiều máy ảo độc lập trên cùng một máy chủ vật lý. Mỗi VM thường có hệ điều hành khách, kernel và các thư viện riêng.

Trong khi đó, **Container** sử dụng Container Engine như Docker để đóng gói ứng dụng cùng các thành phần phụ thuộc, đồng thời chia sẻ kernel của hệ điều hành máy chủ.

Doanh nghiệp có thể kết hợp Container với hạ tầng VM hoặc VPS truyền thống để giảm tài nguyên dành cho hệ điều hành khách, tăng tốc độ triển khai và chuẩn hóa môi trường chạy ứng dụng.

## 2. So sánh kiến trúc VM và Container

| Tiêu chí | Virtual Machine (VM) | Container |
|---|---|---|
| **Dung lượng image** | Thường lớn, có thể từ vài GB trở lên do chứa hệ điều hành khách và các thành phần cần thiết. | Thường nhỏ hơn vì không đóng gói kernel riêng; kích thước phụ thuộc image nền, thư viện và ứng dụng. |
| **Thời gian khởi động** | Thường mất từ vài giây đến vài phút vì phải khởi động một hệ điều hành khách. | Thường khởi động nhanh, có thể chỉ mất vài giây hoặc ít hơn tùy cấu hình và ứng dụng. |
| **Hao phí hệ điều hành** | Mỗi VM thường cần tài nguyên riêng cho hệ điều hành khách. | Nhiều container chia sẻ kernel máy chủ, giúp giảm phần tài nguyên dành cho hệ điều hành. |
| **Mức độ cô lập** | Thường có ranh giới cô lập mạnh nhờ Hypervisor và kernel riêng của từng VM. | Cô lập tiến trình và tài nguyên ở cấp hệ điều hành; mức độ bảo vệ phụ thuộc cấu hình và các biện pháp bảo mật. |
| **Khả năng mở rộng** | Có thể mở rộng nhưng việc tạo VM mới thường tốn thời gian và tài nguyên hơn. | Có thể tạo, thay thế và nhân bản container nhanh, thuận tiện cho triển khai tự động và kiến trúc microservices. |

### Nhận xét

- **Virtual Machine:** Phù hợp với các trường hợp cần chạy nhiều hệ điều hành khác nhau hoặc yêu cầu ranh giới cô lập mạnh.
- **Container:** Phù hợp với các ứng dụng cần triển khai nhanh, nhất quán giữa môi trường phát triển và sản xuất, đồng thời cần mở rộng linh hoạt.
- **Kết hợp VM và Container:** Docker thường được triển khai bên trong VM hoặc VPS, tận dụng khả năng cô lập của máy ảo và sự linh hoạt của container.

## 3. Cơ chế cô lập và quản lý tài nguyên của Linux Container

### 3.1. Namespaces – Cô lập không gian hệ thống

Namespaces là cơ chế của Linux giúp tạo ra các không gian tài nguyên riêng biệt cho từng nhóm tiến trình. Nhờ đó, tiến trình bên trong container có thể nhìn thấy một môi trường hệ thống khác với tiến trình bên ngoài.

Các loại namespaces quan trọng gồm:

- **PID Namespace:** Cô lập không gian Process ID. Tiến trình trong container có thể nhìn thấy hệ thống tiến trình riêng, trong khi kernel vẫn quản lý chúng trên máy chủ.
- **Network Namespace:** Cung cấp không gian mạng riêng, bao gồm giao diện mạng, bảng định tuyến và các quy tắc mạng tương ứng. Docker có thể kết nối các namespace qua virtual Ethernet và bridge.
- **Mount Namespace:** Cô lập góc nhìn về các điểm gắn kết hệ thống tệp (mount points), giúp container có cấu trúc hệ thống tệp riêng.
- **UTS Namespace:** Cho phép cô lập hostname và domain name.
- **IPC Namespace:** Cô lập một số cơ chế giao tiếp liên tiến trình.
- **User Namespace:** Có thể ánh xạ UID/GID giữa bên trong container và máy chủ nhằm hỗ trợ giảm đặc quyền.

Namespaces giúp container có môi trường vận hành riêng nhưng không tạo ra một kernel độc lập như VM.

### 3.2. Control Groups (cgroups) – Quản lý tài nguyên

cgroups là cơ chế của Linux dùng để tổ chức, theo dõi và giới hạn tài nguyên mà một nhóm tiến trình được sử dụng.

Các chức năng tiêu biểu gồm:

- **CPU:** Thiết lập giới hạn hoặc tỷ lệ sử dụng CPU, giúp hạn chế một container chiếm quá nhiều năng lực xử lý.
- **RAM:** Đặt giới hạn bộ nhớ cho nhóm tiến trình. Khi vượt giới hạn, hệ thống có thể áp dụng các cơ chế xử lý áp lực bộ nhớ, bao gồm OOM kill tùy cấu hình.
- **Theo dõi tài nguyên:** Ghi nhận mức sử dụng tài nguyên của các nhóm tiến trình để hỗ trợ giám sát và quản trị.

Namespaces và cgroups có vai trò khác nhau nhưng bổ trợ cho nhau:

| Cơ chế | Chức năng chính |
|---|---|
| Namespaces | Cô lập góc nhìn về tiến trình, mạng, hệ thống tệp và các tài nguyên hệ thống khác. |
| cgroups | Quản lý, theo dõi và giới hạn mức tiêu thụ tài nguyên của các nhóm tiến trình. |

Docker phối hợp các cơ chế này để vận hành container. Tuy nhiên, chúng không thay thế hoàn toàn các biện pháp bảo mật khác như quyền người dùng, Linux capabilities, seccomp và chính sách hệ thống tệp.

## 4. Kết quả kiểm tra Docker

### 4.1. Các lệnh thực hiện

Sử dụng terminal trên máy đã cài đặt Docker để kiểm tra phiên bản và cấu hình Docker Engine.

Kiểm tra phiên bản Docker Client và Server:

```bash
docker version
```

Kiểm tra thông tin tổng quan của Docker Engine:

```bash
docker info
```

Kiểm tra cấu hình cgroups:

```bash
docker info | grep -i cgroup
```

### 4.2. Bảng kết quả thực tế

Điền các giá trị dựa trên kết quả lệnh `docker info` của hệ thống thực hiện.

| Thông số | Giá trị thực tế |
|---|---|
| Server Version | Điền phiên bản Docker Server |
| Cgroup Driver | Điền giá trị Cgroup Driver |
| Cgroup Version | Điền phiên bản cgroups nếu được hiển thị |
| Containers | Điền tổng số container |
| Images | Điền tổng số image |

**Lưu ý:** Các giá trị trên phụ thuộc vào môi trường Docker thực tế, vì vậy cần kiểm tra và điền kết quả sau khi chạy lệnh, không sử dụng số liệu giả định.

### 4.3. Nhận xét kết quả

- Lệnh `docker version` cung cấp thông tin phiên bản Docker Client và Docker Server.
- Lệnh `docker info` cung cấp thông tin tổng quan về Docker Engine, bao gồm cấu hình cgroups, số lượng container và image hiện có.
- Lệnh `docker info | grep -i cgroup` giúp tìm nhanh các dòng thông tin liên quan đến cgroups.
- Các thông số thu được giúp xác định cấu hình quản lý tài nguyên mà Docker đang sử dụng trên hệ thống.

## 5. Kết luận

Virtual Machine và Container đều đóng vai trò quan trọng trong việc triển khai ứng dụng hiện đại, nhưng có kiến trúc và mục tiêu sử dụng khác nhau.

VM cung cấp môi trường máy tính ảo với hệ điều hành khách riêng, phù hợp khi cần chạy nhiều hệ điều hành hoặc tăng cường ranh giới cô lập. Container chia sẻ kernel của hệ điều hành máy chủ, giúp giảm chi phí tài nguyên và tăng tốc độ triển khai ứng dụng.

Namespaces hỗ trợ cô lập môi trường thực thi, trong khi cgroups quản lý và giới hạn tài nguyên của các nhóm tiến trình. Docker kết hợp các cơ chế này để cung cấp nền tảng triển khai ứng dụng linh hoạt, nhất quán và thuận tiện cho tự động hóa.

Việc lựa chọn VM, Container hoặc kết hợp cả hai cần dựa trên yêu cầu về hiệu năng, bảo mật, khả năng mở rộng và chi phí vận hành của hệ thống.
