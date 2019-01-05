---
title: "TIL#1: Cách sử dụng Standard Functions trong Kotlin"
categories:
  - TodayILearned
tags:
  - til
  - kotlin
---

#Lưu ý trong bài viết mình xin phép chèn 1 số từ tiếng Anh vào, vì nếu dịch sang tiếng Việt nghe sẽ rất … ngớ ngẩn. 

Hi anh em! Mở đầu blog sẽ là 1 bài hướng dẫn cơ bản cách sử dụng các hàm standard của Kotlin. Đối với ae lập trình viên khi tiếp cận với Kotlin, sẽ thấy các bài tutorial sử dụng những function này rất là nhiều.

Dễ thấy nhất là apply, also, with, let và run. Ngoài ra thì 1 số code template của Kotlin khi gen ra sẽ có hàm `TODO()` (Vâng nó cũng là 1 function đấy, k phải `//TODO` đâu :v )

Muốn biết nội trong hàm này được implement thế nào ae có thể đọc thêm trong source code của Kotlin tại đây

Nói tiếp về 5 function kia, chúng ta hãy gọi thân thiện là các scope function. Và cách sử dụng của nó cũng đơn giản như tên gọi, cung cấp cho caller (object gọi function đó) một inner scope với caller là chủ thể. Và trong inner scope đấy, chúng ta có thể tuỳ ý sử dụng properties/methods của caller đấy thoải mái.

Vậy tại sao cách dùng giống nhau mà phải đẻ ra tận 5 thằng làm gì …?

Tất nhiên là cái gì sinh ra cũng có mục đích riêng của nó. Và tại sao việc chúng ta xem xét và sử dụng đúng cách từng function lại vô cùng quan trọng, thì mời ae đọc tiếp

# Cách sử dụng

Lấy hàm `let` làm ví dụ: 

```kotlin
class MainClass {
    val greeting = "Hello, World!"
 
    fun test() {
        val result = greeting.let {
            println(this)
            println(it)
            "Hello with let"
        }
 
        println(result)
    }
}
```

Với đoạn code trên, ta sẽ tạo 1 object `MainClass` có function `test` để có thể gọi được hàm let. Phân tích qua cú pháp của let :


- Ta có `greeting` là 1 `String` object, và đã là object thì có thể gọi đc các standard function trên, ở đây ta gọi let.
- Trong block của hàm let để ý thấy có 2 biến `this` và `it`. 2 biến này sẽ là điểm mấu chốt để thể hiện sự khác nhau giữa các standard functions, sẽ được nói ở đoạn sau.
- Ở cuối hàm `let` có 1 đoạn `String` là `100`, đây chính là giá trị trả về của block. Cơ mà k có `return` ở đây ?:D?   

Anh em bình tĩnh, nếu viết rõ hơn thì block trong hàm `let` sẽ có dạng thế này:

```kotlin
class MainClass {
    val greeting = "Hello, World!"
 
    fun test() {
        val result = greeting.let { it ->
            println(this)
            println(it)
            "100"
        }
 
        println(result)
    }
}
```

block trong `let` là 1 single expression nên nó sẽ tự động nhận giá trị cuối cùng của block là return value. Và trong hàm `let`, `it` được truyền vào, đại diện cho `greeting`. Điều đó giải thích lí do khi chạy đoạn code này sẽ in ra :

```
io.trieulh.basickotlin.MainClass@511d50c0 <-- biến `this`
Hello, World! <-- biến `it`
100 <-- `result` được in ra
```

 Hợp lý chưa ạ? Ok, vậy tổng kết lại cách sử dụng của `let` cũng như các function còn lại, ta có thể khái quát 1 hàm căn bản như sau, với `xxx` là 1 standard function:  

```kotlin
class MainClass {
    val greeting = "Hello, World!"
 
    fun test() {
        val result = greeting.xxx { it ->
            println(this)
            println(it)
            "100"
        }
 
        println(result)
    }
}
```

Thì value được in ra ở 3 hàm `println` tương ứng với các function sẽ là: 

|Function	|println(this)	|println(it)	|println(result)|
|let	|this@MainClass	|“Hello, World!”	|“100”|
|run	|“Hello, World!”	|[N/A]	|“100”|
|with*	|“Hello, World!”	|[N/A]	|“100”|
|apply	|“Hello, World!”	|[N/A]	|“Hello, World!”|
|also	|this@MainClass	|“Hello, World!”	|“Hello, World!”|

