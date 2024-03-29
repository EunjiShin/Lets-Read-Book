# 불변 활용하기: 안정적으로 동작하게 만들기

> ❄️ 불변과 가변을 적절하게 설계하지 못하면 동작을 예측하기 어렵고 혼란스러워진다.

## 4.1 재할당

> 💡 **재할당 (=파괴적 할당)** : 변수에 값을 다시 할당하는 것

- 재할당은 변수의 의미를 바꿔 추측하기 어렵게 하고, 언제 어떻게 변경되었는지 추적하기 힘들게 만든다.

### Before

```java
int damage() {
    int tmp = member.power() + member.weaponAttack();
    tmp = (int)(tmp * (1f + member.speed() / 100f));
    tmp = tmp - (int)(enemy.defence / 2);
    tmp = Math.max(0, tmp);
    return tmp;
  }
```

- 변수 tmp에 공격력, 보정 값, 데미지 등이 재할당되어 값의 의미가 변질된다.
    - 읽는 사람 입장에서 tmp가 정확히 뭘 의미하는지 알기 힘들기에 버그 발생 가능성이 높아진다.

### After : 불변 변수로 만들어 재할당 방지하기

- 변수 하나를 재활용하지 않고, 계속해서 새로운 변수를 만들게 하여 재할당을 방지하자.

```java
int damage() {
    final int basicAttackPower = member.power() + member.weaponAttack();
    final int finalAttackPower = (int)(basicAttackPower * (1f + member.speed() / 100f));
    final int reduction = (int)(enemy.defence / 2);
    final int damage = Math.max(0, finalAttackPower-reduction);
    return damage;
  }
```

### After : 매개변수를 불변으로 만들기

- 마찬가지로 매개변수도 불변으로 만들어, 그 의미가 바뀌지 않게 방지한다.

```java
void addPrice(final int productPrice) {
    final int increaseTotalPrice = totalPrice + productPrice;
    if (MAX_TOTAL_PRICE < increaseTotalPrice) throw new IllegalArgumentException("");
  }
```

---

## 4.2 가변으로 인해 발생하는 의도하지 않은 영향

> 💡 인스턴스가 가변이면, 코드가 변경되었을 때 의도하지 않은 문제가 생겨 예측하지 못한 동작이 발생 가능

### Before : 가변 인스턴스를 재사용하는 경우

```java
class AttackPower {
  static final int MIN = 0;
  int value;
  
  AttackPower(int value) {
    if (value < MIN) throw new IllegalArgumentException();
    this.value = value;
  }
  
}
```

```java
class Weapon {
  final AttackPower attackPower;

  Weapon(AttackPower attackPower) {
    this.attackPower = attackPower;
  }

}
```

- 위와 같이 공격력을 나타내는 클래스와 무기를 나타내는 클래스가 각각 존재한다고 가정하자.

```java
void test() {
    AttackPower attackPower = new AttackPower(20);
    Weapon weaponA = new Weapon(attackPower);
    Weapon weaponB = new Weapon(attackPower);
  }
```

- 처음 구현할 때는 무기 공격력이 고정이었기에, AttackPower 인스턴스 하나를 만들어 재사용했다.

```java
void test() {
    AttackPower attackPower = new AttackPower(20);
    Weapon weaponA = new Weapon(attackPower);
    Weapon weaponB = new Weapon(attackPower);
    
    weaponA.attackPower.value = 25;

    System.out.println("Weapon A attack power : " + weaponA.attackPower); // 25
    System.out.println("Weapon B attack power : " + weaponB.attackPower); // 25
  }
```

- 그런데 이후, 무기 각각의 공격력을 강화할 수 있게 조건이 변경되고, 이후 한 무기의 공격력을 강화하면 다른 무기의 공격력도 강화되는 버그가 발생한다.
- 이는 AttackPower가 가변 인스턴스 변수이기 때문에, 재활용한 모든 코드의 값이 변경되기 때문이다.

### After : 가변 인스턴스를 개별적으로 생성한다.

```java
void test() {
    AttackPower attackPowerA = new AttackPower(20);
    Weapon weaponA = new Weapon(attackPowerA);

    AttackPower attackPowerB = new AttackPower(20);
    Weapon weaponB = new Weapon(attackPowerB);
    
    weaponA.attackPower.value += 5;;

    System.out.println("Weapon A attack power : " + weaponA.attackPower); // 25
    System.out.println("Weapon B attack power : " + weaponB.attackPower); // 20
  }
```

### Before : 함수로 가변 인스턴스를 조작하는 경우

```java
class AttackPower {
  static final int MIN = 0;
  int value;

  AttackPower(int value) {
    if (value < MIN) throw new IllegalArgumentException();
    this.value = value;
  }
  
  void reinforce(int increment) {
    value += increment;
  }
  
  void disable() {
    value = MIN;
  }

}
```

- 상기 공격력 클래스에 공격력 값을 변화시키는 두 메서드를 추가한다.

```java
void changeAttackPower() {
    AttackPower attackPower = new AttackPower(20);
    // 생략
    attackPower.reinforce(15);
    System.out.println("attack power : " + attackPower.value);
  }
```

- 위와 같이 클래스 메서드를 호출하여 인스턴스 변수의 값을 바꾸는 로직을 구현한다.
    - 처음에는 잘 동작하지만 (35), 어느 순간부터 이상한 값 (0)이 할당된다.
