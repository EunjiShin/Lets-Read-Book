# 5. 응집도: 흩어져 있는 것들

## 5.1 static 메서드 오용

### Before

```java
class OrderManager {
  static int add(int moneyAmount1, int moneyAmount2) {
    return moneyAmount1 + moneyAmount2;
  }
}

// 클라이언트
void useCase() {
    moneyData1.amount = OrderManager.add(moneyData1.amount, moneyData2.amount);
  }
```

- **static 메서드는 인스턴스 변수를 사용할 수 없기에, 데이터와 데이터를 조작하는 로직의 응집도가 낮아진다.**
    - OrderManager 인스턴스 없이도 add 메서드를 호출할 수 있게 static 메서드로 만들었다.
    - 이 경우, 데이터는 MoneyData에, 로직은 OrderManager에 있게 되어 응집도가 낮아진다.

### After : 인스턴스 변수를 사용하는 구조로 만들기

> 💡 응집도  <br>
> = 클래스 내부에서 데이터와 로직의 관계가 얼마나 강한가 <br>
> = 인스턴스 변수와 인스턴스 변수를 사용하는 로직을 같은 클래스에 만들면 강해진다

```java
class OrderManager {

  final int amount;

  public OrderManager(int amount) {
    this.amount = amount;
  }

  OrderManager add(final OrderManager other) {
    final int added = amount + other.amount;
    return new OrderManager(added);
  }
}
```

### Before : static 키워드는 없지만, static 처럼 쓰이는 인스턴스 메서드

```java
public class PaymentManager {
  private int discountRate;

  int add(int moneyAmount1, int moneyAmount2) {
    return moneyAmount1 + moneyAmount2;
  }
}
```

- add가 인스턴스 메서드지만, 실제로는 인스턴스 변수를 전혀 사용하지 않는 케이스
- 인스턴스 메서드 앞에 static 키워드를 붙였을 때 컴파일 에러가 발생하지 않는다면 이런 케이스에 해당한다.

> ❄️ `static 메서드를 사용하는 이유` <br>
> - 절차지향 언어의 특징을 객체 지향 언어에 적용하여, 클래스 인스턴스 없이 간편하게 코딩하기 위함 <br>
> 
> `static 메서드를 사용하면 좋은 경우` <br> 
> - 응집도의 영향을 받지 않는 경우
> - ex. 로그 출력 전용 메서드, 포맷 변환 전용 메서드 등 + 팩토리 메서드

---

## 5.2 초기화 로직 분산

### Before

```java
public class GiftPoint {
  private static final int MIN_POINT = 0;
  final int value;

  GiftPoint(final int point) {
    if (point < MIN_POINT) {
      throw new IllegalArgumentException();
    }
    value = point;
  }

  GiftPoint add(final GiftPoint other) {
    return new GiftPoint(value + other.value);
  }

  boolean isEnough(final ConsumptionPoint point) {
    return point.value <= value;
  }

  GiftPoint consume(final ConsumptionPoint point) {
    if (!isEnough(point)) {
      throw new IllegalArgumentException();
    }
    return new GiftPoint(value - point.value);
  }
}
```

- GiftPoint에는 포인트와 포인트를 조작하는 메서드가 정의되어 있어 응집된 클래스처럼 보인다.
- 하지만 실제론 아래와 같은 문제가 발생한다.

    ```java
    // 표준 회원은 3000포인트를 제공받는다
    GiftPoint standardMembershipPoint = new GiftPoint(3000);
    
    // 프리미엄 회원은 10000포인트를 제공받는다
    GiftPoint premiumMembershipPoint = new GiftPoint(10000);
    ```

    - 만약 회원가입 시 부여되는 포인트를 변경하고 싶으면, 생성자가 사용된 소스 코드 전체를 확인해야 한다.
    - 즉, **생성자가 public이기에 의도하지 않은 용도로 사용되기 쉽고, 따라서 로직이 분산됨에 따라 유지 보수 하기 힘들어진 것**.

### After : private 생성자 + 팩토리 메서드를 활용해 목적에 따라 초기화

```java
public class GiftPoint {
  private static final int MIN_POINT = 0;
	private static final int STANDARD_MEMBERSHIP_POINT = 3000;
	private static final int PREMIUM_MEMBERSHIP_POINT = 10000;
  final int value;

  private GiftPoint(final int point) {
    if (point < MIN_POINT) {
      throw new IllegalArgumentException();
    }
    value = point;
  }

	static GiftPoint forStandardMembership() {
		return new GiftPoint(STANDARD_MEMBERSHIP_POINT);
	}

	static GiftPoint forPremiumdMembership() {
		return new GiftPoint(PREMIUM_MEMBERSHIP_POINT);
	}
}
```

