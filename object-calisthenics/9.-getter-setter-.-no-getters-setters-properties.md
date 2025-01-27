# 규칙 9. getter/setter/프로퍼티를 사용하지 않는다. (No Getters/Setters/Properties)

규칙 9는 **TDA (Tell, Don't Ask)** 원칙과 관련이 있습니다. \
TDA 원칙은 객체의 상태를 외부에서 직접 조회하거나 변경하지 않고, 객체에 "무엇을 할지"만 요청하는 방식으로 설계를 권장합니다. \
즉, 객체가 자신의 상태를 스스로 처리하도록 맡기고, 외부에서는 객체에게 작업을 "tell"하고, \
객체가 내부적으로 이를 처리하도록 해야 합니다.

### Before: **잘못된 예시 (Getter/Setter 사용 및 외부에서 점수 변경)**

```dart
class Game {
  int _score = 0;

  // Getter와 Setter 사용
  int get score => _score;
  set score(int value) => _score = value;
}

void main() {
  var game = Game();

  // 외부에서 점수를 조회하고 변경
  game.score = game.score + 10;  // 외부에서 점수 직접 변경
  
  // 외부에서 조건 처리
  if (game.score > 100) {
    print('You won!');
  } else {
    print('Keep playing!');
  }
  
  print(game.score);
}
```

**문제점:**

* **점수 변경 책임이 외부에 있음**: `score`를 외부에서 직접 조회하고 변경하고 있습니다. `Game` 클래스는 점수 변경에 대한 책임을 가지지 않으며, 모든 로직이 외부에서 처리되고 있습니다.
* **조건 비교 외부 처리**: 점수에 따른 조건 비교도 외부에서 이루어집니다. 객체가 자신의 상태를 관리하는 것이 아니라 외부에서 상태를 변경하고 처리하고 있습니다.

***

### After:  리팩토링된 코드  (Getter/Setter 제거 및 메서드로 처리)

```dart
class Game {
  int _score = 0;

  // 점수 변경 로직을 내부로 이동
  void addScore(int delta) {
    _score += delta;
  }

  // 점수 조회 메서드 제공
  int getScore() => _score;

  // 게임 상태를 내부에서 처리
  String checkGameStatus() {
    if (_score > 100) {
      return 'You won!';
    } else {
      return 'Keep playing!';
    }
  }
}

void main() {
  var game = Game();

  // 점수 변경을 외부에서 처리하지 않고 메서드를 호출
  game.addScore(10);  // 점수 증가
  print(game.getScore());  // 점수 출력
  
  // 게임 상태를 내부에서 처리
  print(game.checkGameStatus());  // "Keep playing!" 출력
}
```

**개선점:**

* **상태 관리 책임 이동**: `addScore()` 메서드에서 점수 변경 로직을 처리하고, 게임 상태 체크는 `checkGameStatus()` 메서드에서 처리합니다. 이제 `Game` 객체가 자신의 상태를 관리하고 있습니다.
* **불필요한 Getter/Setter 제거**: `score`에 대한 직접적인 접근을 막고, 점수 변경은 `addScore()` 메서드를 통해 관리됩니다. 객체의 상태가 외부에서 직접적으로 변경되지 않도록 했습니다.
* **조건 처리 내부화**: 점수에 따른 게임 상태는 `Game` 객체 내부에서 처리하고, 외부에서는 단순히 결과만 요청합니다. `Game` 객체가 모든 책임을 내부에서 처리합니다.
* **Open/Closed Principle**: 점수 관리 로직이나 조건을 변경해야 할 경우 `addScore()` 또는 `checkGameStatus()`메서드만 수정하면 되며, 외부 코드의 수정 없이 확장 가능합니다.

### **결론**

#### 객체의 상태 변경과 관련된 로직은 객체 내부에서 처리해야 하며, 외부에서 직접 접근하거나 변경하는 것을 피하는 방식으로 설계해야 합니다.

***

