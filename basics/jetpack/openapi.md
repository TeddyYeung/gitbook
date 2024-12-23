---
icon: network-wired
---

# JetPack Compose

**구성 가능한 함수**

구성 가능한 함수는 `@Composable` 주석이 붙은 일반 함수다.

```kotlin
@Composable
private fun Greeting(name: String) {
   Text(text = "Hello $name!")
}
```

위 `Greeting` 함수는 전달받은 문자열을 표시하는 UI 계층 구조를 생성한다.\
여기서 `Text`는 라이브러리에서 제공하는 구성 가능한 함수  == 플러터 위젯&#x20;

#### Android 앱의 Compose <a href="#android-compose" id="android-compose"></a>

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            BasicsCodelabTheme {
                // 테마에서 정의된 배경색을 사용하는 Surface 컨테이너
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    Greeting("Android") // 구성 가능한 함수 Greeting 호출
                }
            }
        }
    }
}
```

`setContent` 함수를 사용하여 XML 파일 대신 구성 가능한 함수를 호출해 레이아웃을 정의한다.\
`BasicsCodelabTheme`은 구성 가능한 함수의 스타일을 지정하는 역할을 한다.



**Android Studio에서 미리보기**

```kotlin
@Preview(showBackground = true, name = "Text preview")
@Composable
fun DefaultPreview() {
    BasicsCodelabTheme {
        Greeting(name = "Android")
    }
}
```

Android Studio에서 미리보기를 사용하려면 구성 가능한 함수에 `@Preview` 주석을 추가하고 프로젝트를 빌드 하면 된다. 또한, 동일한 파일에 여러 개의 미리보기를 생성하고 이름을 지정할 수도 있다.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>



\
\
\
