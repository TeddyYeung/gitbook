---
description: >-
  "일급 컬렉션"이란 컬렉션(리스트, 맵 등)을 별도의 클래스로 감싸, 컬렉션과 관련된 비즈니스 로직을 한 곳에 캡슐화하는 원칙입니다. 이를
  통해 컬렉션의 불필요한 외부 노출을 방지하고, 데이터를 구조화하며, 도메인 규칙을 명확히 표현할 수 있습니다.
---

# 규칙 4: 일급 콜렉션 사용 (First Class Collection)

#### Flutter 예제: `Order`와 `OrderItem`을 활용한 일급 컬렉션

***

### &#x20;**Before: 일반 컬렉션 사용**

```dart
class OrderItem {
  final String name;
  final int quantity;
  final double price;

  OrderItem(this.name, this.quantity, this.price);
}

// 문제점: Order 클래스가 컬렉션 관리와 비즈니스 로직 모두를 처리
class Order {
  final List<OrderItem> items; // 컬렉션 직접 노출 (문제 발생 가능)

  Order(this.items);

  // 컬렉션 관리 로직
  void addItem(OrderItem item) {
    items.add(item); // 컬렉션 관리 책임을 Order 클래스가 담당
  }

  // 비즈니스 로직: 총 가격 계산
  double calculateTotalPrice() {
    return items.fold(0, (total, item) => total + (item.price * item.quantity));
  }
}

void main() {
  final order = Order([]);
  
  // 컬렉션 관리 로직을 직접 호출
  order.addItem(OrderItem("Apple", 3, 2.5));
  order.addItem(OrderItem("Banana", 5, 1.2));

  // 비즈니스 로직 호출
  print("Total Price: ${order.calculateTotalPrice()}"); // Total Price: 13.0
}

```

### **문제점**&#x20;

1. &#x20;**컬렉션 관리와 비즈니스 로직의 혼재**

`Order` 클래스는 본래 **"주문"이라는 도메인을 표현**해야 합니다. 하지만 현재 코드는 다음과 같은 문제가 있습니다:

* `addItem`은 컬렉션 관리 로직으로, `List<OrderItem>`과 관련된 책임.
  * 이 로직은 컬렉션 자체의 관리에 국한되므로, 별도의 컬렉션 관리 클래스에서 처리해야 더 구조적입니다.
* `calculateTotalPrice`는 주문의 총 가격을 계산하는 **비즈니스 로직**으로, 도메인에 대한 책임을 다룹니다.
  * 하지만 현재는 컬렉션과 직접 상호작용하기 때문에 컬렉션 관리와 얽혀 있습니다.

**결과적으로**:

* `Order` 클래스가 단일 책임 원칙(SRP)을 위반하며, 역할이 혼란스러워집니다.
* 비즈니스 로직이 복잡해지면 유지보수가 어려워지고, 코드가 점점 더 비효율적으로 변할 수 있습니다.

**2. 컬렉션 외부 노출: 무분별한 수정 가능**

```dart
final List<OrderItem> items;
```

`List<OrderItem>`이 `Order` 클래스의 `items` 속성으로 **직접 노출**되고 있기 때문에, 외부에서 다음과 같은 작업이 가능합니다:

```dart
void main() {
  final order = Order([]);
  order.items.add(OrderItem("Apple", 3, 2.5)); // 외부에서 직접 추가
  order.items.removeAt(0); // 외부에서 직접 삭제
  order.items.clear(); // 컬렉션을 완전히 비움

  print(order.items); // 비어 있음
}
```

**문제점**:

* 외부에서 `items` 컬렉션에 **직접 접근**하여 요소를 추가, 삭제, 수정할 수 있습니다.
* 이로 인해 `Order` 클래스 내부에서 관리해야 할 **컬렉션 상태**가 **외부로 유출**됩니다.
* 예기치 않은 변경이 발생하면, `Order` 클래스의 의도된 동작이 깨지고, 데이터가 비정상적으로 처리될 위험이 있습니다.

**3. 불변성 보장 부족**

`final` 키워드는 **객체의 재할당을 방지**하지만, 객체의 **내부 상태 변경**을 막지 않습니다. 즉, `final List<OrderItem> items`는 한 번만 할당할 수 있지만, **리스트 내부의 요소**는 변경할 수 있습니다.

**문제 예시:**

