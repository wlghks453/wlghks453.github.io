---
layout: default
title: JPA 조회
parent: JPA
grand_parent: 스프링
nav_order: 4
---

## 목차
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# N+1 조회 성능 올리기

FETCH JOIN 으로 마무리!!

# COLLECTION 조회일 경우

## FETCH JOIN 으로 마무리!! + 키워드 distinct를 추가해줌.

페치 조인으로 SQL이 1번만 실행됨
distinct 를 사용한 이유는 1대다 조인이 있으므로 데이터베이스 row가 증가한다. 그 결과 같은 order 엔티티의 조회 수도 증가하게 된다. JPA의 distinct는 SQL에 distinct를 추가하고, 
더해서 같은 엔티티가 조회되면, 애플리케이션에서 중복을 걸러준다. 이 예에서 order가 컬렉션 페치 조인 때문에 중복 조회 되는 것을 막아준다.

단점
페이징 불가능
참고: 컬렉션 페치 조인을 사용하면 페이징이 불가능하다. 하이버네이트는 경고 로그를 남기면서 모든 데이터를 DB에서 읽어오고, 메모리에서 페이징 해버린다(매우 위험하다). oom 나면서 사망각!
참고: 컬렉션 페치 조인은 1개만 사용할 수 있다. 컬렉션 둘 이상에 페치 조인을 사용하면 안된다. 데이터가 부정 합하게 조회될 수 있다.  1 x n x m 이런 곳에서 사용하면 부정확해지는듯...

## 페이징 가능하면서 최적화

먼저 ToOne(OneToOne, ManyToOne) 관계를 모두 페치조인 한다. ToOne 관계는 row수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지 않는다.

지연 로딩 성능 최적화를 위해 hibernate.default_batch_fetch_size , @BatchSize 를 적용한다.
hibernate.default_batch_fetch_size: 글로벌 설정
@BatchSize: 개별 최적화
이 옵션을 사용하면 컬렉션이나, 프록시 객체를 한꺼번에 설정한 size 만큼 IN 쿼리로 조회한다.

장점
쿼리 호출 수가 1 + 1 + 1 로 최적화 된다.  (fetch보다 크다면 그 만큼 쿼리가 나가긴 함.)
조인보다 DB 데이터 전송량이 최적화 된다. (Order와 OrderItem을 조인하면 Order가 OrderItem 만큼 중복해서 조회된다. 이 방법은 각각 조회하므로 전송해야할 중복 데이터가 없다.)
페치 조인 방식과 비교해서 쿼리 호출 수가 약간 증가하지만, DB 데이터 전송량이 감소한다.
컬렉션 페치 조인은 페이징이 불가능 하지만 이 방법은 페이징이 가능하다.
결론
ToOne 관계는 페치 조인해도 페이징에 영향을 주지 않는다. 따라서 ToOne 관계는 페치조인으로 쿼리 수를 줄이고 해결하고, 나머지는 hibernate.default_batch_fetch_size 로 최적화 하자.
페이징으로 조금 가져오고 나머지는 IN 방식으로 LAZY 로딩한다는 것!!!
