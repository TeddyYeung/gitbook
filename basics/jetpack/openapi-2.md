---
icon: network-wired
description: 'Jetpack Compose 상태 관리: 핵심 개념'
---

# Understand State in Compose

**Compose  Runtime**

<div align="left"><figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption><p>Compose  Runtime은  화면을 다시 렌더링 해야하는 업데이트를 항상 주시합니다.</p></figcaption></figure></div>

Compose  Runtime은 화면의 렌더링 업데이트를 주시하는가?

<div align="left"><figure><img src="../../.gitbook/assets/image (4) (1).png" alt="" width="563"><figcaption><p>Compose  는 호출되는 모든 구성 가능한 함수들을 추적합니다. </p></figcaption></figure></div>

* 입력된 데이터의 스냅샷을 생성합니다.
* 각 Composable 은 상태와 함께 테이블에 저장됩니다.&#x20;
* Compose는 새로운 상태를 이전 상태와 비교해서 화면을 업데이트합니다.&#x20;



1. **상태 관리 (State Management)**
   * 동적이고 반응형 UI를 만들기 위해 상태 관리는 필수적입니다.
   *   **예시:**

       ```kotlin
       @Composable
       fun Counter() {
           var count by remember { mutableStateOf(0) }
           Button(onClick = { count++ }) {
               Text("Count: $count")
           }
       }

       ```
2. **재구성 (Recomposition)**
   * Compose는 컴포저블 함수와 그 상태를 추적하며, 상태가 변경될 때 UI를 자동으로 갱신합니다.
   * **예시:** 사용자가 버튼을 누를 때 `count` 값이 변경되면 버튼의 텍스트가 자동으로 업데이트됩니다.
3. **관찰 가능한 상태 (Observable State)**
   * Compose는 일반 Kotlin 변수 대신 **mutableState**와 같은 관찰 가능한 상태를 사용해야 UI 변경을 감지하고 업데이트할 수 있습니다.
   *   **피해야 할 코드:**

       ```kotlin
       var count = 0 // Compose가 상태 변화를 추적하지 못함
       ```
   *   **권장 코드:**

       ```kotlin
       var count by remember { mutableStateOf(0) } // Compose가 상태를 감지
       ```

\
\
\
