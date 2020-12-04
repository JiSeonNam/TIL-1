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