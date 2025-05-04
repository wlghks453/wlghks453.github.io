---
layout: default
title: 사용자 정의 Repository
parent: SPRING DATA JPA
grand_parent: 스프링
nav_order: 7
---

## 목차
{: .no_toc .text-delta }

1. TOC 
{:toc}

---

# 📚 사용자 정의 Repository 구현

Spring Data JPA에서는 기본적인 CRUD 메소드 외에도, **사용자 정의 Repository**를 구현하여 복잡한 로직이나 특정한 쿼리 처리 방법을 추가할 수 있습니다. 사용자 정의 Repository는 기본 `JpaRepository`의 기능을 확장하고, 원하는 커스텀 메소드를 구현하는 데 사용됩니다.

## 사용자 정의 Repository 구현 단계

사용자 정의 Repository를 구현하기 위한 단계는 크게 3단계로 나눌 수 있습니다:

1. **인터페이스 정의**: 사용자 정의 메소드를 포함하는 인터페이스를 정의합니다.
2. **구현 클래스 작성**: 위에서 정의한 인터페이스의 구현체 클래스를 작성하여 로직을 구현합니다.
3. **기본 Repository와 연결**: Spring Data JPA의 기본 Repository 인터페이스와 사용자 정의 Repository를 연결합니다.


### 1. 사용자 정의 Repository 인터페이스 정의

```java
public interface UserRepositoryCustom {
    List<User> findUsersByCustomCriteria(String criteria);
}
```

### 2. 사용자 정의 Repository 구현 클래스 작성

`UserRepositoryCustom` 인터페이스의 구현체는 **반드시 `UserRepositoryImpl`**로 명명해야 합니다.

```java
public class UserRepositoryImpl implements UserRepositoryCustom {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public List<User> findUsersByCustomCriteria(String criteria) {
        String query = "SELECT u FROM User u WHERE u.someField = :criteria";
        return entityManager.createQuery(query, User.class)
                .setParameter("criteria", criteria)
                .getResultList();
    }
}
```

### 3. 기본 Repository와 사용자 정의 Repository 연결

기본 `UserRepository` 인터페이스에 `JpaRepository`와 사용자 정의 인터페이스를 상속시킵니다.

```java
public interface UserRepository extends JpaRepository<User, Long>, UserRepositoryCustom {

}
```

이렇게 해야 `UserRepository`는 기본 CRUD 메소드와 함께, 커스텀 메소드를 사용할 수 있습니다.

### 명명 규칙

- **기본 Repository 인터페이스**: `UserRepository`
- **사용자 정의 Repository 인터페이스**: `UserRepositoryCustom`
- **사용자 정의 Repository 구현체**: `UserRepositoryImpl` (이 규칙에 따라 이름을 지어야 동작합니다)

## 무조건 사용해야 할까?

사용자 정의 Repository 구현을 위해 반드시 **Custom 인터페이스와 Impl 클래스를 사용해야 하는 것은 아닙니다**. 일반적인 **`@Repository`** 클래스를 만들어서 그 안에 필요한 로직을 구현하고 사용할 수도 있습니다. 이 방법을 통해 더 간단하게 커스텀 로직을 구현할 수 있습니다.

### @Repository를 활용한 방식

 `@Repository`로 필요한 로직을 구현하여 사용할 수 있습니다.

```java
@Repository
public class UserRepositoryCustom {

    @PersistenceContext
    private EntityManager entityManager;

    public List<User> findUsersByCustomCriteria(String criteria) {
        String query = "SELECT u FROM User u WHERE u.someField = :criteria";
        return entityManager.createQuery(query, User.class)
                .setParameter("criteria", criteria)
                .getResultList();
    }
}
```

이 방식은 사용자 정의 Repository 인터페이스를 따로 정의하지 않고, `@Repository`로 필요한 로직을 구현하여 사용하는 방법입니다. 일반적인 Spring Bean으로 등록되므로, 서비스 계층에서 바로 주입받아 사용할 수 있습니다.