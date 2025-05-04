---
layout: default
title: JPA 반환타입
parent: JPQL
grand_parent: 스프링
nav_order: 5
---

## 목차
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 🎯 반환 타입에 따른 JPQL 사용법

JPQL을 사용할 때, **반환 타입이 명확한지 여부**에 따라 두 가지 다른 접근 방식을 사용할 수 있습니다: **TypedQuery**와 **Query**.

### 🔹 **반환 타입이 명확할 때: TypedQuery**

`TypedQuery`는 쿼리 실행 결과의 **반환 타입이 명확할 때** 사용됩니다.

예를 들어, 특정 엔티티 클래스를 조회할 때 그 결과가 항상 해당 엔티티 타입의 객체 리스트로 반환될 것을 기대할 수 있습니다.

이 경우 `TypedQuery`를 사용하면 **컴파일 시점에 타입 안전성**을 보장받을 수 있습니다.

*예시:*

```java
TypedQuery<Employee> query = em.createQuery("SELECT e FROM Employee e WHERE e.salary > :salary", Employee.class);
List<Employee> results = query.setParameter("salary", 1000).getResultList();
```

### 🔹 **반환 타입이 명확하지 않을 때: Query**

반환 타입이 명확하지 않거나 **여러 가지 타입이 섞여 있을 가능성이 있을 때**는 `Query`를 사용합니다.

예를 들어, 다양한 데이터 타입이 혼합된 결과를 반환하거나 특정 엔티티가 아닌 **값 타입**(예: `Object[]`, `Long` 등)을 반환할 때 사용됩니다.

*예시:*

```java
Query query = em.createQuery("SELECT e.name, e.salary FROM Employee e WHERE e.department.id = :deptId");
List<Object[]> results = query.setParameter("deptId", 1).getResultList();
```

---

## 🎯 JPQL에서의 프로젝션 방법

JPQL(Java Persistence Query Language)을 사용하여 쿼리를 작성할 때, SELECT 절에서 조회할 대상을 지정하는 것을 **프로젝션**이라고 합니다. 

프로젝션 대상에는 엔티티, 임베디드 타입, 그리고 스칼라 타입(숫자, 문자 등 기본 데이터 타입)이 포함됩니다.

### 🎂 Query 타입으로 조회

`Query` 타입을 사용하여 조회할 때는, JPQL 쿼리의 결과가 특정 엔티티의 객체 리스트로 반환됩니다. 이 방법은 반환 타입이 명확하지 않거나, 여러 가지 데이터 타입이 혼합된 결과를 처리할 때 유용합니다. 그러나, 이 방식은 타입 안전성이 보장되지 않기 때문에 결과를 적절한 타입으로 캐스팅하는 것이 중요합니다.

**예시:**

```java
Query query = em.createQuery("SELECT e.name, e.salary FROM Employee e WHERE e.department.id = :deptId");
List<Object[]> results = query.setParameter("deptId", 1).getResultList();
```

위 예제에서는 `Employee` 엔티티의 이름과 급여를 조회한 후, 결과를 `Object[]` 배열로 반환받습니다. 각 배열 요소는 쿼리에서 선택한 컬럼 값들을 담고 있습니다.

### 😊 Object[] 타입으로 조회

이 방식은 여러 필드를 선택하여 그 결과를 `Object[]` 배열로 받는 일반적인 프로젝션 방법입니다. 각 `Object[]` 배열의 인덱스는 선택한 필드에 대응되며, 이를 통해 여러 개의 필드 값을 동시에 조회하고, 배열 형태로 처리할 수 있습니다.

**예시:**

```java
List<Object[]> results = em.createQuery("SELECT e.name, e.salary FROM Employee e").getResultList();
for (Object[] result : results) {
String name = (String) result[0];
Double salary = (Double) result[1];
//...
}
```

이 방법은 간단하게 여러 컬럼 값을 조회하고 이를 배열로 다루기에 적합합니다. 다만, 각 값을 배열에서 인덱스를 사용해 꺼내기 때문에 실수할 가능성이 있습니다.