- **생성자를 private으로 만들어 클래스 내부에서만 인스턴스를 생성**할 수 있게 만든다.
- **목적에 따라 static 팩토리 메서드를 만들고, 메서드 내부에서 생성자를 호출해 인스턴스를 생성**한다.
    - 이를 통해 신규 가입 포인트와 관련된 로직이 GiftPoint 클래스에 응집된다.

> ❄️ 상황에 따라 생성 로직이 너무 많아진다면, 클래스가 하는 일이 불분명해진다. <br>
> 따라서 이 경우 **생성 전용 팩토리 클래스를 분리**하는 방법을 고려해보자.

---

## 5.3 범용 처리 클래스 (Common / Util)

- 똑같은 일을 수행하는 코드가 많으면, 재사용을 위해 범용 클래스로 만들곤 한다.
    - 이때 static 메서드로 구현하는 경우가 많고 응집도가 낮아지는 문제가 생긴다.

### Before : 너무 많은 로직이 한 클래스에 모이는 문제

```java
public class Common {
  
  // 세금 포함 금액 계산
  static BigDecimal calcAmountIncludingTax(BigDecimal amountExcludingTax, BigDecimal taxRate) {
    ... 
  }
  
  // 사용자가 이미 탈퇴했다면 true
  static boolean hasResigned(User user) {
    ...
  }
  
  // 상품 주문하기
  static void createOrder(Product product) {
    ...
  }
  
  // 유효한 전화번호라면 true
  static boolean IsValidPhoneNumber(String phoneNumber) {
    ...
  }
}
```

- 세금 계산하는 메서드 이외에, 탈퇴 판별/ 상품 주문/ 유효한 전화번호 등 관련 없는 로직이 모여있다.
- Common, Util이라는 이름 자체가 읽는 사람으로 하여금 ‘범용적으로 사용하고 싶은 로직을 모아두는 곳’ 이라는 인식을 준다.
- **재사용성은 설계의 응집도를 높이면 저절로 높아진다는 점**을 기억하자.

### After : 꼭 필요한 경우가 아니라면 범용 처리 클래스를 만들지 말자

```java
class AmountIncludingTax {
  final BigDecimal vallue;
  
  AmountIncludingTax(
      final AmountIncludingTax amouontExcludingTax,
      final TaxRate taxRate
  ) {
    value = amouontExcludingTax.value.multiply(taxRate.value);
  }
}
```

### After : 어플리케이션 전체에서 사용되는 공통 기능은 범용 코드로 만들어도 OK

> ❄️ **`횡단 관심사 (cross-cutting concern)`** <br>
> : 어플리케이션 모든 동작에 대해 넓게 활용되는 기능들 <br>
> : ex. 로그 출력, 오류 확인, 디버깅, 예외 처리, 캐시, 동기화, 분산 처리 등..

---

## 5.4 결과를 리턴하는 데 매개변수 사용하지 않기

### Before

> ❄️ `**출력 매개 변수**` <br>
> : 출력으로 사용되는 매개변수. 잘못 사용하면 응집도가 낮아짐 <br>
> : ex. 이동 대상 인스턴스를 매개변수로 전달받고, 그 값을 변경하는 경우

```java
public class ActorManager {
  // 게임 캐릭터 위치를 이동
  void shift(Location location, int shiftX, int shiftY) {
    location.x _= shiftX;
    location.y = shiftY;
  }
}

public class SpecialAttackManager {
  void shift(Location location, int shiftX, int shiftY) {
    location.x _= shiftX;
    location.y = shiftY;
  }
}
```

- 데이터 조작 대상은 Location, 조작 로직은 ActorManager와 SpecialAttackManager에 위치

```java
public class DiscountManager {
  void set(MoneyData money) {
    money.amount -= 2000;
    if (money.amount < 0) {
			money.amount = 0;
		}
  }
}

discountManager.set(money);
```

- 전달한 매개변수 money의 값을 변경하는 코드.
- 매개변수는 입력으로 전달하는 것이 일반적인데, 이를 출력으로 활용한 경우에 해당
    - 즉 매개변수가 입력인지 출력인지 확인하기 위해 메서드 내부 로직을 하나하나 확인해야 함
    - 전체 로직을 읽어야만 이해할 수 있기에 가독성이 나쁜 코드

### After

```java
public class Location {

	final int x;
	final int y;

	Location(final int x, final int y) {
		this.x = x;
		thix.y = y;
	}
  // 게임 캐릭터 위치를 이동
  Location shift(final int shiftX, final int shiftY) {
    final int nextX = x + shiftX;
		final int nextY = y + shiftY;
		return new Location(nextX, nextY);
  }
}
```

- 위치 데이터와 위치 데이터를 조작하는 로직을 모두 한 클래스에 집합시키기

---

