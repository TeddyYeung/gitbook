---
description: 'Jetpack Compose 상태 호이스팅 (State Hoisting): 핵심 개념'
icon: network-wired
---

# Understand state hoisting in Compose

* **상태를 가지는 컴포저블 vs. 상태를 가지지 않는 컴포저블**
  * **상태를 가지는 컴포저블 (Stateful Composables):**
    * 자체적으로 상태를 관리합니다.
    * 재사용성과 테스트 용이성이 떨어집니다.
    *   **예시:**

        ```kotlin
        kotlin코드 복사@Composable
        fun Counter() {
            var count by remember { mutableStateOf(0) }
            Button(onClick = { count++ }) {
                Text("Count: $count")
            }
        }
        ```
  * **상태를 가지지 않는 컴포저블 (Stateless Composables):**
    * 상태를 외부에서 받아 사용하며 상태 변경 로직도 외부에서 처리합니다.
    * 재사용성과 테스트가 더 쉬워집니다.
    *   **예시:**

        ```kotlin
        kotlin코드 복사@Composable
        fun CounterDisplay(count: Int, onIncrement: () -> Unit) {
            Button(onClick = { onIncrement() }) {
                Text("Count: $count")
            }
        }
        ```
* **상태 호이스팅 (State Hoisting)**
  * 상태를 하위 컴포저블에서 상위 컴포저블로 옮기는 과정입니다.
  * 하위 컴포저블에서 상태 변수를 제거하고,
    * 현재 상태 값
    * 상태 변경을 처리하는 람다 함수를 **매개변수**로 받도록 합니다.
  *   **예시:**

      ```kotlin
      kotlin코드 복사@Composable
      fun CounterApp() {
          var count by remember { mutableStateOf(0) }
          CounterDisplay(count = count, onIncrement = { count++ })
      }
      ```
* **상태 호이스팅의 이점**
  * **재사용성:** 상태를 가지지 않는 컴포저블은 여러 화면이나 기능에서 재사용할 수 있습니다.
  * **유지보수성:** 상태 관리와 UI 로직이 분리되어 코드가 더 깔끔하고 이해하기 쉬워집니다.\


**플러터와의 유사성**

Jetpack Compose의 상태 호이스팅 개념은 **Flutter의 상태 관리 패턴**과 매우 유사합니다.\
Flutter에서도 `StatefulWidget` 대신 상태를 부모로 올리고, `StatelessWidget`으로 UI를 구성하면 **테스트 가능성**과 **재사용성**이 크게 향상됩니다.

*   **Flutter 예시:**

    ```dart
    dart코드 복사class CounterApp extends StatelessWidget {
      @override
      Widget build(BuildContext context) {
        int count = 0;
        return CounterDisplay(
          count: count,
          onIncrement: () {
            count++;
          },
        );
      }
    }

    class CounterDisplay extends StatelessWidget {
      final int count;
      final VoidCallback onIncrement;

      CounterDisplay({required this.count, required this.onIncrement});

      @override
      Widget build(BuildContext context) {
        return ElevatedButton(
          onPressed: onIncrement,
          child: Text('Count: $count'),
        );
      }
    }
    ```

#### 요약

* 상태 호이스팅은 Jetpack Compose와 Flutter 모두에서 **재사용성과 유지보수성**을 개선하는 중요한 원칙입니다.&#x20;
* 더 **구조적인 코드**와 **유연한 UI 컴포넌트**를 설계할 수 있습니다.

\
\
