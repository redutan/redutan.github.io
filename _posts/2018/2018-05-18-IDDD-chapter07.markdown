---
layout: "post"
title: "IDDD 7장. 서비스"
date: "2018-05-18 00:55"
tags:
    - IDDD
    - DDD
    - BOOK
---

**도메인 서비스** : 도메인 모델 내 무상태 오퍼레이션 제공

* 애그리게잇이나 값 객체의 행위로써 어울리지 않는 것

*before*
```java
class Product {
    Set<BacklogItem> backlogItems;

    BusinessPriorityTotals businessPriorityTotals() {
        //...
    }
}
```

* 모델 정제를 통해 `Set<BacklogItem> backlogItems`를 분리
* 그렇다면 `businessPriorityTotals()` 메서드는 어떻게 되는가?

*after*
```java
class Product {
    static BusinessPriorityTotals businessPriorityTotals(
            Set<BacklogItem> aBacklogItems) {
        // ...
    }
}
```

* 정적 메소드로 변경 - 하지만 과연 이것은 옳은가?

**도메인 서비스** 가 필요한 시점이다!

## 도메인 서비스란 무엇인가

> 하지만 그보다 먼저 도메인 서비스가 아닌 것은 무엇인가?

* 도메인 서비스 : 도메인 로직이 있다.
* 애플리케이션 서비스 : 도메인 로직이 없다. (보안, 트랜잭션 등)

*정의*

* 엔터티나 값객체의 책임이 아닌 (도메인적) 행위
* 유비쿼터스 언어로써 표현
* 무상태!

*예시*

* *중요* 비즈니스 프로세스
    * *복수 개*의 도메인 객체에서 필요로 하는 계산
* 한 조합에서 다른 조합으로 도메인 객체를 *변형*할 때

## 서비스가 필요할지 확인하자

도메인 특화 지식은 클라이언트로 절대 유촐돼선 안된다.(클라이언트가 애플리케이션 서비스일지라도)
도메인 객체에 위치 시키기 애매하면 *도메인 서비스*에 담는다.

`주의`: 도메인 서비스를 남용하면 빈약한(Anemic) 도메인 모델이 된다.

## 도메인에서 서비스를 모델링하기

*AuthenticationService*
```java
package com.saasovation.identityaccess.domain.model.identity;   // !!

interface AuthenticationService {
    UserDescriptor authenticate(
        TenantId tenantId,
        String username,
        Strin password);
}
```

*EncryptionAuthenticationService*
```java
package com.saasovation.identityaccess.infra.services;   // !!

class EncryptionAuthenticationService implements AuthenticationService {
    @Override
    UserDescriptor authenticate(
            @NonNull TenantId tenantId,
            @NonNull String username,
            @NonNull Strin password) {

        Tenant tenant = tenantRepository.findById(tenantId);
        if (tenant == null || !tenant.isActive()) {
            throw new TenantNotFoundException(tenantId);
        }
        String encryptedPassword = 
                encryptService.encryptedValue(passwod); // 암호화 책임도 도메인 서비스?
        User user = userRepository.findByTenantAndUsernameAndPassword(
            tenant, username, encryptedPassword);
        if (user == null || !user.isEnabled()) {
            throw new UserNotFoundException(username);
        }
        return user.userDescriptor();
    }
}
```

### 분리된 인터페이스가 꼭 필요할까

다형성이 요구되지 않는다면 굳이 분리된 인터페이스는 필요하지 않다고 본다.
다형성 등으로 인한 추상화가 요구될 때 분리된 인터페이스로 리팩토링 하자

`생각` : 구현클래스에 `Impl` 접미사를 붙이는 것은 가능하면 피하는 것이 좋다고 본다.

DI 프레임워크(ex:Spring)를 사용하면 분리된 인터페이스를 사용하기 더 편하다.

### 계산 프로세스

`생각`: 계산(연산)은 변하지 않는 행동이므로 *분리된 인터페이스*는 필요하지 않다.

대부분 경우 계산 프로세스를 **정적 메서드**로 해결하는 경향이 많다.
이는 잘못된 행동이며 계산이 도메인 로직을 표현한다면 당연히 도메인 모델 내에 **도메인 서비스로써 존재**해야 한다.

`참고`: p.369 ~ 371 소스코드

### 변환 서비스

타 서비스와 통합을 위해 사용 
* 이것도 *도메인 서비스*였다니

`Adapter`

```java
package com.saasovation.collaboration.infra.services;

class UserInRoleAdapter {
    <T extends Collaborator> T toCollaborator(
            Tenant tenant, String identity, String role, 
            Class<T> collaboratorClass);
}
```

`Translator`

```java
package com.sasovation.collaboration.infra.services;

class CollbaratorTransaltor {
    <T extends Collaborator> T toCollaboratorFrom(
            String json, Class<T> collaboratorClass);
}
```

`생각`: translator도 도메인 서비스 인가?

### 도메인 서비스의 미니 계층 사용하기

가능하면 미니 계층은 사용하지 말자 - 안티패턴!

* 하지만 필요하다면 특정 도메인만 제한적으로 사용하는 것도 좋다.

## 마무리

* 도메인 서비스는 필요할 때 사용
    * 남용하면 빈약한 도메인 모델화
* 무상태
* VS 애플리케이션 서비스