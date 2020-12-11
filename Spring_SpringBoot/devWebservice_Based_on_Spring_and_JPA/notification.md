# 알림

### 주요 기능
- 읽지 않은 / 읽은 알림 목록 조회 
- 읽은 알림 삭제
- 이메일 / 웹 알림
- 관심 주제 && 활동 지역이 매칭 되는 스터디 생성시 알림 받기
- 참여중인 스터디의 변경 사항에 대한 알림 받기
- 모임 참가 신청 결과에 대한 알림 받기
<br>

## 알림 도메인
- Notification에서 Account로 ManyToOne 단방형 관계
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/notification_1.jpg"></p>

<br>

### 구현
- Notification 엔티티 생성
```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
public class Notification {

    @Id @GeneratedValue
    private Long id;

    private String title; // 제목

    private String link; // 링크

    private String message; // 짧은 메시지

    private boolean checked; // 확인 여부

    @ManyToOne
    private Account account; // 누구에게 알림을 보낼 것인지

    private LocalDateTime createdLocalDateTime; // 언제 알림이 왔는지(정렬)

    @Enumerated(EnumType.STRING)
    private NotificationType notificationType; // 알림 타입(새 스터디, 참여 중인 스터디, 모임 참가 신청 결과)
    
}
```
- ENUM 타입 NotificationType 생성
    * STUDY_CREATED : 새 스터디
    * STUDY_UPDATED : 참여 중인 스터디
    * EVENT_ENROLLMENT : 모임 참가 신청 결과 
```java
public enum NotificationType {

    STUDY_CREATED, STUDY_UPDATED, EVENT_ENROLLMENT;

}
```
<br>

## 알림 인프라 설정
- 이미 구현된 StudyService의 메서드 내에서 알림을 보내도 되지만 굳이 트랜잭션 안에 포함시키고 싶지 않기 때문에 비동기 기반으로 처리한다.
- 알림이 서비스 로직에 영향을 주지 않게 처리한다.
- 따라서 스프링이 제공하는 ApplicationEventPublisher와 스프링 `@Async`기능을 사용해서 비동기적인 이벤트 기반으로 알림 처리.
<br>

### 구현
- StudyService에 ApplicationEventPublisher사용
```java
@Service
@Transactional
@RequiredArgsConstructor
public class StudyService {

    ...
    private final ApplicationEventPublisher eventPublisher;

    public Study createNewStudy(Study study, Account account) {
        Study newStudy = repository.save(study);
        newStudy.addManager(account);
        eventPublisher.publishEvent(new StudyCreatedEvent(newStudy)); // 이벤트 던지기
        return newStudy;
    }

    ...
}
```
- 이벤트를 처리할 StudyCreatedEvent 생성
    * 커스텀한 오브젝트 이벤트 핸들링을 지원하기 때문에 굳이 ApplicationEvent를 상속 받을 필요없다.
```java
@Getter
public class StudyCreatedEvent {

    private Study study;

    public StudyCreatedEvent(Study study) {
        this.study = study;
    }
}
```
- 이벤트를 받는 리스너 StudyEventListener 생성
```java
@Slf4j
@Component
@Transactional(readOnly = true)
public class StudyEventListener {

    @EventListener
    public void handleStudyCreatedEvent(StudyCreatedEvent studyCreatedEvent) {
        Study study = studyCreatedEvent.getStudy();
        log.info(study.getTitle() + "is created.");
        // TODO 이메일 보내거나, DB에 Notification 정보를 저장하면 된다.
        throw new RuntimeException();
    }
}
```
- 지금은 메인 로직에 영향을 준다.
    * 같은 쓰레드에서 처리되기 때문에 만약 StudyEventListener에서 예외가 발생하면 rollback되어 스터디 생성이 되지 않는다.
    * 같은 쓰레드에서 처리되지 않게 하면 영향을 주지 않을 것이다.
    * 따라서 스프링 `@Async`기능을 사용한다.
