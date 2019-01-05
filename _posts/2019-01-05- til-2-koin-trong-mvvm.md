---
title: "TIL#2: Koin và cách sử dụng trong MVVM"
categories:
  - TodayILearned
tags:
  - til
  - kotlin
  - koin
  - mvvm
---

#Lưu ý trong bài viết mình xin phép chèn 1 số từ tiếng Anh vào, vì nếu dịch sang tiếng Việt nghe sẽ rất … ngớ ngẩn. 

Koin là gì? và tại sao lại dùng Koin? Trong bài viết này, ngoài việc định nghĩa cũng như hướng dẫn cách sử dụng Koin, mình sẽ đi qua cũng như trộn lẫn với khái niệm về MVVM, mộn architecture không cũ cũng không mới đối với lập trình viên Mobile nói chung cũng như Android nói riêng.

# Mục tiêu bài viết
- Khái niệm về Koin
- Lợi thế của Koin so với các lib tương đương
- Cách cài đặt và sử dụng Koin trong việc triển khai MVVM
- Sử dụng Koin trong việc viết Unit Test


##  Khái niệm về Koin – Dependency Injection Library sử dụng Kotlin
Okay okay, Dependency Injection(DI), Nếu như bạn là 1 lập trình viên Mobile, thì những năm gần đây 2 từ DI được nói đến và sử dụng rất nhiều trong nhiều dự án lớn (và nhỏ, maybe…). Vậy trước khi tìm hiểu về Koin, hãy đi sơ qua khái niệm về DI.

> Dependency Injection là kĩ thuật (hay phương pháp) sử dụng 1 đối tượng(object) hay 1 hàm tĩnh (static method) để cung cấp dependency cho 1 đối tượng khác.

Đọc xong khái niệm, lại lật lại dăm ba cái nguyên tắc S.O.L.I.D nào, chữ D là gì? D – Dependency Inversion, nguyên tắc này bao gồm 2 ý chính:

> Ý 1: Các module cấp cao không nên phụ thuộc vào các module cấp thấp, cả 2 nên phụ thuộc vào abstraction
> 
> Ý 2: Các module giao tiếp với nhau thông qua interface.


*Giải thích nhanh: Module cấp cao là thằng gọi Module cấp thấp, và gọi qua interface, không thông qua implementation của thằng module cấp thấp. Điều này đảm bảo việc thay đổi implementation của Module cấp thấp sẽ k ảnh hưởng đến thằng cấp cao. Ví dụ thì search google nha.

1 lợi ích nữa của việc giao tiếp thông qua interface, đó là thằng module cấp cao sẽ không phải (chính xác hơn là không thể) triển khai, khởi tạo module cấp thấp, mà chỉ có nhiệm vụ duy nhất là sử dụng. Vậy ai sẽ là người khởi tạo các module cấp thấp cho bọn này? Đấy là lí do mà Koin, hay những dependency injector/ service locator ra đời. (Shiet, 2 cái này google tiếp đi nha)

Koin – đơn giản là 1 framework đóng vai trò như 1 Service locator, phụ trách phân phát dependency cho bất cứ module nào cần.

Song hành với Koin, còn có 1 framework khác là Dagger/Dagger 2, được phát triển rất mạnh cho nền tảng Android và được các nha sĩ, à nhầm developer tin dùng. Vậy Koin có gì vượt trội mà người ta lại phải tạo ra nó trong khi Dagger đang làm vương làm tướng trong rất nhiều dự án Android hiện đại. Ok, let’s check !

## Lợi thế của Koin so với Dagger
Hãy so sánh 2 framework này 1 cách công bằng nào.

### Dagger/Dagger 2:
- Dagger sử dụng annotation,
- Và chính vì sử dụng annotation, code sẽ được sinh ra trong quá trình compile, chính vì vậy ta có thể check được lỗi trong quá trình build framework


### Koin:
- Không sử dụng annotation, lợi ích rõ nhất là build phase sẽ rất là nhanh và không sinh code trong quá trình build sẽ làm giảm dụng lượng của ứng dụng.
- Triển khai đơn giản (Đơn giản cực kì)
- Dễ học (Dễ học cực kì)


