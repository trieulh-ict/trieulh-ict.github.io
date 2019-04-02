---
title: "RxBestPractices#2: Áp dụng Rx vào các bài toán API request"
categories:
  - TodayILearned
tags:
  - android
  - kotlin
  - rxjava
---

#Lưu ý trong bài viết mình xin phép chèn 1 số từ tiếng Anh vào, vì nếu dịch sang tiếng Việt nghe sẽ rất … ngớ ngẩn. 


# Lời nói đầu 

Chao xìn anh em! Hôm nay mình sẽ tiếp tục series Rx Best Practices với chủ đề thứ 2 tương đối phổ biến: Sử dụng RxJava trong request API.

RxJava + Retrofit thì chúng ta không còn lạ gì nữa rồi, 1 giải pháp kết hợp vô cùng mạnh mẽ trong việc xử lý API ở Android. Bản chất của việc tương tác với API service về cơ bản cũng chỉ là bài toán về xử lý dữ liệu. Tuy nhiên lý thuyết là vậy, ứng dụng Rx vào trong nhiều bài toán thực tế trở nên khó khăn rât nhiều nếu ta không nắm rõ nó.

Trong bài viết này, mình sẽ mặc định là các bạn đã nắm rõ phần config Retrofit trong project, và sẽ chỉ tập trung vào vai trò của Rx (hay RxJava) trong việc xử lý request và response ở client.

# Mục tiêu bài viết

Mục tiêu cũng như tiêu đề rồi, trong bài này mình sẽ liệt kê ra 1 số case study mà sự xuất hiện của Rx sẽ giúp việc triển khai tầng Network của bạn trở nên dễ dàng hơn rất nhiều.

## 1. Sử dụng **Single** hay **Completable** để đóng gói dữ liệu trả về.

Trong nhiều tutorial của *Retrofit*, chúng ta có thể bắt gặp sự xuất hiện của *Observable* trong hầu hết các ví dụ về API response của Retrofit. Ví dụ:

```kotlin
interface APIClient {

    @GET("my/api/path")
    getMyData(): Observable<MyData>
}
```

Và khi gọi API trong use case, chúng ta sẽ implement 3 method:

```kotlin
apiClient.getMyData()
    .subscribe(myData -> {
        // handle data
    }, throwable -> {
        // handle error
    }, () -> {
        // handle after onNext
    });
```

Phần còn lại để Retrofit lo, 1 sự kết hợp vô cùng mạnh mẽ. 

Ở đây việc sử dụng **Observable** hoàn toàn ổn, tuy nhiên trong nhiều trường hợp, ta tự đặt ra câu hỏi: 
- Liệu có cần thiết phải implement cả 3 method không?
- API chỉ trả lại dữ liệu duy nhất 1 lần, liệu sử dụng **Observable** ở đây có cần thiết (over-engineering) quá không?
- `onComplete` luôn được gọi ngay sau `onNext`, liệu ta có thể gộp lại làm 1 không?

Để giải quyết 3 câu hỏi ở trên, tuỳ vào trường hợp, ta có thể thay thế bằng **Single** hoặc **Completable**

### a. Single
Thay **Observable** trong API interface bằng **Single** nếu ta đảm bảo API sẽ chỉ trả kết quả 1 lần trên mỗi lần gọi, và chúng ta chỉ quan tâm đến việc API có trả về response hay lỗi:

```kotlin
interface APIClient {

    @GET("my/api/path")
    getMyData(): Single<MyData>
}
```
Lúc này trong use case:

```kotlin
apiClient.getMyData()
    .subscribe(myData -> {
        // handle data
    }, throwable -> {
        // handle error
    });
```

Code trở nên tường minh và ngắn gọn hơn vì chúng ta không cần thiết phải implement thêm onCompleted nữa.

Trong 1 số case, nếu ta muốn sử dụng 1 số operator của **Observable** mà không có trong **Single**, ta có thể convert sang **Observable** bằng cách sau:

```kotlin
apiClient.getMyData()
    .toObservable()
```

hoặc ngược lại convert **Observable** sang **Single**:

```kotlin
apiClient.getMyData()
    .singleOrError()
```

### b. Completable
Trong trường hợp ta không quan tâm đến giá trị trả về, ví dụ khi gọi PUT/POST request, API thường trả lại chính object được request, việc sử lý dữ liệu này là không cần thiết, ta có thể dùng **Completable**:

```kotlin
interface APIClient {

    @PUT("my/api/updatepath")
    fun updateMyData(@Body data: MyData): Completable<MyData>
}
```

Lúc này trong use case ta chỉ quan tâm xem request có thành công hay lỗi, bỏ qua việc xử lý giá trị trả về:

```kotlin
apiClient.updateMyData(myUpdatedData)
    .subscribe(() -> {
        // handle completion
    }, throwable -> {
        // handle error
    });
```

Về bản chất, **Completable** tương đối khác với **Single** hay **Observable** vì nó không emit dữ liệu. Chính vì thế ta không thể convert trực tiếp **Completable** sang **Observable** vì hiển nhiên không có cách nào để biết khi nào thì **Observable** gọi `onComplete` cả. 
**Single** thì lại được, vì tính chất **Single** chỉ emit dữ liệu 1 lần, ta sẽ xác định được thời điểm gọi `onComplete` là sau `onNext` . Vì thế trong **Single** sẽ support việc đó bằng operator `toCompletable`, cách dùng tương tự `toObservable`.

Trong 1 số trường hợp ta cũng có thể dùng operator `andThen` của **Completable** để chuyển sang **Single** hay **Observable**:

```kotlin
apiClient.updateMyData(myUpdatedData)
    .andThen(Observable.just(ResultCode.SUCCESS))
    ...
```

## 2. Sử dụng startWith() trong việc render thumbnail Image
Thông trường, ta vẫn có thể set source ảnh trực tiếp thông qua UI xml hay sử dụng các method của Picasso, Glide. Tuy nhiên những cách như vậy không mang lại nhiều lợi ích trong việc kiểu soát luồng dữ liệu. Ví dụ:

Trong Xml:
```Java
<ImageView
            android:id="@+id/avatar"
            android:layout_width="60dp"
            android:layout_height="60dp"
            android:src="@drawable/avatar_thumbnail.png"/>
```

Trong Activity:

```kotlin
api.requestAvatarUrl(userId)
    .subscribe { avatarUrl ->
        RequestOptions options = new RequestOptions()
                    .centerCrop()
                    .placeholder(R.mipmap.ic_launcher_round)
                    .error(R.mipmap.ic_launcher_round);

        Glide.with(this).load(avatarUrl).apply(options).into(avatar);
    }
```

Ở đây ta có thể dễ dàng nhận ra đây là 1 case xử lý không tốt. Để hiển thị avatar, ta đã phải render tận 3 lần: Từ Xml, từ placeholder method, và cuối cùng là từ `avatarUrl`. Ngoài việc xem xét về hiệu năng, ta có thể dễ dàng nhận ra việc set Avatar đã bị phân tán ở 2 nơi. 

Để giải quyết vấn đề này, ta có thể sư dụng operator `startWith()`:

```kotlin
api.requestAvatarUrl(userId)
    .startWith(thumbnailUrl)
    .subscribe { avatarUrl ->
        RequestOptions options = new RequestOptions()
                    .centerCrop()
                    .error(R.mipmap.error_image);
        Glide.with(this).load(avatarUrl).apply(options).into(avatar);
    }
```

Điều này mang lại 2 lợi ích:
- Việc quản lý thumbnail giờ là việc của server, `thumbnailUrl` là static và server có thể thay đổi nội dung mà không cần sự can thiệp của client.
- Thống nhất được việc set Avatar vào 1 chỗ duy nhất là ở `Activity`.

`startWith()` về cơ bản rất hữu dụng cho việc ta muốn mock data lên UI trước khi dữ liệu từ API được trả về.

Tuy nhiên, thay vì sử dụng mock data, ta có 1 cách tối ưu hơn nếu ứng dụng có hỗ trợ cached data.

## 3. Sử dụng concat() trong trường hợp ứng dụng hỗ trợ Cache

**Case study:** Trước khi dữ liệu được trả về từ Api, ta muốn hiển thị dữ liệu đã được lưu trữ sẵn trong local database.

Trong trường hợp này ta có thể sử dụng đến operator `concat()` bằng cách sau:

```kotlin
Observable.concat(dataSource.getLocalData(), dataSource.getRemoteData())
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .toObservable()
    .subscribe(data ->
        //Render to UI
    );
```

