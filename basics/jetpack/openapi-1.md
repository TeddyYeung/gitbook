---
description: 'Jetpack Compose 함수 이해: 핵심 원칙'
icon: network-wired
---

# Understand Composable Functions

1. **@Composable 애노테이션**
   * 데이터 → UI 변환 함수에는 반드시 `@Composable` 애노테이션을 추가해야 합니다.
   *   **예시:**

       ```kotlin
       @Composable
       fun Greeting(name: String) {
           Text(text = "Hello, $name!")
       }
       ```
2. **멱등성 (Idempotency)**
   * 같은 인자로 호출하면 항상 동일한 결과를 반환해야 합니다.
   * **예시:** UI를 다시 그리는 상황에서도 동일한 컴포넌트를 생성해야 함.
3. **부작용 없음 (No Side Effects)**
   * 컴포저블 함수는 부작용을 가지면 안 됩니다.
   * 병렬 실행이나 순서에 상관없이 안정적으로 작동해야 합니다.
   *   **피해야 할 코드:**

       ```kotlin
       @Composable
       fun UpdateCounter() {
           counter++ // 부작용 발생
       }
       ```
4. **UI 방출 (UI Emission)**
   * 값을 반환하지 않고 다른 컴포저블을 호출하여 UI 계층을 만듭니다.
   *   **예시:**

       ```kotlin
       @Composable
       fun Profile() {
           Column {
               Text("Name: John Doe")
               Text("Age: 30")
           }
       }
       ```



\
\
\
