> [강의 링크](https://www.inflearn.com/course/spring-framework_core)
 
 # 스프링 프레임워크 핵심 기술

 ## IoC 컨테이너와 빈
- Inversion of Control : 의존 관계 주입(Dependency Injection)이라고도 하며, 어떤 객체가 사용하는 의존 객체를 직접 만들어 사용하는게 아니라, 주입 받아 사용하는 방법을 말한다.

### 스프링 IoC 컨테이너
- [BeanFactory](https://docs.spring.io/spring-framework/docs/5.0.8.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html) : 스프링 IoC 컨테이너의 최상위 인터페이스 
- ApplicationContext
    * BeanFactory를 상속받고 있기 때문에 BeanFactory의 모든 기능을 포함하며, 추가 기능도 제공되기 때문에 일반적으로 BeanFactory보다 추천된다. 
    * 메세지 소스 기능(i18n)
    * 이벤트 발행 기능
    * 리소스 로딩 기능 등
- 애플리케이션 컴포넌트의 중앙 저장소.
- 빈 설정 소스로 부터 빈 정의를 읽어들이고, 빈을 구성하고 제공한다.

### 빈
- Bean : 스프링 IoC 컨테이너가 가지고 있고 관리하는 객체.
- 빈으로 등록되면 좋은 장점
    * 의존성 관리(빈으로 등록되어 있어야만 의존성 주입이 가능하다.)
    * Test하기 편리해진다.
    * 스코프
        - 싱글톤 : 하나의 객체를 사용, 기본적으로 빈으로 등록할 때 싱글톤으로 등록된다.
        - 프로토타입 : 매번 다른 객체를 사용
    * 라이프사이클 인터페이스 지원


