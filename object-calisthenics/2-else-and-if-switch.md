---
description: 2. Don't Use the ELSE Keyword
---

# 2: else 예약어 금지 & 다형성으로 IF문, Switch문 리팩토링하기

### **문제점: 왜 `else`를 피해야 할까?**

1. **불필요한 중첩**: `else`를 사용하면 들여쓰기가 깊어지고 코드의 흐름을 따라가기 어려워집니다.
2. **불명확한 의도**: `else`가 포함된 조건문은 논리적으로 의도를 파악하기 어려울 수 있습니다.
3. **확장성 저하**: 조건이 늘어나거나 복잡해질수록 `else` 블록이 커지면서 유지보수가 어려워집니다.

주로 사용할 수 있는 방식은 3가지가 있겠습니다.&#x20;

## 1.  Return Early Pattern (Early Return & Fail Fast)

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

## 2.  조건문을 다형성으로 바꾸기 (Replace Conditional with _PolyMorphism)_

조건문을 다형성으로 바꾸는 리팩토링 기법은 코드의 유지보수성을 높이고 확장성을 개선하는 데 유용합니다.

&#x20;아래에서 `if-else`나 `switch-case`를 다형성으로 대체하는 방법을 설명하겠습니다. \
이 방식은 보통 상태에 따라 다르게 행동해야 하는 경우나, 반복적으로 조건문을 사용하는 경우에 효과적입니다.

### 2-1. IF문 개선하기&#x20;

### 기존 코드 (Before) : 조건문과 `else`를 사용하여 상태에 따라 출력 메시지를 처리하는 코드

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

### 다형성 기반으로 리팩토링 (After)

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

### 2-2. Enum 기반 조건문에서 다형성으로 리팩토링

### 기존 코드 (Before) : **Enum 기반 조건문 사용**

먼저, 기존 코드에서는 `enum`과 `switch-case`를 사용하여 알림 유형에 따라 메시지를 처리합니다.&#x20;

```dart
enum NotificationType {
  email,
  sms,
  push,
}

void sendNotification(NotificationType type, String message) {
  switch (type) {
    case NotificationType.email:
      print('Sending email: $message');
      break;
    case NotificationType.sms:
      print('Sending SMS: $message');
      break;
    case NotificationType.push:
      print('Sending push notification: $message');
      break;
  }
}
```

#### **문제점**

**1. 새로운 상태가 추가될 때 기존 코드를 수정해야 함**

예를 들어, 기존 코드에서 새로운 상태 `NotificationType.inApp`이 추가된다고 가정합니다.

1. `enum NotificationType`에 새로운 값을 추가.
2. `sendNotification` 함수에 새로운 `case` 분기 추가.

```dart
enum NotificationType {
  email,
  sms,
  push,
  inApp, // 새로운 상태 추가
}

void sendNotification(NotificationType type, String message) {
  switch (type) {
    case NotificationType.email:
      print('Sending email: $message');
      break;
    case NotificationType.sms:
      print('Sending SMS: $message');
      break;
    case NotificationType.push:
      print('Sending push notification: $message');
      break;
    case NotificationType.inApp: // 새로운 상태에 대한 분기 추가
      print('Sending in-app notification: $message');
      break;
  }
}
```

* 새로운 상태를 처리하기 위해 **기존의 `sendNotification` 함수 내부를 수정해야만 동작을 확장**할 수 있습니다.
* 기존 코드를 수정할수록 의도치 않은 버그가 발생하거나, 다른 동작에 영향을 미칠 위험이 증가합니다.
* 이는 OCP 원칙의 "기존 코드는 수정하지 않는다"는 원칙에 어긋납니다.

***

**2. 조건문이 늘어나면서 코드 복잡도가 증가**

* `switch-case` 블록은 상태가 추가될수록 분기 조건이 많아지고 코드가 장황해집니다. 예를 들어:

```dart
case NotificationType.type1:
case NotificationType.type2:
case NotificationType.type3:
...
```

* 분기 처리가 많아질수록 코드의 **가독성**과 **유지보수성**이 떨어지고, 새로운 상태를 추가하거나 기존 상태를 변경하는 작업이 점점 어려워집니다.
* 이러한 구조는 상태를 확장하려 할 때, 시스템의 복잡도가 기하급수적으로 늘어나는 경향을 보입니다.

