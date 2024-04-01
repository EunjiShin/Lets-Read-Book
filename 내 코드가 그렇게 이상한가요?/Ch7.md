# 7. 컬렉션 : 중첩을 제거하는 구조화 테크닉

## 7.1 이미 존재하는 기능을 다시 구현하지 말기

### Before

```java
boolean hasPrisonKey = false;
for (Item each : items) {
	if (each.name.equals("감옥 열쇠")) {
		hasPrisonKey = true;
		break;
	}
}
```

- for 반복문 내부에 if 조건문이 중첩되어 가독성이 좋지 않다.

### After

```java
boolean hasPrisonKey = items.stream().anyMatch(
	item -> item.name.equals("감옥 열쇠))
);
```

- stream을 이용해 복잡한 로직을 생략할 수 있다.
- 표준 컬렉션 라이브러리에 내가 구현하고자 하는 로직과 같은 기능을 하는 메서드가 있는지 확인해보자!

---

## 7.2 반복 처리 내부의 조건 분기 중첩

- 컬렉션 내부에서 특정 조건을 만족시키는 요소에 대해서만 어떤 작업을 수행하고 싶은 경우

### Before

```java
for (Member member : members) {
	if (0 < member.hitPoint) {
		if (member.containsState(StateType.poison)) {
			member.hitPoint -= 10;
			if (member.hitPoint <= 0) {
				member.hitPoint = 0;
				member.addState(StateType.dead);
				member.removeState(StateType.poison);
			}
		}
	}

}
```

- for문 내부에 if 조건문이 중첩되어 가독성이 떨어진다.

### After : 조건 컨티뉴로 조건 분기 중첩 제거하기

> ❄️ **`조건 컨티뉴`** : 조건을 만족하지 않는 경우 다음 반복으로 넘어간다.

```java
for (Member member : members) {
  if (member.hitPoint == 0) continue;
  if (!member.containsState(StateType.poison)) continue;

  member.hitPoint -= 10;

  if (0 < member.hitPoint) continue;
  member.hitPoint = 0;
  member.addState(StateType.dead);
  member.removeState(StateType.poison);
}
```

### After : 조건 브레이크로 중첩 제거하기

```java
int totalDamage = 0;
for (Member member: members) {
  if (!member.hasTeamAttackSucceeded()) break;
  int damage = (int)(member.attack() * 1.1);
  if (damage < 30) break;
  totalDamage += damage;
}
```

---

## 7.3 응집도가 낮은 컬렉션 처리

- 컬렉션과 관련된 작업을 처리하는 코드가 시스템 여기저기에 구현되어 응집도가 낮아질 수 있다.

### After : 컬렉션 처리를 캡슐화 하기

> ❄️ **`일급 컬렉션 (First Class Collection)`** : 컬렉션과 관련된 로직을 캡슐화하는 디자인 패턴

- 클래스 설계 원리를 반영했을 때, 일급 컬렉션은 다음과 같은 요소로 구성된다.
    - 컬렉션 자료형의 인스턴스 변수
    - 컬렉션 자료형의 인스턴스 변수에 잘못된 값이 할당되지 않게 막고, 정상적으로 조작하는 메서드

```java
class Party {
	private final List<Member> members;
	
	Party() {
		members = new ArrayList<Member>();
	}

	Party add(final Member newMember) {
		List<Member> adding = new ArrayList<>(members);
		adding.add(newMember);
		return new Party(adding);
	}
}
```

- 컬렉션을 인스턴스 변수로 가지는 클래스를 만든다.
- 인스턴스 변수를 조작하는 로직도 추가한다.

### After : 외부로 전달할 때 컬렉션의 변경 막기

```java
class Party {
	List<Member> members() {
		return members;
	}
}
```

- 위와 같이, 인스턴스 변수를 그대로 외부에 전달하면 외부에서 마음대로 변수를 추가하고 제거할 수 있다.

```java
class Party {
	List<Member> members() {
		return members.unmodifiableList();
	}
}
```

- 따라서 **외부로 컬렉션을 전달할 때는, unmodifiableList를 이용해 컬렉션 요소를 변경하지 못하게 막자**.