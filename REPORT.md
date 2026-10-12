# BÁO CÁO KỸ THUẬT: PHÂN TÍCH KIẾN TRÚC CONTAINER VS VIRTUAL MACHINE


## 1. Tổng quan



Trong quá trình phát triển và triển khai phần mềm, doanh nghiệp cần một môi trường vận hành ổn định, dễ mở rộng và tối ưu chi phí hạ tầng. Virtual Machine (VM) và Container là hai công nghệ quan trọng giúp giải quyết nhu cầu này.



VM sử dụng công nghệ ảo hóa phần cứng thông qua Hypervisor để chạy nhiều máy ảo độc lập trên cùng một máy chủ vật lý. Mỗi VM thường có hệ điều hành khách, kernel và các thư viện riêng. Trong khi đó, Container sử dụng Container Engine như Docker để đóng gói ứng dụng cùng các thành phần phụ thuộc, đồng thời chia sẻ kernel của hệ điều hành máy chủ.



Doanh nghiệp có thể chuyển từ mô hình triển khai VPS truyền thống sang Docker để giảm tài nguyên dành cho hệ điều hành, tăng tốc độ triển khai và chuẩn hóa môi trường chạy ứng dụng.



## 2. So sánh kiến trúc VM và Container



| Tiêu chí | Virtual Machine (VM) | Container |

|---|---|---|

| Dung lượng Image | Thường lớn, có thể từ vài GB trở lên do chứa hệ điều hành khách và các thành phần cần thiết. | Thường nhỏ hơn vì không đóng gói kernel riêng; kích thước phụ thuộc image nền, thư viện và ứng dụng. |

| Thời gian khởi động | Thường mất từ vài giây đến vài phút vì phải khởi động một hệ điều hành khách. | Thường chỉ mất vài giây hoặc ít hơn đối với ứng dụng đơn giản, tùy cấu hình và ứng dụng. |

| Hao phí OS | Mỗi VM thường cần tài nguyên riêng cho hệ điều hành khách. | Nhiều container chia sẻ kernel máy chủ nên giảm phần tài nguyên dành cho hệ điều hành. |

| Mức độ cô lập | Thường có ranh giới cô lập mạnh nhờ Hypervisor và kernel riêng của từng VM. | Cô lập tiến trình và tài nguyên ở cấp hệ điều hành; mức độ bảo vệ phụ thuộc cấu hình và các biện pháp bảo mật. |

| Khả năng mở rộng | Có thể mở rộng nhưng việc tạo VM mới thường tốn thời gian và tài nguyên hơn. | Có thể tạo, thay thế và nhân bản container nhanh, thuận tiện cho triển khai tự động và kiến trúc microservices. |



Nhìn chung, VM phù hợp với các trường hợp cần chạy nhiều hệ điều hành khác nhau hoặc yêu cầu ranh giới cô lập mạnh. Container phù hợp với các ứng dụng cần triển khai nhanh, nhất quán giữa môi trường phát triển và sản xuất, cũng như cần mở rộng linh hoạt. Hai công nghệ có thể được sử dụng kết hợp: Docker thường chạy bên trong một VM hoặc VPS.



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



Namespaces giúp container có môi trường vận hành riêng, nhưng không tạo ra một kernel độc lập như VM.



### 3.2. Control Groups (cgroups) – Quản lý tài nguyên



cgroups là cơ chế của Linux dùng để tổ chức, theo dõi và giới hạn tài nguyên mà một nhóm tiến trình được sử dụng.



Các chức năng tiêu biểu gồm:



- **CPU:** Thiết lập giới hạn hoặc tỷ lệ sử dụng CPU, giúp hạn chế một container chiếm quá nhiều năng lực xử lý.

- **RAM:** Đặt giới hạn bộ nhớ cho nhóm tiến trình. Khi vượt giới hạn, hệ thống có thể áp dụng các cơ chế xử lý áp lực bộ nhớ, bao gồm OOM kill tùy cấu hình.

- **Theo dõi tài nguyên:** Ghi nhận mức sử dụng tài nguyên của các nhóm tiến trình để hỗ trợ giám sát và quản trị.



Namespaces và cgroups có vai trò khác nhau nhưng bổ trợ cho nhau: namespaces chủ yếu cung cấp khả năng cô lập góc nhìn tài nguyên, còn cgroups quản lý mức tiêu thụ tài nguyên. Docker phối hợp các cơ chế này để vận hành container. Chúng không thay thế hoàn toàn các biện pháp bảo mật khác như quyền người dùng, Linux capabilities, seccomp và chính sách hệ thống tệp.



## 4. Kết quả kiểm tra Docker



Các lệnh được sử dụng:



```bash

docker version

docker info

docker info | grep -i cgroup

```



Điền các giá trị đúng theo kết quả thực tế của hệ thống:



| Thông số | Giá trị thực tế |

|---|---|

| Server Version | \[Điền từ mục Server Version trong docker info] |

| Cgroup Driver | \[Điền từ mục Cgroup Driver trong docker info] |

| Containers | \[Điền tổng số tại mục Containers] |

| Images | \[Điền tổng số tại mục Images] |



**Nhận xét:** Lệnh `docker version` cung cấp thông tin phiên bản Docker Client và Docker Server. Lệnh `docker info` cung cấp thông tin tổng quan về Docker Engine, bao gồm cấu hình cgroups và số lượng container, image hiện có.
