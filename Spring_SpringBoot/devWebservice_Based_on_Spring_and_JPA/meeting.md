# 모임

### 모임 관련 주요 기능
- 모임 (목록) 조회
- 모임 개설
- 모임 수정 / 취소(삭제)
- 모임 참가 신청 / 취소
- 모임 참가 신청 확인 / 거절 / 출석 체크
<br>

## 모임 도메인
- Event
    * Event에서 Study쪽으로 `@ManyToOne` 단방향 관계
    * Event와 Enrollment는 `@OneToMany` `@ManyToOne` 양방향 관계
    * Event는 Account쪽으로 `@ManyToOne` 단방향 관계
    * Enrollment는 Account쪽으로 `@ManyToOne` 단방향 관계
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/meeting_1.jpg"></p>

<br>

### 구현
- Event 엔티티 추가
```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
public class Event {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    private Study study; // 어느 스터디에 속한 모임인지

    @ManyToOne
    private Account createBy; // 모임을 만든 사람

    @Column(nullable = false)
    private String title; // 모임의 제목

    @Lob
    private String description; // 모임의 본문

    @Column(nullable = false)
    private LocalDateTime createdDateTime; // 모임 생성 시간

    @Column(nullable = false)
    private LocalDateTime endEnrollmentDateTime; // 모임 접수 종료 시간

    @Column(nullable = false)
    private LocalDateTime startDateTime; // 모임 시작 일시

    @Column(nullable = false)
    private LocalDateTime endDateTime; // 모임 종료 일시

    @Column
    private Integer limitOfEnrollments; // 최대 참가 인원

    @OneToMany(mappedBy = "event")
    private List<Enrollment> enrollments; // 참가 신청

    @Enumerated(EnumType.STRING)
    private EventType eventType; // 모임 타입
}
```
- 모임 타입 ENUM 생성
```java
public enum EventType {
    // 선착순, 관리자 확인 후 승인
    FCFS, CONFIRMATIVE;

}
```
- 참가 신청 Enrollment
```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
public class Enrollment {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    private Event event; // 어떤 모임에 대한 참가 신청인지 

    @ManyToOne
    private Account account; // 신청자 정보

    private LocalDateTime enrolledAt; // 신청 시간

    private boolean accepted; // 참가 상태가 확정 여부

    private boolean attended; // 실제 참석 여부
}
```
<br>

## 모임 만들기
- EventForm 생성
    * `@DateTimeFormat`
        - 어떤 식으로 formatting해서 받아올지 정할 수 있다. (여기서는 iso format 사용)
```java
@Data
public class EventForm {

    @NotBlank
    @Length(max = 50)
    private String title;

    private String description;

    private EventType eventType = EventType.FCFS;

    @DateTimeFormat(iso = DateTimeFormat.ISO.DATE_TIME)
    private LocalDateTime endEnrollmentDateTime;

    @DateTimeFormat(iso = DateTimeFormat.ISO.DATE_TIME)
    private LocalDateTime startDateTime;

    @DateTimeFormat(iso = DateTimeFormat.ISO.DATE_TIME)
    private LocalDateTime endDateTime;

    @Min(2)
    private Integer limitOfEnrollments = 2;
    
}
```
- EventController 생성
```java
@Controller
@RequestMapping("/study/{path}")
@RequiredArgsConstructor
public class EventController {

    private final StudyService studyService;
    private final EventService eventService;
    private final ModelMapper modelMapper;

    @GetMapping("/new-event")
    public String newEventForm(@CurrentAccount Account account, @PathVariable String path, Model model) {
        Study study = studyService.getStudyToUpdateStatus(account, path); // account가 스터디의 매니저인지만 확인하면 된다.
        model.addAttribute(study);
        model.addAttribute(account);
        model.addAttribute(new EventForm());
        return "event/form";
    }

    @PostMapping("/new-event")
    public String newEventSubmit(@CurrentAccount Account account, @PathVariable String path,
                                 @Valid EventForm eventForm, Errors errors, Model model) {
        Study study = studyService.getStudyToUpdateStatus(account, path);
        if (errors.hasErrors()) {
            model.addAttribute(account);
            model.addAttribute(study);
            return "event/form";
        }

        Event event = eventService.createEvent(modelMapper.map(eventForm, Event.class), study, account);
        return "redirect:/study/" + study.getEncodedPath() + "/events/" + event.getId();
    }
}
```
- EventService 생성
```java
@Service
@Transactional
@RequiredArgsConstructor
public class EventService {

    private final EventRepository eventRepository;

    public Event createEvent(Event event, Study study, Account account) {
        event.setCreateBy(account);
        event.setCreatedDateTime(LocalDateTime.now());
        event.setStudy(study);
        return eventRepository.save(event);
    }
}
```
- EventRepository 생성
```java
@Transactional(readOnly = true)
public interface EventRepository extends JpaRepository<Event, Long> {

}
```
- EventValidator 생성
```java
@Component
public class EventValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        return EventForm.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        EventForm eventForm = (EventForm)target;

        if (isNotValidEndEnrollmentDateTime(eventForm)) {
            errors.rejectValue("endEnrollmentDateTime", "wrong.datetime", "모임 접수 종료 일시를 정확히 입력하세요.");
        }

        if (isNotValidEndDateTime(eventForm)) {
            errors.rejectValue("endDateTime", "wrong.datetime", "모임 종료 일시를 정확히 입력하세요.");
        }

        if (isNotValidStartDateTime(eventForm)) {
            errors.rejectValue("startDateTime", "wrong.datetime", "모임 시작 일시를 정확히 입력하세요.");
        }
    }

    private boolean isNotValidStartDateTime(EventForm eventForm) {
        // 모임 시작 시간이 접수 마감 시간보다 이전일 경우
        return eventForm.getStartDateTime().isBefore(eventForm.getEndEnrollmentDateTime());
    }

    private boolean isNotValidEndEnrollmentDateTime(EventForm eventForm) {
        // 접수 마감 시간이 지금 보다 이전일 경우 
        return eventForm.getEndEnrollmentDateTime().isBefore(LocalDateTime.now());
    }

    private boolean isNotValidEndDateTime(EventForm eventForm) {
        LocalDateTime endDateTime = eventForm.getEndDateTime();
        // 모임 종료 시간이 시작 시간보다 이전일 경우 또는 모임 종료 시간이 접수 종료 시간보다 이전일 경우
        return endDateTime.isBefore(eventForm.getStartDateTime()) || endDateTime.isBefore(eventForm.getEndEnrollmentDateTime());
    }
}
```
- EventController에 Validator 추가
```java
@Controller
@RequestMapping("/study/{path}")
@RequiredArgsConstructor
public class EventController {

    ...
    private final EventValidator eventValidator;

    @InitBinder("eventForm")
    public void initBinder(WebDataBinder webDataBinder) {
        webDataBinder.addValidators(eventValidator);
    }

    ...
}
```
- 모임 form 뷰 추가
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body class="bg-light">
    <nav th:replace="fragments.html :: main-nav"></nav>
    <div th:replace="fragments.html :: study-banner"></div>
    <div class="container">
        <div class="py-5 text-center">
            <h2><a th:href="@{'/study/' + ${study.path}}"><span th:text="${study.title}">스터디</span></a> / 새 모임 만들기</h2>
        </div>
        <div class="row justify-content-center">
            <form class="needs-validation col-sm-10" th:action="@{'/study/' + ${study.path} + '/new-event'}"
                  th:object="${eventForm}" method="post" novalidate>
                <div class="form-group">
                    <label for="title">모임 이름</label>
                    <input id="title" type="text" th:field="*{title}" class="form-control"
                           placeholder="모임 이름" aria-describedby="titleHelp" required>
                    <small id="titleHelp" class="form-text text-muted">
                        모임 이름을 50자 이내로 입력하세요.
                    </small>
                    <small class="invalid-feedback">모임 이름을 입력하세요.</small>
                    <small class="form-text text-danger" th:if="${#fields.hasErrors('title')}" th:errors="*{title}">Error</small>
                </div>
                <div class="form-group">
                    <label for="eventType">모집 방법</label>
                    <select th:field="*{eventType}"  class="custom-select mr-sm-2" id="eventType" aria-describedby="eventTypeHelp">
                        <option th:value="FCFS">선착순</option>
                        <option th:value="CONFIRMATIVE">관리자 확인</option>
                    </select>
                    <small id="eventTypeHelp" class="form-text text-muted">
                        두가지 모집 방법이 있습니다.<br/>
                        <strong>선착순</strong>으로 모집하는 경우, 모집 인원 이내의 접수는 자동으로 확정되며, 제한 인원을 넘는 신청은 대기 신청이 되며 이후에 확정된 신청 중에 취소가 발생하면 선착순으로 대기 신청자를 확정 신청자도 변경합니다. 단, 등록 마감일 이후에는 취소해도 확정 여부가 바뀌지 않습니다.<br/>
                        <strong>확인</strong>으로 모집하는 경우, 모임 및 스터디 관리자가 모임 신청 목록을 조회하고 직접 확정 여부를 정할 수 있습니다. 등록 마감일 이후에는 변경할 수 없습니다.
                    </small>
                </div>
                <div class="form-row">
                    <div class="form-group col-md-3">
                        <label for="limitOfEnrollments">모집 인원</label>
                        <input id="limitOfEnrollments" type="number" th:field="*{limitOfEnrollments}" class="form-control" placeholder="0"
                               aria-describedby="limitOfEnrollmentsHelp">
                        <small id="limitOfEnrollmentsHelp" class="form-text text-muted">
                            최대 수용 가능한 모임 참석 인원을 설정하세요. 최소 2인 이상 모임이어야 합니다.
                        </small>
                        <small class="invalid-feedback">모임 신청 마감 일시를 입력하세요.</small>
                        <small class="form-text text-danger" th:if="${#fields.hasErrors('limitOfEnrollments')}" th:errors="*{limitOfEnrollments}">Error</small>
                    </div>
                    <div class="form-group col-md-3">
                        <label for="endEnrollmentDateTime">등록 마감 일시</label>
                        <input id="endEnrollmentDateTime" type="datetime-local" th:field="*{endEnrollmentDateTime}" class="form-control"
                               aria-describedby="endEnrollmentDateTimeHelp" required>
                        <small id="endEnrollmentDateTimeHelp" class="form-text text-muted">
                            등록 마감 이전에만 스터디 모임 참가 신청을 할 수 있습니다.
                        </small>
                        <small class="invalid-feedback">모임 신청 마감 일시를 입력하세요.</small>
                        <small class="form-text text-danger" th:if="${#fields.hasErrors('endEnrollmentDateTime')}" th:errors="*{endEnrollmentDateTime}">Error</small>
                    </div>
                    <div class="form-group col-md-3">
                        <label for="startDateTime">모임 시작 일시</label>
                        <input id="startDateTime" type="datetime-local" th:field="*{startDateTime}" class="form-control"
                               aria-describedby="startDateTimeHelp" required>
                        <small id="startDateTimeHelp" class="form-text text-muted">
                            모임 시작 일시를 입력하세요. 상세한 모임 일정은 본문에 적어주세요.
                        </small>
                        <small class="invalid-feedback">모임 시작 일시를 입력하세요.</small>
                        <small class="form-text text-danger" th:if="${#fields.hasErrors('startDateTime')}" th:errors="*{startDateTime}">Error</small>
                    </div>
                    <div class="form-group col-md-3">
                        <label for="startDateTime">모임 종료 일시</label>
                        <input id="endDateTime" type="datetime-local" th:field="*{endDateTime}" class="form-control"
                               aria-describedby="endDateTimeHelp" required>
                        <small id="endDateTimeHelp" class="form-text text-muted">
                            모임 종료 일시가 지나면 모임은 자동으로 종료 상태로 바뀝니다.
                        </small>
                        <small class="invalid-feedback">모임 종료 일시를 입력하세요.</small>
                        <small class="form-text text-danger" th:if="${#fields.hasErrors('endDateTime')}" th:errors="*{endDateTime}">Error</small>
                    </div>
                </div>
                <div class="form-group">
                    <label for="description">모임 설명</label>
                    <textarea id="description" type="textarea" th:field="*{description}" class="editor form-control"
                              placeholder="모임을 자세히 설명해 주세요." aria-describedby="descriptionHelp" required></textarea>
                    <small id="descriptionHelp" class="form-text text-muted">
                        모임에서 다루는 주제, 장소, 진행 방식 등을 자세히 적어 주세요.
                    </small>
                    <small class="invalid-feedback">모임 설명을 입력하세요.</small>
                    <small class="form-text text-danger" th:if="${#fields.hasErrors('description')}" th:errors="*{description}">Error</small>
                </div>
                <div class="form-group">
                    <button class="btn btn-primary btn-block" type="submit"
                            aria-describedby="submitHelp">모임 만들기</button>
                </div>
            </form>
        </div>
        <div th:replace="fragments.html :: footer"></div>
    </div>
    <script th:replace="fragments.html :: form-validation"></script>
    <script th:replace="fragments.html :: editor-script"></script>