Đấy là điểm mạnh mỗi thằng, còn muốn đi sâu tìm hiểu về Dagger, tác giả xin phép được từ chối vì đơn giản là mình không thích dùng Dagger cho lắm =))

## Cách cài đặt và sử dụng Koin trong việc triển khai MVVM
Giờ chúng ta sẽ vào phần chính, trong phần này, mình sẽ build thử 1 app có 1 tác vụ duy nhất là truy vấn 1 list thông tin về nhân viên trên server, tải về và đẩy lên UI.

Đầu tiên hay vào phần cơ bản nhất.

### Cài đặt

Ta sẽ khởi tạo 1 project đơn giản sử dụng ngôn ngữ Kotlin và cài đặt thư viện Koin vào trong đó. Cái này các bạn tự làm, miễn là đảm bảo có đủ các phần sau:

```gradle
// build.gradle của app module
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
 
dependencies {
...
  implementation "org.jetbrains.kotlin:kotlin-stdlibjdk7:$kotlin_version"
 
  //lifecycle architecture
  implementation "android.arch.lifecycle:extensions:$lifecycle_version"
  testImplementation "android.arch.core:core-testing:$lifecycle_version"
 
  //Koin
  implementation "org.koin:koin-android:$koin_version"
  //Koin for Android Architecture ViewModel
  implementation "org.koin:koin-android-viewmodel:$koin_version"
  //Koin for JUnit Test
  testImplementation "org.koin:koin-test:$koin_version"
...
}
```

Ở trên là các `dependency` phục vụ cho 2 mục đích: sử dụng MVVM (khung sườn cho toàn bộ project) và sử dụng Koin cho việc triển khai và test. Ngoài ra trong phần demo này, mình sẽ sử dụng thêm 1 số lib phục vụ cho feature. Ae tự tìm hiểu chức năng nhé.

```gradle
dependencies {
    ....
 
    //Rx
    implementation "io.reactivex.rxjava2:rxandroid:$rxandroid_version"
    implementation "io.reactivex.rxjava2:rxjava:$rxjava_version"
 
    //Logger
    implementation "com.jakewharton.timber:timber:$timber_version"
 
    //Retrofit
    implementation "com.squareup.okhttp3:okhttp:$okhttp_version"
    implementation "com.squareup.okhttp3:logging-interceptor:$okhttp_version"
    implementation ("com.squareup.retrofit2:retrofit:$retrofit_version"){
        // exclude Retrofit’s OkHttp peer-dependency module and define your own module import
        exclude module: 'okhttp'
    }
    implementation "com.squareup.retrofit2:converter-gson:$retrofit_version"
    implementation "com.squareup.retrofit2:adapter-rxjava2:$retrofit_version"
    implementation "com.squareup.retrofit2:converter-moshi:$retrofit_version"
    implementation "com.squareup.moshi:moshi-kotlin:$moshi_version"
}
```

### Triển khai MVVM

MVVM về cơ bản sẽ gồm 3 thành phần chính là `Model` – `View` – `ViewModel` tương tác với nhau như mô tả ở hình dưới (Chi tiết lại mời các bạn google)

![mvvm-architecture]({{site.url}}/assets/images/mvvm-architecture.png)

*Source: Internet*

Trong demo này, hãy tạo UI trước. UI về cơ bản chỉ gồm 1 Container Activity và 1 Feature Fragment:

```kotlin
class MainActivity : AppCompatActivity() {
 
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        if (savedInstanceState == null) {
            supportFragmentManager.beginTransaction()
                .replace(R.id.container, EmployeesListFragment.newInstance())
                .commitNow()
        }
    }
}
```

```kotlin
class EmployeesListFragment : Fragment() {
 
    companion object {
        fun newInstance() = EmployeesListFragment()
    }
    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        return inflater.inflate(R.layout.fragment_employees_list, container, false)
    }
}
```

Không có gì đặc biệt.

