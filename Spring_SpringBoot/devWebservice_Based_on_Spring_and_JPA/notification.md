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