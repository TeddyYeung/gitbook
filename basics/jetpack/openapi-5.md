---
icon: network-wired
---

# Add a ViewModel to a Composable



<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

**라이프사이클**:\
ViewModel은 UI 데이터를 라이프사이클에 맞춰 관리하도록 설계되어, \
화면 회전과 같은 구성 변경 중에도 데이터를 안전하게 유지할 수 있습니다.\
\


<div align="center"><figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure></div>

**필수 의존성**:\
Jetpack Compose에서 ViewModel을 사용하려면 프로젝트에 `lifecycle-runtime-compose`와 `lifecycle-viewmodel-compose` 의존성을 추가해야 합니다.



**구현 방법**:

* `ViewModel` 클래스를 확장하여 ViewModel 클래스를 생성합니다.
* 생성한 ViewModel을 컴포저블 함수나 액티비티에서 초기화하고 참조합니다.
* 이를 통해 UI 데이터를 효율적으로 관리하고 앱 라이프사이클 동안 데이터를 유지할 수 있습니다.

#### **Compose State & Kotlin StateFlow**

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

#### **Compose State 예제**

```kotlin
kotlin코드 복사import androidx.compose.runtime.*
import androidx.compose.material3.*
import androidx.compose.foundation.layout.*
import androidx.compose.ui.unit.dp
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.runtime.Composable

@Composable
fun ComposeStateExample() {
    // Compose State: 상태를 관리하는 변수
    var count by remember { mutableStateOf(0) }

    Column(
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally,
        modifier = Modifier.fillMaxSize()
    ) {
        Text(text = "Count: $count", style = MaterialTheme.typography.titleLarge)
        Spacer(modifier = Modifier.height(16.dp))
        Button(onClick = { count++ }) { // 버튼 클릭 시 상태 업데이트
            Text("Increment")
        }
    }
}

@Preview(showBackground = true)
@Composable
fun PreviewComposeStateExample() {
    ComposeStateExample()
}
```

**설명**:

* `remember { mutableStateOf(0) }`를 사용해 UI 상태를 관리합니다.
* 버튼을 클릭하면 `count` 값이 증가하고, 텍스트는 즉시 업데이트됩니다.

***

#### **Kotlin StateFlow 예제**

```kotlin
kotlin코드 복사import androidx.compose.runtime.*
import androidx.compose.material3.*
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewmodel.compose.viewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.StateFlow
import androidx.compose.runtime.collectAsState
import androidx.compose.foundation.layout.*
import androidx.compose.ui.unit.dp

// ViewModel 정의
class StateFlowViewModel : ViewModel() {
    private val _count = MutableStateFlow(0) // StateFlow 생성
    val count: StateFlow<Int> = _count.asStateFlow() // 외부에 StateFlow 제공

    fun increment() {
        _count.value++ // 값 업데이트
    }
}

@Composable
fun StateFlowExample(viewModel: StateFlowViewModel = viewModel()) {
    // StateFlow를 Compose에서 구독
    val count by viewModel.count.collectAsState()

    Column(
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally,
        modifier = Modifier.fillMaxSize()
    ) {
        Text(text = "Count: $count", style = MaterialTheme.typography.titleLarge)
        Spacer(modifier = Modifier.height(16.dp))
        Button(onClick = { viewModel.increment() }) { // 버튼 클릭 시 ViewModel 상태 업데이트
            Text("Increment")
        }
    }
}
```

**설명**:

1. **ViewModel**에서 `MutableStateFlow`를 생성하고 값을 업데이트합니다.
2. Compose에서 `collectAsState()`를 사용해 StateFlow를 구독합니다.
3. 값이 변경되면 Compose UI가 즉시 업데이트됩니다.

***

#### **Compose State와 StateFlow 차이점**

**Compose State**는 단일 Compose 함수 안에서 상태를 관리합니다.\
\
\- 쉬운 예시) \
Compose의 상태는 교실의 칠판과 같습니다. 칠판에 숫자나 글자를 쓰면 모두가 즉시 볼 수 있고, 내용을 지우고 새로 쓰면 즉시 반영됩니다. UI가 데이터를 보고 즉시 업데이트되는 방식이 Compose State의 작동 원리입니다.

**Kotlin StateFlow**는 ViewModel 또는 데이터 계층에서 상태를 관리하며, 여러 구독자가 데이터를 공유할 수 있습니다.\
\
\- 쉬운 예시) \
StateFlow는 뉴스 방송과 비슷합니다. 뉴스 채널이 계속 새로운 소식을 보내면 여러 사람들이 이를 동시에 시청할 수 있습니다. 뉴스가 업데이트되면 모든 시청자가 이를 즉시 확인할 수 있죠. StateFlow는 앱의 여러 부분이 데이터를 듣고 반응할 수 있도록 업데이트를 전달하는 방식으로 작동합니다.\
