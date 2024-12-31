---
icon: network-wired
---

# Kotlin의 Property Delegation과 Compose의 remember 함수 이해하기

***

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Jetpack Compose에서 **Composable의 생명주기**는 다음과 같은 단계로 이루어집니다:

#### (1) Enter (진입)

* Composable이 처음 생성되고 컴포지션에 추가됩니다.

#### (2) Recompose (재구성)

* Composable은 상태(state)가 변경될 때마다 재구성됩니다.
* 이 과정에서 UI는 변경된 상태를 반영하며 업데이트됩니다.

#### (3) Exit (종료)

* 더 이상 필요하지 않은 Composable은 컴포지션에서 제거됩니다.

`이떄,remember` 함수는 이 생명주기 동안 값이 효율적으로 유지되도록 도와줍니다. \
만약 `remember`를 사용하지 않는다면, 비싼 연산이 재구성될 때마다 반복적으로 수행되어 성능 문제가 발생할 수 있습니다.\
\
이렇듯, Kotlin과 Jetpack Compose를 사용하다 보면 효율적인 코드 작성과 성능 최적화를 위해 알아두어야 할 개념들이 있습니다. 특히 **Property Delegation**과 **remember 함수**는 코드 가독성과 성능 향상에 큰 도움을 줍니다.&#x20;

### 1. Property Delegation (속성 위임)

Property Delegation은 Kotlin에서 제공하는 강력한 기능으로, getter와 setter를 커스터마이징하거나 반복적인 코드를 줄이는 데 유용합니다. 대표적인 예로 `lazy`, `observable`, `vetoable` 같은 기본 제공 델리게이트를 사용할 수 있습니다.

#### 주요 예제

**(1) Lazy Initialization**

```kotlin
val lazyValue: String by lazy {
    println("Computed!")
    "Hello, Kotlin!"
}

fun main() {
    println(lazyValue) // "Computed!" 출력 후 "Hello, Kotlin!" 출력
    println(lazyValue) // "Hello, Kotlin!"만 출력
}
```

* **설명**: `lazy`는 값이 처음 호출될 때 초기화되며, 이후에는 캐싱된 값을 반환합니다.

**(2) Observable Delegation**

```kotlin
import kotlin.properties.Delegates

var name: String by Delegates.observable("Initial") { _, old, new ->
    println("$old -> $new")
}

fun main() {
    name = "Kotlin"
    name = "Compose"
}
```

* **설명**: 속성이 변경될 때마다 콜백이 호출되어 변화 내용을 추적할 수 있습니다.

***

### 2. remember 함수

Jetpack Compose에서 `remember` 함수는 컴포지션 내에서 상태를 저장하는 데 사용됩니다. 이는 불필요한 재구성을 방지하고 효율적인 상태 관리를 가능하게 합니다.

#### 주요 특징

* **Composition Scope**: `remember`는 해당 컴포지션이 살아있는 동안 값을 저장합니다.
* **재구성 방지**: 값이 변경되지 않는 한 재구성(recomposition) 시에도 저장된 값을 그대로 유지합니다.
* **효율성**: 반복적으로 수행되는 비싼 연산을 피할 수 있습니다.

#### 주요 예제

**(1) 간단한 상태 저장**

```kotlin
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }

    Button(onClick = { count++ }) {
        Text("Clicked $count times")
    }
}
```

* **설명**: `remember`를 사용하여 `mutableStateOf`를 감싸면, 컴포지션이 재구성되어도 카운터 값이 유지됩니다.

**(2) 비싼 연산 저장**

```kotlin
@Composable
fun ExpensiveOperationDemo() {
    val result = remember {
        performExpensiveOperation()
    }

    Text("Result: $result")
}

fun performExpensiveOperation(): String {
    println("Expensive operation performed")
    return "Operation Complete"
}
```

* **설명**: `performExpensiveOperation` 함수는 `remember` 덕분에 한 번만 호출됩니다. 이후 재구성 시에도 결과값은 유지됩니다.

**(3) remember와 rememberSaveable의 차이**

```kotlin
@Composable
fun RememberVsRememberSaveable() {
    var rememberValue by remember { mutableStateOf(0) }
    var saveableValue by rememberSaveable { mutableStateOf(0) }

    Column {
        Button(onClick = { rememberValue++ }) {
            Text("Remember: $rememberValue")
        }
        Button(onClick = { saveableValue++ }) {
            Text("Saveable: $saveableValue")
        }
    }
}
```

* **설명**: `rememberSaveable`은 프로세스 종료 또는 화면 회전 이후에도 값을 유지합니다. 반면, `remember`는 컴포지션 범위에서만 값을 저장합니다.

***

### 3. 성능 최적화

`remember`와 같은 도구를 활용하면 Jetpack Compose에서 다음과 같은 성능 최적화를 이룰 수 있습니다:

* **불필요한 재구성 방지**: 상태 저장을 통해 비효율적인 UI 업데이트를 방지합니다.
* **비싼 연산 최적화**: 반복적으로 호출될 필요가 없는 연산을 한 번만 수행합니다.
* **코드 간소화**: 명확하고 간결한 상태 관리 코드 작성이 가능합니다.

***

### 결론

Kotlin의 Property Delegation과 Jetpack Compose의 `remember` 같은 함수는 \
각각 다른 영역에서 코드의 효율성과 가독성을 높이는 데 중요한 역할을 합니다.

\
\