- AsyncConfig 생성
    * `@EnableAsync`를 사용하면 `@Async`를 사용할 수 있지만 기본으로 사용하는 Executor가 그다지 유용하지 않기 때문에 [ThreadPoolTaskExecutor](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/concurrent/ThreadPoolTaskExecutor.html)툴을 사용한다.
        - 참고) 기본 Executor는 매번 쓰레드를 만들어서 처리하는 SimpleAsyncTaskExecutor를 사용한다.
        * 예외를 처리하는 ExceptionHandler도 만들 수 있다.
    * CorePoolSize, MaxPoolSize, QueueCapacity
        - 처리할 task(이벤트)가 생겼을 때, 현재 일하고 있는 쓰레드 개수(active thread)가 코어 개수(core pool size)보다 작으면 남아있는 쓰레드를 사용한다.
        - 현재 일하고 있는 쓰레드 개수가 코어 개수만큼 차있으면 큐 용량(queue capacity)이 찰때까지 큐에 쌓아둔다.
        - 큐 용량이 다 차면, 코어 개수를 넘어서 맥스 개수(max pool size)에 다르기 전까지 새로운 쓰레드를 만들어 처리한다.
        - 맥스 개수를 넘기면 task를 처리하지 못한다.
```java
@Slf4j
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor(); // Java의 ThreadPoolTaskExecutor를 사용한 Executor이다.
        int processors = Runtime.getRuntime().availableProcessors(); // 현재 시스템의 프로세서 개수
        log.info("processors count {}", processors);
        // 기본값 세팅
        executor.setCorePoolSize(processors); // 현재 시스템의 프로세서 개수만큼 코어 사이즈 설정
        executor.setMaxPoolSize(processors * 2); // max를 코어 사이즈 2배
        executor.setQueueCapacity(50); // 큐 용량
        executor.setKeepAliveSeconds(60); // QueueCapacity가 꽉 차서 만든 쓰레드를 얼마나 유지할 것인지(60초)
        executor.setThreadNamePrefix("AsyncExecutor-");
        executor.initialize();
        return executor;
    }
}
```
- StudyEventListener에 `@Async` 애노테이션 추가
```java
@Slf4j
@Async
@Component
@Transactional(readOnly = true)
public class StudyEventListener {

    ...
}
```
<br>

## 스터디 개설 알림
- 스터디를 만들때가 아니라 공개할 때 알림
    * 알림 받을 사람: 스터디 주제와 지역에 매칭이 되는 Account
    * 알림 제목: 스터디 이름
    * 알림 메시지: 스터디 짧은 소개
<br>

### 구현
- 현재는 스터디를 만들었을 때 알림을 보내지만 사실 그렇게 하면 안된다.
    * 스터디가 공개되었을 때 알림을 보내야 한다.
    * 따라서 StudyService의 `createNewStudy()` 메서드의 `eventPublisher.publishEvent(new StudyCreatedEvent(newStudy));`를 삭제한다.
- 스터디가 공개되었을 때 알림을 보낸다.
```java
@Service
@Transactional
@RequiredArgsConstructor
public class StudyService {

    ...

    public void publish(Study study) {
        study.publish();
        this.eventPublisher.publishEvent(new StudyCreatedEvent(study));
    }

    ...
}
```
- 알림을 보내면 StudyEventListener에 들어오고 이벤트를 처리해야 한다.
    * 먼저 스터디 정보를 조회해서 tag와 zone의 정보로 맵핑되는 account 정보를 조회해온다.
```java
@Slf4j
@Async
@Component
@Transactional
@RequiredArgsConstructor
public class StudyEventListener {

    private final StudyRepository studyRepository;

    @EventListener
    public void handleStudyCreatedEvent(StudyCreatedEvent studyCreatedEvent) {
        // studyCreatedEvent로 넘어올 때 study는 detached 객체이므로 이벤트를 처리할 때 조회한다.
        Study study = studyRepository.findStudyWithTagsAndZonesById(studyCreatedEvent.getStudy().getId());
        
    }
}
```
- StudyRepository에 `findStudyWithTagsAndZonesById()` 추가
```java
@Transactional(readOnly = true)
public interface StudyRepository extends JpaRepository<Study, Long> {

    ...

    @EntityGraph(value = "Study.withTagsAndZones", type = EntityGraph.EntityGraphType.FETCH)
    Study findStudyWithTagsAndZonesById(Long id);
}
```
- Study 엔티티에 엔티티 그래프 추가
```java
@NamedEntityGraph(name = "Study.withTagsAndZones", attributeNodes = {
        @NamedAttributeNode("tags"),
        @NamedAttributeNode("zones")})
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Study {
    ...
}
```
- account중 스터디와 적절한 tag, zone을 가진 account를 조회하는 것은 쿼리가 매우 복잡하다. 따라서 QueryDSL을 사용한다.
- QueryDSL 설치
    * 복잡한 쿼리를 스프링 데이터 JPA에서 자바 코드를 사용해서 구현할 수 있게 도와주는 라이브러리
    * Type-safe한 쿼리를 만들 때 사용할 수 있는 Q클래스들을 만들어줘야 한다.
        - 생성해주는 작업을 plugin이 해준다.
        - 반드시 메이븐 컴파일 빌드(mvn compile)를 해야 Q클래스를 생성해준다.