## 5.5 매개변수가 너무 많은 경우

### Before

```java
int recoverMagicPoint(
    int currentMagicPoint, int originalMaxMagicPoint,  
    List<Integer> maxMagicPointIncrements, int recoverAmount
  ) {
    int currentMaxMagicPoint = originalMaxMagicPoint;
    for (int each : maxMagicPointIncrements) {
      currentMaxMagicPoint += each;
    }
    return Math.max(currentMagicPoint + recoverAmount, currentMagicPoint);
  }
```

- 정상적으로 동작하지만 구조가 좋지 않다.
    - 너무 많은 매개변수가 있기에 실수로 잘못된 값을 대입할 수 있다.
    - 회복 이외의 기능도 수행하고 있기에, 중복 코드가 발생할 가능성이 높아진다.
- 메서드에 매개변수를 전달한다 = 해당 매개변수를 이용해 어떤 기능을 수행하고 싶다
    - 즉, **매개변수가 많다는 건 많은 기능을 처리한다**는 의미!
    - 처리할게 많아질 수록 로직이 복잡해지고, 중복 코드가 발생하기 쉬워진다.
- 기본 자료형 (primitive type)을 남용하는 기본 자료형 집착(primitive obsession)을 버려야 한다.
    - **기본 자료형을 쓰면 ‘동작하는 코드’는 만들 순 있지만, 관련 데이터와 로직을 집약하기 힘들다**.
    - 데이터는 단순히 존재하는게 아니라, 데이터를 사용해 계산하거나, 판단해 제어 흐름을 전환할 때 사용된다. 기본 자료형을 쓰면 계산과 제어 로직 모두가 분산된다는 점을 기억하자.

### After : 의미 있는 단위는 모두 클래스로 만들자

```java
public class MagicPoint {
  private int currentAmount;
  private int originalMaxAmount;
  private List<Integer> maxIncrements;
  
  int current() {
    return currentAmount;
  }
  
  int max() {
    int amount = originalMaxAmount;
    for (int each : maxIncrements) {
      amount += each;
    }
    return amount;
  }
  
  void recover(final int recoveryAmount) {
    currentAmount = Math.min(currentAmount + recoveryAmount, max());
  }
  
  void consume(final int consumeAmount) {
    ...
  }
  
}
```

- 매개변수가 많다면, 데이터 하나하나를 매개변수로 다루지 말고, 그 **데이터를 인스턴스 변수로 갖는 클래스를 만들고 활용**하자.
    - 다른 클래스에서 불필요한 조작을 하지 못하게 인스턴스 변수를 private로 만든다.
    - 인스턴스 변수를 조작하는 로직을 메서드로 정의한다.

---

## 5.6 메서드 체인

> ❄️ `**메서드 체인**` <br>
> : .(점)으로 여러 메서드를 연결해서 리턴 값의 요소에 차례차례 접근하는 방법

### Before

```java
void equipArmor(int memberId, Armor newArmor) {
	if(party.members[memberId].equipments.canChange) {
		party.members[memberId].equipments.armor = newArmor;
	}
}
```

- 메서드 체인을 이용해 클래스 깊이 접근하는 케이스
    - 할당하는 코드를 어디에서나 작성할 수 있기에 비슷한 코드가 여러 곳에 중복 작성될 수 있다.
    - 즉 모든 곳에서 요소에 접근할 수 있어, 응집도를 낮출 수 있다.
- **데메테르의 법칙**
    - 사용하는 객체 내부를 알아서는 안된다는 법칙
    - 메서드 체인으로 내부 구조를 돌아다닐 수 있는 설계는 데메테르의 법칙을 위반한다.

### After : 묻지 말고 명령하기 (Tell, Don’t Ask)

> ❄️ **`묻지 말고, 명령하기`** <br>
> : 다른 객체 내부 상태(변수)를 기반으로 판단하거나 제어하려고 하지 말고, 메서드로 명령해 객체가 알아서 판단하고 제어하도록 설계해라.

- 인스턴스 변수를 private으로 만들어 외부에서 접근할 수 없게 하자.
- 인스턴스 변수에 대한 제어는 외부에서 메서드를 이용하는 형태로 만들자.
    - 상세 판단과 제어는 명령을 받는 쪽이 담당하게 한다.

```java
public class Equipments {
  private boolean canChange;
  private Equipment head;
  private Equipment armor;
  private Equipment arm;
  
  void equipArmor(final Equipment newArmor) {
    if (canChange) {
      armor = newArmor;
    }
  }
  
  void deactivateAll() {
    head = Equipment.EMPTY;
    armor = Equipment.EMPTY;
    arm = Equipment.EMPTY;
  }
}
```

- 부위별 방어구와 방어구 탈착과 관련된 로직이 한 클래스에 응집된다.