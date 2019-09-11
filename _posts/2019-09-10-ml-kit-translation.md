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

**Machine Learning** (ML) không phải là khái niệm mới tính tới thời điểm hiện tại, nó đang càng ngày càng đóng 1 vị trí quan trọng trong lĩnh vực công nghệ, đặc biệt là những công việc dựa trên nền tảng dữ liệu lớn (Big Data).

Tuy nhiên, đi kèm với khả năng ưu việt, ML cũng vấp phải 1 số giới hạn nhất định, ví dụ như giới hạn về sức mạnh xử lý, hay giới hạn về khả năng lưu trữ thông tin.v..v.. Chính những điều này đã gây khó khăn không nhỏ cho những nhà phát triển khi muốn ứng dụng ML lên các thiết bị di động, những cỗ máy bỏ túi có hiệu năng nhỏ hơn rất nhiều so với yêu cầu tối thiểu của 1 thiết bị cần phải có để phục vụ cho các tác vụ ML.

Chính vì thế mà **ML Kit** được Google giới thiệu như 1 giải pháp giúp các nhà phát triển di động có thể dễ dàng triển khái các tính năng ML của mình vào trong ứng dụng, mà không phải lo lắng về việc tiêu tốn quá nhiều tài nguyên thiết bị.

## 1. Mục tiêu bài viết

Thôi lan man vậy đủ rồi, chi tiết hơn các bạn có thể lên website của Google để tìm hiểu kĩ hơn về **ML Kit** nhé. Trong bài viết này mình chỉ muốn thử 1 trong các tính năng mà ML Kit đã tích hợp sẵn trong SDK của họ: **Identify text's language** và **Translate Text**

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

### Cài đặt thư viện

