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
