---
title: "TIL#5: Làm việc với Path trong Custom View"
categories:
  - TodayILearned
tags:
  - kotlin
  - ui
  - animation
---

#Lưu ý trong bài viết mình xin phép chèn 1 số từ tiếng Anh vào, vì nếu dịch sang tiếng Việt nghe sẽ rất … ngớ ngẩn.

Hầy zô! Bài hôm nay chúng ta sẽ đến 1 khái niệm khác vừa lạ vừa quen, đó chính là **Custom View**.

## 1. Mục tiêu bài viết

**Custom View** là 1 khái niệm tương đối quen thuộc đối với lập trình viên di động. Do bản chất các bộ Widget có sẵn của platform không thể đáp ứng được yêu cầu thiết kế, cũng như có 1 số thiết kế tương đối phức tạp, mà đòi hỏi lập trình viên phải tự tay "vẽ" theo ý mình.

Trong bài viết này, chúng ta sẽ thử sử dụng **Path**, **Canvas** để tạo 1 **Custom View** xem thế nào nhé.

### Kịch bản

**Custom View** trong bài viết này sẽ gồm những thành phần sau:

- Vẽ các hình polygon đồng tâm với số cạnh từ 3 -> 8
- Bo tròn các đỉnh polygon
- Hiển thị 1 phần của Polygon
- Cho 1 image object chạy dọc theo các cạnh của polygon
- Tạo Animation cho polygon

Còn đây là kết quả cuối cùng cần đạt được:

![path-final]({{site.url}}/assets/images/path-final.gif)

## 2. Triển khai

### a. Khởi tạo Custom View

Custom View có cách sử dụng giống như cách chúng ta nhúng các TextView, ImageView widget vào trong XML vậy. Bản chất nó như nhau. Ở đây mình đặt tên cho **Custom View** này là **PolygonPathView**:

```kotlin
<androidx.appcompat.widget.LinearLayoutCompat xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_marginHorizontal="16dp"
    android:gravity="center"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <io.trieulh.pathanimationexample.view.PolygonPathView
        android:id="@+id/polygonView"
        android:layout_width="240dp"
        android:layout_height="240dp" />

    ...
</androidx.appcompat.widget.LinearLayoutCompat>
```

Trong source code, chúng ta sẽ extend view này từ class **View**:

```kotlin
class PolygonPathView(context: Context, attrs: AttributeSet) : View(context, attrs) {
}
```

Ok vậy là xong phần khởi tạo. Tiếp theo chúng ta sẽ tìm hiểu xem làm thế nào để vẽ được các Polygon đồng tâm.

### b. Sử dụng Path để vẽ Polygon

Để định nghĩa 1 Polygon, chúng ta cần 3 đặc điểm cơ bản : số cạnh, bán kính và màu sắc. Mình sẽ wrap những đặc điểm này vào trong 1 data class **Polygon** cho tiện sử dụng:

```kotlin
data class Polygon(val side: Int, val radius: Float, @ColorInt val color: Int)
```

Sau đó ta sẽ định nghĩa các polygon cần phải vẽ vào trong View. Theo mục tiêu bài toán, ta sẽ có 5 polygon có số cạnh từ 3 đến 8:

```kotlin
class PolygonPathView(context: Context, attrs: AttributeSet) : View(context, attrs) {
    private val polygons = listOf(
        Polygon(3, 20f, Color.BLACK),
        Polygon(4, 40f, Color.BLUE),
        Polygon(5, 60f, Color.GREEN),
        Polygon(6, 80f, Color.RED),
        Polygon(7, 100f, Color.MAGENTA),
        Polygon(8, 120f, Color.CYAN)
    )
}
```

Và tiến hành override `onDraw()` để vẽ:

```kotlin
class PolygonPathView(context: Context, attrs: AttributeSet) : View(context, attrs) {

    private val paint = Paint(Paint.ANTI_ALIAS_FLAG)
    ...
    override fun onDraw(canvas: Canvas) {
        paint.style = Paint.Style.STROKE

        for (i in 0..(maxNumOfSides - 3)) {
            val polygon = polygons[i]
            paint.color = polygon.color
            val path = createPathAndEffect(polygon, paint)
            canvas.drawPath(path, paint)

        }
        super.onDraw(canvas)
    }
}
```

