# 도메인 분석 설계

## 요구사항 분석
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/Domain_Analysis_Design_1.jpg"></p>

- 회원 기능
    * 회원 등록
    * 회원 조회
- 상품 기능
    * 상품 등록
    * 상품 수정
    * 상품 조회
- 주문기능
    * 상품 주문
    * 주문 내역 조회
    * 주문 취소
- 기타 요구사항
    * 상품은 재고 관리가 필요하다.
    * 상품의 종류는 도서, 음반, 영화가 있다.
    * 상품을 카테고리로 구분할 수 있다.
    * 상품 주문 시 배송 정보를 입력할 수 있다.
<br>

## 도메인 모델과 테이블 설계
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/Domain_Analysis_Design_2.jpg"></p>

- 회원은 여러 상품을 주문할 수 있기 때문에 1:N 관계다.
    * 회원 : 주문 = 1 : N
- 회원이 한 번 주문할 때 여러 상품을 선택할 수 있고 상품도 여러 주문에 담길 수 있기 때문에 N:M관계이다.
    * 주문 상품이라는 테이블을 만들어 각각 1:N, N:1 관계로 풀어낸다.
- 상품은 도서, 음반, 영화로 구분되어 상품이라는 공통 속성을 사용하므로 상속 구조로 표현했다.
<br>

### 회원 엔티티 분석
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/Domain_Analysis_Design_3.jpg"></p>

- 회원(Member)
    * 이름과 임베디드 타입인 주소(Address), 주문리스트(orders)를 가진다.
- 주문(Order)
    * 한 번 주문할 때 여러 상품을 주문할 수 있으므로 주문상품(OrderItem)은 1:N 관계다.
    * 상품을 주문한 회원과 배송 정보, 주문 날짜, 주문 상태를 가지고 있다.
- 주문상품(OrderItem)
    * N:M 관계를 1:N, N:1 관계를 풀어내는 역할 뿐만 아니라
    * 주문한 상품 정보와 주문 금액(orderPrice), 주문 수량(count) 정보를 가지고 있다.
- 배송(Delivery)
    * 주문과 배송지 주소, 주문 상태를 가지고 있다.
    * 주문 시 하나의 배송 정보를 생성하며 주문과 배송은 1:1 관계다.
- 상품(Item)   
    * 상품 이름, 가격, 재고수량, 어느 카테고리에 맵핑되어 있는지 카테고리 정보를 가지고 있다.
    * 상속 관계로 Album, Book, Movie를 가진다.
- 카테고리(Category)
    * 카테고리와 아이템은 N:M 관계다.
- 참고
    * 회원이 주문을 하기 때문에, 회원이 주문리스트를 가지는 것은 얼핏보면 잘 설계한것 같지만, 객체 세상은 실제 세계와는 다르다. 
    * 실무에서는 회원이 주문을 참조하지않고, 주문이 회원을 참조하는것으로 충분하다. 
    * 실습에서는 1:N, N:1의 양방향 연관관계를 설명하기 위해서 추가했다.
    * 카테고리와 아이템 관계 또한 실무에서는 N:M 관계를 쓰면 안된다.
<br>

### 회원 테이블 분석
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/Domain_Analysis_Design_4.jpg"></p>

- MEMBER
    * 거의 1:1 맵핑이다.
    * Address 임베디드 타입 정보가 MEMBER 테이블에 그대로 들어갔다.(DELIVERY 테이블도 마찬가지)
- ITEM
    * 앨범, 도서, 영화 타입을 통합해서 하나의 테이블로 만들었다. (싱글 테이블 전략을 썼다.)
    * DTYPE 컬럼으로 타입을 구분한다.
- ORDERS
    * DB에 order by가 예약어로 잡고 있는 경우가 많아 관례상 ORDERS로 사용
- ORDER_ITEM
    * 어느 주문인지, 어느 아이템인지 확인할 FK, ORDER_ID, ITEM_ID가 있다.
- CATEGORY
    * 객체에서는 카테고리와 아이템이 서로의 객체를 가져도 된다.
    * 그러나 RDB에서는 일반적으로 불가능하다. 따라서 중간에 맵핑 테이블 CATEGORY_ITEM을 둬서 1:N, N:1 관계로 풀어낸다.
<br>

### 연관관계 맵핑 분석
- 회원과 주문
    * 회원 입장에서 1:N, 주문입장에서 N:1의 양방향 연관관계이다.
    * 따라서 연관관계의 주인을 정해야 하는데, 외래키가 있는 주문을 연관관계의 주인으로 정하는 것이 좋다.
        - 테이블에서 1:N 관계에서 항상 N 쪽에 외래키가 있다.
    * Order.member를 ORDERS.MEMBER_ID(FK)와 맵핑한다.
- 주문상품과 주문
    * N:1 양방향 관계이다.
    * 외래키가 주문상품에 있으므로 주문상품이 연관관계의 주인이다.
    * OrderItem.order를 ORDER_ITEM.ORFER_ID(FK)와 맵핑한다.
- 주문상품과 상품
    * N:1 단방향 관계이다.
    * OrderItem.item을 ORDER_ITEM_ID(FK)와 맵핑한다.
- 주문과 배송
    * 1:1 단방향 관계이다.
    * Order.delivery를 ORDERS.DELIVERY_ID(FK)와 맵핑한다.
<br>
 