</body>
</html>
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/meeting_2.jpg"></p>

<br>

## 모임 조회
- 팝업창을 띄우기 위해 BootStrap의 [modal](https://getbootstrap.com/docs/4.4/components/modal/) 사용
- 날짜를 여러 형태로 표현해주는 라이브러리인 [Moment.js](https://momentjs.com/) 사용
    * ex) 2020년 12월, 일주일 뒤, 4시간 전, ...
    * npm install moment --save
<br>

### 구현
- EventController에 조회 관련 맵핑 추가
    * `findById()`
        - Optional 타입으로 넘어오기 때문에 있으면 리턴하고 없으면 에러를 리턴하는 `orElseThrow()` 사용
```java
@Controller
@RequestMapping("/study/{path}")
@RequiredArgsConstructor
public class EventController {

    ...
    private final EventRepository eventRepository;

    ...

    @GetMapping("/events/{id}")
    public String getEvent(@CurrentAccount Account account, @PathVariable String path, @PathVariable Long id,
                           Model model) {
        model.addAttribute(account);
        model.addAttribute(eventRepository.findById(id).orElseThrow());
        model.addAttribute(studyService.getStudy(path));
        return "event/view";
    }
}
```
- Event 엔티티에 모임 조회 관련 메서드 추가
```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
public class Event {

    ...

    public boolean isEnrollableFor(UserAccount userAccount) {
        return isNotClosed() && !isAlreadyEnrolled(userAccount);
    }

    public boolean isDisenrollableFor(UserAccount userAccount) {
        return isNotClosed() && isAlreadyEnrolled(userAccount);
    }

    private boolean isNotClosed() {
        return this.endEnrollmentDateTime.isAfter(LocalDateTime.now());
    }

    public boolean isAttended(UserAccount userAccount) {
        Account account = userAccount.getAccount();
        for (Enrollment e : this.enrollments) {
            if (e.getAccount().equals(account) && e.isAttended()) {
                return true;
            }
        }
        return false;
    }

    private boolean isAlreadyEnrolled(UserAccount userAccount) {
        Account account = userAccount.getAccount();
        for (Enrollment e : this.enrollments) {
            if (e.getAccount().equals(account)) {
                return true;
            }
        }
        return false;
    }
}
```
- 모임 조회 뷰 생성
```html
<!-- view.html -->
<!DOCTYPE html>
<html lang="en"
      xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
<head th:replace="fragments.html :: head"></head>
<body>
    <nav th:replace="fragments.html :: main-nav"></nav>
    <div th:replace="fragments.html :: study-banner"></div>
    <div class="container">
        <div class="row py-4 text-left justify-content-center bg-light">
            <div class="col-6">
                    <span class="h2">
                    <a href="#" class="text-decoration-none" th:href="@{'/study/' + ${study.path}}">
                        <span th:text="${study.title}">스터디 이름</span>
                    </a> / </span>
                <span class="h2" th:text="${event.title}"></span>
            </div>
            <div class="col-4 text-right justify-content-end">
                    <span sec:authorize="isAuthenticated()">
                        <button th:if="${event.isEnrollableFor(#authentication.principal)}"
                                class="btn btn-outline-primary" data-toggle="modal" data-target="#enroll">
                            <i class="fa fa-plus-circle"></i> 참가 신청
                        </button>
                        <button th:if="${event.isDisenrollableFor(#authentication.principal)}"
                                class="btn btn-outline-primary" data-toggle="modal" data-target="#disenroll">
                            <i class="fa fa-minus-circle"></i> 참가 신청 취소
                        </button>
                        <span class="text-success" th:if="${event.isAttended(#authentication.principal)}" disabled>
                            <i class="fa fa-check-circle"></i> 참석 완료
                        </span>
                    </span>
            </div>
            <div class="modal fade" id="disenroll" tabindex="-1" role="dialog" aria-labelledby="leaveTitle" aria-hidden="true">
                <div class="modal-dialog modal-dialog-centered" role="document">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h5 class="modal-title" id="leaveTitle" th:text="${event.title}"></h5>
                            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                <span aria-hidden="true">&times;</span>
                            </button>
                        </div>
                        <div class="modal-body">
                            <p>모임 참가 신청을 취소하시겠습니까?</p>
                            <p><strong>확인</strong>하시면 본 참가 신청을 취소하고 다른 대기자에게 참석 기회를 줍니다.</p>
                            <p>감사합니다.</p>
                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-secondary" data-dismiss="modal">닫기</button>
                            <form th:action="@{'/study/' + ${study.path} + '/events/' + ${event.id} + '/leave'}" method="post">
                                <button class="btn btn-primary" type="submit" aria-describedby="submitHelp">확인</button>
                            </form>
                        </div>
                    </div>
                </div>
            </div>
            <div class="modal fade" id="enroll" tabindex="-1" role="dialog" aria-labelledby="enrollmentTitle" aria-hidden="true">
                <div class="modal-dialog modal-dialog-centered" role="document">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h5 class="modal-title" id="enrollmentTitle" th:text="${event.title}"></h5>
                            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                <span aria-hidden="true">&times;</span>
                            </button>
                        </div>
                        <div class="modal-body">
                            <p>모임에 참석하시겠습니까? 일정을 캘린더에 등록해 두시면 좋습니다.</p>
                            <p><strong>확인</strong> 버튼을 클릭하면 모임 참가 신청을 합니다.</p>
                            <p>감사합니다.</p>
                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-secondary" data-dismiss="modal">닫기</button>
                            <form th:action="@{'/study/' + ${study.path} + '/events/' + ${event.id} + '/enroll'}" method="post">
                                <button class="btn btn-primary" type="submit" aria-describedby="submitHelp">확인</button>
                            </form>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="row px-3 justify-content-center">
            <div class="col-7 pt-3">
                <dt class="font-weight-light">상세 모임 설명</dt>
                <dd th:utext="${event.description}"></dd>

                <dt class="font-weight-light">모임 참가 신청 (<span th:text="${event.enrollments.size()}"></span>)</dt>
                <dd>
                    <table class="table table-borderless table-sm" th:if="${event.enrollments.size() > 0}">
                        <thead>
                        <tr>
                            <th scope="col">#</th>
                            <th scope="col">참석자</th>
                            <th scope="col">참가 신청 일시</th>
                            <th scope="col">참가 상태</th>
                            <th th:if="${study.isManager(#authentication.principal)}" scope="col">
                                참가 신청 관리
                            </th>
                            <th th:if="${study.isManager(#authentication.principal)}" scope="col">
                                출석 체크
                            </th>
                        </tr>
                        </thead>
                        <tbody>
                        <tr th:each="enroll: ${event.enrollments}">
                            <th scope="row" th:text="${enrollStat.count}"></th>
                            <td>
                                <a th:href="@{'/profile/' + ${enroll.account.nickname}}"
                                   class="text-decoration-none">
                                    <svg th:if="${#strings.isEmpty(enroll.account?.profileImage)}" data-jdenticon-value="nickname"
                                         th:data-jdenticon-value="${enroll.account.nickname}" width="24" height="24" class="rounded border bg-light"></svg>
                                    <img th:if="${!#strings.isEmpty(enroll.account?.profileImage)}"
                                         th:src="${enroll.account?.profileImage}" width="24" height="24" class="rounded border"/>
                                    <span th:text="${enroll.account.nickname}"></span>
                                </a>
                            </td>
                            <td>
                                <span class="date-time" th:text="${enroll.enrolledAt}"></span>
                            </td>
                            <td>
                                <span th:if="${enroll.accepted}">확정</span>
                                <span th:if="${!enroll.accepted}">대기중</span>
                            </td>
                            <td th:if="${study.isManager(#authentication.principal)}">
                                <a th:if="${event.isAcceptable(enroll)}" href="#" class="text-decoration-none"
                                   th:href="@{'/study/' + ${study.path} + '/events/' + ${event.id} + '/enrollments/' + ${enroll.id} + '/accept'}" >신청 수락</a>
                                <a th:if="${event.isRejectable(enroll)}" href="#" class="text-decoration-none"
                                   th:href="@{'/study/' + ${study.path} + '/events/' + ${event.id} + '/enrollments/' + ${enroll.id} + '/reject'}">취소</a>
                            </td>
                            <td th:if="${study.isManager(#authentication.principal)}">
                                <a th:if="${enroll.accepted && !enroll.attended}" href="#" class="text-decoration-none"
                                   th:href="@{'/study/' + ${study.path} + '/events/' + ${event.id} + '/enrollments/' + ${enroll.id} + '/checkin'}">체크인</a>
                                <a th:if="${enroll.accepted && enroll.attended}" href="#" class="text-decoration-none"
                                   th:href="@{'/study/' + ${study.path} + '/events/' + ${event.id} + '/enrollments/' + ${enroll.id} + '/cancel-checkin'}">체크인 취소</a>
                            </td>
                        </tr>
                        </tbody>
                    </table>
                </dd>
            </div>
            <dl class="col-3 pt-3 text-right">
                <dt class="font-weight-light">모집 방법</dt>
                <dd>
                    <span th:if="${event.eventType == T(com.studyolle.domain.EventType).FCFS}">선착순</span>
                    <span th:if="${event.eventType == T(com.studyolle.domain.EventType).CONFIRMATIVE}">관리자 확인</span>
                </dd>

                <dt class="font-weight-light">모집 인원</dt>
                <dd>
                    <span th:text="${event.limitOfEnrollments}"></span>명
                </dd>

                <dt class="font-weight-light">참가 신청 마감 일시</dt>
                <dd>
                    <span class="date" th:text="${event.endEnrollmentDateTime}"></span>
                    <span class="weekday" th:text="${event.endEnrollmentDateTime}"></span><br/>
                    <span class="time" th:text="${event.endEnrollmentDateTime}"></span>
                </dd>

                <dt class="font-weight-light">모임 일시</dt>
                <dd>
                    <span class="date" th:text="${event.startDateTime}"></span>
                    <span class="weekday" th:text="${event.startDateTime}"></span><br/>
                    <span class="time" th:text="${event.startDateTime}"></span> -
                    <span class="time" th:text="${event.endDateTime}"></span>
                </dd>

                <dt class="font-weight-light">모임장</dt>
                <dd>
                    <a th:href="@{'/profile/' + ${event.createdBy.nickname}}" class="text-decoration-none">
                        <svg th:if="${#strings.isEmpty(event.createdBy?.profileImage)}"
                             th:data-jdenticon-value="${event.createdBy.nickname}" width="24" height="24" class="rounded border bg-light"></svg>
                        <img th:if="${!#strings.isEmpty(event.createdBy?.profileImage)}"
                             th:src="${event.createdBy?.profileImage}" width="24" height="24" class="rounded border"/>
                        <span th:text="${event.createdBy.nickname}"></span>
                    </a>
                </dd>

                <dt th:if="${study.isManager(#authentication.principal)}" class="font-weight-light">모임 관리</dt>
                <dd th:if="${study.isManager(#authentication.principal)}">
                    <a class="btn btn-outline-primary btn-sm my-1"
                       th:href="@{'/study/' + ${study.path} + '/events/' + ${event.id} + '/edit'}" >
                        모임 수정
                    </a> <br/>
                    <button class="btn btn-outline-danger btn-sm" data-toggle="modal" data-target="#cancel">
                        모임 취소
                    </button>
                </dd>
            </dl>
            <div class="modal fade" id="cancel" tabindex="-1" role="dialog" aria-labelledby="cancelTitle" aria-hidden="true">
                <div class="modal-dialog modal-dialog-centered" role="document">
                    <div class="modal-content">
                        <div class="modal-header">
                            <h5 class="modal-title" id="cancelTitle" th:text="${event.title}"></h5>
                            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                <span aria-hidden="true">&times;</span>
                            </button>
                        </div>
                        <div class="modal-body">
                            <p>모임을 취소 하시겠습니까?</p>
                            <p><strong>확인</strong>하시면 본 모임 및 참가 신청 관련 데이터를 삭제합니다.</p>
                            <p>감사합니다.</p>
                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-secondary" data-dismiss="modal">닫기</button>
                            <form th:action="@{'/study/' + ${study.path} + '/events/' + ${event.id}}" th:method="delete">
                                <button class="btn btn-primary" type="submit" aria-describedby="submitHelp">확인</button>
                            </form>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div th:replace="fragments.html :: footer"></div>
    </div>
    <script src="/node_modules/moment/min/moment-with-locales.min.js"></script>
    <script type="application/javascript">
        $(function () {
            $('[data-toggle="tooltip"]').tooltip();

            moment.locale('ko');
            $(".date-time").text(function(index, dateTime) {
                return moment(dateTime, "YYYY-MM-DD`T`hh:mm").format('LLL');
            });
            $(".date").text(function(index, dateTime) {
                return moment(dateTime, "YYYY-MM-DD`T`hh:mm").format('LL');
            });
            $(".weekday").text(function(index, dateTime) {
                return moment(dateTime, "YYYY-MM-DD`T`hh:mm").format('dddd');
            });
            $(".time").text(function(index, dateTime) {
                return moment(dateTime, "YYYY-MM-DD`T`hh:mm").format('LT');
            });
        })
    </script>
</body>
</html>
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/meeting_3.jpg"></p>

<br>

## 모임 목록 조회
- EntityGraph를 사용하지 않으면 N + 1 문제가 발생한다.
    * 발생하는 쿼리
        - 스터디 조회 
        - 모임 목록 조회   
        - 모임 수 마다 enrollment 조회 (N)
    * 해결 후 발생하는 쿼리
        - 스터디 조회
        - 모임 조회
    * enrollment를 조회할 때는 account에 대한 자세한 정보를 가져오지 않고 필요한 정보(account_id)만 가져온다.
<br>

### 구현
- 목록 조회 맵핑
```java
@Controller
@RequestMapping("/study/{path}")
@RequiredArgsConstructor
public class EventController {

    ...
    private final StudyRepository studyRepository;

    @GetMapping("/events")
    public String viewStudyEvents(@CurrentAccount Account account, @PathVariable String path, Model model) {
        Study study = studyService.getStudy(path);
        model.addAttribute(account);
        model.addAttribute(study);

        List<Event> events = eventRepository.findByStudyOrderByStartDateTime(study);
        List<Event> newEvents = new ArrayList<>();
        List<Event> oldEvents = new ArrayList<>();
        events.forEach(e -> {
            if (e.getEndDateTime().isBefore(LocalDateTime.now())) {
                oldEvents.add(e);
            } else {
                newEvents.add(e);
            }
        });

        model.addAttribute("newEvents", newEvents);
        model.addAttribute("oldEvents", oldEvents);

        return "study/events";
    }
}
```
- EventRepository에 `findByStudyOrderByStartDateTime()` `@EntityGraph` 추가
```java
@Transactional(readOnly = true)
public interface EventRepository extends JpaRepository<Event, Long> {

    @EntityGraph(value = "Event.withEnrollments", type = EntityGraph.EntityGraphType.LOAD)
    List<Event> findByStudyOrderByStartDateTime(Study study);
}
```
- Event 엔티티에 `@NamedEntityGraph`, `numberOfRemainSpots()` 추가
```java
@NamedEntityGraph(
        name = "Event.withEnrollments",
        attributeNodes = @NamedAttributeNode("enrollments")
)
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
public class Event {

    ...

    public int numberOfRemainSpots() {
        return this.limitOfEnrollments - (int) this.enrollments.stream().filter(Enrollment::isAccepted).count();
    }
}
```
- 모임 목록 조회 뷰 생성
```html
<!-- framents.html -->
...

<div th:fragment="date-time">
    <script src="/node_modules/moment/min/moment-with-locales.min.js"></script>
    <script type="application/javascript">
        $(function () {
            moment.locale('ko');
            $(".date-time").text(function(index, dateTime) {
                return moment(dateTime, "YYYY-MM-DD`T`hh:mm").format('LLL');
            });
            $(".date").text(function(index, dateTime) {
                return moment(dateTime, "YYYY-MM-DD`T`hh:mm").format('LL');
            });
            $(".weekday").text(function(index, dateTime) {
                return moment(dateTime, "YYYY-MM-DD`T`hh:mm").format('dddd');
            });
            $(".time").text(function(index, dateTime) {
                return moment(dateTime, "YYYY-MM-DD`T`hh:mm").format('LT');
            });
            $(".calendar").text(function(index, dateTime) {
                return moment(dateTime, "YYYY-MM-DD`T`hh:mm").calendar();
            });
            $(".fromNow").text(function(index, dateTime) {
                return moment(dateTime, "YYYY-MM-DD`T`hh:mm").fromNow();
            });
            $(".date-weekday-time").text(function(index, dateTime) {
                return moment(dateTime, "YYYY-MM-DD`T`hh:mm").format('LLLL');
            });
        })
    </script>
</div>
```
```html
<!-- events.html -->
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body>
    <nav th:replace="fragments.html :: main-nav"></nav>
    <div th:replace="fragments.html :: study-banner"></div>
    <div class="container">
        <div th:replace="fragments.html :: study-info"></div>
        <div th:replace="fragments.html :: study-menu(studyMenu='events')"></div>
        <div class="row my-3 mx-3 justify-content-center">
            <div class="col-10 px-0 row">
                <div class="col-2 px-0">
                    <ul class="list-group">
                        <a href="#" class="list-group-item list-group-item-action d-flex justify-content-between align-items-center">
                            새 모임
                            <span th:text="${newEvents.size()}">2</span>
                        </a>
                        <a href="#" class="list-group-item list-group-item-action d-flex justify-content-between align-items-center">
                            지난 모임
                            <span th:text="${oldEvents.size()}">5</span>
                        </a>
                    </ul>
                </div>
                <div class="col-10 row row-cols-1 row-cols-md-2">
                    <div th:if="${newEvents.size() == 0}" class="col">
                        새 모임이 없습니다.
                    </div>
                    <div class="col mb-4 pr-0" th:each="event: ${newEvents}">
                        <div class="card">
                            <div class="card-header">
                                <span th:text="${event.title}">title</span>
                            </div>
                            <ul class="list-group list-group-flush">
                                <li class="list-group-item">
                                    <i class="fa fa-calendar"></i>
                                    <span class="calendar" th:text="${event.startDateTime}"></span> 모임 시작
                                </li>
                                <li class="list-group-item">
                                    <i class="fa fa-hourglass-end"></i> <span class="fromNow" th:text="${event.endEnrollmentDateTime}"></span> 모집 마감,
                                    <span th:if="${event.limitOfEnrollments != 0}">
                                        <span th:text="${event.limitOfEnrollments}"></span>명 모집 중
                                        (<span th:text="${event.numberOfRemainSpots()}"></span> 자리 남음)
                                    </span>
                                </li>
                                <li class="list-group-item">
                                    <a href="#" th:href="@{'/study/' + ${study.path} + '/events/' + ${event.id}}" class="card-link">자세히 보기</a>
                                </li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
            <div class="col-10 px-0 row">
                <div class="col-2"></div>
                <div class="col-10">
                    <table th:if="${oldEvents.size() > 0}" class="table table-hover">
                        <thead>
                        <tr>
                            <th scope="col">#</th>
                            <th scope="col">지난 모임 이름</th>
                            <th scope="col">모임 종료</th>
                            <th scope="col"></th>
                        </tr>
                        </thead>
                        <tbody th:each="event: ${oldEvents}">
                        <tr>
                            <th scope="row" th:text="${eventStat.count}">1</th>
                            <td th:text="${event.title}">Title</td>
                            <td>
                                <span class="date-weekday-time" th:text="${event.endDateTime}"></span>
                            </td>
                            <td>
                                <a href="#" th:href="@{'/study/' + ${study.path} + '/events/' + ${event.id}}" class="card-link">자세히 보기</a>
                            </td>
                        </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>
        <div th:replace="fragments.html :: footer"></div>
    </div>
    <script th:replace="fragments.html :: tooltip"></script>
    <script th:replace="fragments.html :: date-time"></script>
</body>
</html>
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/meeting_4.jpg"></p>

<br>

## 모임 수정
- 모집 방법은 수정할 수 없다.
- 모집 인원은 확정된 참가 신청 수 보다는 커야 한다. 예) 5명의 참가 신청을 확정 상태로 변경했다면, 모임을 수정할 때 모집 인원 수가 5보다는 커야 한다. 3으로 줄이면 안된다.
- 최대한 모임 개설하기 화면의 코드를 재사용한다.
- 모집 인원을 늘린 선착순 모임의 경우에, 자동으로 추가 인원의 참가 신청을 확정 상태로 변경해야 한다. (나중에 할 일)
<br>