Trong mô hình MVVM nếu như bạn để ý kĩ ở trên, các tác vụ liên quan đến tải dữ liệu sẽ không phải do UI phụ trách, mà đó là việc của ViewModel (à thực chất VM không phải là thằng tải, mà chỉ là thằng gọi, chi tiết đọc tiếp sẽ rõ ._. ). Và sau khi tải xong, ViewModel sẽ trigger 1 event để thông báo cho View rằng đã có sự thay đổi về dữ liệu, yêu cầu cập nhật lên UI.

Tiếp theo, hãy tạo 1 class ViewModel để nó có thể tải được dữ liệu về và bắn sang cho UI. 

```kotlin
class EmployeesListViewModel(
    val employeeUseCase: EmployeeUseCase,
    val scheduler: SchedulerProvider
) : BaseViewModel() {
 
    var employeeList: MutableLiveData<List<Employee>> = MutableLiveData()
        private set
 
    init {
        employeeList.default(mutableListOf())
    }
 
    fun getEmployees() {
        employeeUseCase.getEmployeeList()
            .subscribeOn(scheduler.io())
            .observeOn(scheduler.ui())
            .subscribe({ employees ->
                employeeList.value = employees
                Timber.d("${employeeList.value!!.size}")
            }, { throwable ->
                throwable.printStackTrace()
            })
    }
}
```

Ây dà, bắt đầu phức tạp rồi. Trước hết hãy cùng mổ xẻ nội dung của class này để xem nó triển khai việc truy vấn dữ liệu thế nào nào .

Đầu tiên, ta khởi tạo `EmployeesListViewModel` với 2 params: 

- `employeeUseCase`: service phụ trách trực tiếp việc tải dữ liệu. Việc của nó là kết nối với server và lấy dữ liệu về cho ViewModel.
- `scheduler`: hỗ trợ quản lý thread cho `RxJava`, không có gì đặc biệt. Đơn giản chỉ là để tránh UI bị block khi dữ liệu chưa về, ta sẽ đẩy tác vụ này sang 1 thread khác ngoài main thread.

Trong ViewModel này, ta cung sẽ tạo 1 object là `employeeList` để lưu trữ trạng thái của dữ liệu chúng ta cần lấy. Ban đầu, object này sẽ là object rỗng và value của mang giá trị null (tức là `employeeList.value==null` chứ không phải `employeeList==null`). Chính vì thế trong init ta sẽ khởi tạo value của nó là 1 list rỗng

```kotlin
init {
        employeeList.default(mutableListOf())
    }
```

(Method `default` chỉ là 1 extension method mình viết thêm support việc init nhìn đẹp và dễ hiểu hơn thôi)

```kotlin
fun <T : Any?> MutableLiveData<T>.default(initValue: T) = apply {
   value = initValue
}
```

Cuối là hàm chính phụ trách việc gọi lấy dữ liệu, tương đối tường minh rồi. Trong hàm này, ViewModel sẽ sử dụng `employeeUseCase.getEmployeeList()` để lấy dữ liệu, khi dữ liệu được trả về, sẽ đc cập nhật vào trong value của `employeeList`. Sau khi dữ liệu đã dược cập nhật thành công, ViewModel sẽ trigger để bắn sang UI. Hmmmm… trong class này không có chỗ nào thể hiện là có hàm trigger ở đây cả, vậy phải làm nào?

Trước hết hãy truyền ViewModel vào UI, để UI có thể gọi được hàm `getEmployees()` của ViewModel.

Bắt đầu cần đến công việc của `Koin` rồi. Ở đây ta đang muốn UI sử dụng ViewModel, nhưng lại không muốn UI phụ trách việc khởi tạo. Time to shine!!!

Ta sẽ khởi tạo ViewModel trong UI bằng đoạn code sau trong `EmployeesListFragment`:

```kotlin
class EmployeesListFragment : Fragment() {
    private val viewModel: EmployeesListViewModel by viewModel()
...
}
```

Ở đây, `by viewModel()` là 1 lazy initiation, sẽ khởi tạo 1 object `viewModel` để có thể sử dụng trên UI.

Để sử dụng được Koin module, chúng ta sẽ làm lần lượt các bước sau:
- Tạo ra các module
- Khởi tạo các hàm init component cần thiết
- Kích hoạt koin cho Android

