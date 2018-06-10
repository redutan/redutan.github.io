---
layout: "post"
title: "IDDD 12장. 리파지토리"
date: "2018-06-10 16:00"
tags:
    - IDDD
    - DDD
    - BOOK
---

> 전역(global) 액세스가 필요한 각 객체의 타입미다, 해당 타입의 모든 객체가 담긴 인메모리 컬렉션이란 허상을 제공하는 객체를 생성하자.
> 잘 알려진 전역의 인터페이스를 통한 액세스를 설정하자.
> 객체를 추가하거나 제거하는 메소드를 제공하자...
> 일정 조건에 부합되는 특성을 갖는 객체를 선택해 완전히 인스턴스화된 객체나 여러 객체의 컬렉션으로 반환하는 메소드를 제공하자...
> 애그리게잇에 대해서만 리파지토리를 제공하자.. [Evans]

애그리게잇 : 리파지토리 = 1(..N):1

## 컬렉션 지향 리파지토리

```java
package java.util;

public interface Collection {
    public boolean add(Object o);
    public boolean addAll(Collection c);
    public boolean remove(Object o);
    public boolean removeAll(Collection c);
}
```

* 같은 인스턴스를 두번 추가되도록 허용해서는 안된다.
* *재저장*을 할 필요가 없다.
    * 그저 객체의 상태를 변경시키면 자동으로 저장될 뿐이다.
* 결국 `Set`처럼 행동해야 한다.

*재저장*할 필요가 없다면 객체의 상태를 추적해야한다.

*영속성 매커니즘의 암시적 변경 추적 기법*

* 암시적 읽기 시 복사(Copy-on-Read) : 읽을 때 복사본을 만들어 두고 복사본과 비교해서 변경된 것이 있으면 Commit 시킨다.
* 암시적 쓰기 시 복사(Copy-on-Write) : 프록시로 감싸두고 Dirty(상태 변경이 되면)인 경우 Commit 시킨다.

### 하이버네이트 구현

```java
package com.saasovation.collaboration.domain.model.calendar;    //!!

interface CalendarEntryRepository {
    void add(CalendarEntry calendarEntry);
    void addAll(Collection<CalendarEntry> calendarEntries);
    void remove(CalendarEntry calendarEntry);
    void removeAll(Collection<CalendarEntry> calendarEntries);

    CalendarEntry calendarEntryOfId(Tenant tenant, CalendarEntriyId id);
    Collection<CalendarEntry> calendarEntriesOfCalendar(
        Tenant tenant, CalendarId calendarId);
    Collection<CalendarEntry> overlappingCalendarEntries(
        Tenant tenant, CalendarId calendarId, TimeSpan timeSpan);

    CalendarEntryId nextIdentity();
}
```
* `package`는 도메인 모델과 함께한다.
* 물리적 삭제 VS **논리적 삭제**

```java
package com.saasovation.collaboration.infrastructure.persistence;

public class HibernateCalendarEntryRepository 
        implements CalendarEntryRepository {
    
    private final SpringHibernateSessionProvider sessionProvider;

    public HibernateCalendarEntryRepository(
            SpringHibernateSessionProvider sessionProvider) {
        this.sessionProvider = sessionProvider;
    }

    private org.hibernate.Session session() {
        return this.sessionProvider.session();
    }

    @Override
    public void add(CalendarEntry calendarEntry) {
        this.session().saveOrUpdate(calendarEntry);
    }

    @Override
    public void addAll(Collection<CalendarEntry> calendarEntries) {
        for (CalendarEntry each : calendarEntries) {
            this.session().saveOrUpdate(each);
        }
    }

    @Override
    public void remove(CalendarEntry calendarEntry) {
        this.session().delete(calendarEntry);
    }

    @Override
    public void removeAll(Collection<CalendarEntry> calendarEntries) {
        for (CalendarEntry each : calendarEntries) {
            this.session().delete(each);
        }
    }
}
```

* package : `infrastructure.persistence`
* `Set`과 유사한 행위 제공
* `Cascade`를 통한 연관된 엔터티를 같이 변경하는 기능도 있음
* 복잡한 조회의 경우 HQL을 이용
    * 하지만 JPA를 사용한다면 JPQL을 사용하나, 현재까지는 querydsl 과 같은 기술이 좋은 것 같다.

### 탑링크 구현에 대한 고려

