---
icon: plug-circle-plus
---

# 코루틴 개념 및 사용

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption><p>코루틴을 사용하여 메인 스레드와 네트워크 스레드 간 작업을 효율적으로 분리하는 방법</p></figcaption></figure>

1. **Main Thread (메인 스레드)**:
   * 메인 스레드는 UI 작업(예: **Drawing events**)과 같은 사용자 인터페이스 관련 이벤트를 처리합니다.
   * 메인 스레드에서 API 요청을 실행하면 UI가 멈출 수 있기 때문에 비동기 처리가 필요합니다.
2. **Suspend Function (중단 함수)**:
   * 메인 스레드에서 요청(**Request**)을 실행할 때, 이 작업은 **Suspend Function**으로 표시됩니다.
   * 이 중단 함수는 네트워크 작업이 완료될 때까지 메인 스레드의 작업을 차단하지 않고, 다른 작업을 계속할 수 있도록 합니다.
3. **Network Thread (네트워크 스레드)**:
   * API 요청은 네트워크 스레드에서 처리됩니다. 다이어그램에서 **API request**로 표시된 부분이 이 작업을 나타냅니다.
   * 네트워크 작업이 끝나면 결과가 다시 메인 스레드로 반환됩니다.
4. **코루틴의 역할**:
   * 메인 스레드에서 UI가 멈추지 않도록 네트워크 요청을 별도의 스레드에서 처리합니다.
   * API 요청이 끝난 후에는 다시 메인 스레드로 돌아와 결과를 처리하거나 UI를 업데이트합니다.

#### 코루틴이 필요한 경우:

* **UI 작업과 네트워크 요청을 분리해야 할 때**: 사용자가 앱에서 버튼을 클릭하거나 화면을 스크롤할 때, 네트워크 요청으로 인해 앱이 멈추는 것을 방지하기 위해 코루틴이 필요합니다.
* **비동기 작업을 효율적으로 처리할 때**: 네트워크 호출, 데이터베이스 작업 등 시간이 오래 걸릴 수 있는 작업을 처리하면서 메인 스레드가 중단되지 않도록 합니다.
* **복잡한 비동기 흐름을 간결하게 관리할 때**: 예를 들어, 여러 API 요청을 순차적으로 실행하거나 병렬로 실행할 때 코루틴은 코드 가독성을 높여줍니다.



**Retrofit과 Coroutines를 사용해 원격 데이터 가져오기**

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption><p><strong>CoroutineScope</strong></p></figcaption></figure>

**Coroutines와 메인 스레드 반응성**

* Kotlin의 **Coroutine**은 비동기 작업을 메인 스레드를 차단하지 않고 수행할 수 있게 해준다.
* 이를 통해 앱이 사용자 입력과 화면 그리기에 계속 반응하도록 보장할 수 있다.
* 모든 Coroutine은 **CoroutineScope** 내에서 실행되며, Scope를 취소하면 하위 Coroutine도 모두 취소된다.\


**안드로이드에서의 Scope 활용**

<div align="left"><figure><img src="../../.gitbook/assets/image (21).png" alt="" width="371"><figcaption></figcaption></figure></div>

* 안드로이드 **Jetpack** 컴포넌트는 `LifecycleScope`, `ViewModelScope`, `LiveDataScope`와 같은 Scope를 제공한다.
* 이러한 Scope는 Activity나 Fragment 등의 생명주기와 연동되며, 생명주기가 종료되면 관련 Coroutine이 자동으로 취소된다.
* 이를 통해 불필요한 작업을 줄이고 사용자 경험을 향상시킬 수 있다.\


## **Retrofit과 Coroutine을 사용한 Android 비동기 데이터 처리**&#x20;

#### **1. Retrofit 설정**

**Retrofit API 인터페이스**

* API 호출을 정의하는 인터페이스
* `@GET` 등 Retrofit에서 제공하는 어노테이션을 사용해 HTTP 메서드와 경로를 지정한다.
* `suspend` 키워드를 붙여 이 메서드가 Coroutine 내에서 호출 가능하도록 함&#x20;

```kotlin
interface ProductApi {
    @GET("products")
    suspend fun getProducts(): Response<List<Product>>
}
```

* `Response<List<Product>>`는 Retrofit이 HTTP 응답을 캡슐화한 객체를 반환하며, 성공 여부 및 데이터를 포함한다.

***

**Retrofit 객체 생성**

* Retrofit은 HTTP 요청을 처리하는 클라이언트 라이브러리로, 빌더 패턴을 통해 설정을 지정한다.
* `baseUrl`을 지정하고, 데이터를 JSON으로 변환하기 위해 `GsonConverterFactory`를 추가한다.
* `lazy` 키워드를 사용해 Retrofit 객체를 지연 초기화한다.