[N/A] có nghĩa trong block của function không tồn tại biến `it` 

* Hàm with không được gọi theo template trên mà sẽ dùng cách gọi cũ của kotlin như sau:

```kotlin
class MainClass {
    val greeting = "Hello, World!"
 
    fun test() {
        val result = with(greeting) {
            println(this)
            println(it)
            "100"
        }
        println(result)
    }
}
```

# Điểm khác biệt

## Sự khác biệt trong việc gọi hàm theo phương thức cũ và theo dạng extension
Hãy so sánh sự khác biệt trong giá trị giữa `run` với `with` … Giống hệt nhau. 

```kotlin
with(greeting) {
            println(length)
            println(toUpperCase())
        }
 
// Tương tự với
 
greeting.run {
            println(length)
            println(toUpperCase())
        }

```
Tuy nhiên đối với trường hợp `greeting` có thể `null`. Giờ hãy sửa lại đoạn code ở trên để tránh `NullPointerException`:

```kotlin
//Eww...
with(greeting) {
            println(this?.length)
            println(this?.toUpperCase())
        }
 
// Yay
greeting?.run {
            println(length)
            println(toUpperCase())
        }
```

Trong trường hợp này, sử dụng `run` rõ ràng giúp cho code trở nên sạch sẽ hơn rất nhiều. DONE!

## Sự khác biệt giữa this và it
Giờ ta sẽ so sánh giữa `run` và `let`  

```kotlin
greeting?.run {
         println("The length of this String is $length"
}
 
// Tương tự với
 
greeting?.let {
         println("The length of this String is ${it.length}"
}
```

Sự khác biệt ở đây không đáng kể. Tuy nhiên có vài lợi ích của việc sử dụng `this` và `it` bạn có thể cân nhắc. 

`this` có thể được lược bỏ, lúc này các `variable` và `method` của object có thể được gọi trực tiếp. Thay vì `${this.length}` ta có thể rút gọn thành `$length`

`it` lại vượt trội hơn `this` trong việc tường minh code. Trong ví dụ dưới đây:

```kotlin
class MainClass {
    val greeting = "Hello, World!"
 
    fun test() {
        greeting?.run {
          println(this@String)
        }
    }
}
```

`this` cần được đánh dấu thành `this@String` (Điều này không bắt buộc), để không bị nhầm lẫn với `this@MainClass`. Việc này có thể giải quyết bằng `it`:

```kotlin
class MainClass {
    val greeting = "Hello, World!"
 
    fun test() {
        greeting?.let { string ->
          println(string)
        }
    }
}
```

Giúp cho việc debug code trở nên dễ dàng hơn rất nhiều.

## Sự khác biệt trong việc sử dụng returned value

Hãy cùng xét đến vấn đề cuối cùng, khi so sánh giữa let và also  

```kotlin
greeting?.let {
      println("The length of this String is ${it.length}")
}
 
// Tương tự với
 
greeting?.also {
      println("The length of this String is ${it.length}")
}
```

Cả 2 đều hỗ trợ trả về giá trị, nhưng cách trả về tương đối khác nhau. `let` dựa vào giá trị cuối cùng được trả về trong block, trong khi `also` luôn trả về caller object. Để dễ hiểu hơn, hãy thử sử dụng cả 2 trong việc gọi hàm theo chuỗi:  

```kotlin
// `let`
greeting?.let {
    println("The original String is $it") // "Hello, World!"
    it.reversed()
}.let {
    println("The reverse String is $it") // "!dlroW ,olleH"
    it.length
}.let {
    println("The length of the String is $it") // 13
}
 
// `also`
greeting?.also {
    println("The original String is $it") // "Hello, World!"
}.also {
    println("The reverse String is ${it.reversed()}") // "dlroW ,olleH"
}.also {
    println("The length of the String is ${it.length}") // 13
}
```

Easy?

# Kết

Đọc đi đọc lại vẫn chưa hiểu cách dùng? Hãy in cái này ra mà dán lên bàn làm việc.

![standard-function](~/assets/images/standardfunction.png)

Cheer! Thank ae đã chịu khó đọc đến đây. Happy coding!