---
title: "[Design Pattern] Delegate trong kotlin"
categories:
  - Design Pattern
tags:
  - kotlin
  - design-pattern
---

#Lưu ý trong bài viết mình xin phép chèn 1 số từ tiếng Anh vào, vì nếu dịch sang tiếng Việt nghe sẽ rất … ngớ ngẩn. 

Delegation là 1 design pattern phải nói là vừa lạ vừa quen đối với tương đối lập trình viên. Dân làm iOS thì chắc không xa lạ gì rồi nhưng với dân Android thì lại khác. Lạ là do khái niệm `delegate` không được dùng nhiều trong java, còn quen thì pattern này được sử dụng liên tục, nhưng ít ai để ý. Vậy thực hư khái niệm của Design pattern này như thế nào, mời anh em đọc tiếp.

## 1. Delegate là gì?
Về cơ bản, trong 1 class có định nghĩa 1 delegate, thì delegate đó đóng vai trò như 1 đối tượng (`object`) dùng để bao đóngđóng một hoặc nhiều phương thức.

Thôi mấy cái khái niệm định nghĩa anh em tìm thêm trên google nhé, để mình đưa ra 1 ví dụ đơn giản thế này cho dễ hiểu:

### a. Ví dụ về Delegate
Trong 1 công ty nhỏ có 1 cặp `Thư kí (Secrectary)`  và `Giám đốc (Boss)`. Dễ nhận thấy dù ở 2 vị trí khác nhau nhưng khái quát thì cả 2 người đều là `Người lao động` (Gọi mĩ miều là `Worker`). Mô hình hoá các đối tượng này ta có các lớp sau:

```java
public interface Worker() {}

public class Secretary implements Worker(){}

public class Boss implements Worker(){}
```

Giờ có 1 yêu cầu như này, là người lao động thì phải báo cáo kết quả công việc. Vậy ta định nghĩa 1 hàm tạm gọi là `report()` trong `interface` như sau:

```java
public interface Worker() {
    public Result report();
}
```

Tuy nhiên đối với từng lớp đối tượng, cách thức thực hiện hành động `report` sẽ khác nhau. `Thư kí` khi được yêu cầu báo cáo thì phải có trách nhiệm tổng hợp và gửi lại báo cáo trực tiếp:

```java
public class Secretary implements Worker(){
    public Result report(){
        return new Result();
    }
}
```

Còn trường hợp của ông `Giám đốc` thì sao. Ví dụ có 1 ông `Chủ tịch hội đồng quản trị` yêu cầu ông `Giám đốc` báo cáo tình hình công ty chẳng hạn. Đã là `Giám đốc` thì dăm ba cái việc giấy tờ sổ sách này tất nhiên phải để `Thư kí` làm rồi. Thế là ta lại định nghĩa class `Boss` như sau:

```java
public class Boss implements Worker(){
    private Secretary secretary;

    public Result report(){
        return secretary.report();
    }
}
```

Chi tiết hơn, có thể thấy trong class `Boss`, việc `report` đã được uỷ nhiệm(`delegate`) cho `Secretary`, thay vì `Boss` phải tự định nghĩa logic cho việc `report` này.

Done. Đây là 1 ví dụ cơ bản về Delegate trong Java.

### b. Ứng dụng của Delegate
- Thứ nhất, dễ thấy lợi ích nhãn tiền của nó là uỷ nhiệm công việc _đáng nhẽ phải tự làm_ cho 1 đại diện khác. 
- Thứ hai, Delegate được coi như 1 pattern thay thế cho inheritance trong 1 số trường hợp không muốn sử dụng lại tất cả các biến, hàm của lớp cha (super class)

Ý thứ nhất đã được giải thích ở ví dụ trên, để giải thích cho ý thứ 2, ta có thể thử sử dụng _inheritance_ cho ví dụ:

```java
public class Boss extends Secretary(){
    public Result report(){
        return super.report();
    }
}
```

Nghe nó cứ sai sai, mặc dù tác vụ vẫn chạy đúng...

## 2. Delegate trong Kotlin

Kotlin hỗ trợ rất mạnh `Delegation Design Pattern` qua từ khoá `by`.

Viết lại ví dụ trên bằng Kotlin như sau:

```kotlin
interface Worker {
    fun report()
}

class Secretary() : Worker {
    override fun report() {
        return Result()
    }
}

class Boss(secretary: Secretary) : Worker by secretary
```

Không những thể hiện đúng tinh thần `Delegate` mà boilerplate code còn ngắn gọn hơn rất nhiều.

Để ý phần `Worker by secretary` có nghĩa là mọi phương thức của `Worker` đáng lẽ phải được định nghĩa trong class `Boss` thì sẽ được thay thế bằng phương thức tương ứng của `Secretary` (Với `secretary` là 1 object được truyền vào khi khởi tạo object `Boss`)

Ngoài việc hỗ trợ `delegate` cho class, Kotlin còn hỗ trợ nhiệt tình cho property. Cái này mình không đào sâu, anh em có thể đọc thêm tại [đây](https://kotlinlang.org/docs/reference/delegated-properties.html).

## 3. Kết luận
Đây chỉ là bài viết mang tính giới thiệu cơ bản. 

Sau khi đọc bài này, có lẽ anh em đã biết được thứ mà lâu nay mình sử dụng được gọi là `Delegate`, cũng hi vọng khi viết về khái niệm này, anh em sẽ đào sâu và sử dụng nó 1 cách hiệu quả hơn. Thanks đã chịu khó đọc đến đây.

Happy coding!