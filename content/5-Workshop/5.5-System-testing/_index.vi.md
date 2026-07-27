---
title : "Kiểm thử hệ thống"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5 </b> "
---

Sau khi hoàn tất triển khai Frontend và Backend lên AWS ở các chương trước, nhóm tiến hành kiểm thử toàn diện hệ thống **SmartDocAI** trong môi trường production thực tế. Mục tiêu của phần này là xác minh các luồng nghiệp vụ chính (đăng ký/đăng nhập, tải lên và truy vấn tài liệu bằng RAG, quản lý hồ sơ cá nhân) hoạt động đúng như thiết kế, đồng thời đánh giá khả năng giám sát, bảo mật và độ ổn định của quy trình CI/CD.

Khác với các chương triển khai hạ tầng (tạo tài nguyên qua giao diện AWS Console), phần kiểm thử chủ yếu thực hiện qua terminal (`curl`, AWS CLI) và đối chiếu kết quả trên Console, nên nội dung sẽ tập trung vào bảng test case và log thực tế hơn là ảnh chụp màn hình từng bước.

### Nội dung thực hiện

1. [Kiểm thử xác thực](5.5.1-Authentication/)
2. [Kiểm thử tải lên tài liệu & RAG](5.5.2-Document-RAG/)
3. [Kiểm thử bảo mật](5.5.3-Security/)
4. [Kiểm thử hồ sơ cá nhân](5.5.4-Profile/)
5. [Giám sát & Nhật ký hệ thống](5.5.5-Monitoring/)
6. [Kiểm thử tự động CI/CD](5.5.6-CICD/)