### 구현 
- 모임 수정 관련 맵핑 추가
    * `eventForm.setEventType(event.getEventType())`
        - 뷰에는 모임 타입을 수정할 수 없지만 악의적으로 타입을 변경해서 서버에 전송할 경우를 막기 위해서
        - eventForm에 들어오는 모임 타입을 기존에 있던 모임 타입으로 덮어쓴다.
```java
@Controller
@RequestMapping("/study/{path}")
@RequiredArgsConstructor
public class EventController {

    ...

    @GetMapping("/events/{id}/edit")
    public String updateEventForm(@CurrentAccount Account account,
                                  @PathVariable String path, @PathVariable Long id, Model model) {
        Study study = studyService.getStudyToUpdate(account, path);
        Event event = eventRepository.findById(id).orElseThrow();
        model.addAttribute(study);
        model.addAttribute(account);
        model.addAttribute(event);
        model.addAttribute(modelMapper.map(event, EventForm.class));
        return "event/update-form";
    }

    @PostMapping("/events/{id}/edit")
    public String updateEventSubmit(@CurrentAccount Account account, @PathVariable String path,
                                    @PathVariable Long id, @Valid EventForm eventForm, Errors errors,
                                    Model model) {
        Study study = studyService.getStudyToUpdate(account, path);
        Event event = eventRepository.findById(id).orElseThrow();
        eventForm.setEventType(event.getEventType());
        eventValidator.validateUpdateForm(eventForm, event, errors);

        if (errors.hasErrors()) { // 에러가 있으면 폼을 다시 보여준다.
            model.addAttribute(account);
            model.addAttribute(study);
            model.addAttribute(event);
            return "event/update-form";
        }

        eventService.updateEvent(event, eventForm);
        return "redirect:/study/" + study.getEncodedPath() +  "/events/" + event.getId();
    }
}
```
- EventValidator에 `validateUpdateForm()` 메서드 추가
    * 이미 확정된 인원 보다 모집 인원 수가 큰지 검증
