### 소개

- 공식 용어는 아님
- 항상 어플리케이션 컨테이너에서 작업
    - 앱 + 실행 환경
    - 빌드 후 실행 커맨드
- 유틸리티 컨테이너
    - 특정 환경만 포함함
    - Node 환경 등
    - 이미지 뒤에 부가 커맨드로 커스텀
- 예시로 → Node 앱을 만들때
    - npm init으로 설정이 들어간 노드 앱을 수동으로 만들 수 있으나 로컬 머신에 npm이 설치가 되어 있어야함
    - 컨테이너 하나에 npm 설치 후 설정을 완료하면 모든 컨테이너에 npm을 설치할 필요가 없음

### 컨테이너 커맨드

- 컨테이너를 인터액티브 모드로 실행
    - 노드 입력을 받기 위해
- docker container prune → 중지된 컨테이너 제거
- docker run -it -d node → 입력을 입력할 수 없으나 컨테이너는 입력을 기다리므로 계속 실행됨, 연결은 안됨
- docker exec → 실행중인 컨테이너 내에서 기본 명령이 아닌 특정 명령을 실행할 수 있음
    - docker exec <name> npm init
- docker run -it node npm init
    - node 이미지의 디폴트 명령인 npm init을 오버라이딩

### 유틸리티 컨테이너 구축

```docker
# node-util 컨테이너
FROM node:14-alpine
# 슬림, 최적화 노드 버전
WORKDIR /app
```

- docker run -it -v [경로]:/app node-util  npm init
    - npm init을 호스트 머신의 특정 경로에 실행
    - 유틸리티 컨테이너로 지정 호스트 머신 경로에 프로젝트를 생성할 수 있음

### ENTRYPOINT 활용

```docker
FROM node:14-alpine
# 슬림, 최적화 노드 버전
WORKDIR /app

ENTRYPOINT ["npm"]
```

- ENTRYPOINT와 CMD의 차이점
    - 명령에서 docker run 이미지 뒤에 커맨드를 실행 시 →
        - CMD로 지정한 실행은 명령의 커맨드가 오버라이드, 실행되지 않음
        - ENTRYPOINT로 지정한 명령은 기본 명령으로 명령의 커맨드는 ENTRYPOINT 명령 뒤에 붙어 실행됨
    - init만 입력해도 npm init 실행

### Docker Compose

- 매번 바인드 마운트로 긴 커맨드를 작성하기 번거로움 → Docker Compose
- yaml 파일에 서비스 정의
    - 서비스명 npm

```yaml
# docker-compose.yaml
version: "3.8"
services:
	npm:
		build: ./ # 컨테이너에서 Dockerfile이 yaml 파일과 같은 위치에 있음
		stdin_open: true
		tty: true # 입력 받기
		volumes: 
			- ./:/app # 바인드 마운트 위치
	
```

- up으로 서비스 시작, down으로 서비스 종료
    - up init → 에러 발생
        - docker-compose.yaml 파일에 정의된 커맨드만 실행됨
    - down 시 컨테이너 자동 삭제
- 명령 실행 위해 exec
- run → yaml 파일에 여러 서비스가 있는 경우 이름으로 단일 서비스를 지정
    - docker-compose run npm init → 정상 작동
    - 명령이 완료되면 서비스 종료
        - 자동으로 삭제되지는 않음 → —rm 옵션
