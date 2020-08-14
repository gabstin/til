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

[https://lh4.googleusercontent.com/WItbIURzsvJsSNCpA24Wp62oH4Q5ouUcx9UU4SnLwnrpx-Y1lXv6XpE31ICfKKLe9vpQJvPXXs5XifWTcbb2G9odHBKdaML0UWnyMn9-nOLvjGY8KETz4UyM7KozM4gIYGg9NJAA](https://lh4.googleusercontent.com/WItbIURzsvJsSNCpA24Wp62oH4Q5ouUcx9UU4SnLwnrpx-Y1lXv6XpE31ICfKKLe9vpQJvPXXs5XifWTcbb2G9odHBKdaML0UWnyMn9-nOLvjGY8KETz4UyM7KozM4gIYGg9NJAA)

### **1. Member 와 Order 엔티티 설정**

[https://lh3.googleusercontent.com/UQ1u25dmTPYSi9rjfW8dF3NihCpFHneomagYTlA3o31xtSNpFrLk9nmLN5xa2pW6vwQGZIcqSVpvOmvzgEWDDGF4IyxAtUwaJfWj_2ZVgaQASHlgWsXjfH1mniNxLz5h0LyWS4xY](https://lh3.googleusercontent.com/UQ1u25dmTPYSi9rjfW8dF3NihCpFHneomagYTlA3o31xtSNpFrLk9nmLN5xa2pW6vwQGZIcqSVpvOmvzgEWDDGF4IyxAtUwaJfWj_2ZVgaQASHlgWsXjfH1mniNxLz5h0LyWS4xY)

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

```java
@ManyToOne(fetch=FetchType.LAZY) // FetchType을 무조건 지원로딩(Lazy)로 설정
   @JoinColumn(name ="member_id")   // @JoinColumn으로 주인엔티티임을 설정
   private Member member;
```

**@ManyToOne(fetch=FetchType.LAZY)**

@ManyToOne으로 다대일을 설정하고 fetch 설정을 지원로딩으로 변경한다. ManyToOne, OneToOne은 기본 즉시로딩으로 설정되어 있기에 무조건 지원로딩으로 변경해야한다. 즉시로딩은 사용하지 않는편이 좋다.

**@JoinColumn(name ="member_id")**

주인엔티티를 설정하는 단계로 name안에 외래키 값을 넣어준다.

Member.class

```java
@OneToMany(mappedBy = "member") // 다대일로 매핑되어있으니 일대다로 양방향을 설정
   private List<Order> orders = new ArrayList<>();
```

**@OneToMany(mappedBy = "member")**

다대일 양방향의 매핑시 설정하는 방법으로 조회기능만 가능하다.

예를들어 회원를 조회하고 회원이 주문한 내역을 같이 볼때 사용될수 있다.

mappedBy = “member”는 Order.class에서 정의된 member를 말한다.