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
