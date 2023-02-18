# 2.
Thứ nhất ta sẽ build docker image và push lên ecr, sau đó sẽ deploy service, với ECS ta có thể lựa chọn công cụ CI/CD là CodePipeline để có thể deployment Blue-Green, với EKS ta có thể dùng ArgoCD hoặc Argo Rollout hoặc Istio để deployment blue-green và các deployment khác mạnh mẽ hơn.

|Môi trường|Dev và QA|PROD|
|---|----|----|
|trigger từ git để build|Tự động|Thủ công|
|Deployment Blue-Green, quá trình di chuyển traffic không get approval|Không|Cần|

# 3.
## a.
- Sử dụng phân cấp khóa (locking hierarchy) và multi-version concurrency control (MVCC) để tránh block và giải phóng lock trong thời gian ngắn
## b.
### 502 - Bad gateway
Đầu tiên phải đảm bảo rằng application đang chạy vẫn có thể phản hồi request bằng cách gửi request trực tiếp đến upstream mà không phải đi qua load balancer hay api gateway. Nếu response được trả về với status code là:
- 502 thì kiểm tra log của application đang chạy.
- khác 502 thì vấn đề có thể trên load balancer hoặc api gateway, có thể do cấu hình sai target group hoặc sai upstream.
### 504 - Gateway Timeout
Trong trường hợp này có thể do đường truyền mạng, băng thông có vấn đề. Thứ 2, ta cũng sẽ kiểm tra bằng cách gửi request vào upstream của application đang chạy, nếu response được trả về với status code là 504, thì đầu tiên phải kiểm tra log, có thể application đang gọi và đang đợi response của một api khác cho việc xử lý request, thì tiếp tục kiểm tra đến api đang gọi đó, nếu không có thể do lỗi code hoặc do tài nguyên không đủ để application sử dụng. Thứ 3 là khi scale, resource đang ở trạng thái *unavaible* và load balancer chưa thể phân phối traffic dẫn đến một vài request bị timeout.
## c.
Về RDS và EC2 ta có thể đặt mua trước (Reserved instances) để có chi phí vận hành tối ưu, các config của micro-service sẽ được lưu trữ ở AWS System Manager, sẽ đảm bảo tính lưu trữ và bảo mật cao.
# 4.
Để đánh giá được performance của service, dưới môi trường DEV, ta có thể dùng tool để load test, cũng như điều chỉnh resource thích hợp cho service để high performance. Về alert realtime thì cloudwatch của aws hoặc prometheus - alert manager đều có thể làm được, tùy thuộc vào việc chúng ta custom rule.