```dart
void main() {
  final order = Order([]);

  // 외부에서 컬렉션에 요소 추가
  order.items.add(OrderItem("Apple", 3, 2.5));
  
  // 외부에서 컬렉션의 요소를 수정
  order.items[0] = OrderItem("Banana", 1, 1.0);
  
  print(order.items); // [OrderItem("Banana", 1, 1.0)]
}
```

**문제점:**

* `final`로 선언된 `items`는 **재할당을 방지**하지만, 리스트 내부에 저장된 **아이템의 수정**이나 **리스트의 내용 변경**은 여전히 가능합니다.
* 외부에서 리스트의 요소를 수정하거나, 컬렉션을 변경할 수 있어 **불변성**을 보장할 수 없습니다.
* 이로 인해 `Order` 클래스의 데이터 무결성이 **손상될** 수 있으며, 비즈니스 로직에서 예기치 않은 동작을 유발할 수 있습니다.

***

### &#x20;**After: 일급 컬렉션 사용**

```dart
class OrderItem {
  final String name;
  final int quantity;
  final double price;

  OrderItem(this.name, this.quantity, this.price);
}

/// 일급 컬렉션: OrderItems
/// - 컬렉션 관련 로직을 캡슐화하고, 컬렉션 외부 노출을 방지.
/// - 불변성을 보장하기 위해 내부 컬렉션은 private으로 관리하고, 읽기 전용 리스트를 외부에 제공.
class OrderItems {
  final List<OrderItem> _items = []; // 내부 컬렉션은 외부에 직접 노출되지 않음

  /// 컬렉션에 아이템을 추가
  void add(OrderItem item) {
    _items.add(item); // 외부에서는 이 메서드를 통해서만 컬렉션 변경 가능
  }

  /// 총 가격 계산 (비즈니스 로직)
  double get totalPrice {
    return _items.fold(0, (total, item) => total + (item.price * item.quantity));
  }

  /// 아이템 개수 반환
  int get itemCount => _items.length;

  /// 읽기 전용 리스트 제공 (불변성 보장)
  List<OrderItem> get items => List.unmodifiable(_items);

  @override
  String toString() {
    return _items.map((item) => "${item.name} (${item.quantity}x)").join(", ");
  }
}

/// Order 클래스
/// - OrderItems를 통해 컬렉션을 관리.
/// - Order는 도메인 로직에만 집중 (단일 책임 원칙 준수).
class Order {
  final OrderItems orderItems;

  Order(this.orderItems);

  /// 주문 요약 출력 (비즈니스 로직)
  void printSummary() {
    print("Order Summary: $orderItems"); // OrderItems의 toString() 활용
    print("Total Items: ${orderItems.itemCount}");
    print("Total Price: ${orderItems.totalPrice}");
  }
}

void main() {
  // Order와 OrderItems 사용
  final order = Order(OrderItems());

  // OrderItems를 통해 컬렉션 관리
  order.orderItems.add(OrderItem("Apple", 3, 2.5));
  order.orderItems.add(OrderItem("Banana", 5, 1.2));

  // 주문 요약 출력
  order.printSummary();
  // Order Summary: Apple (3x), Banana (5x)
  // Total Items: 2
  // Total Price: 13.0
}

```

***

### 개선된 점&#x20;

**1. 비즈니스 종속적 로직 구조 개선**

* `add`, `totalPrice`, `itemCount` 등 **컬렉션에 종속되었던 비즈니스 로직**이 `OrderItems` 클래스로 이동하여 구조가 명확해짐.
* `Order` 클래스는 오직 주문과 관련된 상위 도메인 로직만 다루며, 역할이 분리됨.

**2. 불변성 보장**

* **일급 컬렉션**을 사용하여, 컬렉션을 **내부에 감추고**, 외부에서는 읽기 전용으로만 접근할 수 있도록 합니다.
* `List.unmodifiable()`을 사용하여 컬렉션을 **불변**으로 만들어, 외부에서 컬렉션을 수정할 수 없게 차단합니다.