#### Tạo module
Đầu tiên cần phải trả lời lần lượt từng câu hỏi:
- Ta sẽ chia thành những component gì? - Ở đây ta có thể thấy ta sẽ có 3 component chính gồm `EmployeesListViewModel`, và 2 component `EmployeeUseCase` và `SchedulerProvider` sử dụng trong `EmployeesListViewModel`


#### Khởi tạo các hàm init component cần thiết (Tên module đặt tuỳ ý)

##### `SchedulerProvider` cho vào trong `AppModule` được sự dụng trong toàn bộ app

```kotlin
val appModule = module {
    single { MainApplication.instance }

    single<SchedulerProvider>(createOnStart = true) { AppSchedulerProvider() }
}
```
##### `EmployeesListViewModel` cho vào trong `viewModelModule` để quản lý viewModel

```kotlin
val viewModelModule = module {
    viewModel { EmployeesListViewModel(get(), get()) }
}
```

##### `EmployeeUseCase` cho vào `NetworkModule` để quản lý các network instance

```kotlin
val networkModule = module {
    //API
    single { EmployeeUseCase(get()) }
}
```

Sau đó wrap toàn bộ các module vào trong 1 list:

```kotlin
val koinModuleList = listOf(
    appModule,
    networkModule,
    viewModelModule
)
```

