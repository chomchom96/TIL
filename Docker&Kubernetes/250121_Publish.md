### 개발 → 배포

- 컨테이너는 애플리케이션 코드 + 실행 환경 포함된 독립 패키지
    - 도커는 도커가 설치된 모든 기기에서 앱을 실행 가능
- 배포 환경의 컨테이너도 100% 똑같은 환경의 앱이 실행
- 돌발상황을 직면하지 않도록 확인
    - 로컬에서 발생하는 상황은 배포 환경에서도 발생
- 유의사항
    - 배포 환경에선 바인드 마운트를 사용하지 않음
    - 배포를 위해 빌드 단계가 바뀔 수 있음
    - multi container 프로젝트의 경우 여러 호스트로 나눠야 할 수 있음
    - control - responsibility tradeoff
        - 리모트 머신에 대한 책임이 적을 때 발생하는 메리트가 있음

### 배포 프로세스 & Provider

- 여러 앱을 가정한 시나리오
- 시작으로 단순한 단일 앱
    - 단일 이미지 & 컨테이너
    - 리모트 서버 설정 + SSH 연결
    - 로컬 호스팅 머신 → 도커 레지스트리로 이미지 푸시 → 리모트 호스트에서 이미지 풀 → 리모트 호스트에서 실행 → 포트 노출, 배포
- 리모트 서버(머신)이 필요
    - 여러 docker host provider 서비스
    - AWS, Azure, Google Cloud 등
    - 실습은 ECS 프리 티어로

### AWS EC2

- 아마존 클라우드 호스팅 서비스
1. EC2 Instance 생성 & 실행
    1. VPC → Virtual Public Cloud, security group
2. Security group 설정으로 웹 노출 포트 설정
3. SSH를 통해 인스턴스와 연결
4. 인스턴스에 도커 설치 & 컨테이너 실행

### 배포 환경에서 바인드 마운트

- 개발 환경에서는 코드를 캡슐화할 필요는 없음
    - 코드가 컨테이너 외부에서 제공
    - 바인드 마운팅으로 개발 중에 코드를 추가 또는 변경이 유기적
        - 컨테이너 재시작 없이 반영
        - 개발 워크플로우 효율성 증가
        
        ```
        # 개발용 Dockerfile
        FROM node:14
        WORKDIR /app
        CMD ["npm", "start"]
        
        ```
        
        ```bash
        # 실행 명령어
        docker run -v $(pwd):/app myapp
        
        ```
        
- 배포 환경에서는 코드 전체가 외부 웹에 노출
    - 컨테이너는 standalone, 리모트 머신에 소스 코드가 노출되면 안됨
    - 이미지 / 컨테이너는 single source of truth → 컨테이너 하나로 앱에 필요한 모든 것을 갖추고 있음
        - 따라서 호스트 머신의 컨테이너 주변에는 아무 것도 없어야 함
    - 바인드 마운트 대신 복사본(COPY)를 사용
    - 배포용 Dockerfile 예시
        
        ```
        FROM node:14
        WORKDIR /app
        COPY package*.json ./
        RUN npm install
        COPY . .
        CMD ["npm", "start"]
        ```
        
- 배포 환경에서 바인드 마운트를 피해야 하는 이유
    1. 보안 취약성
        - 호스트 시스템 파일 접근 권한 노출
        - 악의적 파일 조작 위험
    2. 의존성 및 확장성 문제
        - 호스트 시스템 구조 의존
        - 다중 서버 배포 시 일관성 유지 어려움
        - 컨테이너 오케스트레이션 복잡도 증가
    3. 운영 안정성
        - 환경 간 이식성 저하
        - 버전 관리 및 롤백 어려움
        - 컨테이너 독립성 훼손

### EC2 인스턴스 연결

1. AMI 선택
    1. Amazon Machine Image
    2. 가상 머신의 OS등 선택
2. Instance Type
    1. 본강에선는 Free tier로
3. Configure Instance Details
    1. 인스턴스 설정
    2. VPC가 생성되었는지, 디폴트 설정이 되었는지 → 없을 경우 생성
4. Review and Launch
    1. 설정 확인 후 실행
    2. Launch 화면에서 Key pair 생성 → SSH 연결에 필요한 파일
        1. 이름은 아무렇게 → 다운로드(단 한 번만 가능, 잃어버리면 새 인스턴스 생성)
5. SSH 설치
    1. Mac, Linux는 기본 설치, CLI에서 실행
    2. Windows → WSL 2 설정(Window 10) = 가상 리눅스 장치
6. Connect to your Instance
    1. 인스턴스 화면에서 연결 클릭
    2. 새 창의 커맨드를 CLI에서 실행
        1. 키 파일의 권한 확인
        2. 배포할 폴더의 위치에서 실행중인지 확인
        3. 

### 가상 머신에 Docker 설치하기

- 인스턴스 연결 후 설치

```yaml
sudo yum update -ysudo yum -y
install docker
 
sudo service docker start
 
sudo usermod -a -G docker ec2-user
```

- 로그아웃 후 로그인

```yaml
sudo systemctl enable docker

# docker 버전 확인
docker version
```

### 로컬 이미지를 클라우드에 푸시

- 로컬 이미지 → 클라우드 → 리모트 머신
1. 소스코드 배포
    1. 폴더의 모든 항목을 리모트 머신에 복사
    2. 리모트 머신에서 docuer build 후 run
2. 로컬 머신에서 이미지 빌드 후 이미지 배포
    1. 바로 docker run
- 빌드 전 dockerignore 파일 추가하기
    - node_modules, Dockerfile, pem(키 파일) 등
- docker push <이미지 태그/이름>
    - dockerhub 로그인 후 가능

### 앱 실행 & publish

- 원격에서도 실행 커맨드는 동일
    - 권한 에러 시 (permission denied)
        - sudo로 관리자 실행
        - 이외엔 다음 강의에
- docker run <이름> → 자동 pull 후 실행
- docker ps로 이미지 pull 및 실행 확인
- EC2 인스턴스의 public ipv4/v6 주소
    - 해당 주소로 접근할 수 없음 → 권한 부여 x
- Security Group
    - EC2 대시보드 좌측 하단 드래그
    - 접근 권한 제어
    - 보안 그룹을 인스턴스에 연결해서 사용
        - 인바운드 & 아웃바운드 규칙 설정
        - 인바운드 - 외부에서 접근 가능한 주소 정의
        - 아웃바운드 - 인스턴스 내부에서 접근 가능한 주소 정의
        - 0.0.0.0/0 → 제약없음
    -
