# chapter 1. 영속성 컨텍스트

영속성 컨텍스트는 JPA에서 사용하는 임시공간이라고 생각하면된다.
EntityManager를 생성하면 영속성 컨텍스트에 접근할수 있다.
EntityManager는 영속성 컨텍스트에도 엔티티 정보를 저장하고 
이후 실제 DB에 저장한다.
등록, 조회, 수정을 통해서 어떻게 돌아가는지 살펴보자.


![img01][img01]

[img01]: https://lh5.googleusercontent.com/ihN02vy_jk6VLZ4qpECxrkptqjTWd4szTjNbTOpCgxHT1Pz36Cmbu9rwosnOsIzGK4JEXXLVeBDiQNcMF5aUJ5h-zbRH26L46cS7vft9aGn1IbF7Fon3qQIhAj0LX86FSitFldgW

예를 들어 엔티티 등록을 할 경우
em.persist(member);  →  이때 영속성 컨텍스트 저장이 되고 
tx.commit(); →  commit 을 해야 실제 DB에 저장이 된다.

만약 저장 후 조회의 경우라면
em.persist(member); →영속성 컨텍스트에 저장
Member findMember = em.find(Member.class, 101L); 
→ DB에서 가져오는게 아니라 영속성 컨텍스트에서 값을 가져오는 형태
sysout(findMember.getId()); 
tx.commit(); →  여기에서 이제 실제DB에 저장된다.

조회만 할경우는 어떻게 될까?
Member findMember1 = em.find(Member.class, 101L);
→ 영속성 컨텍스트에 저장해둔 데이터가 없다면 일단 DB에서가져온다.
Member findMember2 = em.find(Member.class, 101L);
→ 이후에 똑같은 쿼리가 중복된다면 영속성 컨텍스트에서 가져온다.
sysout(findMemeber1 == findMember2) 
→ 그렇다면 둘의 객체는 같은것인가? 맞다 둘은 완전히 같은 객체이다.

이번에 등록을 두번 할 경우는 어떻게 될까?
em.persist(memberA); → 영속성 컨텍스트에만 저장
em.persist(memberB); → 마찬가지로 영속성 컨텍스트에만 저장
tx.commit →  위에 내용을 한번에 DB에 저장한다.

이렇게 조회등록의 정보를 영속성 컨텍스트를 중간에 두고 DB에 접근을 최소화한다.
특이한건 수정시에 있다.

예제를 한번보자.
Member member = em.find(Member.class, 150L); → 먼저 기존데이터를 가져온다
member.setName(“JPA수정”); → 가져온 데이터에서 Name값을 변경하다.

그렇다면 이후에 persist를 해줘야 할까? 
아니다. setName() 까지만 한다면 데이터가 수정된다.
// em.persist(member)  → 수정시 해당 내용은 사용하지 않아도 된다. 

이유는 영속성 컨텍스트에서 기존 엔티티를 스냅샷 형태로 저장해두고
자신이 저장해놓은 엔티티 데이터와 다를 경우 UPDATE 쿼리를 가진다.
tx.commit(); →  최종적으로 DB에 update쿼리를 날리면서 수정된다.



