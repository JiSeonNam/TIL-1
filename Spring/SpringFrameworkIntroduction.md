> [강의 링크](https://www.inflearn.com/course/spring_revised_edition/dashboard)

## IoC(Inversion of Control)
- 개발자가 만든 어떤 클래스나 메소드를 다른 프로그램이 대신 실행해주는 것
- IoC가 적용 되지 않은 예
```java
class OwnerController {
   private OwnerRepository repository = new OwnerRepository();
}
```
- IoC가 적용 된 예
```java
class OwnerController {
   private OwnerRepository repo;

   public OwnerController(OwnerRepository repo) {
       this.repo = repo;
   } 
}

class OwnerControllerTest {
   @Test
   public void create() {
         OwnerRepository repo = new OwnerRepository();
         OwnerController controller = new OwnerController(repo);
   }
}
```
<br>

## Spring에서 제공하는 IoC/DI 컨테이너
- Bean을 만들고 Bean들 사이의 의존성을 엮어주며 컨테이너가 가지고 있는 Bean들을 제공해 주는 역할
- 의존성 주입은 Bea끼리만 가능하다. 즉, Spring IoC 컨테이너는 그 안에 들어 있는 객체들끼리만 의존성 주입을 해준다.

- BeanFactory
    * IoC/DI에 대한 기본 기능을 가지고 있다.
- ApplicationContext
    * BeanFactory를 상속받고 있기 때문에 BeanFactory의 모든 기능을 포함하며, 일반적으로 BeanFactory보다 추천된다. 
    * 트랜잭션처리, AOP등에 대한 처리를 할 수 있다. 
    * BeanPostProcessor, BeanFactoryPostProcessor등을 자동으로 등록하고, 국제화 처리, 어플리케이션 이벤트 등을 처리할 수 있다.
- BeanPostProcessor
    * 컨테이너의 기본로직을 오버라이딩하여 인스턴스화 와 의존성 처리 로직 등을 개발자가 원하는 대로 구현 할 수 있도록 한다.
- BeanFactoryPostProcessor
    * 설정된 메타 데이터를 커스터마이징 할 수 있다.
<br>

## 빈(Bean)
- Spring에서 IoC 컨테이너가 관리하는 객체를 Bean이라 한다.
<br>

### Spring 컨테이너에 Bean을 등록하는 방법
1. Component Scanning
- @ComponentScan 애노테이션은 어디서 부터 찾을지 알려주고 그 곳에서부터 모든 클래스를 조회한다.
- @Component 애노테이션이 붙어 있는 클래스를 찾아서 그 클래스의 인스턴스를 만들어 Bean으로 등록한다.
- @Component라는 메타 애노테이션을 사용한 애노테이션도 @Component 애노테이션으로 볼 수 있다.
    * ex) @Repository, @Service, @Controller, @Configuration 등이 있고 직접 정의할 수도 있다.
2. 직접 Bean으로 등록
- Bean 설정 파일이 xml인지 java 설정 파일인지에 따라 달라질 수 있지만 최근에는 java 설정 파일을 많이 쓰는 추세이다. 
- @Configuration 애노테이션 안에다가 @Bean 애노테이션을 사용해 Bean을 직접 정의할 수 있다.
```java
@Configuration
public class SampleConfig {
    @Bean
    public SampleController sampleController() {
        return new SampleController();  // return 하는 객체 자체가 Bean으로 등록된다. 
    }
}
```
<br>

### Spring 컨테이너에 Bean을 꺼내서 쓰는 방법
1. ApplicationContext에서 직접 꺼내서 쓰는 방법
```java
@Test
public void testDI() {
    SampleController bean = ApplicationContext.getBean(SampleController.class);
    assertThat(bean).isNotNull();
}
```
2. @Autowired 애노테이션을 사용하는 방법
- IoC 컨테이너에 있는 Bean을 주입받아서 사용할 수 있다.
<br>

