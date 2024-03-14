---
layout: default
title: JPA 프록시
parent: JPA
grand_parent: 스프링

nav_order: 3
---

# 검색영역
{: .no_toc }

## 목차는 여기인 것 같아요.
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# 프록시

em.find() 는 바로 sql이 날라가 실제 엔티티 객체를 조회한다.

em.getReference(): 데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체 조회

프록시 객체와 실제객체 차이가 있지만 그냥 사용하면 된다.

프록시 객체는 속은 비어 있고 실제 객체를 참조한다. => 프록시 객체를 호출하면 실제 객체를 다시 호출하는 것. (프록시 초기화)

최초 실제 엔티티 객체가 영속성 컨텍스트에 존재하면 em.getReference() 해도 실제 엔티티가 반환된다. (최초 만들어진 타입을 유지함)

준영속 상태일 때 프록시를 초기화하면 문제가 발생한다. (하이버네이트는 org.hibernate.LazyInitializationException 예외 발생)
