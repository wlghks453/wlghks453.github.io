---
layout: default
title: JPA JQPL
parent: JPA
grand_parent: 스프링
nav_order: 5
---

{: .no_toc }
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# 🔍 JPQL (Java Persistence Query Language)

JPQL은 Java Persistence API(JPA)에서 사용되는 객체지향 쿼리 언어입니다. JPQL은 SQL과 유사하지만, SQL이 데이터베이스의 테이블을 대상으로 작업하는 반면, 

JPQL은 엔티티 객체를 대상으로 쿼리를 실행합니다. 따라서 JPQL은 데이터베이스 테이블이 아닌 자바 클래스와 그 속성들에 대해 질의할 수 있습니다.

---

## 🛠️ JPQL의 주요 특징

- **객체 지향 쿼리 언어**: JPQL은 엔티티 객체와 그 속성들을 대상으로 질의합니다. 즉, 쿼리에서 사용되는 모든 항목이 자바 객체와 연관되어 있습니다.

- **데이터베이스 독립적**: JPQL은 데이터베이스에 종속되지 않으며, JPA 구현체가 JPQL을 특정 데이터베이스에 맞게 변환해줍니다.

- **SQL과 유사한 문법**: JPQL은 SQL과 유사한 문법을 사용하기 때문에 SQL에 익숙한 개발자가 쉽게 배울 수 있습니다. SELECT, FROM, WHERE, GROUP BY , HAVING, JOIN 지원

---

## 📋 JPQL 예제

```
String jpql = "SELECT e FROM Employee e WHERE e.salary > :salary";
List<Employee> employees = em.createQuery(jpql, Employee.class)
                             .setParameter("salary", 50000)
                             .getResultList();
```

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

### 연관관계 없는 조인 작성하기