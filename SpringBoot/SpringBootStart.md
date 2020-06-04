# [Srping Boot](https://spring.io/projects/spring-boot)
- Spring Boot는 제품 수준의 Spring 기반인 독립적인 애플리케이션을 빠르고 쉽게 만들 수 있도록 도와준다.
- Opinionated view
    * 스프링 부트가 가진 컨벤션으로 가장 널리 쓰인다고 하는 설정들을 스프링 부트가 기본 설정으로 제공한다. 
    * 설정에는 스프링 프레임워크 설정 뿐 아니라 서드파티 플랫폼(ex : 톰캣)에 대한 설정도 제공해 준다.

# 스프링 부트의 goals
- 모든 스프링 기반의 개발에 더 빠르고 폭넓은 사용을 제공한다.
- 이미 정해진 컨베년으로 설정을 제공해 주기 때문에 일일히 설정을 할 필요가 없고 요구사항에 맞게 설정을 쉽고 빠르게 바꿀 수 있다.(이러한 점 때문에 스프링 부트가 널리 사용된다.)
- 비지니스 로직에 필요한 기능들을 구현하는데 필요한 기능뿐 아니라 non-functional features들도 제공한다.
    * embedded servers, security, metrics, health checks and externalized configuration
- XML 설정을 더 이상 사용하지 않고 코드 제너레이션을 하지 않는다
    * 코드 제너레이션 하지 않으므로 더 쉽고 명확하고 커스터마이징 하기가 쉽다.
- 자바 8 이상, 서블릿 3.1 이상부터 지원한다.

# 스프링 부트 프로젝트 구조
- 메이븐 기본 프로젝트 구조와 동일하다.
    * 소스 코드 (src\main\java)
    * 소스 리소스 (src\main\resource)
    * 테스트 코드 (src\test\java)
    * 테스트 리소스 (src\test\resource)

- 스프링 부트에서는 @SpringBootApplication 애노테이션이 붙어있는 Main 애플리케이션 클래스의 위치를 프로젝트 디폴트 패키지의 루트에 위치시키는 것이 권장된다.
- @ComponentScan 애노테이션이 해당 패키지 부터 시작하기 때문이다. 
- 패키지 하위 패키지들을 컴포넌트 스캔해서 빈을 등록하게 된다.
- src/java/ 아래에 바로 위치하면, 불필요하게 모든 패키지에 대해 @ComponentScan이 동작하게 된다.