```java
@Component
public class EventValidator implements Validator {

    ...

    public void validateUpdateForm(EventForm eventForm, Event event, Errors errors) {
        if (eventForm.getLimitOfEnrollments() < event.getNumberOfAcceptedEnrollments()) {
            errors.rejectValue("limitOfEnrollments", "wrong.value", "확인된 참기 신청보다 모집 인원 수가 커야 합니다.");
        }
    }
}
```
- Event 엔티티에 `getNumberOfAcceptedEnrollments()` 메서드 추가
    * 참가 확정 인원을 반환한다.
```java
...
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
public class Event {

    ...

    public long getNumberOfAcceptedEnrollments() {
        return this.enrollments.stream().filter(Enrollment::isAccepted).count();
    }
}
```
- EventService에 `updateEvent()` 메서드 추가
    * 모임 업데이트는 eventForm에 있는 데이터를 event에 넣어주면 된다.
    * 모임 타입이 선착순인 경우, 대기 인원이 있다면 자동으로 참가 확정을 해줘야 한다.
```java
@Service
@Transactional
@RequiredArgsConstructor
public class EventService {

    ...

    public void updateEvent(Event event, EventForm eventForm) {
        modelMapper.map(eventForm, event);
        // TODO 모집 인원을 늘린 선착순 모임의 경우에, 자동으로 추가 인원의 참가 신청을 확정 상태로 변경해야 한다.
    }
}
```
- 모임 수정 뷰 생성 및 중복 코드 제거
```html
<!-- fragments.html -->
<div th:fragment="event-form (mode, action)">
    <div class="py-5 text-center">
        <h2>
            <a th:href="@{'/study/' + ${study.path}}"><span th:text="${study.title}">스터디</span></a> /
            <span th:if="${mode == 'edit'}" th:text="${event.title}"></span>
            <span th:if="${mode == 'new'}">새 모임 만들기</span>
        </h2>
    </div>
    <div class="row justify-content-center">
        <form class="needs-validation col-sm-10"
              th:action="@{${action}}"
              th:object="${eventForm}" method="post" novalidate>
            <div class="form-group">
                <label for="title">모임 이름</label>
                <input id="title" type="text" th:field="*{title}" class="form-control"
                       placeholder="모임 이름" aria-describedby="titleHelp" required>
                <small id="titleHelp" class="form-text text-muted">
                    모임 이름을 50자 이내로 입력하세요.
                </small>
                <small class="invalid-feedback">모임 이름을 입력하세요.</small>
                <small class="form-text text-danger" th:if="${#fields.hasErrors('title')}" th:errors="*{title}">Error</small>
            </div>
            <div class="form-group" th:if="${mode == 'new'}">
                <label for="eventType">모집 방법</label>
                <select th:field="*{eventType}"  class="custom-select mr-sm-2" id="eventType" aria-describedby="eventTypeHelp">
                    <option th:value="FCFS">선착순</option>
                    <option th:value="CONFIRMATIVE">관리자 확인</option>
                </select>
                <small id="eventTypeHelp" class="form-text text-muted">
                    두가지 모집 방법이 있습니다.<br/>
                    <strong>선착순</strong>으로 모집하는 경우, 모집 인원 이내의 접수는 자동으로 확정되며, 제한 인원을 넘는 신청은 대기 신청이 되며 이후에 확정된 신청 중에 취소가 발생하면 선착순으로 대기 신청자를 확정 신청자도 변경합니다. 단, 등록 마감일 이후에는 취소해도 확정 여부가 바뀌지 않습니다.<br/>
                    <strong>확인</strong>으로 모집하는 경우, 모임 및 스터디 관리자가 모임 신청 목록을 조회하고 직접 확정 여부를 정할 수 있습니다. 등록 마감일 이후에는 변경할 수 없습니다.
                </small>
            </div>
            <div class="form-row">
                <div class="form-group col-md-3">
                    <label for="limitOfEnrollments">모집 인원</label>
                    <input id="limitOfEnrollments" type="number" th:field="*{limitOfEnrollments}" class="form-control" placeholder="0"
                           aria-describedby="limitOfEnrollmentsHelp">
                    <small id="limitOfEnrollmentsHelp" class="form-text text-muted">
                        최대 수용 가능한 모임 참석 인원을 설정하세요. 최소 2인 이상 모임이어야 합니다.
                    </small>
                    <small class="invalid-feedback">모임 신청 마감 일시를 입력하세요.</small>
                    <small class="form-text text-danger" th:if="${#fields.hasErrors('limitOfEnrollments')}" th:errors="*{limitOfEnrollments}">Error</small>
                </div>
                <div class="form-group col-md-3">
                    <label for="endEnrollmentDateTime">등록 마감 일시</label>
                    <input id="endEnrollmentDateTime" type="datetime-local" th:field="*{endEnrollmentDateTime}" class="form-control"
                           aria-describedby="endEnrollmentDateTimeHelp" required>
                    <small id="endEnrollmentDateTimeHelp" class="form-text text-muted">
                        등록 마감 이전에만 스터디 모임 참가 신청을 할 수 있습니다.
                    </small>
                    <small class="invalid-feedback">모임 신청 마감 일시를 입력하세요.</small>
                    <small class="form-text text-danger" th:if="${#fields.hasErrors('endEnrollmentDateTime')}" th:errors="*{endEnrollmentDateTime}">Error</small>
                </div>
                <div class="form-group col-md-3">
                    <label for="startDateTime">모임 시작 일시</label>
                    <input id="startDateTime" type="datetime-local" th:field="*{startDateTime}" class="form-control"
                           aria-describedby="startDateTimeHelp" required>
                    <small id="startDateTimeHelp" class="form-text text-muted">
                        모임 시작 일시를 입력하세요. 상세한 모임 일정은 본문에 적어주세요.
                    </small>
                    <small class="invalid-feedback">모임 시작 일시를 입력하세요.</small>
                    <small class="form-text text-danger" th:if="${#fields.hasErrors('startDateTime')}" th:errors="*{startDateTime}">Error</small>
                </div>
                <div class="form-group col-md-3">
                    <label for="startDateTime">모임 종료 일시</label>
                    <input id="endDateTime" type="datetime-local" th:field="*{endDateTime}" class="form-control"
                           aria-describedby="endDateTimeHelp" required>
                    <small id="endDateTimeHelp" class="form-text text-muted">
                        모임 종료 일시가 지나면 모임은 자동으로 종료 상태로 바뀝니다.
                    </small>
                    <small class="invalid-feedback">모임 종료 일시를 입력하세요.</small>
                    <small class="form-text text-danger" th:if="${#fields.hasErrors('endDateTime')}" th:errors="*{endDateTime}">Error</small>
                </div>
            </div>
            <div class="form-group">
                <label for="description">모임 설명</label>
                <textarea id="description" type="textarea" th:field="*{description}" class="editor form-control"
                          placeholder="모임을 자세히 설명해 주세요." aria-describedby="descriptionHelp" required></textarea>
                <small id="descriptionHelp" class="form-text text-muted">
                    모임에서 다루는 주제, 장소, 진행 방식 등을 자세히 적어 주세요.
                </small>
                <small class="invalid-feedback">모임 설명을 입력하세요.</small>
                <small class="form-text text-danger" th:if="${#fields.hasErrors('description')}" th:errors="*{description}">Error</small>
            </div>
            <div class="form-group">
                <button class="btn btn-primary btn-block" type="submit"
                        aria-describedby="submitHelp" th:text="${mode == 'edit' ? '모임 수정' : '모임 만들기'}">모임 수정</button>
            </div>
        </form>
    </div>
</div>
```
```html
<!-- update-form.html -->
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments.html :: head"></head>
<body class="bg-light">
    <nav th:replace="fragments.html :: main-nav"></nav>
    <div th:replace="fragments.html :: study-banner"></div>
    <div class="container">
        <div th:replace="fragments.html :: event-form (mode='edit', action='/study/' + ${study.path}
            + '/events/' + ${event.id} + '/edit')"></div>
        <div th:replace="fragments.html :: footer"></div>
    </div>
    <script th:replace="fragments.html :: form-validation"></script>
    <script th:replace="fragments.html :: editor-script"></script>
</body>
</html>
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/meeting_5.jpg"></p>

<br>

## 모임 취소
- HTML의 FROM은 method로 GET과 POST만 지원한다.
    * DELEET는 지원하지 않는다.
- 기존의 일관성을 지키려면 Post로 받아야 하지만 DELETE로 받고 싶다면 스프링 부트에게 hiddenmethod를 쓰겠다고 알려줘야 한다.
    * HTML `<FORM>`에 DELETE를 주면 _method라는 hidden field에 넣어주는 기능을 thymeleaf가 제공한다.
    * 사실상 `<form>`의 METHOD는 POST다.
<br>

### 구현
- DELETE METHOD 맵핑
```properties
# HTML <FORM>에서 th:method에서 PUT 또는 DELETE를 사용해서 보내는 _method를 사용해서 @PutMapping과 @DeleteMapping으로 요청을 맵핑.
spring.mvc.hiddenmethod.filter.enabled=true
```
- EventController에 모임 취소 관련 맵핑 추가
```java
@Controller
@RequestMapping("/study/{path}")
@RequiredArgsConstructor
public class EventController {

