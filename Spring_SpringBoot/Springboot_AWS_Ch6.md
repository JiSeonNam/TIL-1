## 6. AWS 서버 환경을 만들어보자 - AWS EC2
- 리전 서울로 변경
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_1.jpg"></p>
<br>

### 6-1. EC2 인스턴스 생성하기
- 인스턴스란 EC2 서비스에 생성된 가상머신이다.
- 가장 먼저 ec2를 검색해 인스턴스 시작을 누른 후 AMI를 선택한다.
    * AMI(Amazon Machine Image)는 EC2 인스턴스를 시작하는 데 필요한 정보를 이미지로 만들어 둔 것이다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_2.jpg"></p>

<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_3.jpg"></p>

- 인스턴스 유형에서는 프리티어로 표기된 t2.micro를 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_4.jpg"></p>

- 세부정보 구성
    * 별다른 설정을 하지 않고 넘어간다.
- 스토리지 추가
    * 서버의 용량을 얼마나 정할지 선택하는 단계
    * 기본값은 8GB지만 프리티어로 30GB까지 가능하다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_5.jpg"></p>

- 태그 추가
    * 웹 콘솔에서 표기될 태그인 Name 태그를 등록한다. 
    * 태그는 해당 인스턴스를 표현하는 여러 이름으로 사용될 수 있다.(EC2의 이름을 정한다고 생각하면 쉽다)
    * 서비스의 인스턴스를 나타낼 수 있는 값으로 등록한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_6.jpg"></p>

- 보안 그룹
    * 방화벽을 이야기한다.
    * 기존에 생성된 보안 그룹이 없으므로 유의미한 이름으로 변경한다.
    * pem 키 관리를 잘해야하고 지정된 IP에서만 ssh 접속이 가능하도록 구성하는 것이 안전하다.
    * 현재 접속한 장소의 IP를 등록하고 카페 또는 다른 장소에서 접속할 때는 해당 장소의 IP를 다시 SSH규칙에 추가하는 것이 안전하다.
    * 현재 프로젝트의 기본 포트인 8080도 추가하고 검토 및 시작 버튼을 누른다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_7.jpg"></p>

- 인스턴스 시작 검토
    * 보안 그룹 경고를 하는데 이는 8080이 전체 오픈 되어서 발생한다.
    * 8080을 열어 놓는 것은 위험한 일이 아니므로 시작하기 버튼 클릭
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_8.jpg"></p>

- 할당할 pem 키 선택
    * 인스턴스로 접근하기 위해서는 pem 키(비밀키)가 필요하다.
    * 인스턴스는 지정된 pem 키와 매칭되는 공개키를 가지고 있어, 해당 pem 키 외에는 접근을 허용하지 않는다.
    * 일종의 마스터 키로 절대 유출되서는 안되며 EC2 서버로 접속할 때 필수 파일이기 떄문에 디렉토리로 저장한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_9.jpg"></p>

- pem키를 다운받으면 인스턴스 id를 눌러 EC2 목록으로 이동
- 인스턴스 생성이 완료되면 다음과 같이 IP와 도메인이 할당된다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_10.jpg"></p>

- 고정 IP 할당
    * AWS의 고정 IP를 EIP(Elastic IP, 탄력적IP)라고 한다.
    * 카테고리에서 탄력적 IP를 선택하고 새 주소 할당을 한다.
    * IPv4 주소 풀은 Amazon 풀
- 생성된 탄력적 IP와 EC2 주소를 연결
    * 탄력적 Ip를 생성하고 EC2 서버에 연결하지 않으면 비용이 발생한다.(반드시 연결해야한다)
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_11.jpg"></p>

- EC2 이름과 IP를 선택하고 연결한 뒤 인스턴스 목록 페이지로 이동하면 다음과 같이 연결된 모습을 볼 수 있다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_12.jpg"></p>
<br>

### 6-2. EC2 서버에 접속하기(window 기준)
- putty 설치
    * putty.exe
    * puttygen.exe
- puttygen.exe 파일을 실행
    * putty는 pem키로 사용이 안되고 pem키를 ppk파일로 변환해야 한다.
    * puttygen이 그 과정을 진행해주는 클라이언트이다.
- puttygen 화면에서 다운로드 받은 pem키를 선택한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_13.jpg"></p>

- 자동으로 변환이 진행되며 Save private key 버튼을 눌러 ppk파일을 생성한다.
    * 경고 창이 뜨면 예를 누르고 넘어간다.
- ppk 파일이 저장될 위치와 ppk 이름을 등록
- ppk 파일이 생성되었으면 putty.exe를 실행하여 다음과 같이 각 항목을 등록한다.
    * HostName : username@public_Ip를 등록한다.
        - ec2-user@탄력적IP주소
    * Port : ssh 접속 포트인 22를 등록한다.
    * Connection type : SSH 선택
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_14.jpg"></p> 

- 왼쪽 사이드바에 Auth를 클릭해서 ppk 파일을 로드할 수 있는 화면으로 이동
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_15.jpg"></p> 

- 방금 생성한 ppk파일을 불러오고 Session 탭으로 이동해서 Saved Sessions에 현재 설정들을 저장할 이름을 등록하고 Save버튼을 클릭한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_16.jpg"></p>

- 저장한 뒤 Load 버튼을 클릭하면 SSH 접속 알림이 뜨고 예를 클릭하면 다음과 같이 SSH 접속이 성공한다.
<p align="center"><img src = "https://github.com/qlalzl9/TIL/blob/master/Spring_SpringBoot/img/Springboot_AWS_Ch6_17.jpg"></p>
<br>

### 6-3. 아마존 리눅스 1 서버 생성 시 꼭 해야 할 설정들
- 프로젝트에 맞는 자바 설치
    * (블로그에 정리 해놓음)
    * [Putty에 jdk 11 설치하기](https://qlalzl9.tistory.com/344)
- 타임존 변경
    * EC2 서버의 기본 타임존은 UTC(세계 표준 시간)으로 한국의 시간과 9시간 차이난다.
    * 서버에서 수행되는 Java 애플리케이션에서 생성되는 시간도 모두 9시간 차이가 나기 때문에 반드시 수정해야 한다.

```
sudo rm /etc/localtime
sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
```
- Hostname 변경
    * 여러 서버를 관리 중일 경우 IP만으로 어떤 서비스의 서버인지 확인이 어렵기 때문에 변경한다.
    * 명령어를 실행 후 HOSTNAME 부분을 원하는 서비스명으로 변경하면 된다.
```
sudo vim /etc/sysconfig/network
```
- 호스트 주소를 찾을 때 가장 먼저 검색해 보는 /etc/hosts에 변경한 hostname을 등록
    * `127.0.0.1` 부분에 등록한 HOSTNAME을 등록한다.
    * 등록 후 `curl 등록한 호스트 이름`으로 확인 가능하다.
    * 잘 등록헀다면 80포트로 접근이 안된다는 에러가 발생한다.
        - 80포트로 실행된 서비스가 없음을 의미하고 curl 호스트 이름으로 실행은 잘 되었음을 의미한다.
```
sudo vim /etc/hosts
```