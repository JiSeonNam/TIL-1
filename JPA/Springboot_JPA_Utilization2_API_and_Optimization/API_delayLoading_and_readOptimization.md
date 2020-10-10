# API 개발 고급 - 지연 로딩과 조회 성능 최적화
- 주문 + 배송정보 + 회원을 조회하는 API를 만들 것이다.
- 지연 로딩 때문에 발생하는 성능 문제를 단계적으로 해결해가는 과정이다.
- 참고로 매우매우 중요하다.
<br>

## V1. 엔티티를 직접 노출
```java
/**
 * xToOne(ManyToOne, OneToOne)관계에서의 성능 최적화
 * Order
 * Order -> Member
 * Order -> Delivery
 */
@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {
    private final OrderRepository orderRepository;

    // V1. 엔티티 직접 노출
    @GetMapping("/api/v1/simple-orders")
    public List<Order> ordersV1() {
        List<Order> all = orderRepository.findAllByCriteria(new OrderSearch());
        for (Order order : all) {
            order.getMember().getName(); //Lazy 강제 초기화
            order.getDelivery().getAddress(); //Lazy 강제 초기환
        }
        return all;
    }
}
```
- Order -> Member, Member -> Order 무한루프에 빠져버린다. 
- 객체를 JSON으로 만드는 jackson 라이브러리 입장에서는 객체를 무한루프를 돌면서 뽑아낸다.(양방향 연관관계 문제 발생)
- 문제를 해결하기 위해 양방향 연관관계에서 둘 중에 하나는 `@JsonIgnore`해야한다.
- 모든 양방향 연관관계에 적용하고 실행해도 Type definition error 에러가 난다.
    * 지연로딩에서는 연관관계로 엮여있는 엔티티의 DB 실제 객체를 가져오지 않고 프록시 객체를 가져온다.
    * jackson 라이브러리는 기본적으로 이 프록시 객체를 json으로 어떻게 생성해야 하는지 모르기 때문에 에러가 발생하는 것이다.
    * [참고 : 프록시와 연관관계 관리](https://github.com/qlalzl9/TIL/blob/b237d5631facf5deda0ec7c8fab01277be2206cc/JPA/Java_ORM_Standard_JPA_Programming/8_Proxy_RelationManaging.md#%ED%94%84%EB%A1%9D%EC%8B%9C%EC%99%80-%EC%A6%89%EC%8B%9C%EB%A1%9C%EB%94%A9-%EC%A3%BC%EC%9D%98%EC%A0%90)
- 이를 해결하기 위해 Hibernate5Module을 설치해야 한다.
    * build.gradle에 `'com.fasterxml.jackson.datatype:jackson-datatype-hibernate5'` 의존성 추가 
    * Hibernate5Module을 스프링 빈으로 등록
    ```java
    @SpringBootApplication
    public class JpashopApplication {
	    public static void main(String[] args) {
	    	SpringApplication.run(JpashopApplication.class, args);
	    }

	    @Bean
	    Hibernate5Module hibernate5Module() {
	    	return new Hibernate5Module();
	    }
    }
    ```
- 실행 후 조회하면 다음과 같이 조회된다.
    * Lazy 로딩이 아예 안되게 하는 것이다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/API_delayLoading_and_readOptimization_1.jpg"></p>

- 다음과 같이 강제로 지연 로딩을 시키면 조회는 할 수 있다.
```java
@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {
    private final OrderRepository orderRepository;

    // V1. 엔티티 직접 노출
    @GetMapping("/api/v1/simple-orders")
    public List<Order> ordersV1() {
        List<Order> all = orderRepository.findAllByCriteria(new OrderSearch());
        // 강제로 Lazy 로딩
        for (Order order : all) {
            order.getMember().getName(); // Lazy 강제 초기화
            order.getDelivery().getAddress(); // Lazy 강제 초기환
        }
        return all;
    }
}
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/API_delayLoading_and_readOptimization_2.jpg"></p>

- 어떻게든 해결한다고 해도 엔티티를 노출시키는 것은 안좋은 방법이다.
    * API 스펙 변경 문제
    * 성능 문제
- **지연 로딩(LAZY)을 피하기 위해 즉시 로딩(EARGR)으로 설정하면 안된다!**
    * 즉시 로딩 때문에 연관관계가 필요 없는 경우에도 데이터를 항상 조회해서 성능 문제가 발생할 수 있다. 
    * 즉시 로딩으로 설정하면 성능 튜닝이 매우 어려워 진다.
- **항상 지연 로딩을 기본으로 하고, 성능 최적화가 필요한 경우에는 페치 조인(fetch join)을 사용하자.(V3)**
<br>

## V2. 엔티티를 DTO로 변환
```java
@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {

    private final OrderRepository orderRepository;

    ...

    // V2. 엔티티를 DTO로 변환
    @GetMapping("/api/v2/simple-orders")
    public List<SimpleOrderDto> ordersV2() {
        List<Order> orders = orderRepository.findAllByCriteria(new OrderSearch());

        List<SimpleOrderDto> result = orders.stream()
                .map(o -> new SimpleOrderDto(o))
                .collect(toList());
        return result;
    }
    @Data
    static class SimpleOrderDto {
        private Long orderId;
        private String name;
        private LocalDateTime orderDate;
        private OrderStatus orderStatus;
        private Address address;

        public SimpleOrderDto(Order order) { // DTO에서 엔티티를 사용하는 것은 별 문제 없다.
            orderId = order.getId();
            name = order.getMember().getName();
            orderDate = order.getOrderDate();
            orderStatus = order.getStatus();
            address = order.getDelivery().getAddress();
        }
    }
}
```
- 실행 후 조회하면 다음과 같이 결과가 나온다.
    * API 스펙에 최적화되어 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/API_delayLoading_and_readOptimization_3.jpg"></p>

- 문제점
    * V1과 마찬가지로 지연 로딩으로 인한 DB 쿼리가 너무 많이 호출된다.
        * 쿼리가 최악의 경우 1 + N + N번 실행된다. (V1과 쿼리 수 동일)
            - (order 조회 1번) + (order->member 지연로딩 조회 N번) + (order->delivery 지연로딩 조회 N번)
        * 물론 즉시 로딩으로 바꾸면 안된다.
    * V3에서 나오는 페치 조인으로 해결해야 한다.
<br>

## V3. 엔티티를 DTO로 변환 - 페치 조인 최적화
- [참고 : fetch join](https://github.com/qlalzl9/TIL/blob/b237d5631facf5deda0ec7c8fab01277be2206cc/JPA/Java_ORM_Standard_JPA_Programming/11_JPQL2.md#%ED%8E%98%EC%B9%98-%EC%A1%B0%EC%9D%B8fetch-join)
- OrderRepository에 페치 조인을 위한 메서드 생성
    * 페치 조인을 사용하면 지연 로딩을 하지 않고 연관된 실제 엔티티를 한번에 다 조회한다.
```java
@Repository
@RequiredArgsConstructor
public class OrderRepository {

    private final EntityManager em;

    ...

    public List<Order> findAllWithMemberDelivery() {
        return em.createQuery(
                "select o from Order o" +
                        " join fetch o.member m" +
                        " join fetch o.delivery d", Order.class)
                .getResultList();
    }
}
```
- V3 맵핑
    * 페치 조인을 사용하는 것을 제외하고는 V2의 로직과 같다.
```java
@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {

    private final OrderRepository orderRepository;

    ...

    // V3. 엔티티를 DTO로 변환 - 페치 조인 최적화
    @GetMapping("/api/v3/simple-orders")
    public List<SimpleOrderDto> ordersV3() {
        List<Order> orders = orderRepository.findAllWithMemberDelivery();
        List<SimpleOrderDto> result = orders.stream()
                .map(o -> new SimpleOrderDto(o))
                .collect(toList());
        return result;
    }
}
```
- 실행하고 조회하면 다음과 같은 결과를 얻을 수 있다.
    * V2와 결과는 같지만 쿼리 발생 수가 다르다. 
        - V2 : 5번, V3 : 1번
    * 페치 조인으로 order->member, order->delivery는 이미 조회된 상태이므로 지연로딩이 일어나지 않는다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/API_delayLoading_and_readOptimization_4.jpg"></p>

- 실무에서 매우 자주 사용하는 기법으로 성능이 굉장히 좋다.
- 그러나 연관되는 엔티티의 모든 컬럼을 가져온다는 단점이 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/API_delayLoading_and_readOptimization_5.jpg"></p>

<br>

## V4. JPA에서 DTO로 바로 조회
- 일반적인 SQL을 사용할 때처럼 원하는 값을 선택해서 조회한다.
- new 명령어를 사용해서 JPQL의 결과를 DTO로 즉시 변환
- SELECT절에서 원하는 데이터를 직접 선택하므로 DB 애플리케이션 네트워크 용량 최적화가 된다.
    * 생각보다는 미비하다.
- 그렇다고 V3보다 V4가 무조건 좋은 것은 아니다.
    * V3의 경우 재사용이 가능하지만 V4의 화면에는 최적화 됐지만 재사용성이 없다. (해당 Dto를 사용할 때만 사용할 수 있다.)
    * API 스펙에 맞춘 코드가 Repository에 들어가는 단점도 있다.
- 개인 스타일이지만 Repository는 가급적 순수한 엔티티를 조회할 때와 성능 최적화를 위해 페치 조인 정도를 사용하는 곳에 쓰고 V4처럼 특정 Dto 전용인 경우 패키지를 따로 생성하는 것이 권장된다.

- OrderSimpleQueryRepository 조회 전용 리포지토리 생성
```java
@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository {

    private final EntityManager em;

    public List<OrderSimpleQueryDto> findOrderDtos() {
        return em.createQuery(
                "select new jpabook.jpashop.repository.order.simplequery.OrderSimpleQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
                " from Order o" +
                        " join o.member m" +
                        " join o.delivery d", OrderSimpleQueryDto.class)
                .getResultList();
    }
}
```
- OrderSimpleQueryDto 생성
```java
@Data
public class OrderSimpleQueryDto {
    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    public OrderSimpleQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }
}
```
- JPA에서 DTO로 바로 조회 맵핑
```java
@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {

    private final OrderRepository orderRepository;
    private final OrderSimpleQueryRepository orderSimpleQueryRepository;

    ...

    //  V4. JPA에서 DTO로 바로 조회
    @GetMapping("/api/v4/simple-orders")
    public List<OrderSimpleQueryDto> ordersV4() {
        return orderSimpleQueryRepository.findOrderDtos();
    }
}
```
- 실행하고 조회하면 결과는 똑같지만 V3와 달리 원하는 컬럼만 SELECT 쿼리가 나가는 것을 볼 수 있다.
    * 직접 쿼리를 다 짰기 때문이다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/JPA/img/API_delayLoading_and_readOptimization_6.jpg"></p>

<br>

## 정리 

### 쿼리 방식 선택 권장 순서
1. 우선 엔티티를 DTO로 변환하는 방법을 선택한다.
2. 필요하면 페치 조인으로 성능을 최적화 한다. (대부분의 성능 이슈가 해결된다.)
3. 그래도 안되면 DTO로 직접 조회하는 방법을 사용한다.
4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template을 사용해서 SQL을 직접 사용한다.
