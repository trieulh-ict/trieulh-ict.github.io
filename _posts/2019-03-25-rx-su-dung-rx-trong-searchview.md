---
title: "RxBestPractices#1: Áp dụng Rx vào Search View trong Android"
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

--- 
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

**As a developer**, dưới con mắt của 1 lập trinh viên, ta sẽ đặt ra 1 số câu hỏi:
- Dữ liệu sẽ được truy xuất liên tục **trong khi** mỗi lần user nhập vào ô search, hay **sau khi** user kết thúc nhập liệu.
- Dữ liệu như thế nào thì được coi là hợp lệ.
- Điều gì xảy ra khi user truy xuất 1 dữ liệu cùng lúc nhiều lần
- Điều gì xảy ra nếu user truy xuất nhiều dữ liệu liên tục trong 1 khoảng thời gian ngắn.

Chốt lại, để đảm bảo tính năng đưa đến tay người dùng được toàn vẹn, trước mắt ta phải giải quyết được tất cả các câu hỏi được đặt ra ở trên.
#### Let's begin!

## 3. Thực thi

Trước hết ta cần tạo 1 UI cơ bản: 

![final-result]({{site.url}}/assets/images/20190325-ui.png)


UI tạo ra chủ yếu để test tính năng của Rx, vì vậy ta tạm thời bỏ qua việc hiển thị kết quả mà tập trung vào 3 thành phần chính:
- `EditText` để nhập String query
- `Api call` text để hiển thị số lần Api sẽ được gọi khi user nhập query
- `Last search` text thể hiện query cuối cùng được gọi lên server

### a. Mock dữ liệu Api
Đầu tiên ta sẽ mock 1 function thể hiện việc truy xuất dữ liệu lên server: 

```kotlin
    private var lastSearch: String? = null

    private var searchCount: Int = 0

    private fun searchData(query: String): List<String> {
        lastSearch = query
        searchCount += 1
        return data.filter { it.contains(query, true) }
    }

    companion object {
        val data = listOf("a", "ab", "bc", "abcd")
    }
```

Function ở đây `searchData` với `query` là dữ liệu đầu vào và Mock Api sẽ trả về 1 List các dữ liệu tương ứng. Mỗi 1 lần Api đc thực thi, ta sẽ tăng `searchCount` lên 1 đơn vị, thể hiện đúng số lần Api được gọi, và lưu `query` gần nhất vào `lastSearch`.

### b. Tạo listener cho EditText
Tiếp theo, ta sẽ tạo listener để lắng nghe tất cả dữ liệu text được user nhập vào EditText.

Thông thường có thể gọi trực tiếp hàm `addTextChangedListener` của `EditText` để implement `TextWatcher` interface. Tuy nhiên để tránh việc override những function không cần thiết, ta sẽ tách việc implement ra ngoài Activity cho gọn bằng cách: 

```kotlin
object SearchViewObservable {
    fun fromView(view: EditText): Observable<String> {
        val subject: PublishSubject<String> = PublishSubject.create()

        view.addTextChangedListener(object : TextWatcher{
            override fun afterTextChanged(s: Editable?) {
            }

            override fun beforeTextChanged(s: CharSequence?, start: Int, count: Int, after: Int) {
            }

            override fun onTextChanged(s: CharSequence?, start: Int, before: Int, count: Int) {
                s?.let { text ->
                    subject.onNext(text.toString())
                }
            }
        })

        return subject
    }
}
```

và ở `Activity`, ta sẽ bind listener vào `EditText` như sau:

```kotlin
    private fun initView() {
        SearchViewObservable.fromView(searchEditText)
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe {
                ...
            }
    }
```

### c. Gọi Api
Triến lược tương đối đơn giản, cứ mỗi lần User thay đổi query trong `EditText`, `PublishSubject` trong `SearchViewObservable` sẽ bắn nội dung query, và ta sẽ sử dụng query đó để truy xuất server, khi dữ liêu trả về, thông tin về `Api call` và `Last search` sẽ được cập nhật:

```kotlin
        SearchViewObservable.fromView(searchEditText)
            .flatMap { text -> Observable.just(searchData(text)).delay(3000, TimeUnit.MILLISECONDS) }
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe {
                lastSearchText.text = "Last search: $lastSearch"
                countText.text = "Api call: $searchCount"
            }
```

Ở đây ta giả đinh là Api sẽ được trả về sau 3000 milliseconds.

Vậy là ta đã implement xong 1 chức năng cơ bản của Search. Công việc tiếp theo là giải quyết tất cả những câu hỏi ở đầu bài.

### d. Tối ưu hoá tính năng
### Dữ liệu sẽ được truy xuất liên tục **trong khi** mỗi lần user nhập vào ô search, hay **sau khi** user kết thúc nhập liệu.

Câu này đã được giải quyết bằng cách đặt `onNext` của `PublishSubject` trong `onTextChanged`, đồng nghĩa với việc Api sẽ được gọi ngay **trong khi** User nhập dữ liệu, giải quyết được 1 vấn đề về UX đó là bỏ qua việc User phải bấm thêm 1 submit button.

### Dữ liệu như thế nào thì được coi là hợp lệ.

Tuỳ vào bài toàn, ta sẽ định nghĩa về khái niệm *hợp lệ* khác nhau. Các khả năng có thể xảy ra:
- Dữ liệu chỉ có alphabet character
- Dữ liệu giới hạn kí tự
- Dữ liệu không được phép xuống dòng

Những vấn đề này hoàn toàn có thể xử lý ở phần UI (xml). Ở đây ta sẽ xét 1 case đơn giản là `query` phải khác rỗng:

```kotlin
    private fun initView() {
        SearchViewObservable.fromView(searchEditText)
            .filter { it.trim().isNotEmpty() } <---- This line
            .flatMap { text -> Observable.just(searchData(text)).delay(3000, TimeUnit.MILLISECONDS) }
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe {
                ...
            }
    }
```

### Điều gì xảy ra khi user truy xuất 1 dữ liệu cùng lúc nhiều lần
1 vấn đề phổ biến là gọi 1 Api với cùng 1 input nhiều lần. Có thể là do lỗi duplicate Api từ developer, hoặc có thể là do user spam nút submit liên tục. Mọi thứ đều có thể xảy ra.

Ta có thể xử lý bằng cách thêm operator `distinctUntilChanged()` vào chain:

```kotlin
    private fun initView() {
        SearchViewObservable.fromView(searchEditText)
            .filter { it.trim().isNotEmpty() }                 
            .distinctUntilChanged() <---- This line
            .flatMap { text -> Observable.just(searchData(text)).delay(3000, TimeUnit.MILLISECONDS) }
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe {
                ...
            }
    }
```

Operator này đảm bảo mọi pending emitting value là duy nhất. Tức là khi ta nhập `abc`, ở đây api chờ 3s mới có dữ liệu trả về. Trong vòng 3s đấy đồng nghĩ với việc query `abc` chưa được hoàn thành, nếu ta nhập thêm `abcd` rồi xoá `d` lại thành `abc`, thì `distinctUntilChanged()` sẽ bỏ qua việc emit `abc` lần nữa. Điều này đã giúp ta tránh việc request 1 dữ liệu cùng lúc nhiều lần, giảm thiểu việc lãng phí request.

### Điều gì xảy ra nếu user truy xuất nhiều dữ liệu liên tục trong 1 khoảng thời gian ngắn.
Hãy để ý vào UX, về lý mà nói, dữ liệu đầu ra sẽ chỉ hiển thị kết quả cho query mới nhất. Vậy điều gì xảy ra mới những request trước đấy? Thay vì bắt Observer xử lý, ta có thẻ sử dụng `switchMap` thay vì `flatMap`.
`switchMap` sẽ chỉ truyền dữ liệu của query mới nhất cho `Observer` và bỏ qua hết những query cũ.