Để có thể sử dụng đc các tính năng này của ML Kit, ta phải kết nối đến 1 firebase project. Bạn có thể tham khảo ở [đây](https://firebase.google.com/docs/android/setup) để có hướng dẫn chi tiết.

Sau khi kết nối xong, ta đã có thể sử dụng các thư viện của ML Kit. Ở đây mình sử dụng 2 bộ thư viện về **Identify text's language** và **Translate Text** nên sẽ import vào file `app/build.gradle` như sau:

```kotlin
dependencies {
    ...
    //ML Kit
    implementation 'com.google.firebase:firebase-ml-natural-language:21.0.2'
    implementation 'com.google.firebase:firebase-ml-natural-language-language-id-model:20.0.5'
    implementation 'com.google.firebase:firebase-ml-natural-language-translate-model:20.0.5'
}
```

OK. đã xong bước cài đặt. Bây giờ ta sẽ tiến hành triển khai các tính năng của nó

### Tính năng xác định ngôn ngữ (Identify text's language)

Tính năng đầu tiên là **Identify text's language**, đại khái là ta sẽ sử dụng nó để xác định xem 1 đoạn văn bản bất kì thuộc ngôn ngữ nào.

Ở màn hình Main (ở đây mình dùng class mặc định `MainActivity`) ta sẽ thực thi việc xác định ngôn ngữ sau khi bấm vào nút **DETECT & TRANSLATE**:

```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        ...
        translateButton.setOnClickListener(this)
    }

    override fun onClick(v: View) {
        when (v.id) {
            R.id.translateButton -> detectLanguage()
        }
    }
```

phương thức `detectLangue()` sẽ được gọi:

```kotlin
    private fun detectLanguage() {
        val languageIdentifier = FirebaseNaturalLanguage.getInstance().languageIdentification
        textStatus.text = "Detecting"
        inputText.text?.let { content ->
            languageIdentifier.identifyLanguage(content.toString())
                .addOnSuccessListener { code ->
                    if (code.equals("und")) {
                        languageText.text =  "None"
                        textStatus.text = "Can't Detect"
                    } else {
                        languageText.text = code
                    }

                }
                .addOnFailureListener {
                    Toast.makeText(this@MainActivity, it.message, Toast.LENGTH_LONG).show()
                    languageText.text = "error"
                    textStatus.text = "Detecting Failed"
                }

        }
    }
```

Trong phương thức này, mình đã tạo instance `languageIdentifier` từ ML Kit và gọi hàm `identifyLanguage()` để xác định đoạn text trong input. Ở đây ta có thể thêm listener khi success hay failure để kiểm tra kết quả. Với đoạn code trên, nếu xác định được ngôn ngữ, ta sẽ hiển thị lên màn hình là `Detected Language: [code]`, còn không sẽ là `Detected Language: None`.

Ok, giờ hãy test thử với đoạn text `¡Hola Mundo!` xem thế nào nhé:

![translator-detect]({{site.url}}/assets/images/translator-detect.gif)

Increíble!! Vậy là nó đã tự động xác định được đây là ngôn ngữ Tây Ban Nha. Giờ ta chuyển sang phần tiếp theo là dịch sang tiếng anh.

### Tính năng dịch (Translate Text)

Sau khi đã xác định được ngôn ngữ của đoạn văn bản, chúng ta sẽ tiến hành tạo 1 model phục vụ việc dịch ngôn ngữ từ source language sang target language (ở đây là ES sang EN).

Về cơ bản, mỗi bộ dịch sẽ tương đương với 1 model khác nhau. Không như tính năng xác định ngôn ngữ chúng ta có thể hoàn toàn thực hiện ở local, thì với tính năng dịch, việc đầu tiên là ta phải tải model dó về trước đã.

```kotlin
    private fun translateLanguage(code: String) {
        toggleFunctionalViews(true)
        FirebaseTranslateLanguage.languageForLanguageCode(code)?.let { sourceLanguageCode ->
            textStatus.text = "Downloading Model"
            val options = FirebaseTranslatorOptions.Builder()
                .setSourceLanguage(sourceLanguageCode)
                .setTargetLanguage(FirebaseTranslateLanguage.EN)
                .build()

            val englishTranslator = FirebaseNaturalLanguage.getInstance().getTranslator(options)

            englishTranslator.downloadModelIfNeeded()
                .addOnSuccessListener {
                    textStatus.text = "Translating"
                }
                .addOnFailureListener {
                    Toast.makeText(this@MainActivity, it.message, Toast.LENGTH_LONG).show()
                    resultText.text = "Error!! Can't download model!!!"
                    textStatus.text = "Download failed"
                }
        }
    }

    private fun toggleFunctionalViews(isLoading: Boolean) {
        progressBar.visibility = if (isLoading) View.VISIBLE else View.GONE
        translateButton.visibility = if (isLoading) View.GONE else View.VISIBLE
    }
```

Hàm `translateLanguage()` được gọi với param `code` lấy từ kết quả của hàm xác định ngôn ngữ:

```kotlin
    private fun detectLanguage() {
        val languageIdentifier = FirebaseNaturalLanguage.getInstance().languageIdentification
        textStatus.text = "Detecting"
        inputText.text?.let { content ->
            languageIdentifier.identifyLanguage(content.toString())
                .addOnSuccessListener { code ->
                    if (code.equals("und")) {
                        languageText.text =  "None"
                        textStatus.text = "Can't Detect"
                    } else {
                        languageText.text = code
                        translateLanguage(code)   <--- ADD HERE
                    }

                }
                .addOnFailureListener {
                    Toast.makeText(this@MainActivity, it.message, Toast.LENGTH_LONG).show()
                    languageText.text = "error"
                    textStatus.text = "Detecting Failed"
                }

        }
    }
```

Quay lại `translateLanguage()`, ở đây ta khởi tạo 1 instance `englishTranslator` với params là `options` được build từ source và target language. Đồng nghĩa với việc bộ translator ở đây sẽ có tính năng translate từ `ES` sang `EN` (Không dịch ngược lại).
model sau khi download sẽ được lưu lại trong local cho lần sử dụng tiếp theo, hạn chế việc ta phải download lại liên tục mỗi lần sử dụng.

Khi nhảy vào `downloadModelIfNeeded()` đồng nghĩa với việc model đã sẵn sàng cho việc dịch, ở đây ta sẽ gọi hàm `translate()` của instance `englishTranslator`:

```kotlin
englishTranslator.downloadModelIfNeeded()
                .addOnSuccessListener {
                    textStatus.text = "Translating"
                    englishTranslator.translate(inputText.text.toString())    <---- ADD HERE
                        .addOnSuccessListener {
                            resultText.text = it
                            toggleFunctionalViews(false)
                            textStatus.text = "Translation success"
                        }
                        .addOnFailureListener {
                            Toast.makeText(this@MainActivity, it.message, Toast.LENGTH_LONG).show()
                            resultText.text = "Error!! Can't translate!!!"
                            textStatus.text = "Translation failed"
                        }
                }
                .addOnFailureListener {
                    Toast.makeText(this@MainActivity, it.message, Toast.LENGTH_LONG).show()
                    resultText.text = "Error!! Can't download model!!!"
                    textStatus.text = "Download failed"
                }
```

Done. Vậy là đã xong phần code, bây giờ hãy thử xem tính năng có hoạt động trơn tru không nào:

![translator-translate]({{site.url}}/assets/images/translator-translate.gif)

Lần đầu download hơi lâu nên mình skip sang lần 2 như ở dưới :P

![translator-translate-2]({{site.url}}/assets/images/translator-translate-2.gif)

Sau khi đã download model về local thì việc dịch gần như là ngay lập tức. Trong thực tế ta có thể init các model cần thiết bằng cách download trước để trải nghiệm người dùng được tốt hơn.

## 3. Kết

Đối với những người chưa có base về ML như mình thì những bộ kit có sẵn như thế này là cực kì hữu ích. Ngoài việc xác định và dịch ngôn ngữ, **ML Kit** còn có sẵn các model khác như Face detection, Barcode scanning, Image labelling, Landmark detection. v...v...

Ngoài ra nếu bạn muốn đưa các model riêng của mình vào trong ứng dụng (bằng TensorFlow chẳng hạn), **ML Kit** cũng hỗ trợ import và compress model để phù hợp hơn vs kích cỡ của ứng dụng di động. Phần này các bạn có thể đọc thêm trên trang chủ của firebase ở [đây](https://firebase.google.com/docs/ml-kit).

Thanks and Happy coding!
