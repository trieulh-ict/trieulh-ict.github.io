---
title: "RxBestPractice #1: Áp dụng Rx vào Search View trong Android"
categories:
  - TodayILearned
tags:
  - android
  - kotlin
  - rxjava
---

#Lưu ý trong bài viết mình xin phép chèn 1 số từ tiếng Anh vào, vì nếu dịch sang tiếng Việt nghe sẽ rất … ngớ ngẩn. 


# Lời nói đầu 

Trong vài năm trở lại đây, Reactive Programming (Viết tắt là Rx) không hoàn toàn là khái niệm mới mẻ gì với Mobile Developer cả. Về mặt khái niệm, định nghĩa xin phép bỏ qua, vì các bạn có thể dễ dàng tìm thấy trên google.

Series **Rx Best Practices** được tạo ra nhằm mục đích cung cấp cho ae cách ứng dụng Rx vào trong các bài toán thực tế, giảm thiểu sự nhàm chán khi phải học cách sử dụng từng *term* hay *operator*. Thực ra thì mình viết cái này coi như seft-taught thôi, chính mình lúc mới tiếp cận Rx học còn thấy chán vãi ra :v.

Hãy nhớ kĩ 1 điều:

> Luôn có cách để sử dụng được Rx.

Trust me.


# Mục tiêu bài viết
- Tổng quan về bài toán áp dụng Rx vào trong việc implement tính năng Search trong Mobile (ở đây là Android).
- Giải thích các operator được sử dụng

## 1. Tổng quan bài toán

Không chỉ trên mobile, mà tất cả các ứng dụng nói chung (kể cả web, desktop) chúng ra đều có thể dễ dàng bắt gặp tính năng này. Search là 1 tính năng vô cùng cơ bản của ứng dụng, giúp user có thể dễ dàng tiếp cận vào những phần thông tin cần thiết, thay vì phải duyệt 1 lượng data lớn. Ngoài ra chúng ta còn có tính năng Filter như 1 tính năng nâng cao/tinh gọn của search.

Tuy nhiên về mặt người dùng, giao diện của search có thể rất đơn giản, thế nhưng để implement được nó 1 cách tối ưu nhất cả về hiệu năng lẫn giao diện thì là 1 bài toán cực kì khó. Mình có thể đảm bảo với các bạn rằng lượng codebase sẽ là vô cùng lớn và phức tạp, nếu không có Rx.

Trước hết, hãy mổ xẻ bài toán 1 chút xem chúng ta cần phải quan tâm những gì sẽ ảnh hưởng đến mức độ tối ưư của tính năng. Chú ý là mình sẽ bỏ qua phần giao diện, mà tập trung vào những thành phần ảnh hưởng đến data flow.

### Về mặt người dùng
-  User nhập thông tin cần search thông qua UI dưới dạng text
-  User nhận dữ liệu và hiển thị lên màn hinh dưới dạng list

### Về mặt kĩ thuật
- App sẽ phân tích dữ liệu đầu vào và chuyển về dạng String để lấy được query
- App gửi query lên server để lấy thông tin tương ứng
- App nhận dữ liệu trả về từ server và cho lên UI

## 2. Phân tích
Requirement tương đối đơn giản, về phía user, chúng ta không phải làm gì nhiều. Vậy ta sẽ tập trung về mặt kĩ thuật, xem Rx sẽ support chúng ta giải quyết những bài toán nào.

Hãy nhớ requirement ở trên mới chỉ là dạng thô, dưới con mắt của 1 developer, bạn cần phải đặt ra rất nhiều câu hỏi, để dảm bảo sẽ cover hết được các trường hợp xử lý đầu vào của user.

> Chúng ta sẽ không bao giờ biết được User sẽ làm cái quái gì với ứng dụng của mình đâu.

**As a developer**, dưới con mắt của 1 lập trinh viên:
- Dữ liệu sẽ được truy xuất liên tục **trong khi** mỗi lần user nhập vào ô search, hay **sau khi** user kết thúc nhập liệu.
- Dữ liệu như thế nào thì được coi là hợp lệ.
- Điều gì xảy ra khi user truy xuất lại dữ liệu đã xử lý trước đấy (cached)
- Điều gì xảy ra nếu user truy xuất nhiều dữ liệu liên tục trong 1 khoảng thời gian ngắn.