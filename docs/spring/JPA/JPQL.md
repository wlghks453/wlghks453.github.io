---
layout: default
title: JPA JQPL
parent: JPA
grand_parent: 스프링
nav_order: 5
---

# 검색영역
{: .no_toc }

## 목차는 여기인 것 같아요.
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# 🔍 JPQL (Java Persistence Query Language)

JPQL은 Java Persistence API(JPA)에서 사용되는 객체지향 쿼리 언어입니다. JPQL은 SQL과 유사하지만, SQL이 데이터베이스의 테이블을 대상으로 작업하는 반면, JPQL은 엔티티 객체를 대상으로 쿼리를 실행합니다. 따라서 JPQL은 데이터베이스 테이블이 아닌 자바 클래스와 그 속성들에 대해 질의할 수 있습니다.

## 🛠️ JPQL의 주요 특징

- **객체 지향 쿼리 언어**: JPQL은 엔티티 객체와 그 속성들을 대상으로 질의합니다. 즉, 쿼리에서 사용되는 모든 항목이 자바 객체와 연관되어 있습니다.

- **데이터베이스 독립적**: JPQL은 데이터베이스에 종속되지 않으며, JPA 구현체가 JPQL을 특정 데이터베이스에 맞게 변환해줍니다.

- **SQL과 유사한 문법**: JPQL은 SQL과 유사한 문법을 사용하기 때문에 SQL에 익숙한 개발자가 쉽게 배울 수 있습니다.

## 📋 JPQL 예제

```
String jpql = "SELECT e FROM Employee e WHERE e.salary > :salary";
List<Employee> employees = em.createQuery(jpql, Employee.class)
                             .setParameter("salary", 50000)
                             .getResultList();
```