    ...

    @DeleteMapping("/events/{id}")
    public String cancelEvent(@CurrentAccount Account account, @PathVariable String path, @PathVariable Long id) {
        Study study = studyService.getStudyToUpdateStatus(account, path);
        eventService.deleteEvent(eventRepository.findById(id).orElseThrow());
        return "redirect:/study/" + study.getEncodedPath() + "/events";
    }
}
```
- EventService에 `deleteEvent()` 메서드 추가
```java
@Service
@Transactional
@RequiredArgsConstructor
public class EventService {

    ...

    public void deleteEvent(Event event) {
        eventRepository.delete(event);
    }
}
```
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/meeting_6.jpg"></p>

<br>

## 모임 참가 신청 및 취소
- 모임 참가 신청 및 취소 시 스터디 조회
    *  스터디 관리자가 아니어도 참가 신청이 가능해야 하므로 조회하는 스터디는 관리자 권한 없이 읽어올 수 있어야 하며 데이터를 필요한 만큼만 가져오도록 주의해야 한다.
- 모임 참가 신청
    * 선착순 모임이고 현재까지 수락한 참가 신청 개수와 총 모집 인원수를 확인하고, 가능하다면 해당 참가 신청을 확정 상태로 저장한다.
- 모임 참가 신청 취소
    * 선착순 모임이라면, 대기 중인 모임 참가 신청 중에 가장 빨리 신청한 것을 확정 상태로 변경한다.
- 모임 수정 로직 보완
    * 선착순 모임 수정시 모집 인원이 늘었고 대기 중인 참가 신청이 있다면 가능한 만큼 대기 중인 신청을 확정 상태로 변경한다.
<br>

### 구현
- EventController에 참가 신청/취소 관련 맵핑 추가
    * 참가 신청은 관리자가 아니어도 할 수 있으므로 관리자 권한이 아니어도 스터디 정보를 가져올 수 있어야 한다.
```java
@Controller
@RequestMapping("/study/{path}")
@RequiredArgsConstructor
public class EventController {

