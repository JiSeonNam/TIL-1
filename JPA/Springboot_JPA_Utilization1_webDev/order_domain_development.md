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

