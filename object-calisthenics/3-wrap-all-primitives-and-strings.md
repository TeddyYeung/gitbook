# 규칙 3 : 원시값과 문자열의 포장 (Wrap All Primitives And Strings)

객체 지향 프로그래밍에서 원시값(Primitive)이나 문자열(String)을 별도 클래스로 감싸 의미를 명확히 하고, 관련 로직을 캡슐화하는 원칙입니다. 이를 통해 **가독성**, **유지보수성**, **타입 안정성**을 높일 수 있습니다.

***

**Before: 원시값 사용**

```dart
class User {
  final String email;

  User(this.email);

  void sendWelcomeEmail() {
    if (!email.contains('@')) {
      throw Exception('Invalid email format');
    }
    print('Welcome email sent to $email');
  }
}

void main() {
  final user = User('example@domain.com');
  user.sendWelcomeEmail(); // 정상 동작

  final invalidUser = User('example.com');
  invalidUser.sendWelcomeEmail(); // 예외 발생
}
```

**문제점:**

* 이메일 검증 로직이 `User` 클래스에 흩어져 있음.
* 단순 문자열로 표현되어 잘못된 데이터 사용 가능.
* 이메일 관련 로직이 중복될 가능성.

***

**After: 원시값 포장**

```dart
class Email {
  final String value;

  Email(this.value) {
    if (!value.contains('@')) {
      throw Exception('Invalid email format');
    }
  }

  @override
  String toString() => value;
}

class User {
  final Email email;

  User(this.email);

  void sendWelcomeEmail() {
    print('Welcome email sent to ${email}');
  }
}

void main() {
  final user = User(Email('example@domain.com'));
  user.sendWelcomeEmail(); // 정상 동작

  // final invalidUser = User(Email('example.com')); // 예외 발생
}
```

***

#### **왜 사용해야 하나?**

1. **도메인 로직 캡슐화**: 이메일 검증 로직을 `Email` 클래스에 집중시켜 중복 제거.
2. **타입 안정성 강화**: 문자열 대신 `Email` 타입을 사용해 실수를 방지.
3. **확장성**: 이메일 관련 로직 추가(예: `obfuscate()`)가 용이.
4. **가독성 향상**: 타입 이름만으로도 의도를 명확히 전달.

***

#### **장점 요약**

* **명확성**: 코드의 의미가 더 직관적.
* **유지보수성**: 관련 로직이 한곳에 모여 관리가 쉬움.
* **버그 감소**: 원시값 사용에서 오는 실수를 예방.
