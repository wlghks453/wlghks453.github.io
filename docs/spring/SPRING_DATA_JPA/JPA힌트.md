---
layout: default
title: JPA 힌트
parent: SPRING DATA JPA
grand_parent: 스프링
nav_order: 6
---

## 목차
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# 📝 JPA Hint에 대해 알아보기

**JPA Hint**는 JPA에서 **영속성 컨텍스트(Persistence Context)의 동작 방식을 제어**하기 위해 사용됩니다. 이는 SQL의 Hint와는 다른 개념으로, JPA 프로바이더가 엔티티를 어떻게 다루어야 하는지를 결정하는 데 도움을 줍니다.

## JPA Hint의 주요 개념

JPA Hint를 사용하면, 특정 쿼리나 엔티티에 대해 **스냅샷을 저장하지 않거나 영속성 컨텍스트에서 관리하지 않도록** 설정할 수 있습니다. 이는 데이터 조회 시 성능 최적화를 위한 방법 중 하나입니다.

### JPA Hint의 주요 목적

- **읽기 전용(Read-Only) 모드 설정**: 엔티티를 읽기 전용으로 설정하여, 영속성 컨텍스트에서 스냅샷을 저장하지 않고 변경 감지를 수행하지 않도록 합니다.
- **변경 감지 방지**: 엔티티의 상태를 변경해도 영속성 컨텍스트가 이를 감지하지 않고, 결과적으로 `flush` 시 데이터베이스에 반영되지 않습니다.
- **쓰기 작업 방지**: 읽기 전용 모드에서는 `flush` 작업 중 데이터베이스에 쓰기 작업이 발생하지 않으므로 성능이 최적화됩니다.

### @QueryHints 어노테이션 사용 예시

`@QueryHints` 어노테이션을 사용하여 JPA Hint를 JPQL 쿼리나 네이티브 SQL 쿼리에 적용할 수 있습니다. 예를 들어, 특정 쿼리를 **읽기 전용(Read-Only)**으로 설정하면 JPA는 영속성 컨텍스트에서 관리하지 않습니다.

```java
public interface UserRepository extends JpaRepository<User, Long> {

    @Query("SELECT u FROM User u WHERE u.status = :status")
    @QueryHints(@QueryHint(name = "org.hibernate.readOnly", value = "true"))
    List<User> findByStatus(@Param("status") String status);
}
```

위 예제에서:

- `@QueryHints` 어노테이션과 `@QueryHint`를 사용하여 `org.hibernate.readOnly`를 `true`로 설정합니다.
- 이 설정은 `User` 엔티티를 **읽기 전용**으로 처리하게 하여, 영속성 컨텍스트에서 엔티티 스냅샷을 저장하지 않도록 합니다.
- 결과적으로 `findByStatus` 메소드를 통해 조회된 엔티티가 변경되어도, JPA는 이를 감지하지 않고 데이터베이스에 반영하지 않습니다.

### JPA Hint의 활용 시 주의사항

- **읽기 전용 힌트**: 읽기 전용 힌트를 적용한 엔티티는 **변경해도 반영되지 않으므로**, 실수로 수정하려고 해도 데이터베이스에 반영되지 않습니다.
- **JPA 제공자마다 다를 수 있음**: Hibernate, EclipseLink 등 각 JPA 제공자는 지원하는 Hint의 종류와 동작이 다를 수 있습니다. 사용 중인 JPA 제공자의 공식 문서를 참고하는 것이 좋습니다.

## 결론

JPA Hint는 엔티티의 변경 감지를 방지하고 영속성 컨텍스트의 성능을 최적화하는 데 유용한 도구입니다. 이러한 기능을 적절히 활용하여 성능을 최적화하고, 불필요한 성능 저하를 방지할 수 있습니다.