```xml
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
</dependency>

...

<plugin>
    <groupId>com.mysema.maven</groupId>
    <artifactId>apt-maven-plugin</artifactId>
    <version>1.1.3</version>
    <executions>
        <execution>
            <goals>
                <goal>process</goal>
            </goals>
            <configuration>
                <outputDirectory>target/generated-sources/java</outputDirectory>
                <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
            </configuration>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>com.querydsl</groupId>
            <artifactId>querydsl-apt</artifactId>
            <version>${querydsl.version}</version>
        </dependency>
    </dependencies>
</plugin>
```
- 스프링 데이터 JPA와 QueryDSL 연동
    * JpaRepository에 QueryDSL 기능을 쓸 수 있게 QuerydslPredicateExecutor 인터페이스를 추가
```java
@Transactional(readOnly = true)
public interface AccountRepository extends JpaRepository<Account, Long>, QuerydslPredicateExecutor<Account> {

    ...
}
```
- Account를 조회하는 Predicate 사용하기
```java
public class AccountPredicates {

    public static Predicate findByTagsAndZones(Set<Tag> tags, Set<Zone> zones) {
        QAccount account = QAccount.account;
        // account가 가지고 있는 zone와 tag에 해당하는 것이 맵핑이 되는지 return
        return account.zones.any().in(zones).and(account.tags.any().in(tags));
    }
}
```
- StudyEventListener에 QueryDSL을 사용하고 웹, 이메일 알림 기능 구현
```java
@Slf4j
@Async
@Component
@Transactional
@RequiredArgsConstructor
public class StudyEventListener {

    private final StudyRepository studyRepository;
    private final AccountRepository accountRepository;
    private final EmailService emailService;
    private final TemplateEngine templateEngine;
    private final AppProperties appProperties;
    private final NotificationRepository notificationRepository;

    @EventListener
    public void handleStudyCreatedEvent(StudyCreatedEvent studyCreatedEvent) {
        Study study = studyRepository.findStudyWithTagsAndZonesById(studyCreatedEvent.getStudy().getId());
        Iterable<Account> accounts = accountRepository.findAll(AccountPredicates.findByTagsAndZones(study.getTags(), study.getZones()));
        accounts.forEach(account -> {
            if (account.isStudyCreatedByEmail()) { // 이메일 알림
                sendStudyCreatedEmail(study, account);
            }

            if (account.isStudyCreatedByWeb()) { // 웹 알림
                saveStudyCreatedNotification(study, account);
            }
        });
    }
    // 웹 알림
    private void saveStudyCreatedNotification(Study study, Account account) {
        Notification notification = new Notification();
        notification.setTitle(study.getTitle());
        notification.setLink("/study/" + study.getEncodedPath());
        notification.setChecked(false);
        notification.setCreatedLocalDateTime(LocalDateTime.now());
        notification.setMessage(study.getShortDescription());
        notification.setAccount(account);
        notification.setNotificationType(NotificationType.STUDY_CREATED);
        notificationRepository.save(notification);
    }
    // 이메일 알림
    private void sendStudyCreatedEmail(Study study, Account account) {
        Context context = new Context();
        context.setVariable("nickname", account.getNickname());
        context.setVariable("link", "/study/" + study.getEncodedPath());
        context.setVariable("linkName", study.getTitle());
        context.setVariable("message", "새로운 스터디가 생겼습니다.");
        context.setVariable("host", appProperties.getHost());
        String message = templateEngine.process("mail/simple-link", context);

        EmailMessage emailMessage = EmailMessage.builder()
                .subject("스터디올래, '" + study.getTitle() + "' 스터디가 생겼습니다.")
                .to(account.getEmail())
                .message(message)
                .build();

        emailService.sendEmail(emailMessage);
    }
}
```
- NotificationRepository 생성
```java
public interface NotificationRepository extends JpaRepository<Notification, Long> {

}
```
- 실행해서 스터디를 만들고 스터디 주제와 지역을 account가 가지고 있는 tag와 zone 중 하나로 설정하면 다음과 같이 알림이 온다.
    * 웹 알림이 왔는지 조회하는 기능은 아직 구현하지 않았지만 DB에는 알림이 저장된 것을 볼 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/notification_2.jpg"></p>
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/notification_3.jpg"></p>