    ...

    @PostMapping("/events/{id}/enroll")
    public String newEnrollment(@CurrentAccount Account account,
                                @PathVariable String path, @PathVariable Long id) {
        Study study = studyService.getStudyToEnroll(path);
        eventService.newEnrollment(eventRepository.findById(id).orElseThrow(), account);
        return "redirect:/study/" + study.getEncodedPath() +  "/events/" + id;
    }

    @PostMapping("/events/{id}/disenroll")
    public String cancelEnrollment(@CurrentAccount Account account,
                                   @PathVariable String path, @PathVariable Long id) {
        Study study = studyService.getStudyToEnroll(path);
        eventService.cancelEnrollment(eventRepository.findById(id).orElseThrow(), account);
        return "redirect:/study/" + study.getEncodedPath() +  "/events/" + id;
    }
}
```
- StudyRepository에 경로를 통해 스터디 정보만을 가져올 수 있도록 메서드 추가
```java
@Transactional(readOnly = true)
public interface StudyRepository extends JpaRepository<Study, Long> {

    ...

    Study findStudyOnlyByPath(String path);
}
```
- StudyService에 `getStudyToEnroll()` 메서드 추가
    * 경로를 통해 스터디 정보를 받아와서 있는지 없는지 체크하고 스터디 반환
```java
@Service
@Transactional
@RequiredArgsConstructor
public class StudyService {

