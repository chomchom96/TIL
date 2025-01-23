### Intro

- 왜 Compose를 사용하는지
    - 현재는 모든 컨테이너마다 빌드와 실행을 해주고 있음
    - 필요시 모든 컨테이너를 하나의 configuration으로 실행하는 orchestration commands
    - 하나의 명령으로 모두 실행 또는 중지
    - Configure 파일은 공유 가능
    - 전체 다중 컨테이너 어플리케이션을 편하게 관리 가능
- 유의사항
    - Compose는 Dockerfile을 대체하지 않음
    - 이미지, 컨테이너를 대체하지 않음
    - 다중 호스트에서 관리하기 어려움
- Docker Compose 파일
    - Service
        - 다중 컨테이너를 구성하는 컨테이너
        - 포트, 환경 변수, 할당 volume, network 등 설정

### Compose 파일 만들기

```yaml
# docker-compose.yaml
# docker-compose version
version: "3.8"
# services : 중첩 value, indent로 종속성 표시
# 소속 컨테이너
services:
	mongodb:
	backend:
	fronend:
```

- [https://docs.docker.com/compose/compose-file](https://docs.docker.com/compose/compose-files)

### Compose configuration

- 기본 실행 설정이 detached이므로 별도 설정 x
- key-value가 있는 여러 값의 경우 - 생략 가능

```yaml
# docker-compose.yaml
services:
	mongodb:
		# 사용하는 이미지
		image: 'mongo'
		volumes:
			# 추가하는 모든 볼륨에 -
			# -v syntax와 동일
			# 다른 서비스에 동일한 볼륨을 사용시 자동 공유
			- data:/data/db
		environment:
			MONGO_USERNAME: name
			MONGO_PASSWORD: pw
		# 또는 env file 경로지정
		env_file:
			- ./env/mongo.env
		# networks는 일반적으로 설정하지 않음 -> compose에서 모든 서비스를 자동으로 동일 네트워크 매핑
	backend:
	fronend:
```

### Docker Compose Up & Down

- docker-compose up → 컴포즈 파일의 모든 서비스 빌드 & 실행
    - 생성한 볼륨명엔 prefix로 docker-compose가 붙음
- docker-compose down → 서비스 종료
    - -v 키워드로 볼륨도 같이 삭제

### 다음 컨테이너 작업

```yaml
	backend:
		# 이미지가 없는 경우 build
		# image: 'goals-node'
		build:
			# 복사할 폴더를 포함하는 폴더 경로
			# Dockerfile이 별도 폴더에 있는 경우 -> 둘다 포함된 더 높은 경로
			context: ./backend
			# 파일명이 Dockerfile이 아닌 경우(dev), 맞으면 Dockerfile
			dockerfile: Dockerfile-dev
				# args를 사용하면 지정
				# args: 
					# some-arg: 1
			ports:
				- '80:80'
			volumes:
				# 일반 볼륨
				- logs:/app/logs
				# 바인드 마운트 -> docker-compose부터 상대경로
				- ./backend/:app
				# 익명 볼륨
				- /app/node_modules
			# docker-compose는 여러 서비스를 생성 -> 여러 컨테이너 생성
			# 컨테이너의 의존 관계가 있는 경우 설정
			# backend -> db에 의존하고 있음 -> db 컨테이너 우선 실행
			depends_on:
				- mongodb
```

### 빌드 & 컨테이너 이름 이해하기

- up 명령에 —build로 이미지 리빌드를 강제함
    - 일반적인 경우 존재하는 이미지를 재사용
- docker compose build → 컨테이너 실행하지 않고, compose 파일의 이미지 빌드
- 컨테이너 이름은 프로젝트 폴더명_서비스명_숫자(1에서 시작)
    - 수동 명명은 container_name으로 지정
