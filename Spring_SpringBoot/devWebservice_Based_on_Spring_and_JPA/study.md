# 스터디

### 주요 기능
- 스터디 만들기
- 스터디 공개 및 종료
- 스터디 인원 모집
- 스터디 설정
    * 배너 이미지
    * 스터디 주제 (Tag)
    * 활동 지역 (Zone)
    * 스터디 관리 (공개 / 경로 변경 / 이름 변경 / 스터디 삭제)
- 스터디 참여 / 떠나기
<br>

## 스터디 도메인
- 객체 관점에서 Study와 다른 엔티티의 관계
    * Study에서 Account 쪽으로 @ManyToMany 단방향 관계 두 개 (managers, members)
    * Study에서 Zone으로 @ManyToMany 단방향 관계
    * Study에서 Tag로 @ManyToMany 단방향 관계
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/study_1.jpg"></p>

```java
@Entity
@Getter @Setter @EqualsAndHashCode(of = "id")
@Builder @AllArgsConstructor @NoArgsConstructor
public class Study {

    @Id @GeneratedValue
    private Long id;

    @ManyToMany
    private Set<Account> managers = new HashSet<>(); // 스터디를 관리하는 스터디장, 매니저

    @ManyToMany
    private Set<Account> members = new HashSet<>(); // 스터디원

    @Column(unique = true)
    private String path; // URL 경로

    private String title; // 제목

    private String shortDescription; // 짧은 설명

    @Lob @Basic(fetch = FetchType.EAGER)
    private String fullDescription; // 본문

    @Lob @Basic(fetch = FetchType.EAGER)
    private String image; // 배너 이미지

    @ManyToMany
    private Set<Tag> tags = new HashSet<>(); // 관심 주제(태그)

    @ManyToMany
    private Set<Zone> zones = new HashSet<>(); // 지역 정보

    private LocalDateTime publishedDateTime; // 스터디 공개 시간

    private LocalDateTime closedDateTime; // 스터디 종료 시간

    private LocalDateTime recruitingUpdatedDateTime; // 인원 모집 시간

    private boolean recruiting; // 인원 모집 중인지 여부

    private boolean published; // 공개 여부

    private boolean closed; // 종료 여부

    private boolean useBanner; // 배너 사용 여부
}
```
<br>
