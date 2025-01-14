### case 1 : WWW 통신 컨테이너

- 컨테이너 내부에서 외부 네트워크로 요청을 보내기
    - 컨테이너 안의 임의의 앱 가정
    - 앱이 외부 도메인과 API 통신을 한다고 가정
    - 컨테이너 내부 → 외부 통신

### case 2 : 컨테이너에서 로컬 호스트 통신

- 로컬 호스트 머신의 자체 DB와 연결, 데이터를 가져온다고 가정
    - 컨테이너 내부 → 호스트 머신

### case 3 : 컨테이너 간 통신

- SQL DB를 관리하는 다른 컨테이너가 있다고 가정
    - DB 또는 다른 서비스를 실행하는 다른 컨테이너와 연결
    - 컨테이너 → 컨테이너
    - 앱을 다중 컨테이너로 구축하는 일은 일반적

### 웹 통신하기

- Node 앱의 MongoDB 연결에서 에러 발생
    - 호스트 머신의 MongoDB와 연결 실패
- 웹 API(WWW) 연결에는 설정 필요 없음

### 호스트 통신하기

- 호스트 요청을 위해 [localhost](http://localhost) 도메인 → host.docker.internal

```jsx
mongodb://localhost:8000/ -> mongodb://host.docker.internal:8000/
```

### 컨테이너 간 통신

- mongodb를 호스트 머신 대신 별도 컨테이너에서 실행 → 일반적
- host.docker.internal은 작동하지 않음 → 당연함

```jsx
docker container inspect mongodb
```

→ 컨테이너 속성 나열

- IPAddress → 해당 컨테이너에 연결하는 IP주소
    - 해당 주소로 도메인 대체
    - 컨테이너마다 IP 주소를 확인해야 하므로 다소 번거로움

### Docker Networks: 우아한 컨테이너 간 통신

- 다중 컨테이너 간 통신을 허용하기
    - 여러 컨테이너를 Network로 묶기
    - docker run —network <name> ..
    - IP 조회 및 해결 자동화

```docker
docker run -d --name mongodb(컨테이너명) --network favorites-network(네트워크이름) mongo(Mongo이미지)
```

→ 볼륨이 존재하지 않음 에러

- 네트워크 생성 시 볼륨을 수동으로 생성해야 함

```docker
docker network create favorites-network
```

- 이후 컨테이너 실행 시 --network 명령어로 해당 네트워크에 추가
- 네트워크간 컨테이너 연결
    - url에 해당 컨테이너명 직접 입력
    
    ```docker
    mongodb://mongodb(컨테이너명=mongodb):8000/
    ```
    

### Docker가 IP 주소를 해결하는 법

- 요청이 컨테이너 → 호스트/컨테이너 이동 시 목표 주소를 명시하는 이름으로 대체
    - 도커는 소스 코드를 내부에서 교체하지 않음
    - 컨테이너 연결 과정은 복잡하며 요청 주소가 특정 명령어일 경우 인식해 자동 resolve