## DI(Dependency Injection)
- 클래스 사이의 의존 관계를 빈(Bean) 설정 정보를 바탕으로 컨테이너가 자동으로 연결해주는 것
- 의존성을 관리하는 제어권이 inversion되었으므로 DI도 일종의 IoC라고 볼 수 있다.

### 의존성 주입 방법
1. 생성자를 통해 주입 받는 방법
- Spring 4.3부터 클래스에 생성자가 하나이고, 그 생성자로 주입받는 매개변수가 Bean으로 등록되어 있다면 그 Bean을 자동으로 주입해준다. 
- Spring Reference에서 권장하는 방법으로 필수적으로 사용해야하는 래퍼런스 없이는 인스턴스를 만들지 못하도록 강제할 수 있다. 즉, 밑의 코드에서 OwnerController는 OwnerRepository가 없으면 동작할 수 없다.
- A가 B를 참조하고 B가 A를 참조하는 순환 참조(Circular Dependency )가 일어날 경우 생성자 injection 대신 field 나 setter 방법을 사용해야 한다. 
```java
public OwnerController(OwnerRepository clinicService) {
    this.owners = clinicSevice;
} 
```
2. field에 바로 주입 받는 방법
```java
@Autowired
private OwnerRepository owners;
```
3. Setter를 사용하는 방법
```java
@Autowired
public void setOwners(OwnerRepository owners) {
    this.owners = owners;
}
```
<br>

## AOP
- 관점 지향 프로그래밍(Aspect Oriented Programming)은 공통 처리 메소드나 기능을 별도의 클래스에 구현하여 관리하는 것을 말한다.
- 구현하고자 하는 기능의 코드를 해당 메소드에 작성하지 않았지만 기능이 별도로 외부에서 관리하며 처리되는 것이다.
<br>

### AOP를 구현하는 방법
1. 컴파일을 이용하는 방법
- A.java ---(AOP)---> A.class : 컴파일 도중 AOP 추가
2. 바이트코드 조작
- A.java ---> A.class ---(AOP)---> 메모리 : 클래스 로더가 클래스파일을 읽으면서 메모리에 적재시 AOP 추가
3. **프록시 패턴** 
- Spring AOP가 사용하는 방법
- 디자인 패턴 중 하나인 Proxy 패턴을 사용해 AOP와 같은 효과를 낸다.
<br>

### AOP 적용 예제
- @LogExcutionTime으로 메소드 처리 시간 로깅하기
```java 
@GetMapping("/owners/find")
    @LogExecutionTime
    public String initFindForm(Map<String, Object> model) {
        model.put("owner", new Owner());
        return "owners/findOwners";
    }
```
- @LogExcutionTime 애노테이션(어디에 적용할지 표시 해두는 용도)
```java
@Target(ElementType.METHOD)     // 이 애노테이션을 어디에 쓸 수 있는지 명시(method에 쓰겠다는 뜻)
@Retention(RetentionPolicy.RUNTIME)     // 이 애노테이션 정보를 언제까지 유지할 것인가 (Runtime까지 유지하겠다는 뜻)
public @interface LogExecutionTime {

}
```
- 실제 Aspect(@LogExcutionTime 애노테이션 달린 곳에 적용)
```java
@Component      // Bean으로 등록 되어야 하기 때문에 붙여준다.
@Aspect
public class LogAspect {

   Logger logger = LoggerFactory.getLogger(LogAspect.class);

   @Around("@annotation(LogExecutionTime)")
   public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable { //joinPoint가 Target이다.
       StopWatch stopWatch = new StopWatch();
       stopWatch.start();

       Object proceed = joinPoint.proceed();

       stopWatch.stop();
       logger.info(stopWatch.prettyPrint());

       return proceed;
   }

}
```

