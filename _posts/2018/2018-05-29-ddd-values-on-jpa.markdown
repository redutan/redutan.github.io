---
layout: "post"
title: "JPA에서 Value Object Collection 3가지 구현"
date: "2018-05-29 23:42"
tags:
    - DDD
    - JPA
---

# 동기

팀 내에서 IDDD(Implementing Domain-Driven Design) 스터디 중 **값 객체 Collection을 ORM(hibernate)을 이용하여 구현하는 예제**를 확인했습니다.
그러던 중 이게 아주 과거 hibernate xml 구성 기준이어서 현재 *JPA(Sprin Data JPA)에서는 어떻게 구현되는지 궁금*해 하던 `@동묘`가 저에게 직접 구현을 보고 싶다고 해서 포스팅 하게 되었습니다.

# 구현

3가지 구현 방법

1. Single Column
2. **Entity**
2. Join Table

**예시 도메인**

![Group과 GroupMember](http://www.plantuml.com/plantuml/svg/LO_12i8m38RlUugmex8Na57cIJo8uC5xreOoxThIfWSHtzreqR5Jal_F_v4CcJ5ncLqJKT_H4XnIA75lRIABJF1ijCESgmnzlpYN45WfMG3OsezxDD9wd4cAeQpJ57aANgRkwvze7YdbvhKWVwA0h-WAmNcyaQxOFuiVaIHKB-Ww1UscNOLtiE8Fv8ryz0O0)

* 그룹은 애그리게잇 루트(엔터티)입니다.
* 그룹맴버는 값 객체입니다.
* 한 그룹에는 여러 그룹맴버가 있습니다. (`Group#groupMembers`)
* 그룹과 그룹맴버는 한 애그리게잇으로 묶입니다.

## Many Values Serialized into a Single Column

`groups.group_members`에 그룹맴버들 객체를 직렬화해서 저장합니다. 여기에서는 `varchar(4000)` 데이터타입으로써 JSON으로 직렬화 하겠습니다.

### Java

*GroupMember.java*

```java
@Embeddable
@Value
public class GroupMember {
    private String name;
    @Enumerated(EnumType.STRING)
    private GroupMemberType type;

    // For JPA
    GroupMember() {
        this.name = null;
        this.type = null;
    }

    // For Jackson
    @JsonCreator
    public GroupMember(@JsonProperty("name") String name, 
                       @JsonProperty("type") GroupMemberType type) {
        this.name = name;
        this.type = type;
    }
}
```

*Group.java*

```java
@Entity
@Table(name = "GROUPS")
@Getter
@EqualsAndHashCode
@ToString
@NoArgsConstructor(access = AccessLevel.PACKAGE)
public class Group {
    @Id
    @GeneratedValue
    private Long groupId;
    private String description;
    private String name;

    /**
     * ORM과 한 열로 직렬화되는 여러 값 : ORM and Many Values Serialized into a Single Column
     */
    @Convert(converter = GroupMembersConverter.class)
    @Column(name = "GROUP_MEMBERS", length = 4000)
    private Set<GroupMember> group1Members = new HashSet<>();
    ...
}
```

*GroupMembersConverter.java*

```java
@Converter
public class GroupMembersConverter implements AttributeConverter<Set<GroupMember>, String> {
    private ObjectMapper om = new ObjectMapper();

    @Override
    public String convertToDatabaseColumn(Set<GroupMember> attribute) {
        return om.writeValueAsString(attribute);
    }

    @Override
    public Set<GroupMember> convertToEntityAttribute(String dbData) {
        return om.readValue(dbData, new TypeReference<Set<GroupMember>>() { });
    }
}
```

### SQL

*Schema*

```sql
create table groups (
    group_id bigint not null, 
    description varchar(255), 
    group_members varchar(4000), /* !!! */
    name varchar(255), 
    primary key (group_id)
);
```

*Insert*

```sql
insert into groups (
    group_id, description, group_members, name
) values (
    ?, ?, ?, ?
);
```

*저장 후 테이블 조회 예시*

| GROUP_ID | DESCRIPTION | GROUP_MEMBERS | NAME |
|----------|-------------|---------------|-------|
| 1 | 설명 | `[{"name":"하위그룹","type":"GROUP"},{"name":"회원1","type":"MEMBER"}]` | 이름 |

## Many Values Backed by a Database Entity 

실질적으로는 *Entity처럼 DB 스키마를 구성*하나 실제 객체지향세계(ex:Java Application)에서는 **값 객체로 보이게 구현**

* 이를 위해서 상속을 이용해서 **식별자 속성을 은닉**시키는 것이 구현의 핵심

### Java

*IdentifiedValueObject.java*

```java
@MappedSuperclass
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public abstract class IdentifiedValueObject {
    @Id
    @GeneratedValue
    @Getter(AccessLevel.PACKAGE)
    @Setter(AccessLevel.PACKAGE)
    private Long id;    // 패키지 접근제어를 통해서 식별자 은닉. 단, hibernate는 접근 가능
}
```

*GroupMember.java*

```java
@Entity
@Table(name = "GROUP_MEMBERS")
@Value
@EqualsAndHashCode(callSuper = false)
public class GroupMember extends IdentifiedValueObject {
    private String name;
    @Enumerated(EnumType.STRING)
    private GroupMemberType type;

    // For JPA
    GroupMember() {
        this.name = null;
        this.type = null;
    }
}
```

*Group.java*

```java
public class Group {
    ...
    /**
     * ORM과 데이터베이스 엔터티로 지원되는 여러 값 : ORM and Many Values Backed by a Database Entity
     */
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true) // Aggregate Root 를 위한 일관성 설정
    @JoinColumn(name = "GROUP_ID")
    private Set<GroupMember> groupMembers = new HashSet<>();
    ...
}
```

### SQL

*Schema*

```sql
create table groups (
    group_id bigint not null, 
    description varchar(255), 
    name varchar(255), 
    primary key (group_id)
);
create table group_members (
    id bigint not null,     // PK!!
    name varchar(255), 
    type varchar(255), 
    group_id bigint, 
    primary key (id)
)
alter table group_members 
    add constraint fk_groups_group_members
    foreign key (group_id) references groups;
```

*Insert*

```sql
insert into groups (group_id, description, name) values (?, ?, ?)
insert into group_members (name, type, id) values (?, ?, ?)
insert into group_members (name, type, id) values (?, ?, ?)
update group_members set group_id=? where id=?  /* Update ?! */
update group_members set group_id=? where id=?
```

* 일반적으로 가장 무난하며, IDDD 책에서는 저자가 가장 추천한 방식

## Many Values Backed by a Join Table

`group_members`을 테이블로 분리하는데 이 테이블은 PK가 없이 `groups`의 PK를 FK로 가지는 Join Table로 구현

### Java

*Group.java*

```java
public class Group {
    @Id
    @GeneratedValue
    private Long groupId;
    private String description;
    private String name;
    /**
     * ORM과 조인 테이블로 지원되는 여러 값 : ORM and Many Values Backed by a Join Table
     */
    @ElementCollection
    @CollectionTable(name = "GROUP_MEMBERS", joinColumns = @JoinColumn(name = "GROUP_ID"))
    private Set<GroupMember> groupMembers = new HashSet<>();
    ...
}
```

### SQL

*Schema*

```sql
create table groups (
    group_id bigint not null, 
    description varchar(255), 
    name varchar(255), 
    primary key (group_id)
);
create table group_members (    /* No PK */
    group_id bigint not null, 
    name varchar(255), 
    type varchar(255)
);
alter table group_members 
    add constraint fk_groups_group_members 
    foreign key (group_id) references groups;
```

*Insert*

```sql
insert into groups (group_id, description, name) values (?, ?, ?);
insert into group_members (group_id, name, type) values (?, ?, ?);
insert into group_members (group_id, name, type) values (?, ?, ?);
```

### 이슈

### 값 객체 Collection에 새로운 값이 추가되거나 기존 값이 변경될 시 All Delete and Re-insert

`@ElementCollection`을 통해 변경이 발생할 시, 전체를 지우고 다시 입력하는 이슈가 있음.
이를 완화시키기 위해서 `@OrderColumn`을 추가하는 방법이 있긴하나 완벽하지는 않음

```java
public class Group {
    ...
    @ElementCollection
    @CollectionTable(name = "GROUP_MEMBERS", joinColumns = @JoinColumn(name = "GROUP_ID"))
    @OrderColumn    // !!!
    private Set<GroupMember> groupMembers = new HashSet<>();
    ...
}
```

## Summary

![Total DB schema](https://github.com/redutan/ddd-values/raw/master/ddd-value-db-schema.png)

1. Single Column
2. Entity
3. Join Table

# Reference

* 소스코드 : [https://github.com/redutan/ddd-values](https://github.com/redutan/ddd-values)
* [Implementing Domain-Driven Design. Vaughn Vernon](http://www.acornpub.co.kr/book/implement-ddd)
* [자바 ORM 표준 JPA 프로그래밍. 김영한](http://acornpub.co.kr/book/jpa-programmig)
* [http://kwonnam.pe.kr/wiki/java/jpa/elementcollection](http://kwonnam.pe.kr/wiki/java/jpa/elementcollection)
