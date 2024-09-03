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

# 🚀 JPQL의 Bulk 연산

JPQL의 Bulk 연산은 대량의 데이터를 한 번에 변경하거나 삭제하는 연산을 말합니다. 

## UPDATE 쿼리

대량의 엔티티 필드를 한 번에 업데이트할 때 사용합니다.

```java
UPDATE Member m SET m.status = 'INACTIVE' WHERE m.age > 30
```

## DELETE 쿼리

대량의 엔티티를 한 번에 삭제할 때 사용합니다.

```java
DELETE FROM Member m WHERE m.status = 'INACTIVE'
```

## INSERT INTO ... SELECT( 하이버네이트 지원 )

JPA 표준스펙에는 없지만 하이버네이트에서는 가능하다.

## Bulk 연산의 주의사항

- **영속성 컨텍스트를 무시**: Bulk 연산은 영속성 컨텍스트를 무시하고 DB에 직접 반영됩니다. 따라서 영속성 컨텍스트와 DB 간의 데이터 불일치가 발생할 수 있습니다.
- **영속성 컨텍스트 초기화 필요**: Bulk 연산 후에는 `EntityManager.clear()`를 사용하여 영속성 컨텍스트를 초기화하거나, 영속성 컨텍스트와 DB 간의 상태를 동기화해야 합니다.
---