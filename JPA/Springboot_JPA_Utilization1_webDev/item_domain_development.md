# 상품 도메인 개발
- 구현 기능
    * 상품 등록
    * 상품 목록 조회
    * 상품 수정
- 개발 순서
    * 상품 엔티티 개발(비즈니스 로직 추가)
    * 상품 repository 개발
    * 상품 서비스 개발
<br>

## 상품 엔티티 개발(비즈니스 로직 추가)
- 재고 수량이 늘리고 줄이는 비즈니스 로직을 추가한다.
- 엔티티 자체에서 해결할 수 있는 것들은 엔티티안에 비즈니스 로직을 넣는 것이 좋다.
    * 객체지향적이다.
    * ex) stockQuantity가 Item 엔티티 안에 있으므로 Item 엔티티에서 비즈니스 로직이 나가는게 응집도 있다.
- Item 엔티티에 비즈니스 로직 추가
    * `addStock()`
        - 파라미터로 넘어온 수만큼 재고를 늘린다.
        - 재고가 증가하거나 상품 주문을 취소해서 재고를 다시 늘려야 할 때 사용한다.
    * `removeStock()`
        - 파라미터로 넘어온 수만큼 재고를 줄인다.
        - 만약 재고가 부족하면 예외 발생하도록 예외 처리한다.
        - 주로 상품을 주문할 때 사용한다.
```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype")
@Getter @Setter
public abstract class Item {

    ...

    //==비즈니스 로직==//
    /**
     * stock 증가
     */
    public void addStock(int quantity) {
        this.stockQuantity += quantity;
    }

    /**
     * stock 감소
     */
    public void removeStock(int quantity) {
        int restStrock = this.stockQuantity - quantity;
        if(restStrock < 0) {
            throw new NotEnoughStockException("need more stock");
        }
        this.stockQuantity = restStrock;
    }

}
```
- exception 패키지를 만들고 NotEnoughStockException클래스 생성
```java
public class NotEnoughStockException extends RuntimeException{

    public NotEnoughStockException() {
        super();
    }

    public NotEnoughStockException(String message) {
        super(message);
    }

    public NotEnoughStockException(String message, Throwable cause) {
        super(message, cause);
    }

    public NotEnoughStockException(Throwable cause) {
        super(cause);
    }
}
```
- 이런식으로 stockQuantity를 변경해야 할 경우 Setter를 사용하는 것이 아니라 비즈니스 메소드를 가지고 변경해야 한다.
    * 객체지향적이다.
<br>

## 상품 repository 개발
- ItemRepository 생성
    * `save()`
        - id가 없으면 신규로 보고 `persist()`를 실행한다.
        - id가 있으면 이미 DB에 저장된 엔티티를 수정한다고 보고, `merge()`를 실행한다.
```java
@Repository
@RequiredArgsConstructor
public class ItemRepository {

    private final EntityManager em;

    public void save(Item item) {
        if (item.getId() == null) {
            em.persist(item);
        } else {
            em.merge(item);
        }
    }

    public Item findOne(Long id) {
        return em.find(Item.class, id);
    }

    public List<Item> findAll() {
        return em.createQuery("select i from Item i",Item.class)
                .getResultList();
    }
}
```
<br>

## 상품 서비스 개발
- ItemService 생성
    * ItemService는 ItemRepository에 단순히 위임만 하는 클래스이다.
```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class ItemService {

    private final ItemRepository itemRepository;

    @Transactional // 저장이므로 읽기전용 해제
    public void saveItem(Item item) {
        itemRepository.save(item);
    }

    public List<Item> findItems() {
        return itemRepository.findAll();
    }

    public Item findOne(Long itemId) {
        return itemRepository.findOne(itemId);
    }
}
```
- 상품 테스트는 회원 테스트와 비슷하므로 생략
