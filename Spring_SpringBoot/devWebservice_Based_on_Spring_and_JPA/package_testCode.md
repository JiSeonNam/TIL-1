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
            .resideInAnyPackage(STUDY, EVENT); // Study 클래스는 Study, Event 클래스에서만 접근이 가능해야 한다.

    @ArchTest
    ArchRule eventPackageRule = classes().that().resideInAPackage(EVENT)
            .should().accessClassesThat().resideInAnyPackage(STUDY, ACCOUNT, EVENT); // Event는 Event, Study, Account를 참조한다.

    @ArchTest
    ArchRule accountPackageRule = classes().that().resideInAPackage(ACCOUNT)
            .should().accessClassesThat().resideInAnyPackage(TAG, ZONE, ACCOUNT); // Account는 TAG, ZONE, ACCOUNT를 참조한다.

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