<br>

## 알림 아이콘 변경
- 읽지 않은 웹 알림이 있는 경우, 메인 네비게이션 바의 알림 아이콘을 색이 있는 아이콘으로 변경한다.
    * 핸들러 처리 이후, 뷰 랜더링 전에 스프링 웹 MVC HandlerInterceptor로 읽지 않은 메시지가 있는지 Model에 담아준다.
- NotificationInterceptor 적용 범위
    * redirect 요청에는 적용 X
    * static 리소스 요청에는 적용 X
<br>

### 구현
- NotificationInterceptor 생성
    * `postHandle()`
        - 뷰 렌더링 전, 핸들러 처리 이후에 동작
```java
@Component
@RequiredArgsConstructor
public class NotificationInterceptor implements HandlerInterceptor {

    private final NotificationRepository notificationRepository;

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        // 인증 정보가 있는 요청에 대한 응답에만 실행할 것이기 때문에 authentication을 가져온다.
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication(); 
        // modelAndView를 쓰는 경우에만 model에 넣는다.
        if (modelAndView != null && !isRedirectView(modelAndView) && authentication != null && authentication.getPrincipal() instanceof UserAccount) {
            Account account = ((UserAccount)authentication.getPrincipal()).getAccount();
            long count = notificationRepository.countByAccountAndChecked(account, false);
            modelAndView.addObject("hasNotification", count > 0);
        }
    }
    // redirect인지 아닌지 확인
    private boolean isRedirectView(ModelAndView modelAndView) {
        return modelAndView.getViewName().startsWith("redirect:") || modelAndView.getView() instanceof RedirectView;
    }
}
```
- NotificationRepository에 `countByAccountAndChecked()` 생성
```java
public interface NotificationRepository extends JpaRepository<Notification, Long> {
    long countByAccountAndChecked(Account account, boolean checked);
}
```
- WebConfig 생성
    * HandlerInterceptor를 만들어서 Bean으로 등록한다고 바로 사용되는 것은 아니다.
    * 스프링 MVC 설정을 해야한다.
    * `@EnableWebMvc`를 쓰면 스프링 부트가 제공하는 웹 MVC 자동설정을 사용하지 않겠다는 의미.(전부 직접 작성해야 한다.)
```java
@Configuration
@RequiredArgsConstructor
public class WebConfig implements WebMvcConfigurer {

    private final NotificationInterceptor notificationInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // StaticResourceLocation이라는 enum이 staticResourcesPath를 가지고 있기 때문에 문자열로 바꿔서 List에 저장한다.
        List<String> staticResourcesPath = Arrays.stream(StaticResourceLocation.values())
                .flatMap(StaticResourceLocation::getPatterns)
                .collect(Collectors.toList());
        staticResourcesPath.add("/node_modules/**"); // 커스텀하게 추가도 가능하다.

        registry.addInterceptor(notificationInterceptor)
                .excludePathPatterns(staticResourcesPath); // static 리소스 요청에는 적용 X 
    }
}
```
- 뷰 수정
```html
<!-- fragments.html -->
<!-- <i class="fa fa-bell-o" aria-hidden="true"></i> -->
<i th:if="${!hasNotification}" class="fa fa-bell-o" aria-hidden="true"></i>
<span class="text-info" th:if="${hasNotification}"><i class="fa fa-bell" aria-hidden="true"></i></span>
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/notification_4.jpg"></p>

<br>

## 알림 목록 조회 및 삭제
- 기본적으로 알림 벨 아이콘을 클릭하면 읽지 않은 알림 보여주는 요청을 보낸다.
    * 읽음 처리 된다.
- 하나의 뷰로 읽지 않은 알림, 읽은 알림을 보여주는 데 쓴다.
<br>

### 구현
- 알림 목록 조회 및 삭제 생성
    * `@GetMapping("/notifications")`
        - 읽지 않은 알림 메시지만 보여준다.
        - 알림 메시지를 카테고리 별로 나눠서 Model에 담아주고 뷰에서 보여준다.
        - 모든 읽지 않은 알림 메시지를 읽은 메시지로 수정한다.
    * `@GetMapping("/notifications/old")`
        - 읽은 알림 메시지를 보여준다.
    * `@DeleteMapping("/notifications")`
        - 읽은 알림 메시지를 삭제한다.
    * `putCategorizedNotifications()`
        - 읽지 않은 알림들을 카테고리로 나눠서 Model에 담는 메서드
```java
@Controller
@RequiredArgsConstructor
public class NotificationController {

