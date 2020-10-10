# API 개발 고급 - 컬렉션 조회 최적화
- 주문내역에서 추가로 주문한 상품 정보를 추가로 조회하자.
- 지금까지는 컬렉션 없이 `@ManyToOne` 또는 `@OneToOne`관계에서 조인해서 페치 조인으로 쉽게 해결할 수 있었지만 
- `@OneToMany` 관계에서 컬렉션 조회는 DB입장에서 데이터가 뻥튀기가 되기 때문에 성능 최적화를 고민할 것이 많다.

## V1 엔티티 직접 노출
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