Trước khi đi chi tiết vào cách vẽ, chúng ra sẽ điểm qua 1 chút hàm `onDraw()` ở trên:

- object `paint` được khởi tạo phụ trách việc định nghĩa các đặc điểm về dường nét, màu sắc.
- Mỗi polygon sẽ có 1 `path` riêng được tạo từ `createPathAndEffect()`
- Path sau đó sẽ được vẽ lên canvas qua hàm `drawPath()`

Ta có thể thấy `path` sẽ quyết định xem các nét vẽ được vẽ lên canvas như thế nào. Tiếp theo chúng ta sẽ đi chi tiết vào hàm `createPathAndEffect()`.

Tuy nhiên ta sẽ cùng đi qua 1 chút khái niệm cơ bản về toán học, để hiểu cách biểu diễn polygon trên trục toạ dộ. Đầu tiên là cách xác định toạ độ của 1 điểm trên trục toạ độ:

![coordinate]({{site.url}}/assets/images/coordinate.png)

1 điểm có thể được xác định dựa vào toạ độ _(x,y)_, hoặc theo góc lượng giác _(r,angle)_ với _r_ là bán kính đường tròn đi qua điểm đó có tâm là _(0,0)_ và _angle_ là góc lượng giác hình thành bới trục _x_ với trục đi qua 2 điểm _(0,0)_ và điểm đó.

Chúng ta có thể convert qua lại giữa 2 cách biểu diễn này qua công thức:

```kotlin
val x = r * Math.cos(angle);
val y = r * Math.sin(angle);
```

Ví dụ ta có 1 polygon có 3 cạnh (hay 3 đỉnh), thì ta có thể xác định toạ độ _(x,y)_ của từng đỉnh dựa vào _(r,angle)_. Giả sử đỉnh đầu tiên nằm trên trục toạ độ _Ox_, tức _(x0,y0)=(r,0)_ hay _(r,angle0)=(r,0)_, Ta có thể xác định đỉnh tiếp theo sẽ có _angle1= 360/3=120_. Suy ra toạ độ của đỉnh đó là _(r,angle1)=(r,120)_. Cuối cùng là dùng công thức trên tính được toạ độ _(x,y)_.

Ok vậy là ta có thể xác định được toạ độ các đỉnh của polygon, công việc còn lại là nối giữa các đỉnh để tạo thành 1 hình polygon khép kín:

```kotlin
    private fun createPathAndEffect(
        polygon: Polygon,
        paint: Paint
    ): Path {
        val path = Path()
        val angle = 2.0 * Math.PI / polygon.side
        val radius = polygon.radius
        val cx = pivotX
        val cy = pivotY

        path.moveTo(
            cx + (radius * cos(0.0)).toFloat(),
            cy + (radius * sin(0.0)).toFloat()
        )
        for (i in 1 until polygon.side) {
            path.lineTo(
                cx + (radius * cos(angle * i)).toFloat(),
                cy + (radius * sin(angle * i)).toFloat()
            )
        }
        path.close()
        return path
    }
```

Phân tích:

- `cx, cy` tương ứng với điểm _(0,0)_
- Khi bắt đầu, `path` sẽ xuất phát từ đỉnh đầu tiên tương ứng _angle=0_ bằng hàm `moveTo()`.
- Đi qua mỗi đỉnh, `path` sẽ nối các đỉnh với nhau bằng `lineTo()`.
- Kết thúc bằng `close()` để tạo thành 1 `path` khép kín.

Và kết quả là:

![coordinate]({{site.url}}/assets/images/path-polygon.png)

Chúng ta có thể thấy được tất cả điểm xuát phát của các polygon đều nằm trên cùng 1 đường thẳng. Vậy là đã xong bước đầu.

Tiếp theo, chúng ta sẽ thử làm quen với **PathEffect** để học cách bo tròn và hiển thị 1 phần của polygon.

### c. CornerPathEffect và DashPathEffect

Ở đây ta sẽ sử dụng 2 class của PathEffect là **CornerPathEffect** và **DashPathEffect**.
Đầu tiên là \*_CornerPathEffect_:

```kotlin
class PolygonPathView(context: Context, attrs: AttributeSet) : View(context, attrs) {
    ...
    private var cornerPathEffect = CornerPathEffect(cornerRadius)
    ...
}
```

`cornerRadius` có thể set là _24f_ chẳng hạn tuỳ vào bạn. Sau đó là apply cái `cornerPathEffect` này vào trong `paint`, vì như mình đã nói ở trên, `paint` phụ trách định nghĩa các đặc điểm của `path` mà:

```kotlin
    private fun createPathAndEffect(
        polygon: Polygon,
        paint: Paint
    ): Path {
        ...
        path.close()

        paint.pathEffect = cornerPathEffect
        return path
    }
```

![coordinate]({{site.url}}/assets/images/path-corner.png)

Trước khi apply `DashPathEffect`, nói qua chút về phương pháp chúng ta sẽ xử lý ở đây. Về bản chất, nguyên bản `DashPathEffect` là để tạo hiệu hứng nét đứt cho path chứ không phải để hiển thị thành phần, nhưng chúng ta có thể lợi dụng đặc điểm của nó được.

Giả sử ta chỉ muốn hiển thị _50%_ polygon, điều đó tương đương với 1 nét đứt có độ dài _50%_ chu vi polygon, và khoảng cách giữa các nét đứt là _100 - 50 = 50%_ còn lại của chu vi.

```kotlin
    private fun createPathAndEffect(
        polygon: Polygon,
        paint: Paint
    ): Path {
        ...
        path.close()

        val pathMeasure = PathMeasure(path, false)
        val length = pathMeasure.length
        intervals =
            floatArrayOf(length * pathPercentage / 100f, length * (100f - pathPercentage) / 100f)
        dashPathEffect = DashPathEffect(intervals, 1f)
        paint.pathEffect = ComposePathEffect(dashPathEffect, cornerPathEffect)
        return path
    }
```

Phân tích:

- Chúng ta sử dụng `PathMeasure` để tính dược độ dài của `path` (hay chu vi của polygon), từ đó có thể xác định đc độ dài nét đứt và khoảng cách của nó.
- 2 giá trị trên được lưu vào 1 Float array là `intervals`
- `pathPercentage` là tỉ lệ ta muốn hiển thị
- Ở đây ta sử dụng `ComposePathEffect` để kết hợp đồng thời 2 effect với nhau. Trong ví dụ trên thì `dashPathEffect` sẽ được apply trước, sau đó đến `cornerPathEffect`.

Và kết quả:

![coordinate]({{site.url}}/assets/images/path-effect.png)

Để ý thấy có vẻ như tỉ lệ hiển thị của mỗi polygon không giống nhau, điều này xảy ra vì `length` tính ở trên là `length` của polygon gốc chứ không phải `length` sau khi đã bo tròn. Nếu bạn muốn polygon có tỉ lệ đồng nhất thì có thể tìm hiểu hàm `PathMeasure.getSegment()`.

### d. Image object trên polygon

Để vẽ được 1 Image object trên `path` của polygon, ta cần làm 1 số công việc:
- Khởi tạo Image object, ở đây mình sử dụng 1 con cánh cam :D :

```kotlin
class PolygonPathView(context: Context, attrs: AttributeSet) : View(context, attrs) {
    ...
    private lateinit var bitmap: Bitmap

    private var pos = FloatArray(size = 2)
    private var tan = FloatArray(size = 2)
    private var bm_offsetX: Float = 0f
    private var bm_offsetY: Float = 0f
    private var imageMatrix: Matrix = Matrix()

    init {
        loadBitmap()
    }

    private fun loadBitmap() {
        bitmap = Bitmap.createScaledBitmap(
            BitmapFactory.decodeResource(resources, R.drawable.ladybug),
            15,
            15,
            false
        )

        bm_offsetX = (bitmap.width / 2).toFloat()
        bm_offsetY = (bitmap.height / 2).toFloat()
    }
    ...

}
```
- Đặt Image object lên điểm cuối cùng của `path`:

```kotlin
    override fun onDraw(canvas: Canvas) {
        ...
        for (i in 0..(maxNumOfSides - 3)) {
            ...
            calculateBitmap(path, canvas)
        }
        ...
    }

    private fun calculateBitmap(path: Path, canvas: Canvas) {
        val pathMeasure = PathMeasure(path, false)
        val length = pathMeasure.length
        pathMeasure.getPosTan(length * pathPercentage/100f, pos, tan)

        imageMatrix.reset()
        val degrees = (Math.atan2(tan[1].toDouble(), tan[0].toDouble()) * 180.0 / Math.PI).toFloat()
        imageMatrix.postRotate(degrees, bm_offsetX, bm_offsetY)
        imageMatrix.postTranslate(pos[0] - bm_offsetX, pos[1] - bm_offsetY)

        canvas.drawBitmap(bitmap, imageMatrix, null)
    }
```

Phân tích:
- Chúng ta sẽ sử dụng chính `path` object được dùng để vẽ polygon
- Vẫn như cách cũ, ta sẽ tính `length` tổng thể của `path` và tính phần `path` được hiển thị
- Hàm `getPosTan()` được gọi để lấy 2 tập giá trị là vị trí của điểm cuối cùng trong `path` và đường tiếp tuyến tại đỉnh đó. 2 tập giá trị này được lưu lần lượt vào `pos` và `tan`
- `imageMatrix` ở đây được sử dụng để xoay apply các tính chất về vị trí cũng như góc quay đê làm sao Image object đặt vào đúng điểm cuối của `path` và hướng của Image object trùng với hướng đi của `path` 
- Chúng ta gọi hàm `drawBitmap` để vẽ Image lên `canvas`

Kết quả cuối cùng:

![coordinate]({{site.url}}/assets/images/path-image.png)

e. Tạo Animation cho Path

Phần này tương đối đơn giản rồi. Để ý ở phần trên mình sử dụng `pathPercentage` để biểu diễn thành phần hiển thị. Nhiệm vụ của ta là tạo 1 **Animator** để thay đổi giá trị này theo thời gian. Ta sẽ sửa lại hàm setter của `pathPercentage` như sau:

```kotlin
    var pathPercentage = 50f
        set(value) {
            if (value < 0f)
                field = 0f
            else if (value > 100f) {
                field = 100f
            } else
                field = value
            invalidate()
        }
```

Giá trị mặc định là _50f_ tương đương _50%_, và mỗi lần hàm `set()` của `pathPercentage` được gọi, ta sẽ gọi thêm `invalidate()` để update lại việc hiển thị của **PolygonPathView**.

Theo kịch bản, khi ấn nút *Animate* trên màn hình sẽ trigger *Animator* của **PolygonPathView**. Ta sẽ làm như sau:

```kotlin
    btnAnimate.setOnClickListener {
        ValueAnimator.ofFloat(0f, 100f).apply {
            duration = 2000L
            interpolator = LinearInterpolator()
            repeatCount = INFINITE
            repeatMode = RESTART
            addUpdateListener {
                polygonView.pathPercentage = it.animatedValue as Float
                progressSeekBar.progress = (it.animatedValue as Float).toInt()
            }
        }.start()
    }
```

Kết quả cuối cùng tương đương với yêu cầu ta đã nêu ở đầu bài.

![path-final]({{site.url}}/assets/images/path-final.gif)

## Kết luận 

`Path` và `Canvas` là các tính năng vô cùng thú vị để ta tuỳ biến **Custom View** của mình. Trong 1 số trường hợp ví dụ như tạo shadow, customize background, v..v.. ta có thể dùng `Path`, `Canvas`, giúp làm giảm kích cỡ ứng dụng cũng như tăng khả năng tuỳ biến, tại sử dụng thay vì phải nhúng thêm các **Asset Image** cồng kềnh. Ngoài hàm `moveTo()` hay `lineTo()` cơ bản, `Path` còn cung cấp rất nhiều các cách thức vẽ khác rất mạnh mẽ, chỉ cần 1 chút toán cũng như chút hiểu biết về hình học thì mình khuyến khích ae nên tận dụng class này thường xuyên hơn. Xin hết.

Happy Coding!

_Bài viết có tham khảo phương pháp từ [Playing with Paths](https://medium.com/androiddevelopers/playing-with-paths-3fbc679a6f77) của **Nick Butcher**. Rất đáng đọc, anh em có thể dành thời gian tìm hiểu._