---
layout: default
title: JPA 제목미정2
parent: JPA
grand_parent: 스프링

nav_order: 1
---

# 검색영역
{: .no_toc }

## 목차는 여기인 것 같아요.
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# 매핑

테이블,  컬럼 매핑은 생략한다.



## MappedBy

• 객체의 두 관계중 하나를 연관관계의 주인으로 지정 
• 연관관계의 주인만이 외래 키를 관리(등록, 수정) 
• 주인이 아닌쪽은 읽기만 가능 
• 주인은 mappedBy 속성 사용X 
• 주인이 아니면 mappedBy 속성으로 주인 지정

[정리] 
외래 키가 있는 있는 곳을 주인으로 정해라 (N:1 관계에서 N을 의미)
 
단방향 매핑만으로도 이미 연관관계 매핑은 완료 (설계 시, N관계에다가 일단 MANY TO ~~ 필요할 때 양방향으로 해도 늦지 않다.) 
