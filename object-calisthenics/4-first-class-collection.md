# 규칙 4: 일급 콜렉션 사용 (First Class Collection)

"일급 컬렉션"이란 컬렉션(리스트, 맵 등)을 감싸는 클래스를 만들어, 컬렉션과 관련된 로직을 캡슐화하는 원칙입니다. 이를 통해 컬렉션의 불필요한 외부 노출을 줄이고, 도메인 로직과 데이터를 더 잘 구조화할 수 있습니다.

***

#### Flutter 예제:

사용자 목록(`List<User>`)을 단순히 다루는 대신, `UserList`라는 클래스로 감싸는 예제를 만들어 보겠습니다.

***

**1️⃣ Before: 일반 컬렉션 사용**

```dart
class User {
  final String name;

  User(this.name);
}

class Group {
  final List<User> users;

  Group(this.users);

  void addUser(User user) {
    users.add(user);
  }

  bool hasUser(String name) {
    return users.any((user) => user.name == name);
  }
}

void main() {
  final group = Group([]);
  group.addUser(User("Alice"));
  group.addUser(User("Bob"));

  print(group.hasUser("Alice")); // true
  print(group.hasUser("Charlie")); // false
}
```

**문제점:**

* 컬렉션(`List<User>`)이 외부에 직접 노출되어, **직접적인 수정**이 가능.
* 컬렉션과 관련된 로직이 그룹 로직과 섞여 유지보수가 어려움.
* 중복 로직이 발생할 가능성.

***

**2️⃣ After: 일급 컬렉션 사용**

```dart
class User {
  final String name;

  User(this.name);
}

class UserList {
  final List<User> _users = [];

  void add(User user) {
    _users.add(user);
  }

  bool contains(String name) {
    return _users.any((user) => user.name == name);
  }

  @override
  String toString() {
    return _users.map((user) => user.name).join(", ");
  }
}

class Group {
  final UserList users;

  Group(this.users);

  void printUsers() {
    print("Users in group: $users");
  }
}

void main() {
  final group = Group(UserList());
  group.users.add(User("Alice"));
  group.users.add(User("Bob"));

  print(group.users.contains("Alice")); // true
  print(group.users.contains("Charlie")); // false

  group.printUsers(); // Users in group: Alice, Bob
}
```

***

#### **왜 사용해야 하나?**

1. **로직 캡슐화**: 컬렉션 관련 로직(`add`, `contains`)이 `UserList` 클래스에 집중되어 유지보수성이 높아짐.
2. **컬렉션 보호**: 내부 `_users`를 외부에서 직접 수정하지 못하도록 보호.
3. **의미 강화**: 단순한 `List<User>`가 아닌 `UserList`라는 타입으로 의도를 명확히 전달.
4. **중복 방지**: 컬렉션 로직이 한곳에 모여, 코드 중복 제거.

***

#### **장점 요약**

* **가독성**: 코드가 의도와 구조적으로 더 명확해짐.
* **확장성**: 컬렉션 관련 로직(정렬, 필터 등) 추가가 쉬움.
* **안정성**: 불필요한 직접 수정 방지로 예기치 못한 에러 감소.
