---
title: "TIL#3: Text Translation sử dụng Firebase ML Kit"
categories:
  - TodayILearned
tags:
  - kotlin
  - firebase
  - ml-kit
---

#Lưu ý trong bài viết mình xin phép chèn 1 số từ tiếng Anh vào, vì nếu dịch sang tiếng Việt nghe sẽ rất … ngớ ngẩn.

Hey hey anh em! Lâu lắm mới viết tiếp blog, mặc dù không biết blog mình có nhiều người đọc không :(

Hôm nay chúng ta sẽ thử vọc vạch 1 thứ vô cùng thú vị, đó là:

![ml-kit]({{site.url}}/assets/images/ml-kit.png)

Đúng rồi đó, chính là ML Kit, bộ toolkit cho phép các nhà phát triển có thể đưa các tính năng về machine learning lên các thiết bị di động.

Machine Learning (ML) không phải là khái niệm mới tính tới thời điểm hiện tại, nó đang càng ngày càng đóng 1 vị trí quan trọng trong lĩnh vực công nghệ, đặc biệt là những công việc dựa trên nền tảng dữ liệu lớn (Big Data).

Tuy nhiên, đi kèm với khả năng ưu việt, ML cũng vấp phải 1 số giới hạn nhất định, ví dụ như giới hạn về sức mạnh xử lý, hay giới hạn về khả năng lưu trữ thông tin.v..v.. Chính những điều này đã gây khó khăn không nhỏ cho những nhà phát triển khi muốn ứng dụng ML lên các thiết bị di động, những cỗ máy bỏ túi có hiệu năng nhỏ hơn rất nhiều so với yêu cầu tối thiểu của 1 thiết bị cần phải có để phục vụ cho các tác vụ ML.

Chính vì thế mà ML Kit được Google giới thiệu như 1 giải pháp giúp các nhà phát triển di động có thể dễ dàng triển khái các tính năng ML của mình vào trong ứng dụng, mà không phải lo lắng về việc tiêu tốn quá nhiều tài nguyên thiết bị.

---

## 1. Mục tiêu bài viết

Thôi lan man vậy đủ rồi, chi tiết hơn các bạn có thể lên website của Google để tìm hiểu kĩ hơn về ML Kit nhé. Trong bài viết này mình chỉ muốn thử 1 trong các tính năng mà ML Kit đã tích hợp sẵn trong SDK của họ: **Identify text's language** và **Translate Text**

---

## 2. Triển khai

### Kịch bản

Ở dây mình đã nghĩ ra 1 kịch bản vô cùng đơn giản để có thể sử dụng được 2 bộ kit này như sau:

- **Bước 1**: Nhập 1 đoạn văn bản bằng ngôn ngữ bất kì
- **Bước 2**: Ấn nút chức năng để tiến hành xác định ngôn ngữ đồng thời dịch ngôn ngữ đó sang tiếng Anh
- **Bước 3**: Hiển thị ngôn ngữ và đoạn văn bản kết quả

### Giao diện

Để đáp ứng được các tính năng trên thì ta có thể tạo 1 giao diện cơ bản như sau:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.appcompat.widget.LinearLayoutCompat xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical"
    android:padding="32dp"
    tools:context=".MainActivity">

    <androidx.appcompat.widget.AppCompatEditText
        android:id="@+id/inputText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="¡Hola Mundo!" />

    <ProgressBar
        android:id="@+id/progressBar"
        android:layout_width="60dp"
        android:layout_height="60dp"
        android:visibility="gone"
        tools:visibility="visible" />

    <androidx.appcompat.widget.AppCompatButton
        android:id="@+id/translateButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="12dp"
        android:text="Detect &amp; Translate" />

    <androidx.appcompat.widget.AppCompatTextView
        android:id="@+id/textStatus"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Idle"
        android:textColor="@android:color/black" />

    <androidx.appcompat.widget.LinearLayoutCompat
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="12dp"
        android:gravity="center"
        android:orientation="horizontal">

        <androidx.appcompat.widget.AppCompatTextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Detected Language: "
            android:textColor="@android:color/black" />

        <androidx.appcompat.widget.AppCompatTextView
            android:id="@+id/languageText"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textColor="@color/colorAccent"
            android:textStyle="bold"
            tools:text="en" />
    </androidx.appcompat.widget.LinearLayoutCompat>

    <androidx.appcompat.widget.AppCompatTextView
        android:id="@+id/resultText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="12dp"
        android:gravity="center"
        android:hint="Tranlated content in English here" />

</androidx.appcompat.widget.LinearLayoutCompat>
```

Và giao diện kết quả:

![translator-ui]({{site.url}}/assets/images/translator-ui.png)