```dart
class OrderItems {
  final List<OrderItem> _items = []; // 내부 컬렉션은 외부에 직접 노출되지 않음

  void add(OrderItem item) {
    _items.add(item); // 외부에서는 이 메서드를 통해서만 컬렉션 변경 가능
  }

  void remove(OrderItem item) {
    _items.remove(item); // 외부에서는 이 메서드를 통해서만 컬렉션 변경 가능
  }

  void clear() {
    _items.clear(); // 컬렉션의 모든 아이템을 제거
  }

  void update(OrderItem oldItem, OrderItem newItem) {
    final index = _items.indexOf(oldItem);
    if (index != -1) {
      _items[index] = newItem; // 해당 아이템을 새로운 아이템으로 대체
    }
  }

  /// 아이템 개수 반환 (불변성 보장)
  int get itemCount => _items.length;

  /// 읽기 전용 리스트 제공 (불변성 보장)
  List<OrderItem> get items => List.unmodifiable(_items);

  @override
  String toString() {
    return _items.map((item) => "${item.name} (${item.quantity}x)").join(", ");
  }
}

// 외부에서의 호출 
void main() {
  // 1. Order와 OrderItems 객체 생성
  final order = Order(OrderItems());

  // 2. OrderItems를 통해 컬렉션 관리
  order.orderItems.add(OrderItem("Apple", 3, 2.5));
  order.orderItems.add(OrderItem("Banana", 5, 1.2));

  // 3. 주문 요약 출력
  order.printSummary();

  // 4. 불변 리스트 조회 가능 (외부에서 리스트 수정 불가)
  final items = order.orderItems.items;
  print(items);  // Apple (3x), Banana (5x)

  // 5. 아이템 제거
  order.orderItems.remove(OrderItem("Apple", 3, 2.5));  // "Apple" 아이템 제거
  print(order.orderItems.items);  // [Banana (5x)]

  // 6. 아이템 업데이트 (예: "Banana"의 수량을 7로 변경)
  final oldItem = OrderItem("Banana", 5, 1.2);
  final newItem = OrderItem("Banana", 7, 1.2);
  order.orderItems.update(oldItem, newItem);
  print(order.orderItems.items);  // [Banana (7x)]

  // 7. 컬렉션 전체 지우기
  order.orderItems.clear();
  print(order.orderItems.items);  // []
  
  // 8. 외부에서 Order 클래스에서 직접 컬렉션 수정 불가
  // order.items.add(OrderItem("Orange", 2, 1.5)); // 오류 발생: 'add'가 정의되지 않음
}
```

* **`OrderItems`**:
  * `OrderItems`는 컬렉션 관리의 책임을 맡고 있습니다. `add`, `remove`, `clear`, `update` 메서드를 통해 데이터를 수정할 수 있습니다.
  * `items` getter는 `List.unmodifiable`을 사용하여, 외부에서는 읽기만 할 수 있도록 합니다.
* **`Order`**:
  * `Order` 클래스는 **ViewModel** 역할을 합니다. `OrderItems` 객체를 포함하고 있으며, **컬렉션의 수정은 불가능**합니다. 외부에서는 `Order` 객체의 `items`를 통해 읽을 수 있지만, 컬렉션을 수정하려면 `OrderItems`를 통해 직접 수정해야 합니다.
  * `order.items.add()`와 같은 **수정**은 `Order` 클래스에서 직접 호출할 수 없습니다. `order.items`는 읽기 전용이기 때문에 수정이 불가능합니다.

**3. 상태와 행위의 캡슐화**

* **유지보수 용이**: `add()`, `totalPrice`, `itemCount`, `clear()와 같이`컬렉션을 수정할 수 있는 메서드들이 일관되게 1개의 클래스 내에 존재하므로, 변경이 필요할 때 해당 메서드만 수정하면 됩니다.
* **상태와 행위의 응집성**: 컬렉션의 상태와 행위를 한 곳에서 처리함으로써 코드의 응집성을 높이고, 외부에서 불필요하게 수정할 수 없게 만들어 안전성을 보장합니다.

```dart
class OrderItems {
  final List<OrderItem> _items = [];

  void add(OrderItem item) {
    _items.add(item);
  }

  double get totalPrice {
    return _items.fold(0, (total, item) => total + (item.price * item.quantity));
  }

  int get itemCount => _items.length;

  void clear() {
    _items.clear(); 
  }
}
```

**4. 명확한 컬렉션 객체 이름**

* 단순한 `List<OrderItem>` 대신 `OrderItems`라는 클래스로 의도를 명확히 표현.
* 코드 읽기만으로도 "이것이 주문 아이템 목록"임을 쉽게 이해 가능.

**5. 확장성**