Ở đây, `concat()` đảm bảo dữ liệu từ Local sẽ được xử lý và render trước khi ta request API.

![zip](http://reactivex.io/documentation/operators/images/concat.png "Zip Operator")

Câu hỏi đặt ra ở đây: Trong trường hợp ta gặp Exception khi lấy dữ liệu trong Local Database, điều gì sẽ xảy ra?

Với operator `concat`, khi xảy ra lỗi, **Observable** sẽ gọi `onError` và dừng tiến trình. Chính vì vậy nếu ta gặp **Exception** (Ví dụ như **IOException**) trong method `getLocalData()`, **Observable** sẽ không gọi tiếp `getRemoteData()` nữa. Vì vậy ta phải chủ động handle **Exception** để đảm bảo tiến trình được thực hiện đầy đủ:

```kotlin

val getLocalDataSource = dataSource.getLocalData()
                            .onErrorResumeNext {e ->
                                return Observable.empty()
                            }

val getRemoteDataSource = dataSource.getRemoteData()

Observable.concat(getLocalDataSource, getRemoteDataSource)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .toObservable()
    .subscribe(data ->
        //Render to UI
    );

```

Operator `onErrorResumeNext` được thêm vào cho riêng local data, trong trường hợp gặp **Exception**, tiến trình sẽ chủ động gọi `Observable.empty()` để đảm bảo viẹc emit data được tiếp tục.

Note: `empty()` không emit data mà huỷ luôn tiến trình lỗi.

Trong trường hợp bạn muốn trả lại 1 giá trị mock data trong trường hợp lỗi, có thể tìm hiểu thêm operator `onErrorReturn`.

## 4. Sử dụng zip() trong trường hợp cần kết hợp dữ liệu từ nhiều API

Trong trường hợp cần kêt hợp dữ liệu từ 2 API, có 1 vấn đề gặp phải là API request là async method, đồng nghĩa với việc thời điểm trả về response là không cố định.

Ví dụ, ta muốn build 1 **UserUIModel** chứa đầy đủ thông tin của 1 user, nhưng server hiện đang không cung cấp sẵn API trực tiếp mà chỉ có 2 API để GET profile cá nhân đến profile công viêc.

Lúc này ta phải sử dụng đến operator `zip`:

```kotlin
val getPersonalDataSource = api.getPersonalData()
val getWorkDataSource = api.getWorkData()

Observable.zip(getLocalDataSource, getRemoteDataSource, { personalData, workData ->
        buildUserUIModel(personalData, workData)
    })
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(userDataModel ->
        //Render to UI
    );
```
Operator `zip` đóng vai trò request 2 API, và đợi đến khi cả 2 hoàn thành, sẽ kết hợp dữ liệu trả về và emit 1 data duy nhất.

![zip](http://reactivex.io/documentation/operators/images/zip.o.png "Zip Operator")

Để ý kỹ Diagram, ta sẽ nhận thấy mặc định 2 Observable sẽ được xử lý tuần tự trên cùng 1 Thread (được gọi thông qua `subscribeOn()`). Tức là Api đầu tiên emit data, chờ Api thứ 2 hoàn thành mới tiền hành kết hợp dữ liệu.

Hừm, vậy làm thế nào để 2 Api này được gọi song song nhỉ?

Bài toán được giải quyết bằng cách ta sẽ cho mỗi Api request được subscribe trên **Thread** riêng biệt như sau:

```kotlin
val getPersonalDataSource = api.getPersonalData()
                                .subscribeOn(Schedulers.newThread())
val getWorkDataSource = api.getWorkData()
                            .subscribeOn(Schedulers.newThread())

Observable.zip(getLocalDataSource, getRemoteDataSource, { personalData, workData ->
        buildUserUIModel(personalData, workData)
    })
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(userDataModel ->
        //Render to UI
    );
```

Mỗi Api sẽ được thực hiện trên 1 **Thread** riêng, sau đó được đẩy về **IO Thread** để kết hợp. Vậy là ta đã giảm thiểu được thời gian chờ như ở cách trước.


## Kết
Trên đây chỉ là 4 cách sử dụng Rx thông dụng trong các trường hợp làm việc với Network. Nếu có thêm bài toán nào cần giải quyết liên quan đến vấn đề này, hi vọng anh em nhiệt tình đóng góp.

Happy Coding!