```kotlin
object RetrofitInstance {
    private const val BASE_URL = "https://your-api-url.com/"

    val api: ProductApi by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(ProductApi::class.java)
    }
}
```

***

#### **2. Repository 작성**

**ProductRepository 클래스**

* API 호출 로직을 분리하여 ViewModel이 Retrofit과 직접 통신하지 않도록 한다.
* `getProducts` 함수는 suspend 함수로 정의되어, Coroutine 내에서 호출된다.
* 응답의 성공 여부를 확인한 뒤, 결과 데이터를 반환하거나 빈 리스트를 반환한다.

```kotlin
class ProductRepository {
    suspend fun getProducts(): List<Product> {
        val response = RetrofitInstance.api.getProducts()
        return if (response.isSuccessful) {
            response.body() ?: emptyList()  // 응답 성공 시 데이터 반환
        } else {
            emptyList()  // 실패 시 빈 리스트 반환
        }
    }
}
```

***

#### **3. ViewModel 작성**

**MainViewModel 클래스**

* ViewModel은 비즈니스 로직을 처리하며 UI와 데이터를 연결한다.
* `viewModelScope`는 ViewModel의 생명주기에 맞춰 Coroutine을 관리한다.
* `fetchProducts` 함수는 Repository에서 데이터를 가져와 `LiveData` 객체에 설정한다.

```kotlin
class MainViewModel(private val repository: ProductRepository) : ViewModel() {

    private val _products = MutableLiveData<List<Product>>()  // 내부 데이터
    val products: LiveData<List<Product>> get() = _products  // 외부에서 관찰 가능한 데이터

    fun fetchProducts() {
        viewModelScope.launch {
            try {
                val productList = repository.getProducts()  // Repository 호출
                _products.value = productList  // LiveData 업데이트
                Log.d("MainViewModel", "Products loaded: $productList")  // 디버그 로그
            } catch (e: Exception) {
                Log.e("MainViewModel", "Error loading products", e)  // 에러 로그
            }
        }
    }
}
```

***

#### **4. View에서 ViewModel 호출**

**MainActivity 또는 Fragment**

* ViewModelProvider를 통해 ViewModel 객체를 생성한다.
* ViewModel의 `LiveData`를 관찰해 데이터를 UI에 반영한다.
* `fetchProducts`를 호출해 데이터를 가져오도록 한다.

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var viewModel: MainViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val repository = ProductRepository()
        viewModel = ViewModelProvider(this, ViewModelFactory(repository)).get(MainViewModel::class.java)

        // ViewModel의 LiveData 관찰
        viewModel.products.observe(this) { productList ->
            productList?.let {
                // RecyclerView 또는 UI 업데이트
                Log.d("MainActivity", "Loaded products: $it")
            }
        }

        // ViewModel에서 데이터 가져오기 호출
        viewModel.fetchProducts()
    }
}
```

***

#### **5. 전체 흐름 설명**

1. **Retrofit 설정**
   * API 호출 로직을 정의한 인터페이스(`ProductApi`)를 Retrofit 객체로 초기화한다.
2. **Repository 작성**
   * ViewModel에서 API 호출 로직을 분리하기 위해 Repository에서 Retrofit 메서드를 호출한다.
   * `suspend` 함수로 비동기 호출을 처리하며, 결과를 반환한다.
3. **ViewModel**
   * ViewModel은 Repository에서 데이터를 가져오고, `LiveData`에 결과를 저장한다.
   * `viewModelScope`를 사용해 ViewModel의 생명주기에 맞춘 Coroutine을 실행한다.
4. **View에서 데이터 관찰**
   * Activity 또는 Fragment에서 ViewModel의 `LiveData`를 관찰해 데이터 변화를 감지하고, UI를 업데이트한다.

***

#### **추가 팁**

1. **에러 처리**
   * 네트워크 요청이 실패할 경우 사용자에게 피드백을 제공할 수 있도록 UI에서 상태를 관리하는 것이 좋다.
   * 예: `sealed class`를 사용해 상태를 Success, Loading, Error로 구분.
2. **DI (Dependency Injection)**
   * Repository나 Retrofit 객체의 생성에 `Hilt` 또는 `Koin` 같은 의존성 주입 라이브러리를 사용하는 것이 권장된다.
3. **테스트**
   * ViewModel과 Repository는 단위 테스트가 가능하도록 설계되었다.
   * Mock API나 Fake Repository를 활용해 테스트를 작성하면 안정성을 높일 수 있다.

\


