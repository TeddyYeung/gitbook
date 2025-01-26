# 2: else 예약어 금지 (Don't Use the ELSE Keyword)

**문제점: 왜 `else`를 피해야 할까?**

1. **불필요한 중첩**: `else`를 사용하면 들여쓰기가 깊어지고 코드의 흐름을 따라가기 어려워집니다.
2. **불명확한 의도**: `else`가 포함된 조건문은 논리적으로 의도를 파악하기 어려울 수 있습니다.
3. **확장성 저하**: 조건이 늘어나거나 복잡해질수록 `else` 블록이 커지면서 유지보수가 어려워집니다.

주로 사용할 수 있는 방식은 3가지가 있겠습니다.&#x20;

### 1.  Return Early Pattern (Early Return & Fail Fast)

**Before: `else`를 사용한 코드**

```dart
void updateUserProfile(String username, BuildContext context) {
  if (username.isNotEmpty) {
    // 서버에 사용자 프로필 업데이트 요청
    saveProfile(username);
  } else {
    // 사용자에게 경고 메시지 표시
    showSnackBar(context, 'Username cannot be empty');
  }
}
```

**After: `else`를 제거한 코드**

```dart
void updateUserProfile(String username, BuildContext context) {
  if (username.isEmpty) {
    showSnackBar(context, 'Username cannot be empty');
    return; // 비정상 상태를 빠르게 처리하고 종료
  }
  saveProfile(username); // 정상 상태에서만 실행
}
```

**개선된 점:**

1. **예외적인 상황을 먼저 처리**:
   * `username`이 비어 있는 경우는 정상적인 흐름이 아니므로, 이 조건을 가장 먼저 확인하여 처리합니다.
2. **정상 흐름을 강조**:
   * 조건이 실패했을 때 즉시 종료(`return`)하기 때문에, 나머지 코드는 정상 상태에서의 작업에만 집중할 수 있습니다.
   * 결과적으로 코드의 논리가 더 직관적으로 보입니다.

### 2.  조건문을 다형성으로 바꾸기 (Replace Conditional with _PolyMorphism)_



조건문을 다형성으로 바꾸는 리팩토링 기법은 코드의 유지보수성을 높이고 확장성을 개선하는 데 유용합니다.

&#x20;아래에서 `if-else`나 `switch-case`를 다형성으로 대체하는 방법을 설명하겠습니다. \
이 방식은 보통 상태에 따라 다르게 행동해야 하는 경우나, 반복적으로 조건문을 사용하는 경우에 효과적입니다.

#### 기존 코드 (Before)

조건문과 `else`를 사용하여 상태에 따라 출력 메시지를 처리하는 코드:

```dart
void handleButtonPress(String status) {
  if (status == 'loading') {
    print('Loading... Please wait');
  } else if (status == 'success') {
    print('Operation successful!');
  } else if (status == 'error') {
    print('An error occurred. Try again.');
  } else {
    print('Unknown status');
  }
}
```

**문제점:**

1. 상태가 늘어날수록 `if-else` 블록이 길어지고 유지보수가 어려워짐.
2. 각 상태의 동작이 분리되지 않아 코드가 응집력이 낮음.

***

#### 다형성 기반으로 리팩토링 (After)

**Step 1: 상태별 동작을 추상화**

상태별 동작을 정의하는 `ButtonState`라는 추상 클래스를 생성합니다.

```dart
abstract class ButtonState {
  void handle();
}
```

**Step 2: 상태별 클래스를 구현**

각 상태에 대해 클래스를 생성하고, 해당 상태에서 수행할 동작을 `handle` 메서드에 정의합니다.

```dart
class LoadingState implements ButtonState {
  @override
  void handle() {
    print('Loading... Please wait');
  }
}

class SuccessState implements ButtonState {
  @override
  void handle() {
    print('Operation successful!');
  }
}

class ErrorState implements ButtonState {
  @override
  void handle() {
    print('An error occurred. Try again.');
  }
}

class UnknownState implements ButtonState {
  @override
  void handle() {
    print('Unknown status');
  }
}
```

**Step 3: 다형성을 활용한 메서드 작성**

`ButtonState` 객체를 받아 해당 상태에 맞는 동작을 실행하도록 메서드를 수정합니다.

```dart
void handleButtonPress(ButtonState state) {
  state.handle();
}
```

***

**Step 4: 상태를 매핑하는 팩토리 메서드**

문자열 `status` 값을 상태 객체로 매핑하는 팩토리 메서드를 추가합니다. 이를 통해 기존 코드의 호환성을 유지합니다.

```dart
ButtonState getStateFromStatus(String status) {
  switch (status) {
    case 'loading':
      return LoadingState();
    case 'success':
      return SuccessState();
    case 'error':
      return ErrorState();
    default:
      return UnknownState();
  }
}
```

***

**Step 5: 리팩토링된 코드 사용**

상태 문자열을 기반으로 상태 객체를 생성하고, `handleButtonPress` 메서드로 전달합니다.

```dart
void main() {
  String status = 'loading';
  ButtonState state = getStateFromStatus(status);
  handleButtonPress(state); // Output: Loading... Please wait

  status = 'success';
  state = getStateFromStatus(status);
  handleButtonPress(state); // Output: Operation successful!

  status = 'error';
  state = getStateFromStatus(status);
  handleButtonPress(state); // Output: An error occurred. Try again.

  status = 'unknown';
  state = getStateFromStatus(status);
  handleButtonPress(state); // Output: Unknown status
}
```

***

#### 개선된 점:

1. **조건문 제거**: 상태별 로직이 객체로 분리되어 `if-else` 문을 없앴습니다.
2. **확장성 증가**: 새로운 상태를 추가할 때 기존 코드를 수정할 필요 없이 새로운 클래스를 추가하기만 하면 됩니다.
3. **응집력 향상**: 각 상태의 동작이 독립적으로 정의되어 코드가 더 모듈화되었습니다.
4. **가독성 향상**: 상태와 동작의 관계가 명확하게 드러나고, 코드의 흐름을 쉽게 이해할 수 있습니다.



