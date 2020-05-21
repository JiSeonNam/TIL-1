> [강의 링크](https://www.inflearn.com/course/spring_revised_edition/dashboard)
<br>

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
 