---
layout: default
title: Auditing
parent: SPRING DATA JPA
grand_parent: 스프링
nav_order: 8
---

## 목차
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# ⏰ Auditing

Spring Data JPA의 **Auditing** 기능은 엔티티의 생성, 수정 시간 및 수정자 등을 자동으로 기록할 수 있는 매우 유용한 기능입니다. 모든 엔티티마다 `createdDate`, `lastModifiedDate` 등을 일일이 추가하는 번거로움을 줄이기 위해, **상속 구조**를 활용하면 반복적인 코드를 줄일 수 있습니다.

## 1. @EnableJpaAuditing

Auditing 기능을 활성화하려면 애플리케이션의 메인 클래스에 **`@EnableJpaAuditing`** 어노테이션을 추가해야 합니다. 이를 통해 JPA Auditing 기능을 전역적으로 사용할 수 있습니다.

```java
@SpringBootApplication
@EnableJpaAuditing
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## 2. 공통 Auditing 필드를 포함한 BaseEntity

모든 엔티티에서 **생성일자**, **수정일자**, **생성자**, **수정자**와 같은 필드들을 일일이 추가하지 않기 위해, 이러한 공통 필드를 상속할 수 있는 **BaseEntity** 클래스를 만들 수 있습니다.

```java
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;

    // Getters and Setters
}
```

- **`@MappedSuperclass`**: 이 클래스는 상속받는 엔티티들이 사용할 공통 필드들을 정의하는 부모 클래스입니다.
- **`@EntityListeners(AuditingEntityListener.class)`**: JPA가 Auditing 기능을 적용하도록 리스너를 설정합니다.
- **`@CreatedDate`, `@LastModifiedDate`**: 생성 및 수정 시간을 자동으로 기록합니다.
- **`@CreatedBy`, `@LastModifiedBy`**: 생성 및 수정자를 기록합니다.

## 3. 엔티티에서 BaseEntity 상속

이제 각 엔티티에서는 `BaseEntity`를 상속받아 Auditing 기능을 간단하게 구현할 수 있습니다. 추가적인 필드를 정의하지 않아도, `BaseEntity`의 필드를 통해 자동으로 생성/수정 시간과 사용자 정보를 관리할 수 있습니다.

### 예시: User 엔티티

```java
@Entity
public class User extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;

    // Getters and Setters
}
```

- `User` 엔티티는 **`BaseEntity`**를 상속받아 공통 필드(`createdDate`, `lastModifiedDate`, `createdBy`, `lastModifiedBy`)를 자동으로 포함하게 됩니다.

## 4. AuditorAware 구현

`@CreatedBy`와 `@LastModifiedBy`를 사용하려면 **AuditorAware** 인터페이스를 구현하여, 현재 로그인한 사용자 정보를 제공해야 합니다. 이를 통해 엔티티의 생성자와 수정자를 기록할 수 있습니다.

### AuditorAware 구현 예시

```java
public class AuditorAwareImpl implements AuditorAware<String> {

    @Override
    public Optional<String> getCurrentAuditor() {
        // 현재 로그인한 사용자 정보를 반환하는 로직 (예: Spring Security와 연동)
        return Optional.of("admin"); // 실제 구현에서는 SecurityContextHolder 등을 활용
    }
}
```

- Spring Security와 연동하면, 현재 로그인한 사용자 정보를 가져와서 생성자/수정자에 기록할 수 있습니다.

### AuditorAware 설정

AuditorAware 구현체를 스프링 빈으로 등록해야 합니다.

```java
@Configuration
public class AuditConfig {

    @Bean
    public AuditorAware<String> auditorProvider() {
        return new AuditorAwareImpl();
    }
}
```

## 5. @Column(updatable = false)

`@Column(updatable = false)` 설정을 사용하면, 필드가 한 번 설정된 후 수정되지 않도록 할 수 있습니다. 주로 **`@CreatedDate`, `@CreatedBy`**와 함께 사용됩니다. 이렇게 하면 엔티티가 처음 생성될 때 값이 설정되고, 이후에는 변경되지 않게 보호됩니다.

```java
@CreatedDate
@Column(updatable = false)
private LocalDateTime createdDate;
```

- **`updatable = false`**: 해당 필드는 엔티티가 처음 저장된 후에는 수정할 수 없습니다.

## 등록자와 수정자, 날짜 정보를 분리한 상속 구조

**등록자, 수정자** 정보를 따로 관리하는 클래스와 **등록일자, 수정일자**를 관리하는 클래스를 따로 생성 후, **등록자, 수정자** 클래스는 **등록일자, 수정일자** 클래스를 상속받아서 사용한다면 유연하게 사용할 수 있습니다.

**이 구조는 대부분 엔티티는 날짜 정보는 있다는 가정** 하에 설계되었습니다.

- **모두 다 필요하다면** 등록자 관련 객체만 상속받아서 사용합니다.
- **등록자, 수정자가 필요 없을 때는** 날짜 정보 엔티티만 상속받아서 사용합니다.