    private final NotificationRepository repository;
    private final NotificationService service;

    @GetMapping("/notifications")
    public String getNotifications(@CurrentAccount Account account, Model model) {
        List<Notification> notifications = repository.findByAccountAndCheckedOrderByCreatedDateTimeDesc(account, false);
        long numberOfChecked = repository.countByAccountAndChecked(account, true);
        putCategorizedNotifications(model, notifications, numberOfChecked, notifications.size());
        model.addAttribute("isNew", true);
        service.markAsRead(notifications);
        return "notification/list";
    }

    @GetMapping("/notifications/old")
    public String getOldNotifications(@CurrentAccount Account account, Model model) {
        List<Notification> notifications = repository.findByAccountAndCheckedOrderByCreatedDateTimeDesc(account, true);
        long numberOfNotChecked = repository.countByAccountAndChecked(account, false);
        putCategorizedNotifications(model, notifications, notifications.size(), numberOfNotChecked);
        model.addAttribute("isNew", false);
        return "notification/list";
    }

    @DeleteMapping("/notifications")
    public String deleteNotifications(@CurrentAccount Account account) {
        repository.deleteByAccountAndChecked(account, true);
        return "redirect:/notifications";
    }

    private void putCategorizedNotifications(Model model, List<Notification> notifications,
                                             long numberOfChecked, long numberOfNotChecked) {
        List<Notification> newStudyNotifications = new ArrayList<>(); // 새 스터디 관련 알림
        List<Notification> eventEnrollmentNotifications = new ArrayList<>(); // 참가 신청 관련 알림
        List<Notification> watchingStudyNotifications = new ArrayList<>(); // 이미 참여 중인 스터디 알림
        for (var notification : notifications) {
            switch (notification.getNotificationType()) {
                case STUDY_CREATED: newStudyNotifications.add(notification); break;
                case EVENT_ENROLLMENT: eventEnrollmentNotifications.add(notification); break;
                case STUDY_UPDATED: watchingStudyNotifications.add(notification); break;
            }
        }

        model.addAttribute("numberOfNotChecked", numberOfNotChecked);
        model.addAttribute("numberOfChecked", numberOfChecked);
        model.addAttribute("notifications", notifications);
        model.addAttribute("newStudyNotifications", newStudyNotifications);
        model.addAttribute("eventEnrollmentNotifications", eventEnrollmentNotifications);
        model.addAttribute("watchingStudyNotifications", watchingStudyNotifications);
    }
}
```
- NotificationRepository
    * `@Transactional`
        - 알림의 상태가 바뀔 것이기 때문에 readOnly가 아니어야 한다.
```java
@Transactional(readOnly = true)
public interface NotificationRepository extends JpaRepository<Notification, Long> {
    long countByAccountAndChecked(Account account, boolean checked); // 읽은 알림 개수

    @Transactional
    List<Notification> findByAccountAndCheckedOrderByCreatedDateTimeDesc(Account account, boolean checked); // 읽지 않은 알림 조회

    @Transactional
    void deleteByAccountAndChecked(Account account, boolean checked);
}
```
- NotificationService 생성
    * `markAsRead()`
        - 읽지 않은 알림을 모두 읽음 처리
```java
@Service
@Transactional
@RequiredArgsConstructor
public class NotificationService {

    private final NotificationRepository notificationRepository;

