---
layout: default
title: JPQL FETCH JOIN
parent: JPQL
grand_parent: 스프링
nav_order: 6
---

{: .no_toc }
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# 🔍 Fetch Join이란?

Fetch Join은 JPQL에서 **JOIN FETCH** 구문을 사용하여 연관된 엔티티나 컬렉션을 한 번의 쿼리로 함께 가져오는 방법입니다. 

일반적인 조인과 달리 Fetch Join은 연관된 엔티티를 함께 조회하여 성능을 최적화하는 데 유용합니다.

## 📚 Fetch Join의 주요 특징

1. **한 번의 쿼리로 관련 데이터를 조회**  
   Fetch Join은 여러 엔티티나 컬렉션을 한 번에 조회하여 N+1 문제를 해결할 수 있습니다. 즉, 연관된 엔티티를 별도로 조회하는 것이 아니라, 조인을 통해 한 번의 쿼리로 필요한 모든 데이터를 가져옵니다.

2. **SQL 조인으로 변환됨**  
   Fetch Join SQL의 조인기능이 아니고 JPQL의 기능이지만, 실제로는 SQL의 INNER JOIN이나 OUTER JOIN으로 변환됩니다. JPQL에서 Fetch Join을 사용하면 내부적으로 SQL 조인이 발생하여 연관된 엔티티를 함께 가져오게 됩니다.

3. **프록시 객체가 아님**  
   일반적인 지연 로딩(Lazy Loading)에서는 프록시 객체를 사용하여 실제 데이터가 필요한 시점에 조회하지만, Fetch Join을 사용하면 연관된 엔티티가 프록시 객체가 아닌 실제 데이터로 즉시 로딩됩니다.

4. **영속성 컨텍스트에 저장**  
   Fetch Join의 결과는 영속성 컨텍스트에 모두 저장됩니다. 따라서 Fetch Join을 통해 가져온 엔티티는 영속성 컨텍스트의 관리 하에 있으며, 이후의 변경 사항은 자동으로 반영될 수 있습니다.

## 🚨 N+1 문제와 Fetch Join의 해결

### N+1 문제란?

N+1 문제는 연관된 엔티티를 조회할 때 발생하는 성능 저하 문제입니다. 예를 들어, 부모 엔티티를 조회한 후 자식 엔티티를 각각 조회할 때, 최초 1번의 쿼리(N)와 이후 N번의 추가 쿼리가 발생하게 되는 문제를 말합니다.

### 예제: N+1 문제 발생 예시

아래와 같은 코드가 있다고 가정해봅시다:

```java
List<Member> members = entityManager.createQuery("SELECT m FROM Member m", Member.class)
        .getResultList();

for (Member member : members) {
    System.out.println(member.getTeam().getName());
}
```

위의 코드는 먼저 멤버 목록을 가져오고, 각 멤버의 팀 이름을 출력합니다. 이 경우 `SELECT m FROM Member m` 쿼리가 1번 실행되고, 각 멤버마다 팀을 조회하기 위해 `SELECT t FROM Team t WHERE t.id = :teamId` 쿼리가 N번 추가로 실행됩니다. 따라서 총 1 + N번의 쿼리가 발생하게 됩니다.

### Fetch Join을 사용한 N+1 문제 해결

N+1 문제를 해결하기 위해 Fetch Join을 사용할 수 있습니다:

```java
List<Member> members = entityManager.createQuery("SELECT m FROM Member m JOIN FETCH m.team", Member.class)
        .getResultList();

for (Member member : members) {
    System.out.println(member.getTeam().getName());
}
```

이제 Fetch Join을 사용하여 멤버와 팀을 한 번의 쿼리로 조회합니다. `SELECT m FROM Member m JOIN FETCH m.team` 쿼리는 모든 멤버와 그들이 속한 팀을 한 번에 가져오므로, N+1 문제가 해결됩니다.

## 🛠️ Fetch Join 중복 문제

### Fetch Join 중복 문제란?

`@xxxToMany` 와 같은 관계에서 Fetch Join을 사용하면, 연관된 엔티티를 조회할 때 중복된 결과가 발생할 수 있습니다. 예를 들어, `Team` 엔티티가 여러 `Member` 엔티티와 연관되어 있을 때, `Fetch Join`을 사용하면 각 멤버마다 팀이 반복적으로 조회되어 중복된 행이 발생합니다.

### 예제: Fetch Join 사용 시 중복된 결과 발생

아래와 같이 `Team`에서 `Member`를 조회하는 Fetch Join을 사용한다고 가정해봅시다.

```java
List<Team> teams = entityManager.createQuery("SELECT t FROM Team t JOIN FETCH t.members", Team.class)
.getResultList();

for (Team team : teams) {
System.out.println(team.getName() + " 멤버 수: " + team.getMembers().size());
}
```

예를 들어, Team A에 멤버가 2명 있고, Team B에 멤버가 1명 있을 때, 쿼리의 결과는 다음과 같이 중복될 수 있습니다:

| 팀 이름 | 멤버 수 |
| ------- | ------- |
| Team A  |    2    |
| Team A  |    2    |
| Team B  |    1    |


이로 인해 Team A가 중복되어 조회됩니다. Fetch Join을 사용하면 연관된 엔티티가 여러 번 조인되면서 동일한 팀이 중복된 결과로 나타납니다.

### 중복 문제 해결: DISTINCT 사용

Fetch Join의 중복 문제를 해결하기 위해 JPQL에서 `DISTINCT` 키워드를 사용할 수 있습니다:

```java
List<Team> teams = entityManager.createQuery("SELECT DISTINCT t FROM Team t JOIN FETCH t.members", Team.class)
.getResultList();

for (Team team : teams) {
System.out.println(team.getName() + " 멤버 수: " + team.getMembers().size());
}
```

`DISTINCT` 키워드를 사용하면 중복된 Team 엔티티가 제거되고, 각 팀은 한 번씩만 조회됩니다. 따라서 중복 없이 올바른 결과를 얻을 수 있습니다:

| 팀 이름 | 멤버 수 |
| ------- | ------- |
| Team A  |    2    |
| Team B  |    1    |

`DISTINCT`를 사용하여 중복된 결과를 제거함으로써 Fetch Join의 문제를 해결할 수 있습니다. 

SQL에서는 DISTINCT가 모든 컬럼의 값이 동일할 때만 중복을 제거할 수 있습니다.

SQL에 DISTINCT를 붙여서 날라가지만 모든 ROW가 같지 않아 중복제거가 안될 수도 있습니다.

JPQL에서는 엔티티의 중복을 제거할 때 어플리케이션에서 엔티티 식별자를 기준으로 비교하여 중복을 제거합니다.

---

이와 같이 Fetch Join에서 발생하는 중복 문제는 `DISTINCT`를 사용하여 간단하게 해결할 수 있습니다. 하지만, 필요에 따라 쿼리의 복잡성이나 성능에 영향을 줄 수 있으므로, 적절히 사용하는 것이 중요합니다.


## 🛠️ Fetch Join의 특징과 한계

Fetch Join은 성능 최적화를 위해 유용하지만, 몇 가지 한계가 있습니다.

### 1. Fetch Join 대상에서는 별칭(alias)을 줄 수 없다.
- JPQL에서 Fetch Join으로 조회하는 엔티티에는 별칭을 줄 수 없습니다. 이는 JPQL 표준 규격에 따른 제한입니다.
- **하이버네이트에서는 별칭을 사용할 수 있지만, 가급적 사용하지 않는 것이 좋습니다.** 별칭을 사용하면 JPQL 표준을 벗어나며, 다른 JPA 구현체와의 호환성이 떨어질 수 있습니다.

### 2. 둘 이상의 컬렉션을 Fetch Join할 수 없다.
- JPQL에서 둘 이상의 컬렉션을 동시에 Fetch Join하면 다중 조인으로 인해 Cartesian Product(데카르트 곱)가 발생하여 결과가 예상치 않게 부풀어 오를 수 있습니다.
- 이는 성능 문제를 초래할 수 있으므로, 일반적으로 하나의 컬렉션만 Fetch Join하는 것이 권장됩니다.

### 3. 컬렉션을 Fetch Join하면 페이징 API를 사용할 수 없다.
- Fetch Join을 사용하여 컬렉션을 조회하면 페이징 기능(`setFirstResult`, `setMaxResults`)을 사용할 수 없습니다. 이유는 페이징이 컬렉션의 결과를 제대로 제어할 수 없기 때문입니다.
   - 예를 들어, 일대다(OneToMany) 관계에서 Fetch Join을 사용할 때, 페이징을 하더라도 데이터의 중복으로 인해 올바른 페이징이 이루어지지 않습니다.
- **일대일(OneToOne), 다대일(ManyToOne) 관계와 같은 단일 값 연관 필드들은 Fetch Join해도 페이징이 가능합니다.**
- 하이버네이트는 컬렉션을 Fetch Join할 때 페이징 요청이 들어오면 경고 로그를 남기고, 메모리에서 페이징을 처리합니다. 이는 대량의 데이터를 메모리에 로딩하여 페이징하기 때문에 **매우 위험**할 수 있습니다.

## 📌 Fetch Join 정리

- **모든 문제를 Fetch Join으로 해결할 수는 없다.**
   - Fetch Join은 객체 그래프를 유지하고 연관된 엔티티를 한 번에 가져올 때 효과적입니다.
- **여러 테이블을 조인하여 엔티티가 가진 모양과 다른 결과를 원할 때는 Fetch Join을 사용하지 말아야 합니다.**
   - 이런 경우에는 일반 조인을 사용하고 필요한 데이터만 조회하여 DTO로 변환하는 것이 일반적입니다. Fetch Join은 엔티티를 그대로 유지하는 데 중점을 두기 때문에, 복잡한 결과 조합에는 적합하지 않습니다.

---