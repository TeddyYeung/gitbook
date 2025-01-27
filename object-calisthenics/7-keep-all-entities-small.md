# 규칙 7 : 모든 엔티티를 작게 유지 (Keep All Entities Small)

클래스나 패키지가 너무 크면 **읽기 어렵고, 유지보수**가 힘들어집니다. \
따라서 아래와 같은 기준으로 관리하는 것이 중요합니다. &#x20;

* **Java 기준**: 클래스는 **50줄 이하**, 패키지는 **10개 파일 이하**로 유지.
* **Flutter 기준**: 클래스는 **300줄 이하**로 유지 가능. 응집력과 책임 분리를 고려해서 관리.

1. **클래스 크기**
   * **Java**: 50줄 이하
   * **Flutter**: 300줄 이하 (기능 분리 중요)
2. **패키지 크기**
   * **Java**: 10개 파일 이하
   * **Flutter**: 하나의 목적을 가진 패키지로 관리
3. **기능 분리**
   * UI와 로직 분리: `UI`와 `데이터 처리`를 별도로 관리

### **Flutter 코드 예시 : To-Do 앱** &#x20;

### **Before: 긴 클래스**

```
lib/
└── todo_manager.dart   
```

**`todo_manager.dart`**

```dart
class ToDoManager {
  final List<ToDo> _todos = [];

  void addToDo(String title) {
    _todos.add(ToDo(
      id: DateTime.now().toString(),
      title: title,
      isCompleted: false,
    ));
  }

  void toggleComplete(String id) {
    final index = _todos.indexWhere((todo) => todo.id == id);
    if (index != -1) {
      _todos[index].isCompleted = !_todos[index].isCompleted;
    }
  }

  void deleteToDo(String id) {
    _todos.removeWhere((todo) => todo.id == id);
  }

  List<ToDo> get todos => List.unmodifiable(_todos);
}

class ToDo {
  final String id;
  final String title;
  bool isCompleted;

  ToDo({
    required this.id,
    required this.title,
    this.isCompleted = false,
  });
}
```

**문제점**

1. **응집도 낮음**:
   * `ToDo` 데이터 모델과 관리 로직(`ToDoManager`)이 하나의 파일에 있어 역할이 혼재되어 있음.
2. **확장성 부족**:
   * 새로운 기능 추가 시 이 파일에 계속 코드를 추가해야 하므로 파일 크기가 커짐.
3. **재사용성 저하**:
   * `ToDo` 모델과 관련 로직을 다른 곳에서 사용하려면, 해당 파일 전체를 가져가야 하는 문제가 있음.

***

### **After: 작은 엔티티로 리팩터링된 코드**&#x20;

**구조를 다음과 같이 분리:**

<pre class="language-arduino"><code class="lang-arduino"><strong>lib/
</strong>├── models/
│   └── todo.dart          // 데이터 모델
├── services/
│   ├── todo_service.dart  // 상태 관리
│   └── todo_repository.dart // 데이터 처리 분리
</code></pre>

1. **`models/todo.dart`**

```dart
class ToDo {
  final String id;
  final String title;
  final bool isCompleted;

  ToDo({
    required this.id,
    required this.title,
    this.isCompleted = false,
  });

  ToDo copyWith({String? title, bool? isCompleted}) {
    return ToDo(
      id: id,
      title: title ?? this.title,
      isCompleted: isCompleted ?? this.isCompleted,
    );
  }
}
```

2. **`services/todo_repository.dart`**

```dart
import '../models/todo.dart';

class ToDoRepository {
  final List<ToDo> _todos = [];

  List<ToDo> fetchTodos() {
    return List.unmodifiable(_todos);
  }

  void saveToDo(ToDo todo) {
    _todos.add(todo);
  }

  void deleteToDo(String id) {
    _todos.removeWhere((todo) => todo.id == id);
  }
}
```

3. **`services/todo_service.dart`**

```dart
import '../models/todo.dart';
import 'todo_repository.dart';

class ToDoService {
  final ToDoRepository _repository = ToDoRepository();

  List<ToDo> get todos => _repository.fetchTodos();

  void addToDo(String title) {
    final newToDo = ToDo(id: DateTime.now().toString(), title: title);
    _repository.saveToDo(newToDo);
  }

  void toggleComplete(String id) {
    final todos = _repository.fetchTodos();
    final index = todos.indexWhere((todo) => todo.id == id);
    if (index != -1) {
      final updatedToDo = todos[index].copyWith(
        isCompleted: !todos[index].isCompleted,
      );
      _repository.saveToDo(updatedToDo);
    }
  }

  void deleteToDo(String id) {
    _repository.deleteToDo(id);
  }
}
```

***

#### **결과**

1. **짧고 단순한 클래스**:
   * 각 파일이 300 줄 이하로 유지됨.
   * 클래스별 역할이 분리되어 코드 가독성이 증가.
2. **응집력 있는 패키지**:
   * `models`, `services`로 분리하여 각 패키지가 명확한 목적을 가짐.
3. **유지보수 용이성**:
   * 새로운 기능 추가 시 관련 클래스만 수정하면 됨.
   * 독립적인 구조로 다른 프로젝트에서도 재사용 가능.