*   컬렉션 로직을 `OrderItems` 클래스에 집중시키면, **추가적인 기능**을 쉽게 확장할 수 있습니다. \
    예를 들어, **정렬**이나 **중복 방지** 로직을 추가할 수 있습니다.

    <pre class="language-dart"><code class="lang-dart">class OrderItems {
      final List&#x3C;OrderItem> _items = [];

      void add(OrderItem item) {
        _items.add(item);
      }

      void sortByPrice() {
        _items.sort((a, b) => a.price.compareTo(b.price)); // 가격 기준으로 정렬 로직 추가 
      }

      // 중복 방지 로직 추가 
      bool containsDuplicate() {
        var seen = &#x3C;String>{};
        for (var item in _items) {
          if (seen.contains(item.name)) return true;
          seen.add(item.name);
        }
    <strong>    return false;
    </strong>  }

      double get totalPrice {
        return _items.fold(0, (total, item) => total + (item.price * item.quantity));
      }

      int get itemCount => _items.length;

      List&#x3C;OrderItem> get items => List.unmodifiable(_items);
    }
    </code></pre>


* 이렇게 로직을 캡슐화하면 **새로운 기능을 추가**할 때, `OrderItems` 클래스 내에서만 작업하면 되므로\
  &#x20;다른 코드에 영향을 주지 않으며, 유지보수가 쉬워집니다.

## 번외 : 만약 객체복사 방식으로 구현한다면&#x20;

```dart
import 'package:equatable/equatable.dart';

class OrderItems {
  final List<OrderItem> _items;

  /// 생성자: 내부 리스트 초기화 (기본값은 빈 리스트)
  OrderItems([List<OrderItem>? items]) : _items = List.unmodifiable(items ?? []);

  /// 새로운 아이템 추가
  OrderItems add(OrderItem item) => OrderItems([..._items, item]);

  /// 특정 아이템 제거
  OrderItems remove(OrderItem item) =>
      OrderItems(_items.where((i) => i != item).toList());

  /// 특정 아이템 업데이트
  OrderItems update(OrderItem oldItem, OrderItem newItem) => OrderItems(
      _items.map((item) => item == oldItem ? newItem : item).toList());

  /// 전체 아이템 제거 - 새 빈 객체로 반환 
  OrderItems clear() => OrderItems();

  /// 읽기 전용 리스트 제공
  List<OrderItem> get items => _items;

  /// 아이템 개수
  int get itemCount => _items.length;

  @override
  String toString() =>
      _items.map((item) => "${item.name} (${item.quantity}x)").join(", ");
}

class Order {
  final OrderItems orderItems;

  const Order(this.orderItems);

  /// 새로운 Order 생성
  Order copyWith(OrderItems newOrderItems) => Order(newOrderItems);

  /// 주문 요약 출력
  void printSummary() {
    print("Order Summary: $orderItems");
  }
}

class OrderItem extends Equatable {
  final String name;
  final int quantity;
  final double price;

  const OrderItem(this.name, this.quantity, this.price);

  @override
  List<Object?> get props => [name, quantity, price];

  @override
  String toString() => "$name ($quantity x)";
}

// 외부에서의 호출
void main() {
  // 1. Order와 OrderItems 생성
  var order = Order(OrderItems());

  // 2. 아이템 추가 (새로운 Order 생성)
  order = order.copyWith(order.orderItems.add(const OrderItem("Apple", 3, 2.5)));
  order = order.copyWith(order.orderItems.add(const OrderItem("Banana", 5, 1.2)));

  // 3. 주문 요약 출력
  order.printSummary(); // Order Summary: Apple (3x), Banana (5x)

  // 4. 아이템 제거
  order = order.copyWith(
      order.orderItems.remove(const OrderItem("Apple", 3, 2.5)));
  print(order.orderItems.items); // [Banana (5x)]

  // 5. 아이템 업데이트
  order = order.copyWith(order.orderItems.update(
    const OrderItem("Banana", 5, 1.2),
    const OrderItem("Banana", 7, 1.2),
  ));
  print(order.orderItems.items); // [Banana (7x)]

  // 6. 전체 아이템 제거
  order = order.copyWith(order.orderItems.clear());
  print(order.orderItems.items); // []
}
```

#### 주요 변경점:

1. **`OrderItem` Equatable 적용**:
   * `props`를 통해 객체 비교를 간소화했습니다.
   * 이제 같은 값의 `OrderItem` 객체는 `==`로 바로 비교 가능합니다.
2. **간결한 메서드 체이닝**:
   * `add`, `remove`, `update`, `clear` 모두 새로운 `OrderItems` 객체를 반환하도록 구현.
   * 불필요한 `final` 변수를 사용하지 않고, 체이닝이 간단해졌습니다.
3. **Const 사용**:
   * `OrderItem`에 `const`를 활용해 객체 생성 시 성능 최적화.
