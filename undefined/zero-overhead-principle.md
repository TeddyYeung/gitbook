# \[개발 원칙] 제로 오버헤드 원칙(Zero Overhead Principle)

제로 오버헤드 원칙(Zero Overhead Principle)는 C++ 같은 고성능 언어에서 주로 언급되는 개념으로, \
프로그래밍 언어의 추상화가 성능에 불필요한 비용(오버헤드)을 초래하지 않아야 한다는 원칙입니다. \
즉, **사용자가 특정 기능을 사용하지 않을 때 비용이 없어야 한다**는 것이 핵심입니다.

Dart나 Flutter에서도 이 원칙을 적용할 수 있습니다. \
주로 성능 최적화, 메모리 효율성, 빌드 효율성을 고려한 코드 작성에 관련됩니다. 다음은 Flutter에서의 예시입니다.

### 오버헤드  개선 사례 **1. 리스트 초기화에서 불필요한 동적 메모리 할당 제거**

**Before (동적 리스트 확장으로 오버헤드 발생)**

```dart
List<int> createList(int size) {
  List<int> list = []; // 크기 미지정 리스트
  for (int i = 0; i < size; i++) {
    list.add(i); // 리스트 크기 확장 오버헤드 발생
  }
  return list;
}
```

**문제**

* 리스트가 동적으로 확장되면서 메모리 재할당과 복사가 발생하므로 성능 오버헤드가 있습니다.

***

**After (고정 크기 리스트 할당)**

```dart
List<int> createList(int size) {
  List<int> list = List.filled(size, 0, growable: false);
  for (int i = 0; i < size; i++) {
    list[i] = i;
  }
  return list;
}
```

**개선**

* 리스트 크기를 미리 할당하여 동적 메모리 재할당과 복사를 방지했습니다. 이는 불필요한 오버헤드를 제거합니다.

***

### 오버헤드  개선 사례 **2. 불필요한 객체 생성 제거**

**Before (객체 재사용 없이 매번 새로 생성)**

```dart
class ExpensiveObject {
  ExpensiveObject() {
    print('객체 생성 비용 발생');
  }
}

void main() {
  for (int i = 0; i < 5; i++) {
    ExpensiveObject(); // 매번 새 객체 생성
  }
}
```

**문제**

* 동일한 객체가 여러 번 생성되면서 불필요한 메모리 및 처리 비용이 발생합니다.

***

**After (객체 재사용)**

```dart
class ExpensiveObject {
  static final ExpensiveObject _instance = ExpensiveObject._internal();

  ExpensiveObject._internal() {
    print('객체 생성 비용 발생');
  }

  factory ExpensiveObject() => _instance;
}

void main() {
  for (int i = 0; i < 5; i++) {
    ExpensiveObject(); // 동일 객체 재사용
  }
}
```

**개선**

* 싱글톤 패턴을 적용하여 객체 생성 비용을 한 번으로 제한했습니다.

***

### 오버헤드  개선 사례 **3. 빌드 최적화 (위젯 재빌드 방지)**

**Before (비효율적 빌드 구조)**

```dart
class CounterWidget extends StatefulWidget {
  @override
  _CounterWidgetState createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    print("CounterWidget 빌드됨");
    return Scaffold(
      appBar: AppBar(title: Text('Counter Example')),
      body: Column(
        children: [
          ElevatedButton(
            onPressed: () => setState(() {
              _counter++;
            }),
            child: Text('Increment'),
          ),
          Text('Count: $_counter'),
        ],
      ),
    );
  }
}
```

**문제**

* `build()` 메소드가 불필요하게 호출되며 모든 위젯이 재빌드됩니다.

***

**After (위젯 빌드 최적화)**

```dart
class CounterWidget extends StatefulWidget {
  @override
  _CounterWidgetState createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _counter = 0;

  @override
  Widget build(BuildContext context) {
    print("CounterWidget 빌드됨");
    return Scaffold(
      appBar: AppBar(title: Text('Counter Example')),
      body: Column(
        children: [
          ElevatedButton(
            onPressed: () => setState(() {
              _counter++;
            }),
            child: Text('Increment'),
          ),
          _CounterText(_counter), // 분리된 Stateless 위젯 사용
        ],
      ),
    );
  }
}

class _CounterText extends StatelessWidget {
  final int counter;

  _CounterText(this.counter);

  @override
  Widget build(BuildContext context) {
    print("CounterText 빌드됨");
    return Text('Count: $counter');
  }
}
```

**개선**

* 텍스트 위젯을 별도의 `StatelessWidget`으로 분리하여 위젯 빌드를 최적화했습니다. 버튼 클릭 시 텍스트 위젯만 빌드되도록 개선하여 불필요한 빌드 오버헤드를 제거했습니다.