***

**3. 책임 분리가 안 된 구조**

`switch-case` 구조에서는 하나의 함수(`sendNotification`)가 모든 상태(NotificationType)에 대한 처리를 담당하고 있습니다. 이로 인해 다음 문제가 발생합니다:

* 함수가 \*\*단일 책임 원칙(SRP)\*\*을 위반합니다.
  * `sendNotification`은 알림을 "전송"하는 것 외에, 각 상태의 구체적인 처리를 "결정"하는 책임도 가지고 있습니다.
* 새로운 상태가 추가될 때마다, 함수가 점점 더 많은 책임을 떠안게 되어 유지보수가 어려워집니다.

***

#### OCP 위반의 근본 원인: 현재 `switch-case문이`**확장이 아닌 수정 기반의 구조**

* **OCP 준수의 핵심은 기존 코드의 변경 없이 새로운 기능을 추가하는 것**입니다.
* 그러나 `switch-case` 블록은 새로운 상태를 추가할 때 **수정**이 필요합니다. 이는 확장을 위해 기존 코드를 수정해야 하는 "수정 기반 구조"이기 때문입니다.

### 다형성 기반으로 리팩토링 (After)

**Step 1: 추상 클래스 정의**

`NotificationSender`라는 추상 클래스를 만들어, 알림 전송의 공통 동작을 정의합니다.

```dart
abstract class NotificationSender {
  void send(String message);
}
```

**Step 2: 각 동작을 클래스로 구현**

`NotificationType`에 따라 알림 전송 방식이 달라지므로 각 유형에 맞는 클래스를 작성하여 `send` 메서드를 구현합니다.

```dart
class EmailNotificationSender implements NotificationSender {
  @override
  void send(String message) {
    print('Sending email: $message');
  }
}

class SmsNotificationSender implements NotificationSender {
  @override
  void send(String message) {
    print('Sending SMS: $message');
  }
}

class PushNotificationSender implements NotificationSender {
  @override
  void send(String message) {
    print('Sending push notification: $message');
  }
}

## 새로운 상태가 필요하다면 기존 코드를 수정하지 않고 새로운 클래스만 추가하면 됩니다
class InAppNotificationSender implements NotificationSender {
  @override
  void send(String message) {
    print('Sending in-app notification: $message');
  }
}
```

**Step 3: 팩토리 패턴으로 Enum 매핑**

`NotificationType`을 받아 적절한 `NotificationSender` 객체를 반환하는 팩토리 메서드를 추가합니다. 이 메서드를 통해 조건문을 없앨 수 있습니다.

```dart
NotificationSender getNotificationSender(NotificationType type) {
  switch (type) {
    case NotificationType.email:
      return EmailNotificationSender();
    case NotificationType.sms:
      return SmsNotificationSender();
    case NotificationType.push:
      return PushNotificationSender();
    case NotificationType.inApp: ## 새로운 상태 추가됨
      return InAppNotificationSender();
  }
}
```

**Step 4: 클라이언트 코드**

이제 `NotificationType`에 따라 적절한 `NotificationSender` 객체를 가져와서 `send` 메서드를 호출합니다. 조건문이 없어지고, 각 알림 유형이 독립적으로 처리됩니다.

```dart
void main() {
  NotificationType type = NotificationType.email;
  String message = "Hello, this is a test notification.";

  NotificationSender sender = getNotificationSender(type);
  sender.send(message); // Output: Sending email: Hello, this is a test notification.
}
```

#### **Step 5: Step 3 좀 더 객체지향적으로 개선해보기**&#x20;

* Step 3에서 여전히 갖고 있는 문제점

1. **OCP 위반**
   * 새로운 `NotificationType`이 추가될 때마다 `switch-case`에 새로운 `case`를 추가해야 합니다. 이는 기존 코드를 수정해야 하는 구조이므로 \*\*개방-폐쇄 원칙(OCP)\*\*에 어긋납니다.
2. **높은 결합도**
   * `NotificationType`과 `NotificationSender` 클래스가 팩토리 메서드 내부에서 강하게 결합되어 있습니다. 이러한 결합은 확장성과 유지보수성을 낮춥니다.
3. **책임 과도**
   * 팩토리 메서드가 모든 타입별 객체 생성을 담당하고 있어 단일 책임 원칙(SRP)을 위반할 가능성이 있습니다.

