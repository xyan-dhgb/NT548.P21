# EKS - Elastic Kubernetes Service 

Nền tảng hàng đầu để chạy cụm Kubernetes, cả trên  Amazon Web Services (AWS) và trong trung tâm dữ liệu.

![AWS EKS](/asset/aws-eks-logo.png)

## Đặt vấn đề 

- Bối cảnh:
    - Ứng dụng có nhiều microservices (backend, frontend, worker, v.v.) cần triển khai.
    - Mỗi dịch vụ chạy trong container (Docker), có yêu cầu tự động scaling, update, rollback, load balancing, secret management, ...
    - Các dịch vụ cần phải đảm bảo chạy ổn định trong môi trường production, phân phối ở nhiều AZ (availability zones), có giám sát, có bảo mật.

- Vấn đề:
    - Chúng ta không thể quản lý thủ công các container bằng một câu lệnh `docker run.`
    - Nếu dùng ECS (Elastic Container Service) thì không đủ linh hoạt khi muốn custom logic (ví dụ: policy, advanced networking, tự define operator,...).

    ![Amazon Elastic Container Server](/asset/ecs-logo.webp)

    - Việc build 1 cụm Kubernetes tự quản lý từ đầu rất phức tạp, tốn công, khó bảo trì, nhất là security (certs, RBAC (Role-Based Access Control), HA,...).

- Đó là lúc EKS được "sinh ra" để khắc phục các nhược điểm đó. 

- EKS cung cấp Kubernetes as a Service, nhà phát triển chỉ lo phần app, AWS lo control plane. Đảm bảo HA (high availability), update, patching được tự động thực hiện.

- EKS có thể tích hợp với các service khác như IAM, CloudWatch, ALB (Application load balancer). NLB (Network Load Balancer), EBS (Elastic Block Storage), EFS (Elastic File System)

## Định nghĩa


### Kubernetes  - K8s (Cần nắm rõ)

### EKS 

## Đặc điểm của Amazon EKS

## Dich vụ tương ứng



## Các công nghệ quản lý tương đồng với EKS