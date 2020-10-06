# 주문 도메인 개발
- 구현 기능
    * 상품 주문
    * 주문 내역 조회
    * 주문 취소
- 순서
    * 주문 엔티티, 주문상품 엔티티 개발
    * 주문 Repository 개발
    * 주문 Service 개발
    * 주문 검색 기능 개발
    * 주문 기능 테스트
<br>

## 주문, 주문상품 엔티티 개발
- Order 엔티티에 생성 메소드, 비즈니스 로직, 조회 로직 추가
    * 생성 메소드 : `createOrder()`
        - 주문 엔티티를 생성할 때 사용한다.
        - 주문 회원, 배송정보, 주문상품의 정보를 받아서 실제 주문 엔티티를 생성한다.
        - 밖에서 Order에 setter를 이용해 값을 넣는 방식이 아니라 생성할 때부터 아예 반드시 `createOrder()` 를 호출해 생성 메소드에서 주문 생성에 대한 복잡한 로직을 완성을 시켜서 return한다.
    * 주문 취소 : `cancel()`
        - 주문 취소시 사용한다.
        - 주문 상태를 취소로 변경하고 주문상품에 주문 취소를 알린다.
        - 만약 이미 배송을 완료한 상품이면 주문을 취소하지 못하도록 예외를 발생시킨다.
        - 상태를 바꾼 뒤 루프를 돌면서 OrderItem엔티티의 `cancel()`를 호출하면 상품의 재고를 원복시킨다.
    * 전체 주문 가격 조회 : `getTotalPrice()`
        - 주문 시 사용한 전체 주문 가격을 조회한다. 
        - 전체 주문 가격을 알려면 각각의 주문상품 가격을 알아야 한다. 
        - 로직을 보면 연관된 주문상품들의 가격을 조회해서 더한 값을 반환한다.
        - (실무에서는 주로 주문에 전체 주문 가격 필드를 두고 역정규화한다.)
```java
@Entity
@Table(name = "orders")
@Getter @Setter
public class Order {

    ...

    //==생성 메서드==//
    public static Order createOrder(Member member, Delivery delivery, OrderItem... orderItems) {
        Order order = new Order();
        order.setMember(member);
        order.setDelivery(delivery);
        for (OrderItem orderItem : orderItems) {
            order.addOrderItem(orderItem);
        }
        order.setStatus(OrderStatus.ORDER);
        order.setOrderDate(LocalDateTime.now());
        return order;
    }

    //==비즈니스 로직==//
    /**
     * 주문 취소
     */
    public void cancel() {
        if (delivery.getStatus() == DeliveryStatus.COMP) {
            throw new IllegalStateException("이미 배송완료된 상품은 취소가 불가능합니다.");
        }
        this.setStatus(OrderStatus.CANCEL);
        for (OrderItem orderItem : orderItems) {
            orderItem.cancel();
        }
    }

    //==조회 로직==//
    /**
     * 전체 주문 가격 조회
     */
    public int getTotalPrice() {
        int totalPrice = 0;
        for (OrderItem orderItem : orderItems) {
            totalPrice += orderItem.getTotalPrice();
        }
// 람다와 스트림을 활용하면 간단하게 표현할 수도 있다.
//        int totalPrice = orderItems.stream()
//                .mapToInt(OrderItem::getTotalPrice)
//                .sum();
        return totalPrice;
    }

}
```
- OrderItem 엔티티에 비즈니스 로직 추가
    * 생성 메소드 : `createOrderItem()`
        - 주문 상품, 가격, 수량 정보를 사용해 주문상품 엔티티를 생성한다.
        - `item.removeStock(count);`으로 주문한 수량만큼 상품의 재고를 줄인다.
    * 주문 취소 : `cancel()`
        - 취소한 주문 수량만큼 상품의 재고를 증가시킨다.
    * 주문 가격 조회 : `getTotalPrice()`
        - 주문 가격에 수량을 곱한 값을 반환한다.
```java
@Entity
@Getter @Setter
public class OrderItem {

    ...

    //==생성 메서드==//
    public static OrderItem createOrderItem(Item item, int orderPrice, int count) {
        OrderItem orderItem = new OrderItem();
        orderItem.setItem(item);
        orderItem.setOrderPirce( orderPrice);
        orderItem.setCount(count);

        item.removeStock(count);
        return orderItem;
    }

    //==비즈니스 로직==//
    public void cancel() {
        getItem().addStock(count);
    }

    //==조회 로직==//
    /**
     * 주문상품 전체 가격 조회
     */
    public int getTotalPrice() {
        return getOrderPirce() * getCount();
    }
}
```
<br>