탑링크는 명시적으로 [작업단위(Unit of work)](https://martinfowler.com/eaaCatalog/unitOfWork.html)를 지정할 수 있다.

* 일종의 트랜잭션의 범위를 추상화 시킨 것

```java
Calendar calendar = session.readObject(...);
UnitOfWork work = session.acquireUnitOfWork();
Calendar calendarToRename = work.registerObject(calendar);
calendarToRename.rename("CollabOvation Project Calendar");
work.commit();
```

```java
package com.saasovation.collaboration.infrastructure.persistence;

public class ToplinkCalendarEntryRepository 
        implements CalendarEntryRepository {
    @Override
    public void add(Calendar calendar) {
        this.unitOfWork().registerNewObject(calendar);
    }

    @Override
    public void editingCopy(Calendar calendar) {
        return (Calendar) this.unitOfWork().registerObject(calendar);
    }
}
```

## 영속성 지향의 리파지토리

* 컬렉션 지향 리파지토리 : `Set`
* 영속성 지향 리파지토리 : `HashMap`
    * 하지만 꼭 `put`를 해야한다. - 원자적 쓰기를 통제할 수 없음(트랜잭션 없음)

No-Sql 종류가 많다.

* 잼파이어
* 코히어런스
* 몽고DB
* 리악 

### 코히어런스 구현

```java
package com.saasovation.agilepm.domain.model.product;

interface ProductRepository {
    ProductId nextIdentity();
    Collection<Product> allProductsOfTenant(Tenant tenant);
    Product productOfId(Tenant tenant, ProductId productId);
    void remove(Product product);
    void removeAll(Collection<Product> products);
    void save(Product product);
    void saveAll(Collection<Product> products);
}
```

```java
package com.saasovation.agilepm.infrastructure.persistence;

class CoherenceProductRepository 
        implements ProductRepository {
    private Map<Tenant, NamedCache> caches;

    public CoherenceProductRepository() {
        this.caches = new HashMap<>();
    }

    private synchronized NamedCache(TenantId tenantId) {
        NamedCache cache = this.caches.get(tenantId);
        if (cache == null) {
            // ageilepm:Product:TenantId
            // 1단계 : 2단계 : 3단계
            cache = CacheFactory.getCache(
                "agilepm.Product." + tenantId.id(),
                Product.class.getClassLoader());
            this.caches.put(tenantId, cache);
        }
        return cache;
    }

    @Override
    public void save(Product product) {
        this.cache(product.tenantId())
            .put(this.idOf(product), product);
    }

    @Override
    public void saveAll(Collection<Product> products) {
        Map<String, Product> productsMap = 
            new HashMap<>(products.size());
        for (Product each : products) {
            if (tenantId == null) {
                tenantId = product.tenantId();
            }
            productsMap.put(this.idOf(product), each);
        }
        this.cache(tenantId).putAll(productsMap);
    }

    private String idOf(Product product) {
        return this.idOf(product.productId());
    }
    private String idOf(ProductId productId) {
        return productId.id();
    }

    @Override
    public void remove(Product product) {
        this.cache(product.tenantId()).remove(this.idOf(product));
    }

    @Override
    public void removeAll(Collection<Product> products) {
        for (Product each : products) {
            this.remove(product);
        }
    }

    @Override
    public Collection<Product> allProductsOfTenant(Tenant tenant) {
        Set<Map.Entry<String, Product>> entries = 
            this.cache(tenant).entrySet();
        Collection<Product> products = 
            new HashSet<Product>(entries.size());
        for (Map.Entry<String, Product> entry : entries) {
            products.add(entry.getValue());
        }
        return products;
    }

    @Override
    public Product productOfId(Tenant tenant, ProductId productId) {
        return (Product) this.cache(tenant).get(this.idOf(productId));
    }
}
```

### 몽고DB 구현

1. 애그리게잇을 -> 몽고DB포맷(직렬화), 몽고DB포맷 -> 애그리게잇(역직렬화)
2. 몽고DB 문서의 고유 식별자(`_id`)
3. 몽고DB 노드/클러스터 참조

```java
class MongoProductRepository 
        extends MongoRepository<Product>
        implements ProductRepository {
    
    public MongoProductRepository() {
        super();
        this.serializer(new BSONSerializer<Product>(Product.class));
    }

    public ProductId nextIdentity() {
        // 몽고DB에서 제공하는 식별자. _id 필드에 매핑시킬수 있음
        return new ProductId(new ObjectId().toString());
    }

    @Override
    public void save(Product product) {
        this.databaseCollection(
                this.collectionName(product.tenantId()))
            .save(this.serialize(product));
    }

    protected String collectionName(TenantId tenantId) {
        return "product" + tenantId.id();
    }

    protected String databaseName() {
        return "agilepm";
    }

    @Override
    public Collection<Product> allProductsOfTenant(
            TenantId tenantId) {
        Collection<Product> products = new ArrayList<>();
        DBCursor cursor = 
            this.databaseCollection(
                this.databaseName(),
                this.collectionName(tenantId)).find();
        while (curosr.hasNext()) {
            DBObject dbObject = cursor.next();
            Product product = this.deserialize(dbObject);
            products.add(product);
        }
        return products;
    }

    @Override
    public Product productOfId(
            TenantId tenantId, ProductId productId) {
        Product product = null;
        BasicDBObject query = new BasicDBObject();
        query.put("producetId", 
            new BasicDBObject("id", productId.id()));
        DBCursor cursor = 
            this.databaseCollection(
                this.databaseName(),
                this.collectionName(tenantId)).find(query);
        if (cursor.hasNext()) {
            product = this.deserialize(cursor.next());
        }
        return product;
    }
}
```

```java
class BSONSerializer<T> {
    DBObject serialize(String key, T object) {
        DBObject serialization = this.serialize(object);
        serialization.put("_id", new ObjectId(key));
        return serialization;
    }
}
```


```java
abstract class MongoRepository<T> {

    protected DBCollection databaseCollection(
            String databaseName, String collectionName) {
        return MongoDatabaseProvider
                .database(databaseName)
                .getCollection(collectionName);
    }
}
```

* `BSONSerializer` 때문에 `Setter`를 제공할 필요가 없음

## 추가적인 행동

* 예를 들면 `count()` or `size()`
* 또는 리파지토리에서 애그리게잇 파트를 쿼리하는 것 (성능상 잇점)
    * 하지만 이건 안티패턴이기 때문에 가능한 사용하지 않는 것이 좋다 (애그리게잇 법칙 위배)
    * 만약 그게 당연시 하다고 느껴지면 애그리게잇을 분리하는 것도 한 방법
* 일반적으로 도메인 서비스 제어하가 어울린다.

하지만 너무 유스케이스에 최적화된 조회 메서드를 많이 제공한다면 악취일 수도 있다

* 그런 경우 **CQRS**를 고려해보자

## 트랜잭션의 관리

트랜잭션 관리는 애플리케이션 계층의 책임이다. = 애플리케이션 서비스

*트랜잭션 관리 방법*

```java
class ApplicationServiceFacade {
    // 명식적인 트랜잭션
    public void doSomeUseCaseTask1() {
        Transaction transaction = null;
        try {
            transaction = this.session().beginTransaction();
            // 도메인 모델을 사용한다.
            transaction.commit();
        } catch (Exception e) {
            if (transaction ! null) {
                transaction.rollback();
            }
        }
    }

    // 선언적인 트랜잭션
    @Transactional
    public void doSomeUseCaseTask2() {
        // 도메인 모델을 사용한다.
    }
}
```

* 선언적인 방식이 더 낫다. 순수하게 도메인 로직에 위임하는 것에 집중할 수 있다.
    * 관심사 분리!!

### 경고

단일 트랜잭션에서 여러 애그리게잇을 수정을 커밋하는 기능을 과도하게 사용하지 않도록 주의하라.

## 타입 계층구조

![Type Hierarchy](http://www.plantuml.com/plantuml/svg/SoWkIImgAStDuKhEIImkLWXEBIhBJ4uDACeloqn9BLAevb9GAAaiIEMgXIe8JonAoab5LvPQKPAQbuAX7QOdFo-RU3qEG56WWm00)

```java
// 도메인 모델의 클라이언트
serviceProviderRepository.providerOf(id)
        .scheduleService(date, description);
```

* 상위 타입(`ServiceProvider`)으로 처리

```java
// 도메인 모델의 클라이언트
if (id.identifiesWarble()) {
    serviceProviderRepository.warbleOf(id)
            .scheduleWarbleService(date, warbleDescription);
} else if (id.identifiesWonkle()) {
    serviceProviderRepository.wonkleOf(id)
            .scheduleWonkleService(date, wonkleDescription);
}
```

* 하위 타입을 클라이언트에서 알고 분기 처리 해야하는 책임이 늘어난다.
* 위 코드는 전형적인 악취이다. 

**그러므로 상위타입은 하위타입에 대한 구분정보(`type` 속성)를 알고 있어야 한다.**

* 그렇게 된다면 Repository 계층에서 캡슐화 시켜서 하위 타입 별 분기 처리를 할 수 있을 것이다.

```java
class ServiceProvider {
    private ServiceType type;

    public void scheduleService(Date date, ServiceDescription description) {
        if (type.isWarble()) {
            this.scheduleWarbleSevice(date, description);
        } else if (type.isWonkle()) {
            this.scheduleWonkleService(date, description);
        } else {
            this.scheduleCommonService(date, description);
        }
    }
}
```

* 나라면 위의 경우에서도 `ServiceType`으로 위임해서 `if`문을 제거했을 것 같다.

## 리파지토리 대 데이터 액세스 객체

Repository != DAO

* 리파지토리 : 객체 지향 > with 도메인 모델 패턴
* DAO : 데이터 지향 > with 트랜잭션 스크립트 패턴

SMART DAO는 DDD 입장에서는 안티패턴

* SMART DAO는 도메인 로직이 DB 쿼리나 DB의 프로시저에 존재하는 경우를 말함

**중요한 것은 데이터 액세스 지향보다는 컬렉션 지향으로 설계하려고 노력해야하는 점**

## 리파지토리의 테스트

* 실제 인프라와 연동하는 테스트 (실제 DB와 통신)
* 인메모리로 연동하는 테스트

> 개인적으로 실제 인프라와 연동하는 것이 중요하다고 봄.

### 인메모리 구현으로 테스트하기

Skip

## 마무리

* 컬랙션 지향 VS 영속성 지향
* 리파지토리의 추가적인 행동 (`count()`)
* 트랜잭션 처리
* 타입 계층과 리파지토리
* Repository vs DAO
* 테스트