    public void markAsRead(List<Notification> notifications) {
        notifications.forEach(n -> n.setChecked(true));
        notificationRepository.saveAll(notifications);
    }
}
```
- 알림 목록 조회 및 삭제 관련 뷰 추가
```html
<!-- fragments.html -->
<ul th:fragment="notification-list (notifications)" class="list-group list-group-flush">
    <a href="#" th:href="@{${noti.link}}" th:each="noti: ${notifications}"
       class="list-group-item list-group-item-action">
        <div class="d-flex w-100 justify-content-between">
            <small class="text-muted" th:text="${noti.title}">Noti title</small>
            <small class="fromNow text-muted" th:text="${noti.createdDateTime}">3 days ago</small>
        </div>
        <p th:text="${noti.message}" class="text-left mb-0 mt-1">message</p>
    </a>
</ul>
```
```html
<!-- list.html -->
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body class="bg-light">
    <nav th:replace="fragments.html :: main-nav"></nav>
    <div class="container">
        <div class="row py-5 text-center">
            <div class="col-3">
                <ul class="list-group">
                    <a href="#" th:href="@{/notifications}" th:classappend="${isNew}? active"
                       class="list-group-item list-group-item-action d-flex justify-content-between align-items-center">
                        읽지 않은 알림
                        <span th:text="${numberOfNotChecked}">3</span>
                    </a>
                    <a href="#" th:href="@{/notifications/old}" th:classappend="${!isNew}? active"
                       class="list-group-item list-group-item-action d-flex justify-content-between align-items-center">
                        읽은 알림
                        <span th:text="${numberOfChecked}">0</span>
                    </a>
                </ul>

                <ul class="list-group mt-4">
                    <a href="#" th:if="${newStudyNotifications.size() > 0}"
                       class="list-group-item list-group-item-action d-flex justify-content-between align-items-center">
                        새 스터디 알림
                        <span th:text="${newStudyNotifications.size()}">3</span>
                    </a>
                    <a href="#" th:if="${eventEnrollmentNotifications.size() > 0}"
                       class="list-group-item list-group-item-action d-flex justify-content-between align-items-center">
                        모임 참가 신청 알림
                        <span th:text="${eventEnrollmentNotifications.size()}">0</span>
                    </a>
                    <a href="#" th:if="${watchingStudyNotifications.size() > 0}"
                       class="list-group-item list-group-item-action d-flex justify-content-between align-items-center">
                        관심있는 스터디 알림
                        <span th:text="${watchingStudyNotifications.size()}">0</span>
                    </a>
                </ul>

                <ul class="list-group mt-4" th:if="${numberOfChecked > 0}">
                    <form th:action="@{/notifications}" th:method="delete">
                        <button type="submit" class="btn btn-block btn-outline-warning" aria-describedby="deleteHelp">
                            읽은 알림 삭제
                        </button>
                        <small id="deleteHelp">삭제하지 않아도 한달이 지난 알림은 사라집니다.</small>
                    </form>
                </ul>
            </div>
            <div class="col-9">
                <div class="card" th:if="${notifications.size() == 0}">
                    <div class="card-header">
                        알림 메시지가 없습니다.
                    </div>
                </div>

                <div class="card" th:if="${newStudyNotifications.size() > 0}">
                    <div class="card-header">
                        주요 활동 지역에 관심있는 주제의 스터디가 생겼습니다.
                    </div>
                    <div th:replace="fragments.html :: notification-list (notifications=${newStudyNotifications})"></div>
                </div>

                <div class="card mt-4" th:if="${eventEnrollmentNotifications.size() > 0}">
                    <div class="card-header">
                        모임 참가 신청 관련 소식이 있습니다.
                    </div>
                    <div th:replace="fragments.html :: notification-list (notifications=${eventEnrollmentNotifications})"></div>
                </div>

                <div class="card mt-4" th:if="${watchingStudyNotifications.size() > 0}">
                    <div class="card-header">
                        참여중인 스터디 관련 소식이 있습니다.
                    </div>
                    <div th:replace="fragments.html :: notification-list (notifications=${watchingStudyNotifications})"></div>
                </div>
            </div>
        </div>
        <div th:replace="fragments.html :: footer"></div>
    </div>
    <script th:replace="fragments.html :: date-time"></script>
</body>
</html>
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/notification_5.jpg"></p>

<br>