## 주문 Repository 개발
- OrderRepository 생성
```java
@Repository
@RequiredArgsConstructor
public class OrderRepository {

    private final EntityManager em;

    public void save(Order order) {
        em.persist(order);
    }

    public Order findOne(Long id) {
        return em.find(Order.class, id);
    }

}
```
<br>

## 주문 Service 개발
- 주문서비스는 주문 엔티티와 주문상품 엔티티의 비즈니스 로직을 활용해서 주문, 주문취소, 주문내역 검색 기능을 제공한다.
- 참고 : 예제의 단순화를 위해 한 번에 하나의 상품만 주문할 수 있다.
- OrderService 생성
    * 주문 : `order()`
        - 주문하는 회원 식별자, 상품 식별자, 주문 수량 정보를 받아서 실제 주문 엔티티를 생성한 후 저장한다.
        - 주문 저장 : `orderRepository.save(order);`
            * Order 엔티티에는 orderItems와 delivery가 Cascade가 걸려있기 때문에 자동으로 persist된다.
            * cascade의 범위는 보통 참조하는 주인이 private owner인 경우에만 사용해야 한다.(라이프 사이클이 같이 돌아갈 때)
                - OrderItem엔티티와 delivery엔티티는 Order엔티티말고는 쓰지 않는다.
    * 주문 취소 : `cancelOrder()`
        - 주문 식별자를 받아서 주문 엔티티를 조회한 후 주문 엔티티에 주문 취소를 요청한다.
        - JPA를 사용하지 않았다면 주문 취소하면 UPDATE 쿼리를 직접 날려줘야 한다.
        - 그러나 JPA에서는 데이터를 바꾸면 dirty checking하여 DB에 UPDATE 쿼리가 알아서 날라간다.
    * 주문 검색 : `findOrder()`
        - OrderSearch라는 검색 조건을 가진 객체로 주문 엔티티를 검색한다.
        - 자세한 내용은 주문 검색 기능에서 알아보자.
```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class OrderService {

    private final MemberRepository memberRepository;
    private final OrderRepository orderRepository;
    private final ItemService itemService;

    /**
     * 주문
     */
    @Transactional
    public Long order(Long memberId, Long itemId, int count) {
        //엔티티 조회
        Member member = memberRepository.findOne(memberId);
        Item item = itemService.findOne(itemId);

        //배송정보 생성
        Delivery delivery = new Delivery();
        delivery.setAddress(member.getAddress());
        delivery.setStatus(DeliveryStatus.READY);

        //주문상품 생성
        OrderItem orderItem = OrderItem.createOrderItem(item, item.getPrice(), count);

        //주문 생성
        Order order = Order.createOrder(member, delivery, orderItem);

        //주문 저장
        orderRepository.save(order);

        return order.getId();
    }

    /**
     * 주문 취소
     */
    @Transactional
    public void cancelOrder(Long orderId) {
        //주문 엔티티 조회
        Order order = orderRepository.findOne(orderId);
        //주문 취소
        order.cancel();
    }

    /**
     * 주문 검색
     */
//    public List<Order> findOrders(OrderSearch orderSearch) {
//        return orderRepository.findAll(orderSearch);
//    }

}
```
- 주문상품을 생성할 때 createOrderItem 메소드를 제외한 다른 방법(setter) 사용을 제한한다.
```java
@Entity
@Getter @Setter
public class OrderItem {

    ...

    // protected로 기본 생성자를 만들게 해서 제한한다.
    protected OrderItem() {
    }

    ...
```
- 위 내용을 롬복으로 더 간단하게 만들수도 있다.
```java
@Entity
@Getter @Setter
// 롬복 사용
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class OrderItem {

    ...
}
```
- Order 엔티티에도 마찬가지로 `createOrder()` 이외의 방법으로 Order를 생성하는 것을 제한한다.
```java
@Entity
@Table(name = "orders")
@Getter @Setter
// 롬복 사용
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Order {

    ...
}
```
- 이렇게 **코드를 제약하는 스타일로 짜면 유지보수가 쉽다.**
<br>

### 도메인 모델 패턴
- 주문 서비스의 주문과 주문 취소 메소드를 보면 비즈니스 로직 대부분이 엔티티에 있다.
    * Service는 단순히 위임하고 호출 정도만 한다.
- 이처럼 엔티티가 비즈니스 로직을 가지고 객체 지향의 특성을 적극 활용하는 것을 도메인 모델 패턴이라고 한다.
- 반대로 엔티티에는 비즈니스 로직이 거의 없고 서비스 게층에서 대부분의 비즈니스 로직을 처리하는 것을 트랜잭션 스크립트 패턴이라고 한다.
- 참고
    * [도메인 모델 패턴](https://martinfowler.com/eaaCatalog/domainModel.html)
    * [트랜잭션 스크립트 패턴](https://martinfowler.com/eaaCatalog/transactionScript.html)
<br>