*Giải thích thêm: hàm `get()` là hàm gọi lấy trực tiếp các instance từ Koin. Đọc thêm tại [đây](https://insert-koin.io/docs/1.0/quick-references/koin-dsl/).

#### Kích hoạt koin cho Android

Ta sẽ kích hoạt ngay trong `MainApplication` của app sử dụng module list ở trên:

```kotlin
class MainApplication : Application() {
    ...
    override fun onCreate() {
        super.onCreate()
        ...
        //Start Koin
        startKoin(this, koinModuleList) <-- Dòng này này
        ...
    }
}
```

Sau khi đã tạo hết các Koin module cần thiết, ta đã có thể gọi hàm `getEmployees()` rồi. Trong `EmployeesListFragment`, ta sẽ trigger method `getEmployees()` bằng cách override `onViewCreated()` để lấy dữ liệu employees ngay khi UI khởi tạo xong:

```kotlin
class EmployeesListFragment : Fragment() {
    ...
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        viewModel.getEmployees()
    }
}
```

Quay lại viewModel, ta thấy sau khi request API để lấy thông tin về employees, dữ liệu trả về sẽ được update vào `employeeList` qua hàm `setValue()` (hay được kotlin simplify thành `.value` )

```kotlin
fun getEmployees() {
        employeeUseCase.getEmployeeList()
            .subscribeOn(scheduler.io())
            .observeOn(scheduler.ui())
            .subscribe({ employees ->
                employeeList.value = employees <--- dòng này này
                Timber.d("${employeeList.value!!.size}")
            }, { throwable ->
                throwable.printStackTrace()
            })
    }
```
*Chi tiết về cách lấy dữ liệu trên server, ae có thể tự implement tuỳ cách, mình sẽ không giải thích rõ ở đây.

Ok vậy là dữ liệu đã được lấy về, việc cuối cùng ta phải làm lúc này là đẩy lên UI.

Giải thích qua 1 chút về cách thức vận hành của `MutableLiveData`. Sau khi ta update dữ liệu thông qua `setValue()` của `MutableLiveData` object, 1 hàm `dispatchingValue()` sẽ được gọi nội trong object này, nhiệm vụ của nó là thông báo cho tất cả các observer đang theo dõi biến này rằng: dữ liệu đã được cập nhật, chúng mày làm gì thì làm đi.

Ồ, vậy để có thể lấy được dữ liệu từ biến `MutableLiveData`, ta sẽ phải tạo 1 observer để theo dõi sự thay đổi về value. Chúng ta sẽ tạo trong UI như sau:

```kotlin
class EmployeesListFragment : Fragment() {

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        viewModel.employeeList.observe(this, Observer {employees -> 
            //TODO Update employees to UI
            //
        })
        
        viewModel.getEmployees()
    }

}
```

1 observer được tạo thồng qua method `observe()`. Method này gồm 2 biến truyền vào:
- `this`: `LifecycleOwner` object mà observer này sẽ được bind vào. khi owner bị destroy, observer này sẽ bị clear theo để tránh leak.
- `Observer{}`: `Observer` interface và chúng ta sẽ implement hàm `onChanged()` để bắt các thay đổi về value của `employeesList`. Mỗi khi có sự thay đổi về value, `onChanged()` sẽ đươc gọi và UI có thể sử dụng nó.

Awesome! Vậy là chúng ta đã tương đối hoàn thành việc implement MVVM 1 cách đơn giản nhất. Việc cuối cùng là tận dụng lợi ích của nó, đó là viết Test.

## Tạo Unit Test cho `EmployeesListViewModel`

Đặc điểm nổi trội của MVVM là ta có thể tách các thành phần về business ra khỏi khái niệm về Android, từ đấy giúp việc viết test cho các tác vụ business trở nên đơn giản hơn rất nhiều.

Các vấn đề về UI hãy để cho UI test hay Manual Test hay Automation Test làm. Lúc này việc của developer đơn giản chỉ là viết test cho viewModel, hay cụ thể là `EmployeesListViewModel` cho ví dụ ở trên.

Chúng ta sẽ đi lần lượt từng bước:

### Mock `SchedulerProvider` phục vụ cho Unit Test

Mặc dù ta đã tách Android context ra khỏi viewModel, nhưng có 1 cái vẫn còn tồn tại, đấy là `scheduler`. Khi chạy Unit Test, sẽ k có khái niệm về UI thread, vì vậy ta phải chuyển đổi UI thread thành 1 thread thông thường. Đấy là lí do ở đầu bài viết ta truyền `scheduler` vào viewModel dưới dạng interface, sẽ tiện cho việc định nghĩa lại `SchedulerProvider` component mà k ảnh hưởng đến logic trong model:

```kotlin
class TestSchedulerProvider : SchedulerProvider {
    override fun io() = Schedulers.trampoline()

    override fun ui() = Schedulers.trampoline()

    override fun computation() = Schedulers.trampoline()
}
```

```kotlin
val testRxModule = module(override = true) {
    single<SchedulerProvider> { TestSchedulerProvider() }
}
```

Sau đó override lên module list hiện tại: 

```kotlin
val testModuleList = koinModuleList + testRxModule
```

Đã chuẩn bị xong cho viêt Unit Test. Việc tạo Test suite giờ tương đối đơn giản rồi:

```kotlin
class EmployeesListVIewModelTest : KoinTest {
    private val viewModel: EmployeesListViewModel by inject()
    private lateinit var mockList: MutableList<Employee>

    @get:Rule
    var instantExecutorRule = InstantTaskExecutorRule()

    @Before
    fun before() {
        startKoin(testModuleList)

        declareMock<EmployeeUseCase>()

        mockList = MutableList(3) { index ->
            Employee(index, "Employee$index")
        }
    }

    @After
    fun after() {
        stopKoin()
    }

    @Test
    fun `Test getEmployees success`() {
        `when`(
            viewModel.employeeUseCase.getEmployeeList()
        ).thenReturn(Single.just(mockList))

        viewModel.getEmployees()
        Assert.assertEquals(mockList.size, viewModel.employeeList.value!!.size)
    }
}
```

Giải  thích thêm:

- interface `KoinTest`: implement interface này để có thẻ sử dụng các Koin module trong test suite (`startKoin`, `stopKoin`...)
- `inject()`: tương tự như `viewModel()`
- `instantExecutorRule`: Chuyển đổi các tác vụ background trong MVVM về dạng synchronous (Xử lý tuần tự).
- `declareMock`: Ta sử dụng hàm này khi muốn mock 1 class chỉ định.


Run thử test và tèn tennn:

![final-result]({{site.url}}/assets/images/20190105-final-result.png)

--- 

## The End. 
Bài tương đối dài. Cảm ơn anh em đã kiên trì đọc bài. Happy Coding!