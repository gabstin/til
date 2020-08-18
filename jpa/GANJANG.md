---
layout: post  
title: "JPA 엔티티 관계 설정하기"  
subtitle: "[Tips]"  
date: 2020-03-02 17:00  
background:   
tag: [Tips, Github io, Notion]
---


# JPA 엔티티 관계 설정하기

목차 
1. Member 와 Order 엔티티 설정
1-1 Member와 Order 엔티티 일대다 관계 설정 방법

2. Order와 Delivery 엔티티 설정 
2-1 Enumerated 을 이용한 Enum클래스 설정 

3. Item 상속관계 클래스 만들기

4. Order와 OrderItem과 Item 매핑 

5. Category와 Item 매핑

- [소스자료 https://github.com/jnhnC/entityrelation.git](https://github.com/jnhnC/entityrelation.git)

![https://lh4.googleusercontent.com/WItbIURzsvJsSNCpA24Wp62oH4Q5ouUcx9UU4SnLwnrpx-Y1lXv6XpE31ICfKKLe9vpQJvPXXs5XifWTcbb2G9odHBKdaML0UWnyMn9-nOLvjGY8KETz4UyM7KozM4gIYGg9NJAA](https://lh4.googleusercontent.com/WItbIURzsvJsSNCpA24Wp62oH4Q5ouUcx9UU4SnLwnrpx-Y1lXv6XpE31ICfKKLe9vpQJvPXXs5XifWTcbb2G9odHBKdaML0UWnyMn9-nOLvjGY8KETz4UyM7KozM4gIYGg9NJAA)

### **1. Member 와 Order 엔티티 설정**

![https://lh3.googleusercontent.com/UQ1u25dmTPYSi9rjfW8dF3NihCpFHneomagYTlA3o31xtSNpFrLk9nmLN5xa2pW6vwQGZIcqSVpvOmvzgEWDDGF4IyxAtUwaJfWj_2ZVgaQASHlgWsXjfH1mniNxLz5h0LyWS4xY](https://lh3.googleusercontent.com/UQ1u25dmTPYSi9rjfW8dF3NihCpFHneomagYTlA3o31xtSNpFrLk9nmLN5xa2pW6vwQGZIcqSVpvOmvzgEWDDGF4IyxAtUwaJfWj_2ZVgaQASHlgWsXjfH1mniNxLz5h0LyWS4xY)

**`다대일 관계`**

Meber와 Order의 관계매핑은 다대일 관계로 Order(다) Member(일) 의 관계로 정의 되어 있다,

다대일의 관계는 가장 많이 사용되는 연관관계이며 @ManyToOne으로 다대일의 관계를 정의한다.

양방향 다대일로 매핑하는 경우 ‘다’쪽에 @ManyToOne, ‘일’쪽에 @OneToMany로 정의한다.

외래키가 있는 쪽을 주인엔티티, 그 반대엔티티를 가짜엔티티라고 말하며

주인엔티티는 @JoinColunm 으로 가짜엔티티는 mappedBy로 설정한다.

주인엔티티로 설정되면 등록, 수정, 조회, 삭제 함께 이루어지고

가짜엔티티는 조회기능만 가능하다.

Member.class

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Member {
   @Id   @GeneratedValue
   @Column(name="member_id")
   private Long id;

   private String name;

   //가장 중요한 부분
   @OneToMany(mappedBy = "member")
   private List<Order> orders = new ArrayList<>();
}
```

Order.class

```java
@Entity
@Getter
@Setter
@Table(name="orders")
public class Order {
   @Id    @GeneratedValue
   @Column(name="order_id")
   private Long id;
   
   //가장 중요한 부분
   @ManyToOne(fetch=FetchType.LAZY)
   @JoinColumn(name ="member_id")
   private Member member;
}
```

* 설명

다대일 양방향 연관관계에서 가장 중요한 부분은 연관관계의 주인엔티티를 설정하는 부분이다.

Order.class

```
@ManyToOne(fetch=FetchType.LAZY) // FetchType을 무조건 지원로딩(Lazy)로 설정
   @JoinColumn(name ="member_id")   // @JoinColumn으로 주인엔티티임을 설정
   private Member member;
```

**@ManyToOne(fetch=FetchType.LAZY)**

@ManyToOne으로 다대일을 설정하고 fetch 설정을 지원로딩으로 변경한다. ManyToOne, OneToOne은 기본 즉시로딩으로 설정되어 있기에 무조건 지원로딩으로 변경해야한다. 즉시로딩은 사용하지 않는편이 좋다.

- **@JoinColumn(name ="member_id")**

주인엔티티를 설정하는 단계로 name안에 외래키 값을 넣어준다.

Member.class

```
@OneToMany(mappedBy = "member") // 다대일로 매핑되어있으니 일대다로 양방향을 설정
   private List<Order> orders = new ArrayList<>();
```

**@OneToMany(mappedBy = "member")**

다대일 양방향의 매핑시 설정하는 방법으로 조회기능만 가능하다.

예를들어 회원를 조회하고 회원이 주문한 내역을 같이 볼때 사용될수 있다.

mappedBy = “member”는 Order.class에서 정의된 member를 말한다.

### 1- 2. Embedded 클래스 정의

Integer, String 처럼 사용자가 직접 값 타입을 정의할 수 있는 클래스 이다.
@Embedded 선언하여 임베디드 값임을 정의 한다.
대표적으로 주소라고 회원엔티티에 정의하고 도로명, 지번, 상세주소는 다른클래스에 정의하여 가져와
사용하는 방식이다.

Member.class

```java
@Entity
   @Getter
   @Setter
   @NoArgsConstructor
   public class Member {

      @Id
      @GeneratedValue
      @Column(name="member_id")
      private Long id;

      private String name;

      @Embedded //  //가장 중요한 부분 Embedded 설정 
      private Address address;

      @OneToMany(mappedBy = "member")
      private List<Order> orders = new ArrayList<>();
    }
```

Address.class

```java
@Embeddable //  //가장 중요한 부분 Embedded 클래스 정의
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Address {
   private String city;
   private String street;
   private String zipcode;

   public Address(String city, String street, String zipcode){
       this.city =city;
       this.street = street;
       this.zipcode = zipcode;
   }
}
```

* 설명

Embedded 클래스 설정으로 깔끔하게 엔티티를 관리 할 수 있다.

추가하고 싶은 엔티티에 @Embedded로 설정한다

Member.class

```
@Embedded // Embedded 설정 
  private Address address;
```

Address.class

```
@Embeddable // Embedded 클래스 선언
  public class Address {
```

엔티티클래스는 상단에 @Embeddable로 정의해서 사용한다.

### 2. Order와 Delivery 엔티티 설정

![https://lh6.googleusercontent.com/ThZp1g5cLJhIcgQD-esPAsh69ler0cbT7S7NPxtPFf92-nluDHUM7iXzmUjFigUsqZGoTDM77bAMiqNLiEVKv67JakAGZUsTrtZNVY_R8CaIYYyTytUqqWcGqKpYFUQtIsZ7liE1](https://lh6.googleusercontent.com/ThZp1g5cLJhIcgQD-esPAsh69ler0cbT7S7NPxtPFf92-nluDHUM7iXzmUjFigUsqZGoTDM77bAMiqNLiEVKv67JakAGZUsTrtZNVY_R8CaIYYyTytUqqWcGqKpYFUQtIsZ7liE1)

`**일대일 관계**`

Orderd와 Delivery의 관계매핑은 일대일 관계로 Order(일) Delivery(일) 의 관계로 정의 되어 있다,

일대일의 관계는 @OneToOne으로 일대일의 관계를 정의한다.

양방향 일대일로 매핑하는 경우 @OanyToOne하고

외래키가 있는 주인엔티티는 @JoinColunm 으로 가짜엔티티는 mappedBy로 설정한다는 면에서 다대일과 동일하게 설정하면 된다.

다만 외래키를 어디다 둘것인가의 고민을 해야한다.

일대일이기에 어느쪽이든 외래키를 두는 건 가능하다.

주로 사용되는 테이블에 둘것인지, 아니면 대상테이블에 둘것인가를 고민해야한다. 장단점을 비교해보자

**일대일 관계 외래키 설정 장단점**

1. 주 테이블에 외래키를 둘 경우

장점 : 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능

단점 : 값이 없으면 외래 키에 “null 허용”

2. 대상 테이블에 외래키를 둘 경우

장점 : 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조유지

단점 : 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 “즉시 로딩”됨(프록시는 뒤에서 설명)

간단하게 다시 설명하자면
   주테이블에 외래키 사용할 경우 성능상 우위
   대상테이블에 외래키 사용할 경우 null 값 없이 테이블 데이터 관리 효율적이다.

어떻게 보면

주테이블에 넣은것은 개발자 입장에 맞는 방법이고 대상테이블에 넣는것은 DBA에게 맞는 방법이다.

개발자입장에서 일대일 관계일땐 주테이블에 넣는 것이 맞다.

그렇다면 이 문제에 대해서 미래에 어떻게 바뀔것인가를 생각보자

미래에는 다대일 관계로 변경될 가능성이 없을까?

주문 하나에 배송지 두개로 가질 경우, 또는 여러주문을 배송지 하나로 보낼 경우를 생각해보고 다대일로 변경될 가능성까지 확인해야한다.

그 경우가 없다면 DBA를 잘 상의해서 주테이블에 넣는 방법으로 잘 설득하자!

Order.class

```java
@Entity
@Getter
@Setter
@Table(name="orders")
public class Order {
   @Id    @GeneratedValue
   @Column(name="order_id")
   private Long id;
   private LocalDateTime orderDate;

   //Member와 다대일 매핑
   @ManyToOne(fetch = FetchType.LAZY)
   @JoinColumn(name ="member_id")
   private Member member;

   //Delivery와 일대일 매핑
   //가장 중요한 부분
   @OneToOne(fetch = FetchType.LAZY)
   @JoinColumn(name="delivery_id")
   private Delivery delivery;
}
```

Delivery.class

```java
@Entity
@Getter
@Setter
public class Delivery {
   @Id   @GeneratedValue
   @Column(name="delivery_id")
   private Long id;

   private String status;

   //일대일 양방향 매핑 관계 설정
   //가장 중요한 부분
   @OneToOne(mappedBy = "order")
   private Order order;

}
```

* 설명

일대일 양방향 연관관계에서 가장 중요한 부분은 어디에 외래키를 둘 것인가이다.

주엔티티에 넣는것이 성능상 우위에 있으나 null 데이터가 들어가는 단점이 있다.

설정부분은 다대일과 거의 동일하다.

Order.class

```
//Delivery와 일대일 매핑
@OneToOne(fetch = FetchType.LAZY)
  @JoinColumn(name="delivery_id")
  private Delivery delivery;
```

**@OnyToOne(fetch=FetchType.LAZY)**

@OanyToOne으로 일대일을 설정하고 fetch 설정을 지원로딩으로 변경한다. ManyToOne, OneToOne은 기본 즉시로딩으로 설정되어 있기에 무조건 지원로딩으로 변경해야한다. 즉시로딩은 사용하지 않는편이 좋다.

**@JoinColumn(name="delivery_id")**

주인엔티티를 설정하는 단계로 name안에 외래키 값을 넣어준다.

Delivery.class

```
@OneToOne(mappedBy = "order")
  private Order order;
```

**@OneToMany(mappedBy = "order")**

양방향의 매핑시 설정하는 방법으로 조회기능만 가능하다.

예를들어 배송목록을 조회하고 주문한 내역을 같이 볼 때 사용될수 있다.

mappedBy = “order”는 Order.class에서 정의된 order을 말한다.

### 2-1. Enumerated 을 이용한 Enum클래스 설정

배송상태의 경우 배송준비중, 배송중, 배송완료 이런 식으로 배송상태의 문구가 정해져 있는경우가 많다.

이런 경우 배송상태가 동일하게 저장되는 걸 유지하기 위해 자바에서는Enum을 이용하여 상태문구를 지정하는 경우가 있다.

JPA에서 이것을 지원하는데 @Enumerated라는 어노테이션으로 설정이 가능하다.

Delivery.class

```
@Entity
@Getter
@Setter
public class Delivery {
   @Id
   @GeneratedValue
   @Column(name="delivery_id")
   private Long id;

   // 가장 중요한 부분 해당 배송상태를 Enum으로 변경
   // private String status;

   @Enumerated
   private DeliveryStatus status;
   //일대일 양방향 매핑 관계 설정
   @OneToOne(mappedBy = "delivery")
   private Order order;
```

DeliveryStatus.class

```
public enum DeliveryStatus {
   READY, COMP
}
```

* 설명

@Enumerated를 Enum으로 사용할 엔티티에 설정하고

enum.class를 생성하여 문구를 정의해준다.

delivery.class

```
@Enumerated
   private DeliveryStatus status;
```

**@Enumerated**

Enum을 설정할 엔티티 변수에 정의해준다.

DeliveryStatus.class

```
public enum DeliveryStatus {
     READY, COMP
  }
```

**public enum DeliveryStatus{}**

enum.class를 만들어서 해당 문구를 정의한다.

![https://lh6.googleusercontent.com/Qz_fsTZ0KFEO1jj3wwDObyEEaD3UsHnXQfql9bgsRows3WkTbr8RHB5mJIS8EK2XM0P_S7tnReeGemhUuNMDubE3aOwAKuvEzbjAqoZkfYopBnI2-ttKdQaCH_aOAoelTqVEFx0Z](https://lh6.googleusercontent.com/Qz_fsTZ0KFEO1jj3wwDObyEEaD3UsHnXQfql9bgsRows3WkTbr8RHB5mJIS8EK2XM0P_S7tnReeGemhUuNMDubE3aOwAKuvEzbjAqoZkfYopBnI2-ttKdQaCH_aOAoelTqVEFx0Z)

`**상속관계**`

책, 영화, 앨범 등 상품을 테이블로 정의할때 공통으로 사용하는게 많다.

이름, 수량, 가격 등 상품안에 중복되는 내용을 위해 객체에선 상속관계로 정의 할수있는데 DB에서는 상속관계가 없기에 이런 경우를 구현이 필요하다.

상속관계 구현 방식은 3가지로 나눌 수 있다.

조인전략, 단일전략, 각자테이블전략 으로 나뉜다.

각자테이블전략은 쓰지 않는게 좋고 조인전략, 단일전략 둘중 선택하여 사용하면된다.

조인전략의 경우 아무래도 조인을 하여 가져와야함으로 구조적으로 중요한 테이블경우 사용하면된다.

단일전략은 크게 비중이 없다고 생각이 들면 단일전략으로 사용하면 좋다.

여기서는 조인전략으로 사용하여 구현해보겠다.

Item.class

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)  //가장 중요한 부분
@DiscriminatorColumn
@Getter @Setter
public abstract class Item {

   @Id @GeneratedValue
   @Column(name = "item_id")
   private Long id;

   private String name;

   private int price;

   private int stockQuantity;

  //가장 중요한 부분
   @Column(insertable = false, updatable = false)
   private String dtype;

}
```

Book.class

```java
@Entity
@Getter
@Setter
public class Book extends Item {  //가장 중요한 부분

   private String author;

   private String isbn;
}
```

Album.class

```java
@Entity
@Getter
@Setter
public class Album extends Item { //가장 중요한 부분

   private String artist;

   private String etc;
}
```

Movie.class

```java
@Entity
@Getter
@Setter
public class Movie extends Item { //가장 중요한 부분

   private String director;

   private String actor;
}
```

* 설명

공통이 되는 Item 클래스에 해당 상속관계 어노테이션을 정의한다.

부모가 되는 엔티티에 @Inheritance를 설정하고 옵션을 InheritanceType옵션을 JOINED, SINGLE_TABLE

둘 중에 하나로 설정한다.

Item.class

```
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn
```

**@Inheritance(strategy = InheritanceType.JOINED)**

상속의 부모엔티티임을 설정한다. JOINED, SINGLE_TABLE을 설정한다.

JOINED : 자식엔티티를 각각 테이블을 만들고 조인으로 가져오는 형태

SINGLE_TABLE : 자식엔티티을 단일 테이블로 설정하여 사용하는 형태(조인필요없음)

**@DiscriminatorColumn**

자식 엔티티의 이름을 부모 엔티티에 저장하는 역할을 한다.

item 테이블에 dtype컬럼으로 저장되면 Book테이블에 데이터가 들어가면 Book이라고 저장된다.

```
@Column(insertable = false, updatable = false)
private String dtype;
```

변수 dtype을 정의해야 가져와서 사용할 수 있다.

```
public class Book extends Item {
public class Album extends Item {
public class Movie extends Item {
```

자식엔티티는 부모엔티티를 상속만 받으면 설정이 완료된다.

참고: @DiscriminatorValue("A") 상속받을 테이블에 해당 어노테이션을 설정한 경우 dtype에 A로 임의의

값을 정의할수 있다. 사용하지 않으면 테이블이름이 저장된다. 여기서는 사용하지 않겠다.

### **4. Order와 OrderItem과 Item 매핑**

![https://lh4.googleusercontent.com/mvShsT49feSmbRs9zSswOQQVAOyGIZ-Z_EdnK-olbMz0nXf72Sb43qiduG7ZStSJM1TdVgbTpNKa82BAFw61IdyWxmcNM8jenAK6EMB5BJzwdvAZ5JAuLNvmhuMzdJ1g59stNgpd](https://lh4.googleusercontent.com/mvShsT49feSmbRs9zSswOQQVAOyGIZ-Z_EdnK-olbMz0nXf72Sb43qiduG7ZStSJM1TdVgbTpNKa82BAFw61IdyWxmcNM8jenAK6EMB5BJzwdvAZ5JAuLNvmhuMzdJ1g59stNgpd)

`**양방향 다대일 매핑**`

양방향 다대일 매핑은 위에 다대일 매핑에서 설명했다.

Order.class를 확인하면 해당 매핑관계를 설정하는데 도움이 될것이다.

다시 한번 설명하겠지만 먼저 다대일 매핑관계를 먼저 설정 후에 일대다 관계를 설정하는게 좋다.

그림에서 보면 다대일 관계는 OrderItem이 ‘다’ 이며 Order와 Item이 ‘일’ 관계이다.

그럼 OrderItem에 외래키가 있을 것이고 외래키에 있는 쪽에 @JoinColumn 설정을 하므로써

주인 엔티티임을 설정하면 된다.

다대일 매핑이 설정되었으면 양방향 관계인 일대다 관계를 Order와 Item에서 설정해준다.

설정방법은 @OneToMany(mappedby = “변수명”) 으로 설정하면 된다.

OrderItem.class

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class OrderItem {

   @Id  @GeneratedValue
   @Column(name = "order_item_id")
   private Long id;

   private int orderPrice;

   private int count;

   //가장 중요한 부분
   @ManyToOne(fetch = FetchType.LAZY)
   @JoinColumn(name = "item_id")
   private Item item;

   //가장 중요한 부분
   @ManyToOne(fetch = FetchType.LAZY)
   @JoinColumn(name = "order_id")
   private Order order;
}
```

Order.class

```java
@Entity
@Getter
@Setter
@Table(name="orders")
@NoArgsConstructor
public class Order {

   @Id    @GeneratedValue
   @Column(name="order_id")
   private Long id;

   private LocalDateTime orderDate;

   //Member와 다대일 매핑
   @ManyToOne(fetch = FetchType.LAZY)
   @JoinColumn(name ="member_id")
   private Member member;

   //Delivery와 일대일 매핑
   @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
   @JoinColumn(name="delivery_id")
   private Delivery delivery;

   //가장 중요한 부분
   @OneToMany(mappedBy = "order")
   private List<OrderItem> orderItems = new ArrayList<>();
}
```

Item.class

```java
@Entity
@Getter
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn
@Getter @Setter
public abstract class Item {

   @Id @GeneratedValue
   @Column(name = "item_id")
   private Long id;

   private String name;

   private int price;

   private int stockQuantity;

   @Column(insertable = false, updatable = false)
   private String dtype;

   //가장 중요한 부분
   @OneToMany(mappedBy = "items")
   private List<OrderItem> orderItems = new ArrayList<>();

}
```

* 설명

다대일 양방향 연관관계에서 가장 중요한 부분은 연관관계의 주인엔티티를 설정하는 부분이다.

OrderItem.class

```
   @ManyToOne(fetch = FetchType.LAZY)
   @JoinColumn(name = "item_id")
   private Item item;

   @ManyToOne(fetch = FetchType.LAZY)
   @JoinColumn(name = "order_id")
   private Order order;
```

**@ManyToOne(fetch=FetchType.LAZY)**

@ManyToOne으로 다대일을 설정하고 fetch 설정을 지원로딩으로 변경한다. ManyToOne, OneToOne은 기본 즉시로딩으로 설정되어 있기에 무조건 지원로딩으로 변경해야한다. 즉시로딩은 사용하지 않는편이 좋다.

**@JoinColumn(name ="item_id")**

주인엔티티를 설정하는 단계로 name안에 외래키 값을 넣어준다.

order.class

```
@OneToMany(mappedBy = "order")
   private List<OrderItem> orderItems = new ArrayList<>();
```

item.class

```
@OneToMany(mappedBy = "items")
   private List<OrderItem> orderItems = new ArrayList<>();
```

**@OneToMany(mappedBy = "order")**

**@OneToMany(mappedBy = "items")**

다대일 양방향의 매핑시 설정하는 방법으로 조회기능만 가능하다.

예를들어 회원를 조회하고 회원이 주문한 내역을 같이 볼때 사용될수 있다.

### 5. Category와 Item 매핑

![https://lh5.googleusercontent.com/jSYkngLev2Vgaoub185U2R50fPL0SDnSyJayhn9iPrBAkjZvxapAf3iK2PqLDETiSaHi8HPZhKs29aof3rTr8AeAYhMB8r2cnE93hdVbcP-AN6d5ClfLmxIPjylzTzWPtAwnVwlg](https://lh5.googleusercontent.com/jSYkngLev2Vgaoub185U2R50fPL0SDnSyJayhn9iPrBAkjZvxapAf3iK2PqLDETiSaHi8HPZhKs29aof3rTr8AeAYhMB8r2cnE93hdVbcP-AN6d5ClfLmxIPjylzTzWPtAwnVwlg)

`**다대다 매핑**`

다대다 매핑은 사용하지 않는게 좋다 .

다대다 매핑을 객체에서 구현이 되지만 DB에서 적용할 수 없는 관계이므로 사용하기가 매우 까다롭다.

다대다 매핑의 해결책은 중간에 엔티티를 하나 만들고

다대일 관계로 변경해주는 것이 좋다.

물론 가상테이블을 만드는 방법이 있지만 가상테이블을 만들시

등록일시, 수정일시 등 추가 컬럼에 대해서 추가할수가 없다.

해결책은 엔티티 하나를 더 만드는게 최선이다.

---

![https://lh4.googleusercontent.com/bIYFL1s-jyVX4gMdLrxv76ttPD0IyBdqNK21xu44BQjOP3_9Qn_I8LelYUnWuUGK2ZwrNkxq44AdKz2FFGOj70w5G_iQ555nKSRzOkjCBz7atPFNp-M-9jfjTgw4Giix3ECLQoYv](https://lh4.googleusercontent.com/bIYFL1s-jyVX4gMdLrxv76ttPD0IyBdqNK21xu44BQjOP3_9Qn_I8LelYUnWuUGK2ZwrNkxq44AdKz2FFGOj70w5G_iQ555nKSRzOkjCBz7atPFNp-M-9jfjTgw4Giix3ECLQoYv)

기존에 Category와 Item에 관계 중간에 CategoryItem 으로 엔티티를

만들고 CategoryItem을 기준으로 다대일로 Category와 Item 엔티티의 관계를 매핑한다.

그러므로 다대일관계로 매핑이 가능하다.

Category.class

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Category {

   @Id   @GeneratedValue
   private Long id;
   private String name;

   //가장중요한부분
   @OneToMany(mappedBy = "categories")
   private List<CategoryItem> categoryItems = new ArrayList<>();
}
```

Item.class

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn
@Getter @Setter
public abstract class Item {

   @Id @GeneratedValue
   @Column(name = "item_id")
   private Long id;

   private String name;

   private int price;

   private int stockQuantity;

   @Column(insertable = false, updatable = false)
   private String dtype;

   @OneToMany(mappedBy = "items")
   private List<OrderItem> orderItems = new ArrayList<>();
 
   //가장중요한부분
   @OneToMany(mappedBy = "items")

   private List<CategoryItem> categoryItems = new ArrayList<>();

}
```

CategoryItem.class

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class CategoryItem {

   @Id   @GeneratedValue
   @Column(name= "category_item_id")
   private Long id;

   //가장중요한부분
   @ManyToOne(fetch = FetchType.LAZY)
   @JoinColumn(name ="category_id")
   private Category categories;

   //가장중요한부분
   @ManyToOne(fetch = FetchType.LAZY)
   @JoinColumn(name="item_id")

   private Item items;
}
```

이후 설명은 다대일과 같으니 생략하도록 하겠다.