### 😂 new로 Dto 바로 조회

이 방식은 쿼리 결과를 DTO(Data Transfer Object)로 직접 매핑하여 반환하는 방법입니다. JPQL 쿼리에서 `new` 키워드를 사용하여 DTO 객체를 생성하고, 필요한 필드들을 초기화할 수 있습니다. 

**예시:**

```java
List<EmployeeDTO> results = em.createQuery("SELECT new com.example.EmployeeDTO(e.name, e.salary) FROM Employee e", EmployeeDTO.class).getResultList();
```

---

## 🎯 JPQL에서의 페이징

JPQL은 페이징도 지원해줍니다. 

지원하는 벤더에 따라서 적절하게 페이징 쿼리를 해줍니다.

-끝-

## 🎯 JPQL에서의 조인

데이터베이스에서 여러 테이블의 데이터를 결합하여 조회할 때 사용하는 **조인(Join)**은 JPQL에서도 자주 사용됩니다. JPQL에서는 SQL과 마찬가지로 여러 가지 조인 방법을 제공합니다. 이 포스팅에서는 **Inner Join**, **Outer Join**, 그리고 **Cross Join**에 대해 간략히 살펴보겠습니다.

### 🔹 Inner Join

inner 는 생략 가능합니다.

**예시:**

```java
SELECT e FROM Employee e JOIN e.department d WHERE d.name = 'Sales'
```

### 🔹 Outer Join

**Outer Join**은 한쪽 테이블의 모든 데이터를 포함시키고, 다른 테이블에서 일치하는 데이터가 있으면 결합하고, 없으면 NULL로 채우는 방식입니다. 

**예시:**

```java
SELECT e FROM Employee e LEFT JOIN e.department d
```

### 🔹 Cross Join

**예시:**

```java
SELECT e, d FROM Employee e CROSS JOIN Department d
```

---

## 🐱‍ JPQL에서의 서브쿼리

서브쿼리는 SQL에서와 마찬가지로 JPQL에서도 사용됩니다.

### 서브쿼리 지원 함수

JPQL에서 서브쿼리를 사용할 때는 여러 가지 함수를 활용할 수 있습니다. 대표적인 서브쿼리 지원 함수는 다음과 같습니다:

- **EXISTS**: 서브쿼리의 결과가 존재하는지 확인할 때 사용합니다.
- **ALL**: 서브쿼리의 모든 결과와 비교할 때 사용합니다.
- **ANY, SOME**: 서브쿼리의 일부 결과와 비교할 때 사용합니다.
- **IN**: 서브쿼리의 결과가 특정 값 집합에 포함되는지 확인할 때 사용합니다.

이들 함수는 서브쿼리를 통해 복잡한 조건을 처리할 때 매우 유용하게 사용됩니다.

### 서브쿼리 한계

JPQL에서 서브쿼리를 사용할 때는 몇 가지 제한사항이 있습니다:

- **WHERE, HAVING 절에서만 서브쿼리 사용 가능**: JPQL에서는 서브쿼리를 주로 `WHERE` 절과 `HAVING` 절에서 사용할 수 있습니다.
- **SELECT 절에서 서브쿼리 사용 (하이버네이트에서만 가능)**: 표준 JPA에서는 `SELECT` 절에서 서브쿼리를 사용할 수 없지만, 하이버네이트와 같은 특정 JPA 구현체에서는 `SELECT` 절에서 서브쿼리를 사용할 수 있습니다.
- **FROM 절의 서브쿼리는 불가능**: JPQL에서는 `FROM` 절에 서브쿼리를 사용할 수 없습니다. 이 점은 SQL과의 큰 차이점 중 하나입니다.

### 서브쿼리 예시

아래는 `EXISTS` 함수를 사용한 서브쿼리 예시입니다. 이 예시는 특정 부서에 소속된 직원들이 존재하는지 확인하는 쿼리입니다.

**예시:**

```java
SELECT d FROM Department d WHERE EXISTS (SELECT e FROM Employee e WHERE e.department = d)
```