    ...

    public Study getStudyToEnroll(String path) {
        Study study = repository.findStudyOnlyByPath(path);
        checkIfExistingStudy(path, study);
        return study;
    }
}
```
- EnrollmentRepository 생성
```java
public interface EnrollmentRepository extends JpaRepository<Enrollment, Long> {
    boolean existsByEventAndAccount(Event event, Account account);

    Enrollment findByEventAndAccount(Event event, Account account);
}
```
- Event 엔티티에 참가 신청/취소 관련 메서드 추가
    * `addEnrollment()`
        - 양방향 관계를 맺어주고 있는 것.
        - event에 새로 만든 enrollment 추가해주고 enrollment에 event를 this로 세팅해준다.
    * `isAbleToAcceptWaitingEnrollment()`
        - 선착순이면서 제한인원을 넘지 않았으면 참가 신청이 바로 확정날 수 있도록 true로 변경해준다.
    * `removeEnrollment()`
        - enrollment 관계를 끊어준다.
    * `acceptNextWaitingEnrollment()`
        - 모임이 추가적으로 인원을 받을 수 있으면 가장 첫 번째 대기 enrollment를 가져와서 확정(true)시켜준다.
    * `acceptWaitingList()`
        - 모집 인원이 늘어난 숫자만큼 자동으로 참가 확정해준다.
```java
...
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
public class Event {

    ...

    public void addEnrollment(Enrollment enrollment) {
        this.enrollments.add(enrollment);
        enrollment.setEvent(this);
    }

    public void removeEnrollment(Enrollment enrollment) {
        this.enrollments.remove(enrollment);
        enrollment.setEvent(null);
    }

    public boolean isAbleToAcceptWaitingEnrollment() {
        return this.eventType == EventType.FCFS && this.limitOfEnrollments > this.getNumberOfAcceptedEnrollments();
    }

    public boolean canAccept(Enrollment enrollment) {
        return this.eventType == EventType.CONFIRMATIVE
                && this.enrollments.contains(enrollment)
                && !enrollment.isAttended()
                && !enrollment.isAccepted();
    }

    public boolean canReject(Enrollment enrollment) {
        return this.eventType == EventType.CONFIRMATIVE
                && this.enrollments.contains(enrollment)
                && !enrollment.isAttended()
                && enrollment.isAccepted();
    }

    private List<Enrollment> getWaitingList() {
        return this.enrollments.stream().filter(enrollment -> !enrollment.isAccepted()).collect(Collectors.toList());
    }

    public void acceptWaitingList() {
        if (this.isAbleToAcceptWaitingEnrollment()) {
            var waitingList = getWaitingList();
            int numberToAccept = (int) Math.min(this.limitOfEnrollments - this.getNumberOfAcceptedEnrollments(), waitingList.size());
            waitingList.subList(0, numberToAccept).forEach(e -> e.setAccepted(true));
        }
    }

    public void acceptNextWaitingEnrollment() {
        if (this.isAbleToAcceptWaitingEnrollment()) {
            Enrollment enrollmentToAccept = this.getTheFirstWaitingEnrollment();
            if (enrollmentToAccept != null) {
                enrollmentToAccept.setAccepted(true);
            }
        }
    }

    private Enrollment getTheFirstWaitingEnrollment() {
        for (Enrollment e : this.enrollments) {
            if (!e.isAccepted()) {
                return e;
            }
        }

        return null;
    }
}
```
- EventService에 참가 신청/ 취소 메서드 추가
    * `newEnrollment()`
        - 혹시나 enrollment가 있으면 무시하고 없으면 참가 신청 객체(Enrollment)를 새로 만든다.
    * `cancelEnrollment()`
        - enrollment 관계를 끊어주고 삭제
```java
@Service
@Transactional
@RequiredArgsConstructor
public class EventService {

    ...

    public void updateEvent(Event event, EventForm eventForm) {
        modelMapper.map(eventForm, event);
        event.acceptWaitingList(); // 모임 수정 때 acceptWaitingList()를 호출해서 참가 확정시켜준다.
    }

    ...

    public void newEnrollment(Event event, Account account) {
        if (!enrollmentRepository.existsByEventAndAccount(event, account)) {
            Enrollment enrollment = new Enrollment();
            enrollment.setEnrolledAt(LocalDateTime.now());
            enrollment.setAccepted(event.isAbleToAcceptWaitingEnrollment());
            enrollment.setAccount(account);
            event.addEnrollment(enrollment);
            enrollmentRepository.save(enrollment);
        }
    }

    public void cancelEnrollment(Event event, Account account) {
        Enrollment enrollment = enrollmentRepository.findByEventAndAccount(event, account);
        event.removeEnrollment(enrollment);
        enrollmentRepository.delete(enrollment);
        event.acceptNextWaitingEnrollment();
    }
}
```
<br>

### 테스트 코드 작성
```java
class EventControllerTest extends StudyControllerTest {

    @Autowired EventService eventService;
    @Autowired EnrollmentRepository enrollmentRepository;

    @Test
    @DisplayName("선착순 모임에 참가 신청 - 자동 수락")
    @WithAccount("hayoung")
    void newEnrollment_to_FCFS_event_accepted() throws Exception {
        Account kimhayoung = createAccount("kimhayoung");
        Study study = createStudy("test-study", kimhayoung);
        Event event = createEvent("test-event", EventType.FCFS, 2, study, kimhayoung);

        mockMvc.perform(post("/study/" + study.getPath() + "/events/" + event.getId() + "/enroll")
                .with(csrf()))
                .andExpect(status().is3xxRedirection())
                .andExpect(redirectedUrl("/study/" + study.getPath() + "/events/" + event.getId()));

        Account hayoung = accountRepository.findByNickname("hayoung");
        isAccepted(hayoung, event);
    }

    @Test
    @DisplayName("선착순 모임에 참가 신청 - 대기중 (이미 인원이 꽉차서)")
    @WithAccount("hayoung")
    void newEnrollment_to_FCFS_event_not_accepted() throws Exception {
        Account kimhayoung = createAccount("kimhayoung");
        Study study = createStudy("test-study", kimhayoung);
        Event event = createEvent("test-event", EventType.FCFS, 2, study, kimhayoung);

        Account may = createAccount("may");
        Account june = createAccount("june");
        eventService.newEnrollment(event, may);
        eventService.newEnrollment(event, june);

        mockMvc.perform(post("/study/" + study.getPath() + "/events/" + event.getId() + "/enroll")
                .with(csrf()))
                .andExpect(status().is3xxRedirection())
                .andExpect(redirectedUrl("/study/" + study.getPath() + "/events/" + event.getId()));

        Account hayoung = accountRepository.findByNickname("hayoung");
        isNotAccepted(hayoung, event);
    }

    @Test
    @DisplayName("참가신청 확정자가 선착순 모임에 참가 신청을 취소하는 경우, 바로 다음 대기자를 자동으로 신청 확인한다.")
    @WithAccount("hayoung")
    void accepted_account_cancelEnrollment_to_FCFS_event_not_accepted() throws Exception {
        Account hayoung = accountRepository.findByNickname("hayoung");
        Account kimhayoung = createAccount("kimhayoung");
        Account may = createAccount("may");
        Study study = createStudy("test-study", kimhayoung);
        Event event = createEvent("test-event", EventType.FCFS, 2, study, kimhayoung);

        eventService.newEnrollment(event, may);
        eventService.newEnrollment(event, hayoung);
        eventService.newEnrollment(event, kimhayoung);

        isAccepted(may, event);
        isAccepted(hayoung, event);
        isNotAccepted(kimhayoung, event);

        mockMvc.perform(post("/study/" + study.getPath() + "/events/" + event.getId() + "/disenroll")
                .with(csrf()))
                .andExpect(status().is3xxRedirection())
                .andExpect(redirectedUrl("/study/" + study.getPath() + "/events/" + event.getId()));

        isAccepted(may, event);
        isAccepted(kimhayoung, event);
        assertNull(enrollmentRepository.findByEventAndAccount(event, hayoung));
    }

