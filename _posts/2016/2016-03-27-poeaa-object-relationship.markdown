---
layout: post
title: "POEAA : 객체-관계형 동작 패턴"
date: '2016-03-27 17:34'
tags:
  - book
  - POEAA
---

# 작업 단위

_비즈니스 트랜잭션의 영향을 받은 객체의 리스트를 유지 관리하고, 변경 내용을 기록하는 일과 동시성 문제를 해결하는 일을 조율한다._

**작업의 일관성 유지를 위함**

_example 단위작업 기본 구현_

{% highlight java %}
class UnitOfWork {
    private List newObjects = new ArrayList();
    private List dirtyObjects = new ArrayLis();
    private List removedObjects = new ArrayList();

    // 추가된 객체
    public void registerNew(DomainObject obj) {
        Assert.notNull("id not null", obj.getId());
        Assert.isTrue("object not dirty", !dirtyObjects.contains(obj));<kbd></kbd>
        Assert.isTrue("object not removed", !removedObjects.contains(obj));
        Assert.isTrue("object not already registered new", !newObjects.contains(obj));
        newObjects.add(obj);
    }

    // 수정된 객체
    public void registerDirty(DomainObject obj) {
        Assert.notNull("id not null", obj.getId());
        Assert.isTrue("boject not removed", !removedObjects.contains(obj));
        if (!dirtyObjects.contains(obj) && !newObjects.contains(obj)) {
            dirtyObjects.add(obj);
        }
    }

    // 삭제된 객체
    public void registerRemoved(DomainObject obj) {
        Assert.notNull("id not null", obj.getId());
        if (newObjects.remove(obj)) return;
        dirtyObjects.remove(obj);
        if (!removedObjects.contains(obj)) {
            removedObjects.add(obj);
        }
    }

    // 조회된 객체
    public void registerClean(DomainObject ob) {
        AssertnotNull("id not null", obj.getId());
    }

    // 커밋
    public void commit() {
        insertNew();
        updateDirty();
        deleteRemoved();
    }

    private void insertNew() {
        for (Iterator objects = newObjects.Iterator(); objects.hasNext(); ) {
            DomainObject obj = (DomainObject) objects.next();
            MapperRegistry.getMapper(obj.getClass)).insert(obj);
        }
    }
}
{% endhighlight %}

_example 단위작업 조회 설정 구현(TheadLocal)_

{% highlight java %}
class UnitOfWork {
    ...
    private static ThreadLocal current = new ThreadLocal();

    public static void newCurrent() {
        setCurrent(new UnitOfWork());
    }

    public static void setCurrent(UnitOfWork uow) {
        current.set(uow);
    }

    public static UnitOfWork getCurrent() {
        return (UnitOfWork) current.get();
    }
}
{% endhighlight %}

_example DomainObject_
{% highlight java %}
class DomainObject {
    protected void markNew() {
        UnitOfWork.getCurrent().registerNew(this);
    }

    protected void markClean() {
        UnitOfWork.getCurrent().registerClean(this);
    }

    protected void markDirty() {
        UnitOfWork.getCurrent().registerDirty(this);
    }

    protected void markRemoved() {
        UnitOfWork.getCurrent().registereRemoved(this);
    }
}
{% endhighlight %}

_example Album_
{% highlight java %}
class Album extends DomainObject {
    public static Album create(String name) {
        Album obj = new Album(IdGenerator.nextId(), name);
        obj.markNew();
        return obj;
    }

    public void setTitle(String title) {
        this.title = title;
        markDirty();
    }
}
{% endhighlight %}

_example EditAlbumScript_
{% highlight java %}
class EditAlbumScript {
    public static void updateTitle(Long albumId, String title) {
        // 1. 작업단위 생성
        UnitOfWork.newCurrent();
        // 2. 작업 진행 : 수정을 통한 dirty 상태
        Mapper mapper = MapperRegistry.getMapper(Album.class);
        Album album = (Album) mapper.find(albumId);
        album.setTitle(title);
        // 3. 작업 완료 : 실질적 update 실행
        UnitOfWork.getCurrent().commit();
    }
}
{% endhighlight %}

서비스 계층에서 작업단위 핸들링

_example UnitOfWorkServlet_
{% highlight java %}
class UnitOfWorkServlet {
    final protected void doGet(HttpServletRequest request
            HttpservletResponse response) throws ServletException, IOException {
        try {
            UnitOfWork.newCurrent();
            handleGet(request, response);
            UnitOfWork.getCurrent().commit();
        } finally {
            UnitOfWork.setCurrent(null);
        }
    }

    abstract void handleGet(HttpServletRequest request,
            HttpservletResponse response) throws ServletException, IOException;
}
{% endhighlight %}

프레젠테이션 계층에서 작업단위 핸들링

# 식별자 맵

_모든 객체를 한 맵에 로드해 각 객체가 한 번씩만 로드되게 한다. 객체를 참조할 때는 맵을 이용해 객체를 조회한다._

## 고려사항

1. 키의 선택
    2. 일반적으로 테이블의 기본식별자(PK)
2. 명시적 또는 범용적
    3. 명시적 : `findPerson(1)`
    4. 범용적 : `find("Person", 1)`
3. 식별자 맵의 수
    4. 클래스 당 하나 : 명시적
    5. 전체 세션에 하나 : 범용적
4. 식별자 맵의 위치
    5. 작업 단위 위치
    6. 세션과 연결된 레지스트리
    7. **ThreadLocal** 추천 : 쓰레드에 안전해서 동기화 처리가 필요없음

**명시적 식별자(클래스 당 하나)를 추천**

값객체(Value Object)에는 식별자 맵을 사용할 필요가 없다.

## 식별자 맵의 장점

- 캐시를 통한 성능
- 엔티티에 `==` 가능

# 지연 로드

_필요한 데이터를 모두 포함하지는 않지만 데이터가 필요할 때 가져오는 방법을 아는 객체_

## 구현방법

1. 지연 초기화 : null 비교
2. 가상 프록시 : Proxy
3. 값 홀더 : Wrapper
4. 고스트 : 초기에는 비어있는(null아님)객체이나, 조회 시 실제 DB 접근
