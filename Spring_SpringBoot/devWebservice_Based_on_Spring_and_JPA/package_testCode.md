# 패키지 및 테스트 코드 정리

## 패키지 정리
- 지금까지 만든 도메인 엔티티 클래스 간의 관계는 그림과 같이 5개의 묶음으로 나눌 수 있다.
    * 단방향
- 양방향이 되거나 circular한 관계가 되버리면 모듈화로 분리하기가 힘들다.
    * 따라서 처음부터 패키지 모듈을 고민하면서 의존성을 고민하면 편하다.
<br>

### 스터디 올래의 패키지 정리
- 애플리케이션을 크게 보면 2가지로 나눠볼 수 있다.
    * Infra
    * Modules
- config와 mail을 infra패키지로 옮기고 domain에 있는 클래스들을 각자의 패키지로 분리한다.
- 그런다음 infra를 제외한 모든 패키지를 modules에 옮긴다.
- 이 때 config쪽에는 modules에 있는 것을 참조하고 있는데 infra는 Spring, JPA나 third party libraries를 참조하도록 바꾸고 가급적이면 modules은 infra를 참조하지만 infra는 modules를 참조하지 않도록 바꾼다.
- SecurityConfig의 AccountService를 UserDetailsService로 변경
    * AccountService가 UserDetailsService 타입을 구현한 것이기 때문
    * Spring Security에서도 UserDetailsService를 필요로 한다.
```java
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    // private final AccountService accountService;
    private final UserDetailsService userDetailsService;
    
    ...
}
```
- 나머지는 클래스가 많기 때문에 눈으로 하기 힘들기 때문에 tool의 도움을 받아야 한다.
    * [ArchUnit](https://www.archunit.org/)
    * 아키텍처 테스트 유틸리티 (JUnit 5 지원)
```xml
<dependency>
    <groupId>com.tngtech.archunit</groupId>
    <artifactId>archunit-junit5</artifactId>
    <version>0.13.1</version>
    <scope>test</scope>
</dependency>
```
- PackageDependencyTests를 만들고 ArchUnit으로 테스트 코드 작성
```java
@AnalyzeClasses(packagesOf = App.class)
public class PackageDependencyTests {

    private static final String STUDY = "..modules.study..";
    private static final String EVENT = "..modules.event..";
    private static final String ACCOUNT = "..modules.account..";
    private static final String TAG = "..modules.tag..";
    private static final String ZONE = "..modules.zone..";

    @ArchTest
    ArchRule modulesPackageRule = classes().that().resideInAPackage("com.studyolle.modules..")
            .should().onlyBeAccessed().byClassesThat()
            .resideInAnyPackage("com.studyolle.modules.."); // modules에 있는 건 modules에만 참조하도록 테스트

    @ArchTest
    ArchRule studyPackageRule = classes().that().resideInAPackage("..modules.study..")
            .should().onlyBeAccessed().byClassesThat()
            .resideInAnyPackage(STUDY, EVENT); // Study 패키지에 있는 클래스는 Event, Study에 들어있는 클래스에서만 사용한다.

    @ArchTest
    ArchRule eventPackageRule = classes().that().resideInAPackage(EVENT)
            .should().accessClassesThat().resideInAnyPackage(STUDY, ACCOUNT, EVENT); // Event패키지에 있는 클래스는 Study, Account, Event패키지에 들어있는 클래스를 사용한다.
    @ArchTest
    ArchRule accountPackageRule = classes().that().resideInAPackage(ACCOUNT)
            .should().accessClassesThat().resideInAnyPackage(TAG, ZONE, ACCOUNT); // Account패키지에 있는 클래스는 TAG, ZONE, ACCOUNT패키지에 들어있는 클래스를 사용한다.

    @ArchTest
    ArchRule cycleCheck = slices().matching("com.studyolle.modules.(*)..") // 각각의 모듈 간의 순환 참조 문제가 없는지 확인
            .should().beFreeOfCycles();
}
```
- 실행해보면 Account 엔티티 `isManagerOf()` 메서드에서 account가 study를 참조하고 있다고 오류가 난다.
    * 따라서 Account 엔티티의 `isManagerOf()`를 삭제하고 Study 엔티티에 `isManagedBy()`메서드 생성하고
    * StudyService에 `checkIfManager()`메서드에서 `isManagedBy()`를 사용하도록 코드를 수정한다.
```java
...
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Study {

    ...
    
    public boolean isManagedBy(Account account) {
        return this.getManagers().contains(account);
    }
}
```
```java
@Service
@Transactional
@RequiredArgsConstructor
public class StudyService {

    ...

    private void checkIfManager(Account account, Study study) {
        if (!study.isManagedBy(account)) {
            throw new AccessDeniedException("해당 기능을 사용할 수 없습니다.");
        }
    }

    ...
}
```
- 또한 settings 패키지 관련 순환 참조 오류도 발생한다.
    * settings 패키지가 모두 Account와 관련된 것이므로 Account 패키지로 옮겨준다.
- modulesPackage 에러는 WithAccountSecurityContextFactory와 WithAccount클래스를 account쪽으로 옮겨주면 해결 가능하다.
    * Account와 관련된 것
<br>

## 테스트 클래스 정리
- 테스트 코드 간의 상속을 만들어서 하위 클래스를 실행하면 상위 클래스의 테스트가 실행되는 문제점이 생겼다.
- 따라서 Martin Fowler가 썼던 [ObjectMother](https://martinfowler.com/bliki/ObjectMother.html)라는 글을 참고하여 정리한다.
    * 여기서는 Mother라는 표현보다는 Factory라는 단어를 사용한다.
- 애노테이션 중복 사용을 막기 위해 MockMvcTest 인터페이스 생성
    * 커스텀 애노테이션으로 테스트 애노테이션 묶음 만들기
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Transactional
@SpringBootTest
@AutoConfigureMockMvc
public @interface MockMvcTest {
    
}
```
- AccountFactory 생성
```java
@Component
@RequiredArgsConstructor
public class AccountFactory {

    @Autowired AccountRepository accountRepository;

    public Account createAccount(String nickname) {
        Account kimhayoung = new Account();
        kimhayoung.setNickname(nickname);
        kimhayoung.setEmail(nickname + "@email.com");
        accountRepository.save(kimhayoung);
        return kimhayoung;
    }
}
```
- StudyFactory 생성
```java
@Component
@RequiredArgsConstructor
public class StudyFactory {

    @Autowired StudyService studyService;
    @Autowired StudyRepository studyRepository;

    public Study createStudy(String path, Account manager) {
        Study study = new Study();
        study.setPath(path);
        studyService.createNewStudy(study, manager);
        return study;
    }
}
```
- 이제 나머지 클래스들에 테스트 클래스들의 상속 관계를 없애고 `@MockMvcTest`, AccountFactory, StudyFactory를 사용하면 된다.
    * [코드 커밋 참고](https://github.com/qlalzl9/studyolle/commit/e0c2097c88ef76beb4bc23b8ce00aafc07b73725)
    * 수정 후 전체 테스트를 실행하면 성공적으로 수행된다.
<br>
