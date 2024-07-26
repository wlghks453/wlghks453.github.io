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

## 지연로딩과 즉시로딩

@ManyToOne(fetch = FetchType.LAZY) : 지연로딩
@ManyToOne(fetch = FetchType.EAGER) : 즉시로딩


지연로딩으로 설정하면 연관설정된 엔티티는 프록시 객체로 넘어온다.

즉시로딩으로 설정하면 연관설정된 엔티티는 디비에서 즉시 조회되어 엔티티로 넘어온다.

• 가급적 지연 로딩만 사용(특히 실무에서) 

• 즉시 로딩을 적용하면 예상하지 못한 SQL이 발생 

• 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다. (fetch, 엔티티 그래프를 이용해서 N+1문제 해결) 
즉시로딩일 경우
SELECT * FROM A 했을 때 ROW가 100개가 나왔다고 치자.
ROW 100개에 대해 프록시객체로 채우면 안되니(즉시로딩이므로) SQL이 100개가 또 나가게 되는 것이다.

• @ManyToOne, @OneToOne은 기본이 즉시 로딩 -> LAZY로 설정 

• @OneToMany, @ManyToMany는 기본이 지연 로딩

무조건 지연로딩을 사용하도록 하자.

## 영속성 전이: CASCADE

특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들도 싶을 때
EX. 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장

사용 방법 
@OneToMany(mappedBy="parent", cascade=CascadeType.PERSIST)

CASCADE의 종류

ALL: 모두 적용 
PERSIST: 영속 
REMOVE: 삭제 
MERGE: 병합 
REFRESH: REFRESH 
DETACH: DETACH

자식 엔티티가 하나의 엔티티에서만 관리될 때 사용하면 편하다.

## 고아 객체

부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제

사용 방법
orphanRemoval = true

자식 엔티티가 하나의 엔티티에서만 관리될 때 사용하면 편하다.