## PSA(Portable Service Abstraction)
- 추상화 계층을 사용해서 어떤 기술을 내부에 숨기고 개발자에게 편의성을 제공해주는 것을 Service Abstraction이라 한다.
-  Service Abstraction으로 제공되는 기술을 다른 기술 스택으로 간편하게 바꿀 수 있는 확장성을 갖고 있는 것이 Portable Service Abstraction이다.
- Spring은 Spring Web MVC, Spring Transaction, Spring Cache 등의 다양한 PSA를 제공한다.

#### Spring Web MVC에서의 PSA
- Spring의 PetClinic 예제를 보면 서블릿 어플리케이션임에도 불구하고 서블릿이 전혀 존재하지 않는다. 단지 @Controller 애노테이션이 붙어있는 클래스에서 @GetMapping, @PostMapping과 같은 @RequestMapping 애노테이션을 사용해서 요청을 매핑한다.
- 실제로는 내부적으로 서블릿 기반으로 코드가 동작하지만 서블릿 기술은 추상화 계층에 의해 숨겨져 있는 것이다.
```java
@Controller
class OwnerController {

	private static final String VIEWS_OWNER_CREATE_OR_UPDATE_FORM = "owners/createOrUpdateOwnerForm";

	private final OwnerRepository owners;

	private VisitRepository visits;

	public OwnerController(OwnerRepository clinicService, VisitRepository visits) {
		this.owners = clinicService;
		this.visits = visits;
	}

	@InitBinder
	public void setAllowedFields(WebDataBinder dataBinder) {
		dataBinder.setDisallowedFields("id");
	}

	@GetMapping("/owners/new")
	public String initCreationForm(Map<String, Object> model) {
		Owner owner = new Owner();
		model.put("owner", owner);
		return VIEWS_OWNER_CREATE_OR_UPDATE_FORM;
	}

	@PostMapping("/owners/new")
	public String processCreationForm(@Valid Owner owner, BindingResult result) {
		if (result.hasErrors()) {
			return VIEWS_OWNER_CREATE_OR_UPDATE_FORM;
		}
		else {
			this.owners.save(owner);
			return "redirect:/owners/" + owner.getId();
		}
	}
    // ...
```

#### Spring Transaction에서의 PSA
- Low level로 트랜잭션 처리를 하려면 setAutoCommit()과 commit(), rollback()을 명시적으로 호출해야 한다.
- Spring이 제공하는 @Transactional 애노테이션을 사용하면 단순히 메소드에 애노테이션을 붙여줌으로써 트랜잭션 처리가 이루어진다.
```java
public interface PetRepository extends Repository<Pet, Integer> {

	/**
	 * Retrieve all {@link PetType}s from the data store.
	 * @return a Collection of {@link PetType}s.
	 */
	@Query("SELECT ptype FROM PetType ptype ORDER BY ptype.name")
	@Transactional(readOnly = true)
	List<PetType> findPetTypes();

	/**
	 * Retrieve a {@link Pet} from the data store by id.
	 * @param id the id to search for
	 * @return the {@link Pet} if found
	 */
	@Transactional(readOnly = true)
	Pet findById(Integer id);

	/**
	 * Save a {@link Pet} to the data store, either inserting or updating it.
	 * @param pet the {@link Pet} to save
	 */
	void save(Pet pet);

}
```

#### Spring Cache에서의 PSA
- Cache도 마찬가지로 JCacheManager, ConcurrentMapCacheManager, EhCacheCacheManager와 같은 여러가지 구현체를 사용할 수 있다.
- 사용자는 @Cacheable 애노테이션을 붙여줌으로써 구현체를 크게 신경쓰지 않아도 필요에 따라 바꿔 쓸 수 있다.
```java
public interface VetRepository extends Repository<Vet, Integer> {

	/**
	 * Retrieve all <code>Vet</code>s from the data store.
	 * @return a <code>Collection</code> of <code>Vet</code>s
	 */
	@Transactional(readOnly = true)
	@Cacheable("vets")
	Collection<Vet> findAll() throws DataAccessException;

}
```