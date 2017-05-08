---
title: "[Design Pattern] Singleton và cách sử dụng trong Java"
categories:
  - Design Pattern
  - Java
  - Vietnamese
tags:
  - design-pattern
  - beginner
  - java
---

Mở đầu series về Design Pattern, chúng ta sẽ bắt đầu đi từ những pattern đơn giản nhất, phổ biến nhất mà tất cả lập trình viên Java cần phải nắm **cực kì rõ**. Và Pattern đầu tiên trong số đó là **Singleton** :smile:.
## Vậy Singleton Pattern là gì?
![](http://cdn.journaldev.com/wp-content/uploads/2013/03/java-singleton-pattern.jpg)

Đối với các môn sinh tìm hiểu về Design Pattern, **Singleton** có lẽ là pattern thuộc dạng dễ hiểu nhất, dễ thực hiện nhất. Tuy nhiên để áp dụng đúng cách, đúng trường hợp thì lại không đơn giản. Ờm nói cho nguy hiểm vậy thôi chứ cũng k đến nỗi lằng nhằng lắm, đại khái là có 1 số trường hợp chúng ta cần phải chú ý để có thể tối ưu được việc sử dụng **Singleton** trong project. :smile:

### Định nghĩa
Nếu bạn đã hiểu khái niệm về [Instance](https://en.wikipedia.org/wiki/Instance_(computer_science)) thì việc hiểu công dụng của **Singleton** cũng không gặp nhiều khó khăn.
>Đại khái là, **Singleton** đảm bảo sẽ chỉ có duy nhất 1 **Instance** của _class_ được khởi tạo và sử dụng trên máy ảo JVM.

Sound good? Công dụng nó chỉ có vậy thoai :smile:

Tiếp theo, các quy định cơ bản chúng ta phải tuân thủ khi sử dụng **Singleton**:

- `Constructor` của _class_ phải luôn là `private` `method`, nhằm hạn chế việc khởi tạo ở các _class_ khác.
- Các _variable_ được khởi tạo và sử dụng cùng `constructor` phải luôn là `private` `static` `variable`.
- Luôn phải có 1 `public` `method` phục vụ cho việc trả lại _Instance_ của class, đây sẽ là _method_ duy nhất mà các _class_ khác có thể sử dụng để khởi tạo _class_ sử dụng **Singleton**.

Hừm, vẫn còn thấy khó hiểu? Đang ngồi lẩm bẩm sao thằng tác giả nói gì mà lằng nhằng thế? Okidoki, vậy bây giờ ta sẽ đi vào từng ví dụ cụ thể, để hiểu cách sử dụng của **Singleton** trong từng trường hợp nhé, để đỡ phải ngồi chửi thầm...

### Cách sử dụng
Nói thẳng ra là mình cũng chẳng sử dụng hết các cách này đâu, cũng tham khảo 1 số nguồn và tổng hợp lại cho mọi người thôi. :smile:

**Singleton** mình liệt kê ở đây sẽ có 4 cách cơ bản và phổ biến nhất (Sử dụng tên tiếng Anh cho mọi người dễ search):

1. [Eager initialization](#1-eager-initialization)
2. [Static block initialization](#2-static-block-initialization)
3. [Lazy Initialization](#3-lazy-initialization)
4. [Thread Safe Singleton](#4-thread-safe-singleton)

Ngoài ra còn mấy cách tricky kiểu như sử dụng **Enum** thì cho qua đi, khó nhớ lắm...

### 1. Eager initialization

Cách đầu tiên cũng là cách sơ đẳng, đơn giản cmn nhất, đúng kiểu đọc định nghĩa phát có thể viết được luôn rồi.

```
public class EagerInitializedSingleton {

    private static final EagerInitializedSingleton instance = new EagerInitializedSingleton();

    //private constructor to avoid client applications to use constructor
    private EagerInitializedSingleton(){}

    public static EagerInitializedSingleton getInstance(){
        return instance;
    }
}
```

Quá đơn giản phải không ạ? Áp dụng đúng theo các qui định ở trên là xong, có biến `private` nè, có `private` `constructor` nè, có `private` `method` để lấy **Instance** nè. Ờ nhưng mà cách này thì độ thực tế không cao lắm, à mà có thể cũng cao, vì nhiều ông dev lười để ý, cứ phệt toẹt như sách vào thế này... Lí do vì sao mà mềnh nói lợi bất cập hại, đơn giản là vì... vì... **Java** và **JVM** nó tồn tại 1 cái gọi là [Class loading](http://www.onjava.com/pub/a/onjava/2005/01/26/classloading.html)...

Tức là, Ứng dụng trước khi hoạt động được, **JVM** sẽ tải hết thông tin về **Class** vào bộ nhớ (bao gồm cả _name, method, variable_ đi kèm blo bla...).

Nếu sử dụng cách ở trên, **Instance** của **Class** sẽ được khởi tạo sẵn và nạp vào bộ nhớ cùng vs **Class**. Quá nguy hiểm nếu **Instance** có dung lượng quá lớn, nếu trong quá trình sử dụng app mà ta không sử dụng đến **Instance** này thì đấy quả là 1 sự lãng phí k cần thiết...

Chưa hết, vẫn còn 1 điểm hại nữa... Điều gì xảy ra nếu việc khởi tạo **Instance** gặp vấn đề, hay còn gọi là **Exception**? Đúng rồi, bật app lên cái chết nhe răng chứ còn gì nữa... :sob:

Thế này cách này chỉ để cho biết thôi, chứ dùng thì vứt, vứt nhé...

### 2. Static block initialization
Cách này thông minh hơn cách 1 một tí, đó là có thể xử lý **Exception**! Wohooo!!! :ok_hand:


```
public class StaticBlockSingleton {

    private static StaticBlockSingleton instance;

    private StaticBlockSingleton(){}

    //static block initialization for exception handling
    static{
        try{
            instance = new StaticBlockSingleton();
        }catch(Exception e){
            throw new RuntimeException("Exception occured in creating singleton instance");
        }
    }

    public static StaticBlockSingleton getInstance(){
        return instance;
    }
}
```

1 cách sử dụng khá là lạ, `static` block trong class, Mình cũng chẳng mấy khi sử dụng _syntax_ này trong Java...

Tuy nhiên nó vẫn chưa giải quyết được vấn đề về tối ưu hoá sử dụng bộ nhớ... Next thôi next thôi.

### 3. Lazy Initialization
Khái niệm **Lazy...** được sử dụng rất nhiều trong trong lập trình, hiểu nôm na là:

> Tôi sẽ tạo 1 cái **Instance** rỗng xong để đấy làm đại diện thôi, bao giờ dùng thì mới cấp bộ nhớ.

Đấy, lười chưa, tạo thì cấp cho luôn đi, lại còn phải khi nào dùng ms cấp cơ...

```
public class LazyInitializedSingleton {

    private static LazyInitializedSingleton instance;

    private LazyInitializedSingleton(){}

    public static LazyInitializedSingleton getInstance(){
        if(instance == null){
            instance = new LazyInitializedSingleton();
        }
        return instance;
    }
}
```
Cách này nghe có vẻ hợp lý rồi đấy, k còn phải lăn tăn việc **Exception** hay tối ưu bộ nhớ nữa. Tuy nhiên thịt chó hay ăn với lá mơ, và đời cũng không như là mơ. Phương pháp này trong 1 số trường hợp sẽ là điểm yếu chết người. Đó là khi được sử dụng trong **Distributed system**, hay gọn hơn là trong **Multi-thread system**.

Lí do là sao, là đây:
> Điều gì xảy ra khi 2 _Thread_ riêng biệt cùng gọi method `getInstance` và check được rằng `instance == null` trả về `true`. Lúc đấy `constructor` sẽ bị gọi 2 lần (hoặc tệ hơn là n lần nếu có n _Thread_). Và thế là quy tắc về **Instance** sẽ bị phá vỡ.

Vậy chúng ta phải làm gì? Đến với cách 4 chứ làm gì...

### 4. Thread Safe Singleton
Phương pháp này sẽ giải quyết bài toán bằng cách:
> Cho lần lượt từng _Thread_ truy cập vào method `getInstance()`

```
public class ThreadSafeSingleton {

    private static ThreadSafeSingleton instance;

    private ThreadSafeSingleton(){}

    public static synchronized ThreadSafeSingleton getInstance(){
        if(instance == null){
            instance = new ThreadSafeSingleton();
        }
        return instance;
    }

}
```

thêm `synchronized` vào, thế là ta đã giải quyết xong. Việc cho `synchronized` sẽ đảm bảo trong 1 thời điểm sẽ chỉ có 1 _Thread_ được phép thực thi method này.
...
...
...
Ahihi, bạn tin người vcd... Nghĩ gì mà lại đi dùng cách đấy... Chẳng lẽ mỗi gần gọi **Instance** chúng ta đều phải đưa về chế độ _synchronization_? Performance có vẻ không ổn lắm...

Dùng cách này này:

```
public static ThreadSafeSingleton getInstanceUsingDoubleLocking(){
    if(instance == null){
        synchronized (ThreadSafeSingleton.class) {
            if(instance == null){
                instance = new ThreadSafeSingleton();
            }
        }
    }
    return instance;
}
```
Phương pháp này được gọi là **double checked locking** (**Google** bảo thế...). Nguyên tắc rất cơ bản:
> Với trường hợp **Instance** chưa được tạo, tất cả _Thread_ sẽ phải chờ 1 _Thread_ đến trước nhất khởi tạo xong **Instance**, sau đó sử dụng như bình thường.
> Với trường hợp **Instance** đã được tạo, không cần phải đưa về **synchronization**.

Giờ thì có lẽ ok rồi đấy.

![](https://cdn.meme.am/cache/instances/folder847/250x250/41665847/too-damn-high-the-number-of-singletons-in-our-codebase-is-too-damn-high.jpg)

---

Kết thúc bài viết đầu tiên. Nếu bạn cảm thấy mình viết vẫn còn khó hiểu, có thể liên hệ trực tiếp qua Facebook hoặc Email của mềnh (Mọi thức đã được đính kèm ở đâu đấy trong web :see_no_evil:)

Happy Coding!

_Reference: Internet_
