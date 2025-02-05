### 멀티 스테이지 빌드

- 파일 내부에 스테이지라는 빌드 단위를 정의
- 스테이지끼리 서로 결과를 복사할 수 있음
- 최적화된 파일을 생성하는 스테이지, 이를 사용하는 스테이지 등을 정의
- Dockerfile과 유사, CMD → RUN 이후 추가 동작
    - FROM문으로 새로운 스테이지 정의
    - as문으로 스테이지 명명
    - COPY —from으로 사용

```yaml
FROM node:14-alphine as build

...

RUN npm run build
# 베이스 이미지 교체
FROM nginx:stable-alphine
# 이전 스테이지의 결과 사용
# 소스 경로 -> 복사 경로
COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### 프론트엔드 standalone 배포

- 컨테이너 생성 → 이미지, 포트, 환경변수 등
- Startup dependency ordering → 백엔드 컨테이너 시작 후 프론트엔드 컨테이너를 시작하도록 설정할 수 있음
- 백엔드와 프론트엔드가 포트 80을 중복해서 사용하고 있음 → 요청을 구분할 수 없음, ECS 태스크 중복
    1. 백엔드와 프론트엔드 서버 병합
    2. 백엔드 포트 번호 변경
    3. 새로운 태스크 생성
        1. 동일 task role로 프론트엔드용 태스크 생성
        2. 환경 변수 사용 + 배포 url 추가 설정
            1. 백엔드 load balancer에서 설정한 url 사용
        
        ```yaml
        proces.env.NODE_ENV === "development"? "localhost:80": "load_balancer_url"
        ```
        
        c. 프론트엔드를 위한 load balancer 생성 및 사용
        
        - 이전 로드 밸런서와 동일 보안 그룹을 사용(동일 포트 트래픽)
        
        d. 태스크 기반 서비스 생성
        

### 멀티-스테이지 빌드 Target

- Dockerfile이 있는 경우 전체 빌드 또는 하나의 이미지만 빌드할 수 있음
    - —target

```yaml
# 위의 Dockerfile에서 build 스테이지만 빌드하기
docker build --target build ...
```

### 이미지 & 컨테이너

- 컨테이너
    - 앱 코드 + 실행 환경 패키지
    - 독립적
    - Single-task focused
    - 공유 및 재생산 가능한 패키지
    - 무상태(stateless)
        - 데이터는 컨테이너 종료 시 삭제
        - 볼륨으로 미러링해서 방지
- 이미지
    - 컨테이너를 실행하는 blueprint
    - Dockerhub 등에서 pull 또는 자체 빌드
    - 공유 가능, 기반으로 다중 컨테이너 생산
    - read only
        - 컨테이너에서 작성한 데이터는 이미지에 영향 X
        - 변경되지 않음
    - 호출 시 layer(=컨테이너) 생성

### 주요 명령

- docker build
    - Dockerfile로 이미지 빌드
    - 폴더 + 경로 지정
    - 이미지 이름, 태그 지정
    - category + 하위 버전(node:alpine-14) 등
- docker run
    - 이미지를 컨테이너로 실행
    - —rm으로 자동 삭제
    - -d detached
    - 기반 이미지 이름, 태그를 지정해야 함
- docker push/pull
    - Dockerhub 이미지 push/pull
    - Dockerhub가 아닌 경우 전체 url로 명시

### 데이터, 볼륨, 네트워킹

- 컨테이너는 isolated & stateless
    - 작성한 데이터는 공유되지 않음 & 저장되지 않음
    - isolated - 자체 데이터와 File System
        - BInd mount로 호스트에 연결
        - -v /local/path:/container/path
    - stateless - 데이터를 내부에 저장하므로 삭제되면 같이 삭제
        - Volume으로 데이터 유지
        - -v name:/container/path
            - name is optional
- 컨테이너 간 통신
    - 컨테이너 IP 사용하기
    - Docker network
        - 컨테이너명으로 url 연결

### Docker Compose

- docker build, run 등 명령어가 복잡해짐, 옵션이 매우 많음
- Docker Compose로 컨테이너 시작 구성을 미리 설정
    - docker-compose up시 정의된 모든 컨테이너 시작
    - docker-compose down으로 동시 정지
    - 다중 컨테이너 시 유용

### 배포

- 바인드 마운트 사용 금지, 대신 일반 볼륨 등
- 다중 컨테이너의 경우 다중 호스트 환경이 될 수 있음
    - docker만으로 처리할 수 없음
- multi-stage build
    - 빌드 단계가 필요할 경우(리액트 등)
    - 이미지 빌드 중 중간 빌드 결과물을 활용
- Control vs Ease-of-use tradeoff
    - AWS에서 자체 서버 + Docker, Container 실행
        - 제어권이 많으나 보안, 모니터링 등을 수동
    - 관리형 서비스
        - 자체 서버는 필요 없으나 책임, 지식 등 줄음