#### &#x20; 리팩토링된 코드 : Map 기반 동적 팩토리&#x20;

```dart
// 동적 매핑 기반 팩토리
class NotificationSenderFactory {
  static final Map<NotificationType, NotificationSender> _senderMap = {
    NotificationType.email: EmailNotificationSender(),
    NotificationType.sms: SmsNotificationSender(),
    NotificationType.push: PushNotificationSender(),
    NotificationType.inApp: InAppNotificationSender(), // 새로운 상태 추가
  };

  // 객체 반환 메서드
  static NotificationSender getNotificationSender(NotificationType type) {
    return _senderMap[type] ??
        (throw Exception("NotificationSender not found for type: $type"));
  }
}
```

***

#### ##: 최종 사용 예시

```dart
void main() {
  NotificationType type = NotificationType.inApp;
  String message = "You have a new in-app notification.";

  // 팩토리에서 객체를 가져와 실행
  NotificationSender sender =
      NotificationSenderFactory.getNotificationSender(type);
  sender.send(message);
}
```

Step3과 비교해서 개선된 점:

* **OCP 준수**
  * 새로운 `NotificationType`이 추가될 경우, 기존 팩토리 메서드를 수정할 필요가 없습니다. \
    새로운 클래스와 맵핑만 추가하면 됩니다.
  * 기존 코드는 닫혀 있고, 확장만 가능하므로 **OCP**를 만족합니다.
* **결합도 감소**
  * `NotificationSenderFactory`와 각 `NotificationSender` 클래스 간의 결합이 느슨해졌습니다. 팩토리의 역할은 맵핑을 관리하는 것으로 제한되며, 객체 생성 책임은 각각의 클래스가 가집니다.
* **코드 가독성 향상**
  * `switch-case`가 제거되어 팩토리 메서드가 간결하고 명확합니다.
  * 맵핑으로 인해 상태와 객체 생성 간의 관계를 한눈에 확인할 수 있습니다.
* **테스트 용이성**
  * 팩토리 로직은 단순화되어 테스트가 용이하며, 각 `NotificationSender` 클래스는 독립적으로 테스트할 수 있습니다.

### 결론

* `switch-case`와 `if-else`는 상태별 분기를 처리할 때, 새로운 상태가 추가되면 기존 코드를 수정해야 하므로 OCP(개방-폐쇄 원칙)\*\*을 위반합니다. 그러나 **상태가 고정적이고 로직이 단순**할 경우 간결하고 빠른 해결책이 됩니다.
* **다형성**은 상태별 동작을 독립적으로 분리하여, 새로운 상태 추가 시 기존 코드를 수정하지 않고 확장할 수 있어 **OCP를 준수**합니다. 상태가 자주 변경되거나 확장이 필요한 경우에 유리하며, **유지보수성과 유연성**을 제공합니다.

따라서 상황에 따라 두 접근법의 복잡도와 유지보수성을 고려하여 선택하는 것이 중요합니다.

#### 다형성을 사용하면 좋은 경우

* **확장이 자주 발생하는 경우**:\
  새로운 상태(예: 알림 유형)가 자주 추가될 가능성이 있다면 다형성을 사용하는 것이 적합합니다.
* **복잡한 비즈니스 로직**:\
  상태별 동작이 복잡하거나 서로 다른 로직을 수행해야 한다면 다형성으로 분리하는 것이 유지보수에 유리합니다.
* **가독성과 재사용성을 중시하는 경우**:\
  코드가 길고 복잡해질 경우 다형성을 통해 단순화 및 재사용성을 높일 수 있습니다.

***

#### `if-else` 또는 `switch-case`가 더 적합한 경우

1. **상태가 고정적이고 적은 경우**:\
   상태가 2\~3개로 고정적이라면 다형성을 사용하는 것은 오히려 과한 설계인 것 같습니다.
2. **단순한 로직**:\
   각 상태의 처리 로직이 매우 간단한 경우 `if-else`나 `switch-case`가 더 직관적이고 효율적입니다.
3. **성능 우선**:\
   다형성은 객체 생성과 추가 메모리를 사용하므로, 성능이 중요한 경우 간단한 조건문이 더 나을 수 있습니다.

