# 규칙 4: 일급 콜렉션 사용 (First Class Collection)

"일급 컬렉션"이란 컬렉션(리스트, 맵 등)을 별도의 클래스로 감싸, 컬렉션과 관련된 **비즈니스 로직**을 한 곳에 캡슐화하는 원칙입니다. 이를 통해 컬렉션의 **불필요한 외부 노출을 방지**하고, 데이터를 구조화하며, **도메인 규칙을 명확히 표현**할 수 있습니다.

***

#### Flutter 예제: `Order`와 `OrderItem`을 활용한 일급 컬렉션

**1️⃣ Before: 일반 컬렉션 사용**

```dart
class OrderItem {
  final String name;
  final int quantity;
  final double price;

  OrderItem(this.name, this.quantity, this.price);
}

class Order {
  final List<OrderItem> items;

  Order(this.items);

  void addItem(OrderItem item) {
    items.add(item);
  }

  double calculateTotalPrice() {
    return items.fold(0, (total, item) => total + (item.price * item.quantity));
  }
}

void main() {
  final order = Order([]);
  order.addItem(OrderItem("Apple", 3, 2.5));
  order.addItem(OrderItem("Banana", 5, 1.2));

  print("Total Price: ${order.calculateTotalPrice()}"); // Total Price: 13.0
}
```

문제점 \


**1. 비즈니스 종속적 로직 분산: 컬렉션 로직과 혼재**

`Order` 클래스는 본래 **"주문"이라는 도메인을 표현**해야 합니다. 하지만 현재 코드는 다음과 같은 문제가 있습니다:

* **컬렉션 관리 로직**(예: `addItem`)과 **도메인 비즈니스 로직**(예: `calculateTotalPrice`)이 `Order` 클래스에 혼재되어 있음.
* `addItem`은 컬렉션의 관리와 관련된 로직으로, 이는 `List<OrderItem>` 컬렉션 자체와 밀접한 연관이 있음.\
  이 로직은 `Order` 클래스가 아니라 컬렉션 관리에 특화된 별도 클래스에서 담당해야 유지보수가 용이합니다.
* `calculateTotalPrice`는 도메인 로직이지만, 컬렉션과 직접 상호작용하기 때문에 컬렉션 로직과 밀접히 얽혀 있음.\
  이는 컬렉션 관리 책임과 도메인 책임이 분리되지 않았음을 나타냅니다.

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

**3. 불변성 보장 부족: 데이터 무결성 깨질 위험**

`List<OrderItem>`이 외부에 노출되고, 내부적으로 가변적인 컬렉션(List)을 사용하고 있기 때문에 **데이터 무결성**문제가 발생할 수 있습니다:

* 외부에서 컬렉션을 **변경 가능한 상태로 접근**하므로, `Order` 클래스는 데이터 무결성을 보장할 수 없습니다.
*   예를 들어:

    ```dart
    final order = Order([]);
    order.items.add(OrderItem("Apple", 3, 2.5)); // 정상 추가
    order.items[0] = OrderItem("Banana", 1, 1.0); // 외부에서 수정
    print(order.items); // [OrderItem("Banana", 1, 1.0)]
    ```

    외부에서 직접 요소를 수정하거나, 컬렉션을 비워버리는 작업이 가능하며, 이는 `Order` 클래스가 원하지 않는 상태로 변할 수 있음을 의미합니다.
*   또한, 이러한 무분별한 변경은 `calculateTotalPrice` 같은 메서드가 의도한 대로 작동하지 않을 가능성을 증가시킵니다. 예를 들어:

    ```dart
    final order = Order([]);
    order.addItem(OrderItem("Apple", 3, 2.5)); 
    order.items.clear(); // 컬렉션 비움
    print(order.calculateTotalPrice()); // 결과: 0.0 (예기치 않은 상태)
    ```



***

**2️⃣ After: 일급 컬렉션 사용**

```dart
class OrderItem {
  final String name;
  final int quantity;
  final double price;

  OrderItem(this.name, this.quantity, this.price);
}

class OrderItems {
  final List<OrderItem> _items = [];

  void add(OrderItem item) {
    _items.add(item);
  }

  double get totalPrice {
    return _items.fold(0, (total, item) => total + (item.price * item.quantity));
  }

  int get itemCount => _items.length;

  List<OrderItem> get items => List.unmodifiable(_items);

  @override
  String toString() {
    return _items.map((item) => "${item.name} (${item.quantity}x)").join(", ");
  }
}

class Order {
  final OrderItems orderItems;

  Order(this.orderItems);

  void printSummary() {
    print("Order Summary: $orderItems");
    print("Total Items: ${orderItems.itemCount}");
    print("Total Price: ${orderItems.totalPrice}");
  }
}

void main() {
  final order = Order(OrderItems());
  order.orderItems.add(OrderItem("Apple", 3, 2.5));
  order.orderItems.add(OrderItem("Banana", 5, 1.2));

  order.printSummary();
  // Order Summary: Apple (3x), Banana (5x)
  // Total Items: 2
  // Total Price: 13.0
}
```

***

#### 개선된 부분과 장점

**1. 비즈니스 종속적 로직 구조 개선**

* `add`, `totalPrice`, `itemCount` 등 **컬렉션에 종속된 비즈니스 로직**이 `OrderItems` 클래스로 이동하여 구조가 명확해짐.
* `Order` 클래스는 오직 주문과 관련된 상위 도메인 로직만 다루며, 역할이 분리됨.

**2. 불변성 보장**

일급 컬렉션은 컬렉션의 불변성(immutable)을 보장하기 위해 \
단순히 `final` 키워드만 사용하는 것이 아니라 **캡슐화**를 통해 값을 안전하게 보호합니다.

* `final`은 변수의 **재할당만 금지**할 뿐, 객체 내부의 상태(컬렉션의 요소 추가, 삭제 등)는 변경될 수 있습니다.
* 일급 컬렉션에서는 컬렉션(`List<OrderItem>`)을 내부에 감추고, 외부에 노출하지 않습니다.
* `List.unmodifiable`을 사용해 읽기 전용 컬렉션으로 제공함으로써, 외부에서 컬렉션을 수정하려는 모든 시도를 차단합니다.
* 이를 통해 데이터 무결성을 보장하고, 예기치 못한 변경으로 인한 버그를 방지할 수 있습니다.

**3. 상태와 행위의 캡슐화**

* 컬렉션의 상태(아이템 목록)와 행위(추가, 총합 계산 등)를 `OrderItems`에서 **한 곳**에서 처리.
* 유지보수가 쉬워지고, 변경으로 인한 부작용 최소화.

**4. 명확한 컬렉션 객체 이름**

* 단순한 `List<OrderItem>` 대신 `OrderItems`라는 클래스로 의도를 명확히 표현.
* 코드 읽기만으로도 "이것이 주문 아이템 목록"임을 쉽게 이해 가능.

**5. 확장성**

* 컬렉션 로직(정렬, 필터링, 중복 방지 등)을 `OrderItems` 클래스 내부에 추가하기 쉬움.
* 예를 들어, 특정 조건에 따라 아이템을 필터링하거나, 중복된 아이템을 합치는 로직을 추가 가능.