```kotlin
        SearchViewObservable.fromView(searchEditText)
            .filter { it.trim().isNotEmpty() }
            .distinctUntilChanged()
            .switchMap { text -> Observable.just(searchData(text)).delay(3000, TimeUnit.MILLISECONDS) } <---- This line
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe {
                ...
            }
```

### Hạn chế số lần xử lý dữ liệu trong 1 khoảng thời gian.
Phương pháp cuối cùng chưa được đề cập ở trên, điều này tuỳ vào ý định của developer. Trong trường hợp ta muốn giới hạn số lần gọi Api, giả sứ là tối đã 1 query/giây. Lợi ích ở đây là hạn chế lượng công việc mà Rx phải xử lý (filter, switch, distinct...), giúp cải thiện hiệu năng cho CPU tương đối lớn trong 1 số trường hợp (mặc dù trong ví dụ này thì k cần thiết vì dữ liệu tương đối đơn giản).

Operator được sử dụng ở đây là `debound`. Về cơ bản, `debound` sẽ tiến hành kiểm tra xem trong 1 khoảng thời gian nhất định **tính từ lần emit mới nhất**, nếu không có thêm query nào được emit, nó sẽ tiến hành xử lý chain. Trường hợp khoảng thời gian đấy có thêm data được emit, `debound` sẽ tính lại từ đầu.

```kotlin
        SearchViewObservable.fromView(searchEditText)
            .debounce(1000, TimeUnit.MILLISECONDS) <---- This line
            .filter { it.trim().isNotEmpty() }
            .distinctUntilChanged()
            .switchMap { text -> Observable.just(searchData(text)).delay(3000, TimeUnit.MILLISECONDS) }
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe {
                ...
            }
```

### e. So sánh hiệu năng
Lý thuyết mãi rồi, đến bước cuối cùng, giờ ta sẽ so sánh xem việc sử dụng cơ số operator trên sẽ mang lại lợi ích thế nào so với việc không dùng.

Chiến lược test sẽ thực hiện liên tục các bước như sau trong khoảng thời gian ngắn:
- Nhập `123`
- Thêm `4`
- Xoá `4`
- Thêm `45678`

#### Trường hợp không dùng thêm operator

```kotlin
        SearchViewObservable.fromView(searchEditText)
            .flatMap { text -> Observable.just(searchData(text)).delay(3000, TimeUnit.MILLISECONDS) }
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe {
                lastSearchText.text = "Last search: $lastSearch"
                countText.text = "Api call: " + searchCount.toString()
            }
```

Kết quả: 

![before]({{site.url}}/assets/images/20190325-before.png)

Api được gọi đến 10 lần.

#### Trường hợp dùng thêm operator

```kotlin
        SearchViewObservable.fromView(searchEditText)
            .debounce(1000, TimeUnit.MILLISECONDS)
            .filter { it.trim().isNotEmpty() }
            .distinctUntilChanged()
            .switchMap { text -> Observable.just(searchData(text)).delay(3000, TimeUnit.MILLISECONDS) }
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe {
                lastSearchText.text = "Last search: $lastSearch"
                countText.text = "Api call: " + searchCount.toString()
            }
```

Kết quả: 

![after]({{site.url}}/assets/images/20190325-after.png)

Api được gọi duy nhất 1 lần.

Ngon lành cành đào. :v

---
# Kết luận

OK bài viết đầu tiên về series **Rx Best Practices** kết thúc ở đây. Chúng ta đã thấy được Rx đã hỗ trợ rất đa dạng và tinh gọn trong việc xử lý dữ liệu theo chuỗi như ví dụ ở trên.

Cảm ơn anh em đã dành thời gian đọc. Néu có bất kì 1 case study nào muốn mình viết ở các bài sau thì cứ nhắn tin qua tài khoản Social của mình ở Profile nhé.

Happy coding!