---

## 🐱‍ JPQL에서의 조건식

SQL처럼 다양한 조건식을 제공합니다. 표준 함수라서 어떤 DB에서 사용가능합니다.

JPQL에서 자주 사용되는 조건식인 **CASE WHEN**, **COALESCE**, **NULLIF**가 있습니다.

### 🔹 CASE WHEN

SQL의 `CASE WHEN`과 유사하게 동작한다.

**예시:**

```java
SELECT e.name,
CASE
WHEN e.salary >= 100000 THEN 'High'
WHEN e.salary >= 50000 THEN 'Medium'
ELSE 'Low'
END
FROM Employee e
```

### 🔹 COALESCE

`COALESCE`는 인수 중 첫 번째로 NULL이 아닌 값을 반환하는 함수입니다. 여러 개의 값을 받아서 그 중에서 NULL이 아닌 첫 번째 값을 반환하므로, NULL 값을 대체하는 데 유용합니다.

**예시:**

```java
SELECT e.name, COALESCE(e.nickname, e.name)
FROM Employee e
```

이 쿼리는 직원의 닉네임이 존재하면 닉네임을 반환하고, 그렇지 않으면 본래 이름을 반환합니다.

### 🔹 NULLIF

`NULLIF`는 두 인수가 동일하면 NULL을 반환하고, 그렇지 않으면 첫 번째 인수를 반환하는 함수입니다. 이 함수는 특정 값이 특정 조건과 동일할 때 그 값을 NULL로 처리하고 싶을 때 사용됩니다.

**예시:**

```java
SELECT e.name, NULLIF(e.department.name, 'Unassigned')
FROM Employee e
```

이 쿼리는 부서 이름이 'Unassigned'인 경우에 NULL을 반환하고, 그렇지 않은 경우에는 실제 부서 이름을 반환합니다.

---

# 🐱‍💻 JPA 경로 표현식(Path Expression)

JPA에서 경로 표현식(Path Expression)은 엔티티와 그 엔티티에 속한 필드 또는 연관된 엔티티를 탐색하는 데 사용됩니다.

## 📚 경로 표현식의 개념

경로 표현식은 주로 다음과 같은 필드와 관계를 탐색할 때 사용됩니다.

- **단일 값 연관 필드**
- **컬렉션 값 연관 필드**
- **임베디드 타입**

### 1. 단일 값 연관 필드

단일 값 연관 필드는 다른 엔티티와의 `@ManyToOne` 또는 `@OneToOne` 관계를 의미합니다.

```java
SELECT e.department.name FROM Employee e
```

이 경로 표현식을 통해 `Department` 엔티티의 `name` 필드에 접근할 수 있습니다.

- **실제 SQL은 INNER JOIN 이 발생합니다.**


### 2. 컬렉션 값 연관 필드

컬렉션 값 연관 필드는 다른 엔티티와의 `@OneToMany` 또는 `@ManyToMany` 관계를 나타냅니다.

team 앤티티는 멤버 엔티티의 컬렉션을 가지고 있습니다.

```java
SELECT t.members FROM Team t
```

컬렉션 값 연관 필드입니다.

- **컬렉션 값 경로는 단일 값으로 확장될 수 없습니다.** 컬렉션 경로는 그 자체로 컬렉션을 나타내며, 추가적으로 경로를 따라갈 수 없습니다. 이를 해결하려면 명시적으로 조인을 사용해야 합니다.

### 3. 임베디드 타입

임베디드 타입은 엔티티 내부에 포함된 다른 객체입니다. 보통 `@Embedded` 애노테이션을 사용하여 엔티티 내부에 객체를 포함시키고, 경로 표현식을 통해 임베디드 객체의 필드에 접근할 수 있습니다.

```java
SELECT o.customer.address.city FROM Order o
```

## 🛠️ 경로 표현식 사용할 때 주의사항

 - **묵시적 조인(안보이는 SQL) 보단 명시적 조인(직접 JOIN을 명시하여 한눈에 보이게)** 을 사용합시다.

---