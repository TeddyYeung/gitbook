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



