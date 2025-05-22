# Aqua Trivy
Vulnerability and Misconfiguration Scanning. The all-in-one open source security scanner

![Trivy logo](/asset/trivy-logo.jpg)

## Đặt vấn đề

- Trong quy trình DevOps hiện đại, đặc biệt là CI Pipeline,  sau khi mã nguồn của chúng ta được build, test tĩnh với các tiêu chuẩn an toàn, nó sẽ được đóng gói thành container Docker hay 1 Pod trong Kubernetes. 

- Tuy nhiên, nếu không được kiểm soát kỹ, những image này có thể mang theo lỗ hổng bảo mật (CVE - Common Vulnerabilities and Exposures) tiềm ẩn, trở thành điểm yếu trong toàn bộ hệ thống. Đây chính là lúc các công cụ như Trivy phát huy vai trò – giúp phát hiện sớm các  vấn đề, rủi ro an ninh bảo mật trong Image sử dụng. 

## Định nghĩa

- Là một công cụ bảo mật mã nguồn mở của Aqua Security.

- Dùng để tìm lỗ hổng (CVE) và cấu hình sai (IaC) trên các kho lưu trữ của GitHub, các tập tin nhị phân, container images, cụm Kubernetes, v.v. 

- Trivy được thiết kế để sử dụng trong CI. Trước khi đẩy lên kho lưu trữ container hoặc triển khai ứng dụng, chúng ta có thể dễ dàng quét hình ảnh container ở cục bộ và các đối tượng khác một cách dễ dàng.

## Lợi ích

- Phát hiện lỗ hổng bảo mật sớm

- Hỗ trợ nhiều loại mục tiêu (Target)

- Tích hợp dễ dàng vào pipeline CI/CD

- Tốc độ quét nhanh, không cần cấu hình phức tạp

- Tìm secrets bị lộ

## Mục tiêu của Trivy

- Container Image 

- Filesystem

- Git repository (remote)

- Kubernetes cluster or resource

## Loại quét

- Các gói hệ điều hành và phần mềm phụ thuộc đang sử dụng (SBOM - Software Bill of Materials)

- Các lỗ hổng đã biết (CVE)
    - Cơ sở dữ liệu CVE của Trivy được cập nhật từ các nguồn: NVD, Red Hat, Debian, Ubuntu, Alpine,...

- Cấu hình sai:
    - Có thể quét file cấu hình YAML, Terraform, Dockerfile...

- Thông tin nhạy cảm và bí mật
    - Có thể phát hiện API key, password, token, private key bị đẩy nhầm lên repo hoặc image.

## Độ nguy hiểm

- Output sẽ chia các lỗi theo severity:

    - CRITICAL: lỗi nghiêm trọng, có thể bị khai thác ngay.

    - HIGH: lỗi nghiêm trọng, thường ảnh hưởng đến quyền truy cập.

    - MEDIUM: lỗi vừa phải.

    - LOW/UNKNOWN: ít nghiêm trọng hoặc chưa xác định.

## Cách cài đặt trên Ubuntu

- Cài đặt các công cụ cần thiết
```bash
$ sudo apt-get install wget apt-transport-https gnupg lsb-release
```

- Tải public key từ Aqua Security.
```bash
$ wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
```

- Thêm Trivy repository vào danh sách nguồn phần mềm của hệ thống
```bash
$ echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
```

- Cập nhật lại danh sách gói, bao gồm repo Trivy vừa thêm.
```bash
$ sudo apt-get update
```

- Cài đặt Trivy từ repository mới.
```bash
$ sudo apt-get install trivy
```

- Cài đặt trên các nguồn khác: [Xem ở đây](https://trivy.dev/v0.34/getting-started/installation/)

## Ví dụ

### Quét lỗ hỏng bảo mật của docker image
- Câu lệnh:
```bash
$ trivy image [YOUR_IMAGE_NAME]
```

- Kết quả:
```bash
$ trivy image myimage:1.0.0
2022-05-16T13:25:17.826+0100    INFO    Detected OS: alpine
2022-05-16T13:25:17.826+0100    INFO    Detecting Alpine vulnerabilities...
2022-05-16T13:25:17.826+0100    INFO    Number of language-specific files: 0

myimage:1.0.0 (alpine 3.15.3)

Total: 2 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 0, CRITICAL: 2)

┌────────────┬────────────────┬──────────┬───────────────────┬───────────────┬─────────────────────────────────────────────────────────┐
│  Library   │ Vulnerability  │ Severity │ Installed Version │ Fixed Version │                          Title                          │
├────────────┼────────────────┼──────────┼───────────────────┼───────────────┼─────────────────────────────────────────────────────────┤
│ busybox    │ CVE-2022-28391 │ CRITICAL │ 1.34.1-r4         │ 1.34.1-r5     │ busybox: remote attackers may execute arbitrary code if │
│            │                │          │                   │               │ netstat is used                                         │
│            │                │          │                   │               │ https://avd.aquasec.com/nvd/cve-2022-28391              │
├────────────┤                │          │                   │               │                                                         │
│ ssl_client │                │          │                   │               │                                                         │
│            │                │          │                   │               │                                                         │
│            │                │          │                   │               │                                                         │
└────────────┴────────────────┴──────────┴───────────────────┴───────────────┴─────────────────────────────────────────────────────────┘

app/deploy.sh (secrets)

Total: 1 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 0, CRITICAL: 1)

┌──────────┬───────────────────┬──────────┬─────────┬────────────────────────────────┐
│ Category │    Description    │ Severity │ Line No │             Match              │
├──────────┼───────────────────┼──────────┼─────────┼────────────────────────────────┤
│   AWS    │ AWS Access Key ID │ CRITICAL │    3    │ export AWS_ACCESS_KEY_ID=***** │
└──────────┴───────────────────┴──────────┴─────────┴────────────────────────────────┘
```

### Quét cấu hình sai của thư mục làm việc

- Câu lệnh:
```bash
$ trivy config [YOUR_IAC_DIR]
```

- Kết quả:
```bash
$ ls build/
Dockerfile
$ trivy config ./build
2022-05-16T13:29:29.952+0100    INFO    Detected config files: 1

Dockerfile (dockerfile)
=======================
Tests: 23 (SUCCESSES: 22, FAILURES: 1, EXCEPTIONS: 0)
Failures: 1 (UNKNOWN: 0, LOW: 0, MEDIUM: 1, HIGH: 0, CRITICAL: 0)

MEDIUM: Specify a tag in the 'FROM' statement for image 'alpine'
══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
When using a 'FROM' statement you should use a specific tag to avoid uncontrolled behavior when the image is updated.

See https://avd.aquasec.com/misconfig/ds001
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 Dockerfile:1
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1 [ FROM alpine:latest
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```

## Tham khảo

[1]  https://blog.devopsviet.com/2023/11/26/huong-dan-phat-hien-van-de-bao-mat-trong-docker-image-voi-trivy/

[2] https://trivy.dev/latest/

[3] https://trivy.dev/v0.34/

[4] https://trivy.dev/v0.34/getting-started/quickstart/