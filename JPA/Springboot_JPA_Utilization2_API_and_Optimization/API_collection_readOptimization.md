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

## V3. 엔티티를 DTO로 변환 - 페치 조인 최적화
- OrderRepository에 페치 조인을 위한 메서드 생성
    * `findAllWithItem()`
```java
@Repository
@RequiredArgsConstructor
public class OrderRepository {

    private final EntityManager em;

    ...

    public List<Order> findAllWithItem() {
        return em.createQuery(
                "select distinct o from Order o" +
                        " join fetch o.member m" +
                        " join fetch o.delivery d" +
                        " join fetch o.orderItems oi" +
                        " join fetch oi.item i", Order.class)
                .getResultList();
    }
}
```
- V3 맵핑
```java
@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;

    ...

    // V3. 엔티티를 DTO로 변환 - 페치 조인 최적화
    @GetMapping("/api/v3/orders")
    public List<OrderDto> ordersV3() {
        List<Order> orders = orderRepository.findAllWithItem();
        List<OrderDto> result = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(toList());
        return result;
    }
}
```
- 페치 조인으로 SQL이 1번만 실행된다.
- `distinct`를 사용한 이유는 일대다 조인이 있으므로 데이터베이스 row가 증가한다.(데이터 뻥튀기)
- 그 결과 같은 order엔티티의 조회 수도 증가하게 된다. 
- JPA의 distinct는 SQL에 distinct를 추가하고, 더해서 같은 엔티티가 조회되면, 애플리케이션에서 중복을 걸러준다.
    * SQL의 distinct는 모든 컬럼이 동일해야 중복을 걸러준다.
- 이 예에서 order가 컬렉션 페치 조인 때문에 중복 조회 되는 것을 막아준다.
- 그러나 컬렉션 페지 조인을 사용하면 **페이징이 불가능**하다.
    * 하이버네이트는 경고 로그를 남기면서 모든 데이터를 DB에서 읽어오고, 메모리에서 페이징 해버린다(매우 위험하다).
- 또한 컬렉션 페치 조인은 1개만 사용할 수 있다.
    * 컬렉션 둘 이상에 페치 조인을 사용하면 안된다. 
    * 데이터가 부정합하게 조회될 수 있다
<br>

## V3.1 엔티티를 DTO로 변환 - 페이징과 한계 돌파

### 페이징과 한계 돌파
- 컬렉션을 페치 조인하면 페이징이 불가능하다.
    * 컬렉션을 페치 조인하면 일대다 조인이 발생하므로 데이터가 예측할 수 없이 증가한다.
    * 일다대에서 일(1)을 기준으로 페이징을 하는 것이 목적이다. 그런데 데이터는 다(N)를 기준으로 row가 생성된다.
    * Order를 기준으로 페이징 하고 싶은데, 다(N)인 OrderItem을 조인하면 OrderItem이 기준이 되어버린다.
    * (더 자세한 내용은 자바 ORM 표준 JPA 프로그래밍 - 페치 조인 한계 참조)
- 이 경우 하이버네이트는 경고 로그를 남기고 모든 DB 데이터를 읽어서 메모리에서 페이징을 시도한다. 최악의 경우 장애로 이어질 수 있다.
<br>

### 한계 돌파
- 그러면 페이징 + 컬렉션 엔티티를 함께 조회하려면 어떻게 해야할까?
- 지금부터 코드도 단순하고, 성능 최적화도 보장하는 매우 강력한 방법이 있다
    * 대부분의 페이징 + 컬렉션 엔티티 조회 문제는 이 방법으로 해결할 수 있다.
- 먼저 XToOne(OneToOne, ManyToOne) 관계를 모두 페치조인 한다. ToOne 관계는 row수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지 않는다.
- 컬렉션은 지연 로딩으로 조회한다.
- 지연 로딩 성능 최적화를 위해 hibernate.default_batch_fetch_size , @BatchSize 를 적용한다.
    * hibernate.default_batch_fetch_size: 글로벌 설정
    * @BatchSize : 개별 최적화
    * 이 옵션을 사용하면 컬렉션이나, 프록시 객체를 한꺼번에 설정한 size 만큼 IN 쿼리로 조회한다.
```java
@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;

    ...

    // V3.1 엔티티를 DTO로 변환 - 페이징과 한계 돌파
    @GetMapping("/api/v3.1/orders")
    public List<OrderDto> ordersV3_page(@RequestParam(value = "offset",
            defaultValue = "0") int offset,
                                        @RequestParam(value = "limit", defaultValue = "100") int limit) {
        List<Order> orders = orderRepository.findAllWithMemberDelivery(offset, limit);
        List<OrderDto> result = orders.stream()
                .map(o -> new OrderDto(o))
                .collect(toList());
        return result;
    }

    ...
}
```
```java
@Repository
@RequiredArgsConstructor
public class OrderRepository {

    private final EntityManager em;

    ...

    public List<Order> findAllWithMemberDelivery(int offset, int limit) {
        return em.createQuery(
                "select o from Order o" +
                        " join fetch o.member m" +
                        " join fetch o.delivery d", Order.class)
                .setFirstResult(offset)
                .setMaxResults(limit)
                .getResultList();
    }
}
```
- 실행해서 offset과 limit을 입력해 조회하면 다음과 같다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/API_collection_readOptimization_2.jpg"></p>

- 최적화 옵션 : application.yml 파일에 다음과 같이 default_batch_fetch_size를 1000으로 설정하면 한번에 DB에 있는 IN 쿼리를 가져온다.
    * 참고)
        - default_batch_fetch_size 의 크기는 적당한 사이즈를 골라야 하는데, 100~1000 사이를 선택하는 것이 권장된다.
        - 이 전략을 SQL IN 절을 사용하는데, 데이터베이스에 따라 IN 절 파라미터를 1000으로 제한하기도 한다.
        - 1000으로 잡으면 한번에 1000개를 DB에서 애플리케이션에 불러오므로 DB에 순간 부하가 증가할 수 있다. 
        - 하지만 애플리케이션은 100이든 1000이든 결국 전체 데이터를 로딩해야 하므로 메모리 사용량이 같다.
        - 1000으로 설정하는 것이 성능상 가장 좋지만, 결국 DB든 애플리케이션이든 순간 부하를 어디까지 견딜 수 있는지로 결정하면 된다.
```yml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 1000
```
- 개별로 설정하려면 `@BatchSize`를 적용하면 된다. (컬렉션은 컬렉션 필드에, 엔티티는 엔티티 클래스에 적용)
- 장점
    * 쿼리 호출 수가 1 + N -> 1 + 1 로 최적화 된다.
    * 조인보다 DB 데이터 전송량이 최적화 된다. (Order와 OrderItem을 조인하면 Order가 OrderItem 만큼 중복해서 조회된다. 이 방법은 각각 조회하므로 전송해야할 중복 데이터가 없다.)
    * 페치 조인 방식과 비교해서 쿼리 호출 수가 약간 증가하지만, DB 데이터 전송량이 감소한다.
    * 컬렉션 페치 조인은 페이징이 불가능 하지만 이 방법은 페이징이 가능하다.
- **결론**
    * ToOne 관계는 페치 조인해도 페이징에 영향을 주지 않는다. 
    * 따라서 ToOne 관계는 페치조인으로 쿼리 수를 줄이고 해결하고, 나머지는 hibernate.default_batch_fetch_size 로 최적화 하자.
<br>