    @Test
    @DisplayName("참가신청 비확정자가 선착순 모임에 참가 신청을 취소하는 경우, 기존 확정자를 그대로 유지하고 새로운 확정자는 없다.")
    @WithAccount("hayoung")
    void not_accepterd_account_cancelEnrollment_to_FCFS_event_not_accepted() throws Exception {
        Account hayoung = accountRepository.findByNickname("hayoung");
        Account kimhayoung = createAccount("kimhayoung");
        Account may = createAccount("may");
        Study study = createStudy("test-study", kimhayoung);
        Event event = createEvent("test-event", EventType.FCFS, 2, study, kimhayoung);

        eventService.newEnrollment(event, may);
        eventService.newEnrollment(event, kimhayoung);
        eventService.newEnrollment(event, hayoung);

        isAccepted(may, event);
        isAccepted(kimhayoung, event);
        isNotAccepted(hayoung, event);

        mockMvc.perform(post("/study/" + study.getPath() + "/events/" + event.getId() + "/disenroll")
                .with(csrf()))
                .andExpect(status().is3xxRedirection())
                .andExpect(redirectedUrl("/study/" + study.getPath() + "/events/" + event.getId()));

        isAccepted(may, event);
        isAccepted(kimhayoung, event);
        assertNull(enrollmentRepository.findByEventAndAccount(event, hayoung));
    }

    private void isNotAccepted(Account kimhayoung, Event event) {
        assertFalse(enrollmentRepository.findByEventAndAccount(event, kimhayoung).isAccepted());
    }

    private void isAccepted(Account account, Event event) {
        assertTrue(enrollmentRepository.findByEventAndAccount(event, account).isAccepted());
    }

    @Test
    @DisplayName("관리자 확인 모임에 참가 신청 - 대기중")
    @WithAccount("hayoung")
    void newEnrollment_to_CONFIMATIVE_event_not_accepted() throws Exception {
        Account kimhayoung = createAccount("kimhayoung");
        Study study = createStudy("test-study", kimhayoung);
        Event event = createEvent("test-event", EventType.CONFIRMATIVE, 2, study, kimhayoung);

        mockMvc.perform(post("/study/" + study.getPath() + "/events/" + event.getId() + "/enroll")
                .with(csrf()))
                .andExpect(status().is3xxRedirection())
                .andExpect(redirectedUrl("/study/" + study.getPath() + "/events/" + event.getId()));

        Account hayoung = accountRepository.findByNickname("hayoung");
        isNotAccepted(hayoung, event);
    }

    private Event createEvent(String eventTitle, EventType eventType, int limit, Study study, Account account) {
        Event event = new Event();
        event.setEventType(eventType);
        event.setLimitOfEnrollments(limit);
        event.setTitle(eventTitle);
        event.setCreatedDateTime(LocalDateTime.now());
        event.setEndEnrollmentDateTime(LocalDateTime.now().plusDays(1));
        event.setStartDateTime(LocalDateTime.now().plusDays(1).plusHours(5));
        event.setEndDateTime(LocalDateTime.now().plusDays(1).plusHours(7));
        return eventService.createEvent(event, study, account);
    }
}
```
<br>

## 모임 참가 신청 수락 취소 및 출석 체크
- 정석은 POST로 하는 것이지만 프론트 쪽의 코드를 간결하게 하고 줄맞춤을 맞추기 위해 GET맵핑을 사용한다.
<br>

### 구현
- EventController에 모임 참가 신청 수락 취소 및 출석 체크 관련 맵핑 추가
    * 맵핑을 추가할 때 다음 코드와 같이 eventId와 enrollmentId를 가지고 해당하는 enrollment의 정보를 바꿔줘도 된다.
```java
@GetMapping("events/{eventId}/enrollments/{enrollmentId}/accept")
public String acceptEnrollment(@CurrentAccount Account account, @PathVariable String path,
                               @PathVariable Long eventId, @PathVariable Long enrollmentId) {
    Study study = studyService.getStudyToUpdate(account, path);
    Event event = eventRepository.findById(evendId).orElseThrow();
    Enrollment enrollment = enrollmentRepository.findById(enrollmentId).orElseThrow();
    eventService.acceptEnrollment(event, enrollment);
    return "redirect:/study/" + study.getEncodedPath() + "/events/" + eventId();
}
```
- 그러나 DB에 저장되어 있는 id값으로 가져오는 경우 Spring Data JPA가 제공하는 Entity Converter를 사용해서 코드를 줄일 수 있다.
    * `@PathVariable("eventId") Event event`와 같이 eventId에 해당하는 event를 바로 바인딩 받을 수 있다.
    * 참고) 다른 id를 가져오는 부분도 코드 수정을 한다.(refactoring)
```java
@Controller
@RequestMapping("/study/{path}")
@RequiredArgsConstructor
public class EventController {

    ...

    @GetMapping("events/{eventId}/enrollments/{enrollmentId}/accept")
    public String acceptEnrollment(@CurrentAccount Account account, @PathVariable String path,
                                   @PathVariable("eventId") Event event, @PathVariable("enrollmentId") Enrollment enrollment) {
        Study study = studyService.getStudyToUpdate(account, path);
        eventService.acceptEnrollment(event, enrollment);
        return "redirect:/study/" + study.getEncodedPath() + "/events/" + event.getId();
    }

    @GetMapping("/events/{eventId}/enrollments/{enrollmentId}/reject")
    public String rejectEnrollment(@CurrentAccount Account account, @PathVariable String path,
                                   @PathVariable("eventId") Event event, @PathVariable("enrollmentId") Enrollment enrollment) {
        Study study = studyService.getStudyToUpdate(account, path);
        eventService.rejectEnrollment(event, enrollment);
        return "redirect:/study/" + study.getEncodedPath() + "/events/" + event.getId();
    }

    @GetMapping("/events/{eventId}/enrollments/{enrollmentId}/checkin")
    public String checkInEnrollment(@CurrentAccount Account account, @PathVariable String path,
                                    @PathVariable("eventId") Event event, @PathVariable("enrollmentId") Enrollment enrollment) {
        Study study = studyService.getStudyToUpdate(account, path);
        eventService.checkInEnrollment(enrollment);
        return "redirect:/study/" + study.getEncodedPath() + "/events/" + event.getId();
    }

    @GetMapping("/events/{eventId}/enrollments/{enrollmentId}/cancel-checkin")
    public String cancelCheckInEnrollment(@CurrentAccount Account account, @PathVariable String path,
                                          @PathVariable("eventId") Event event, @PathVariable("enrollmentId") Enrollment enrollment) {
        Study study = studyService.getStudyToUpdate(account, path);
        eventService.cancelCheckInEnrollment(enrollment);
        return "redirect:/study/" + study.getEncodedPath() + "/events/" + event.getId();
    }
}
```
- EventService에 메서드 추가
```java
@Service
@Transactional
@RequiredArgsConstructor
public class EventService {

    ...

    public void acceptEnrollment(Event event, Enrollment enrollment) {
        event.accept(enrollment);
    }

    public void rejectEnrollment(Event event, Enrollment enrollment) {
        event.reject(enrollment);
    }

    public void checkInEnrollment(Enrollment enrollment) {
        enrollment.setAttended(true);
    }

    public void cancelCheckInEnrollment(Enrollment enrollment) {
        enrollment.setAttended(false);
    }
}
```
- Event에 메서드 추가
```java
...
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
public class Event {

    ...

    public void accept(Enrollment enrollment) {
        if (this.eventType == EventType.CONFIRMATIVE
                && this.limitOfEnrollments > this.getNumberOfAcceptedEnrollments()) {
            enrollment.setAccepted(true);
        }
    }

    public void reject(Enrollment enrollment) {
        if (this.eventType == EventType.CONFIRMATIVE) {
            enrollment.setAccepted(false);
        }
    }
}
```