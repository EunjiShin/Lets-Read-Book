# 2. 설계 첫 걸음

## 2.1 의도를 분명히 전달할 수 있는 이름 설계하기

### Before

```java
int d = 0;
d = p1 + p2;
d = d - ((d1+d2) / 2);
if (d < 0) d = 0;
```

- 이름이 짧아 입력은 쉽지만, 보는 사람의 입장에선 의미를 파악하기 힘들다.
- 심지어 코드를 짠 나조차도 시간이 지난 후엔 못 알아볼 지도 모른다.

### After

```java
int damageAmount = 0;
damageAmount = playerArmPower + playerWeaponPower;
damageAmount = damageAmount - ((enemyBodyDefence + enemyArmorDefence) / 2);
if (damageAmount < 0) damageAmount = 0;
```

- **의도를 쉽게 파악할 수 있는 이름**을 사용하여 누가 보더라도 코드 의미를 쉽게 알 수 있다.

---

## 2.2 목적별로 변수 따로 만들어 사용하기

### Before

<aside>
💡 **재할당** : 변수에 값을 다시 할당하는 행위

</aside>

```java
int damageAmount = 0;
damageAmount = playerArmPower + playerWeaponPower; // 1
damageAmount = damageAmount - ((enemyBodyDefence + enemyArmorDefence) / 2); // 2
if (damageAmount < 0) damageAmount = 0;
```

- 이해하긴 쉽지만, damageAmount에 재할당이 여러 번 발생한다.
    - 재할당은 변수의 용도를 바꿔, 코드를 읽는 사람을 혼란스럽게 만들고, 버그를 만들 수도 있다.

### After

```java
int totalPlayerAttackPower = playerArmPower + playerWeaponPower;
int totalEnemyDefence = enemyBodyDefence + enemyArmorDefence;
damageAmount = damageAmount - (totalEnemyDefence / 2);
if (damageAmount < 0) damageAmount = 0;
```

- 기존 코드에서 의미가 바뀌는 부분 (1과 2)을 새로운 변수로 만든다.
    - ex. 1은 플레이어 공격력의 총합, 2는 적 방어력의 총합
- **목적에 따라 변수를 만들어 사용**하면, 어떤 값을 계산하는 데 어떤 값을 사용하는지 관계를 파악하기 쉬워진다.

---

## 2.3 단순 나열이 아니라, 의미 있는 것을 모아 메서드로 만들기

### Before

```java
int totalPlayerAttackPower = playerArmPower + playerWeaponPower;
int totalEnemyDefence = enemyBodyDefence + enemyArmorDefence;
damageAmount = damageAmount - (totalEnemyDefence / 2);
if (damageAmount < 0) damageAmount = 0;
```

- 모든 로직이 하나의 메서드 안에 모여있는 상황
    - ex. 1) 공격력과 방어력을 계산하는 로직 2) 계산 결과를 다른 변수에 저장하는 로직
- 로직이 어디에서 시작하고 어디서 끝나는지, 무슨 일을 하는지 파악하기 어려운 코드

### After

```java
// 플레이어 공격력 합계 계산
int sumUpPlayerAttackPower(int playerArmPower, int playerWeaponPower) {
	return playerArmPower + playerWeaponPower;
}

// 적의 방어력 합계 계산
int sumUpEnemyDefence(int enemyBodyDefence, int enemyArmorDefence) {
	return enemyBodyDefence + enemyArmorDefence;
}

// 데미지 평가
int estimateDamage(int totalPlayerAttackPower, int totalEnemyDefence) {
	int damageAmount = totalPlayerAttackPower - (totalEnemyDefence / 2);
	if (damageAmount < 0) return 0;
	return damageAmount;
}

// 계산 결과 저장
void main() {
	...
	int totalPlayerAttackPower = sumUpPlayerAttackPower(playerBodyPower, playerWeaponPower);
	int totalEnemyDefence = sumUpEnemyDefence(enemyBodyDefence, enemyArmorDefence);
	int damageAmount = estimateDamage(totalPlayerAttackPower, totalEnemyDefence);
}
```

- 세부 계산 로직을 메서드로 분리 & 서로 다른 계산 작업을 각각의 메서드로 분리했다.
- 전체 코드 양은 많아졌지만, 전체 흐름이 쉽게 읽히고 이해하기도 쉬워졌다.

---

## 2.4 관련된 데이터와 로직을 클래스로 모으기

### Before

```java
void main() {
	// 플레이어의 히트포인트
	int hitPoint;
}

// 어딘가에 작성된 히트포인트 감소 로직
int method(int hitPoint, int damageAmount) {
	hitPoint = hitPoint - damageAmount;
	if (hitPoint < 0) return 0;
	return hitPoint;
}

// 어딘가에 작성된 히트포인트 증가 로직
int anotherMethod(int hitPoint, int recoveryAmount) {
	hitPoint = hitPoint + recoveryAmount;
	if (hitPoint > 999) return 999;
	return hitPoint;
}
```

- 변수와 변수를 조작하는 로직이 사방팔방에 흩어져 있어, 관련 로직을 찾는데 시간이 오래 걸린다.
- 변수에 잘못된 값이 섞일 수도 있다.
    - ex. hitPoint는 무조건 양수여야 하는데, 음수값이 할당될 수도 있다.

### After

```java
class HitPoint {
	private static final int MIN = 0;
	private static final int MAX = 999;
	final int value;
	
	HitPoint(final int value) {
		if (value < MIN) throw new IllegalArgumentException("최소값 미만");
		if (value > MAX) throw new IllegalArgumentException("최대값 초과");
		this.value = value;
	}

	HitPoint damage(final int damageAmount) {
		final int damaged = value - damageAmount;
		final int corrected = damaged < MIN ? MIN : damaged;
		return new HitPoint(corrected);
	}

	HitPoint recover(final int recoveryAmount) {
		final int recovered = value + recoveryAmount;
		final int corrected = MAX < recovered ? MAX : recoverd;
		return new HitPoint(corrected);
	}
}
```

- **밀접한 데이터, 로직, 유효값 검증 과정을 하나의 클래스**에 모아두었다.
- 잘못된 값이 유입되지 않으며, 관련 로직이 모아져 있으므로 여기저기 찾지 않아도 되어 유지보수와 변경이 쉽다.