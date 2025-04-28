# Sonarqube

![SonarQube logo](/asset/sonarqube-logo.jpg)

## Định nghĩa: 

- SonarQube là một nền tảng mã nguồn mở do SonarSource phát triển, giúp kiểm tra chất lượng mã nguồn liên tục. Công cụ này thực hiện đánh giá tự động bằng phân tích tĩnh (static analysis) nhằm phát hiện lỗi và các vấn đề về chất lượng mã (code smells) trên 29 ngôn ngữ lập trình.

- Nền tảng này còn tích hợp các công cụ phát hiện thông tin nhạy cảm mạnh mẽ giúp lập trình viên ngăn chặn kịp thời rủi ro bảo mật trong mã nguồn. 

- Có thể tích hợp dễ dàng vào quy trình CI/CD.

## Định nghĩa SonarQube Server:

- Là trung tâm của hệ thống, bao gồm giao diện web dùng để đánh giá mã nguồn và phân tích tĩnh tự động theo tiêu chuẩn trong doanh nghiệp, được triển khai tại chỗ (on-premise).

- Bằng cách tích hợp trực tiếp vào quá trình CI hoặc một trong các nền tảng DevOps được hỗ trợ, mã nguồn sẽ được kiểm tra dựa trên một bộ quy tắc rộng, bao gồm các thuộc tính về khả năng bảo trì, độ tin cậy và các vấn đề bảo mật trong từng nhất merge/pull request.

## Ưu điểm:

- **Phát hiện lỗi sớm**: Phân tích tĩnh giúp phát hiện lỗi sớm bằng cách xem xét các tài liệu và hiện vật trước khi thực hiện, tiết kiệm thời gian và công sức sau này trong quá trình phát triển.

- **Hiệu quả về chi phí**: Hiệu quả về chi phí hơn các kỹ thuật kiểm tra động vì các lỗi được tìm thấy trong quá trình kiểm tra tĩnh rẻ hơn để sửa.

- **Tăng năng suất phát triển**: Tăng năng suất phát triển nhờ tài liệu chất lượng và dễ hiểu, và thiết kế được cải thiện

## Nhược điểm:

- **Có thể không phát hiện ra tất cả các vấn đề**: Phân tích tĩnh có thể không phát hiện ra tất cả các vấn đề có thể phát sinh trong thời gian chạy.

- **Phụ thuộc vào kỹ năng của người đánh giá**: Hiệu quả của phân tích tĩnh phụ thuộc vào kỹ năng, kinh nghiệm và kiến ​​thức của người đánh giá.

## Kiến trúc:

- **SonarQube Server**: Đây là trung tâm của hệ thống, chịu trách nhiệm xử lý và hiển thị kết quả phân tích mã nguồn. Nó bao gồm giao diện web để quản lý dự án, xem báo cáo và cấu hình hệ thống.

- **Compute Engine**: Thành phần này thực hiện phân tích mã nguồn bằng cách áp dụng các quy tắc kiểm tra, phát hiện lỗi, lỗ hổng bảo mật và các vấn đề về chất lượng mã.

- **Database**: SonarQube sử dụng cơ sở dữ liệu để lưu trữ kết quả phân tích, lịch sử kiểm tra và cấu hình hệ thống, giúp duy trì tính liên tục trong việc theo dõi chất lượng mã nguồn.

- **Sonar Scanner**: Đây là công cụ quét mã nguồn, có thể được tích hợp vào các quy trình CI/CD để tự động kiểm tra chất lượng mã trước khi triển khai.

## Nên dùng SonarQube trong các trường hợp sau:

- **Dự án lớn với nhiều lập trình viên**: SonarQube giúp duy trì chất lượng mã nguồn trong các nhóm phát triển lớn.

- **Tích hợp CI/CD**: Nếu có nhu cầu muốn tự động kiểm tra mã nguồn trong quy trình DevOps, SonarQube là một lựa chọn lý tưởng.

- **Phát hiện lỗi bảo mật**: Công cụ này giúp xác định các lỗ hổng bảo mật, đặc biệt quan trọng trong DevSecOps.

- **Cải thiện chất lượng mã**:  Khi muốn đảm bảo mã nguồn tuân thủ các tiêu chuẩn về bảo trì, độ tin cậy và hiệu suất.

- **Hỗ trợ nhiều ngôn ngữ lập trình**: SonarQube có thể phân tích hơn 30 ngôn ngữ, phù hợp với các dự án đa nền tảng.

## Không nên dùng trong các trường hợp sau:

- **Dự án nhỏ hoặc cá nhân**: Nếu chỉ làm việc một mình hoặc dự án có quy mô nhỏ, SonarQube có thể quá phức tạp và tốn tài nguyên.

- **Không cần phân tích mã nguồn sâu**: Nếu chỉ cần kiểm tra lỗi cơ bản, các công cụ nhẹ hơn như ESLint hoặc Pylint có thể phù hợp hơn.

- **Hạn chế tài nguyên hệ thống**: SonarQube yêu cầu nhiều tài nguyên, đặc biệt khi phân tích các dự án lớn.

- **Chi phí cao cho phiên bản thương mại**: Nếu cần các tính năng nâng cao, phiên bản trả phí có thể không phù hợp với ngân sách.

## Tham khảo:

[1] https://docs.sonarsource.com/sonarqube-server/latest/design-and-architecture/overview/

[2] https://docs.sonarsource.com/sonarqube-server/latest/\

[3] https://www.techgeeknext.com/tools/sonarqube-architecture