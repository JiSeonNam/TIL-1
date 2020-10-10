# API 개발 고급 - 컬렉션 조회 최적화
- 주문내역에서 추가로 주문한 상품 정보를 추가로 조회하자.
- 지금까지는 컬렉션 없이 `@ManyToOne` 또는 `@OneToOne`관계에서 조인해서 페치 조인으로 쉽게 해결할 수 있었지만 
- `@OneToMany` 관계에서 컬렉션 조회는 DB입장에서 데이터가 뻥튀기가 되기 때문에 성능 최적화를 고민할 것이 많다.

## V1. 엔티티 직접 노출
- OrderApiController 생성
```java
@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;

    @GetMapping("/api/v1/orders")
    public List<Order> ordersV1() {
        List<Order> all = orderRepository.findAllByCriteria(new OrderSearch());
        // 강제로 Lazy 로딩
        for (Order order : all) {
            order.getMember().getName(); //Lazy 강제 초기화
            order.getDelivery().getAddress(); //Lazy 강제 초기화
            List<OrderItem> orderItems = order.getOrderItems();
            orderItems.stream().forEach(o -> o.getItem().getName()); //Lazy 강제 초기화
        }
        return all;
    }
}
```
- 실행 후 조회하면 다음과 같이 원래 데이터에서 컬렉션이 추가된 조회 정보가 나온다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/API_collection_readOptimization_1.jpg"></p>

- 물론 양방향 연관관계에서 둘 중에 하나는 @JsonIgnore해야한다.
- 지금까지 배워왔듯이 이 방법은 사용하면 안된다.
<br>

## V2. 엔티티를 DTO로 변환
- [지난 단원](https://github.com/qlalzl9/TIL/blob/master/JPA/Springboot_JPA_Utilization2_API_and_Optimization/API_delayLoading_and_readOptimization.md#v2-%EC%97%94%ED%8B%B0%ED%8B%B0%EB%A5%BC-dto%EB%A1%9C-%EB%B3%80%ED%99%98)에서 한 것처럼 OrderDto를 만들어서 orderItems를 추가하면 된다고 생각하면 안된다.
- orderItems 또한 엔티티가 노출되지 않도록 OrderItemDto를 만들어서 엔티티에 대한 의존을 완전히 분리 시켜야한다.
    * 참고) Address 같은 값 타입은 변하지 않으므로 상관없다.
```java
@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;

    ...

    // V2. 엔티티를 DTO로 변환
    @GetMapping("/api/v2/orders")
    public List<OrderDto> ordersV2() {
        List<Order> orders = orderRepository.findAllByCriteria(new OrderSearch());
        List<OrderDto> result = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(Collectors.toList());
        return result;
    }

    @Data
    static class OrderDto {
        private Long orderId;
        private String name;
        private LocalDateTime orderDate; //주문시간
        private OrderStatus orderStatus;
        private Address address;
        private List<OrderItemDto> orderItems;
        
        public OrderDto(Order order) {
            orderId = order.getId();
            name = order.getMember().getName();
            orderDate = order.getOrderDate();
            orderStatus = order.getStatus();
            address = order.getDelivery().getAddress();
            orderItems = order.getOrderItems().stream()
                    .map(orderItem -> new OrderItemDto(orderItem))
                    .collect(Collectors.toList());
        }
    }

    @Data
    static class OrderItemDto {
        // 굳이 다 넣을 필요없이 원하는 항목만 넣으면 된다.
        private String itemName; // 상품 명
        private int orderPrice; // 주문 가격
        private int count; // 주문 수량

        public OrderItemDto(OrderItem orderItem) {
            itemName = orderItem.getItem().getName();
            orderPrice = orderItem.getOrderPrice();
            count = orderItem.getCount();
        }
    }
}
```
- 이 방법 또한 쿼리가 매우 많이 나간다.
    * 최악의 경우 SQL 실행 수
        - order 1번
        - member , address N번(order 조회 수 만큼)
        - orderItem N번(order 조회 수 만큼)
        - item N번(orderItem 조회 수 만큼
<br>