- AttackPower 인스턴스가 다른 스레드에서 사용되면서 attackPower.value에 대해 reinforce()와 disable()이 **각각의 스레드에서 실행되며 부수 효과가 발생**한 것!
    - 즉, 동일한 결과를 내기 위해선 reinforce()와 disable()이 항상 동일한 순서로 실행되어야 한다.
    - **코드의 결과가 작업 실행 순서에 의존하게 되기에 결과를 예측하기 힘들고, 유지보수 하기도 힘들다.**

> ❄️ **`함수의 주요 작용`** <br>
> : 함수(메서드)가 매개변수를 전달받고 값을 리턴하는 것 <br><br>
> **`함수의 부수 효과`** 
> : 주요 작용 이외의 상태 변경을 일으키는 것 <br>
> : ex. 인스턴스 변수 변경, 전역 변수 변경, 매개변수 변경, I/O 조작 등

- 부수 효과가 있는 함수는 예측하기 힘들기 때문에, 함수가 영향을 주거나 받을 수 있는 범위를 한정해야 한다.
- **함수는 다음을 만족하게 설계하는 것이 좋다.**
    - 데이터(상태)는 매개변수로 받는다.
    - 상태를 변경하지 않는다.
    - 값은 함수의 리턴 값으로 돌려준다.

### After : 불변으로 만들어 예기치 못한 동작 막기

```java
class AttackPower {
  static final int MIN = 0;
  final int value;

  AttackPower(final int value) {
    if (value < MIN) throw new IllegalArgumentException();
    this.value = value;
  }

  AttackPower reinforce(final AttackPower increment) {
    return new AttackPower(this.value + increment.value);
  }

  AttackPower disable() {
    return new AttackPower(MIN);
  }

}
```

```java
void changeAttackPower() {
    AttackPower attackPower = new AttackPower(20);
    // 생략
    final AttackPower reinforced = attackPower.reinforce(
        new AttackPower(15)
    );
    System.out.println("attack power : " + attackPower.value);
  }
```

- **부수 효과의 여지 자체를 없앨 수 있게, 인스턴스 변수를 불변으로 만든다.**
    - 기능 변경 때 의도하지 않은 부수 효과가 있는 함수가 만들어져, 예상치 못한 동작을 일으킬 수 있기 때문
- 이제 공격력 값을 변경할 때 새로운 AttackPower 인스턴스를 생성하므로, 변경 전과 후의 공격력을 서로 영향을 주지 않는다. 스레드 세이프해짐!

---

## 4.3 불변과 가변은 어떻게 다루어야 할까

### 기본적으로 불변으로 만들자

- **불변 변수의 장점**
    1. 변수의 의미가 변하지 않아 혼란을 줄일 수 있다.
    2. 동작이 안정적이게 되므로 결과를 예측하기 쉽다.
    3. 코드의 영향 범위가 한정적이므로 유지 보수가 편리하다.

### 가변이 필요한 경우는?

- **성능이 중요한 경우엔 가변이 필요하다.**
    - 불변이면 값을 변경하려면 인스턴스를 새로 생성해야 한다. 이때 크기가 큰 인스턴스라면 시간이 오래 걸려 성능이 저하될 수 있다.
    - ex. 대량의 데이터를 빠르게 처리하거나, 이미지를 처리하거나, 리소스 제약이 큰 임베디드 소프트웨어를 다루거나…
- 스코프가 국소적인 경우엔 가변을 써도 된다.
    - ex. 반복문 카운터 등 반복 처리 스코프에서만 쓰이는 지역 변수

### 상태를 변경하는 메서드 설계하기

> 💡 **뮤테이터(mutater)** : 상태를 변화시키는 메서드

```java
class HitPoint {
  int amount; 
}

public class Member {
  final HitPoint hitPoint; // 0 이상. 만약 0이 되면 사망 상태로 변경 필요
  final States states;
  
  void damage(int damageAmount) {
    hitPoint.amount -= damageAmount;
  }
}
```

- damage()는 멀쩡해보이지만, 실제로는 HitPoint.amount를 음수로 만들 수도 있다.
- 또한 amount가 0이 되어도 사망 상태로 바뀌지 않는다.

```java
class HitPoint {
  private static final int MIN = 0
  int amount;

  HitPoint(final int amount) {
    if (amount < MIN) {
      throw new IllegalArgumentException("");
    }
    this.amount = amount;
  }
  
  void damage(final int damagedAmount) {
    final int nextAmouont = amount - damagedAmount;
    amount = Math.max(MIN, nextAmouont);
  }
  
  boolean isZero() {
    return amount == MIN;
  }
}

public class Member {
  final HitPoint hitPoint; // 0 이상. 만약 0이 되면 사망 상태로 변경 필요
  final States states;
  
  void damage(final int damageAmount) {
    hitPoint.damage(damageAmount);
    if (hitPoint.isZero()) {
			stated.add(StateType.dead);
    }
  }
}
```

### 코드 외부와 데이터 교환은 국소화하기

- 불변을 써서 신중하게 설계했더라도, 코드 외부와의 데이터 교환은 주의가 필요하다.
    - ex. I/O 조작은 코드 외부 상태에 의존한다. 웹 애플리케이션은 필수로 DB를 사용한다.
- 코드를 주의 깊게 작성했어도 파일이나 DB는 코드 외부이기에, **코드 단에선 외부 동작을 제어할 수 없다**.
    - 따라서 특별한 이유 없이 외부에 의존하는 코드를 쓰면 동작 예측이 힘들어진다.
    - 코드 외부와 데이터 교환을 국소화하는 테크닉으로는 레포지터리 패턴(repository pattern)이 있다.