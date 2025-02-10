### 수동 배포의 잠재적 문제점

- 쿠버네티스
    - kubernetes.io
    - 컨테이너화된 애플리케이션의 배포, scaling, management 오픈 소스
- 수동 배포의 경우
    - 자체 EC2 인스턴스 생성, 도커 설치, 컨테이너 실행
    - EC2 인스턴스를 자체 관리, 구성, 내부 SW, OS 최신화를 해야 함
    - 컨테이너 충돌, 다운 시 새 컨테이너로 교체해야 함
        - 충돌 발생을 직접 모니터링 + 수동 재시작으로 해결해야 함
    - 트래픽 증가 시 컨테이너 수동 추가
        - 트래픽에 따라 추가 컨테이너에 요청 처리 및 트래픽 감소 시 제거
        - 모든 컨테이너가 처리하는 트래픽이 균등해야 함

### 왜 Kubernetes인지

- AWS ECS로 해결하기
    - 컨테이너 충돌 시 자동으로 재시작 + health check
    - Autoscaling → 컨테이너 인스턴스 수 자동 조정
    - Load Balander → 도메인을 제공하고 트래픽을 컨테이너 인스턴스에 분배
- ECS의 단점
    - 클라우드 서비스에 한정됨
    - AWS에서 정의한 대로 구성해야 함
        - 다른 프로바이더로 옮기려면 세부 정보, 구성 파일 등을 다시 학습하고 수동 변환
        - 따라서 Docker + alpha 학습 수요

→ Kubernetes가 해결

### Kubernetes 정의

- 배포, 스케일링, 모니터, 재시작, 컨테이너 교체 등을 정의
    - 클라우드 서비스와 독립적
    - 컨테이너 관리 오픈 소스의 표준이므로 가능함
- 특정 도구로 해당 구성을 클라우드 프로바이더 or Remote Machine으로 전달
    - 클라우드마다 설정에 특화 코드를 추가, 교체 시 같이 교체
- 따라서 표준화된 배포 가능
- 혼동하지 말 것
    - Service provider가 아님, 배포에 도움을 주는 오픈소스 프로젝트
    - Service provider가 제공하는 서비스가 아님
    - Docker의 대안이 아님 → Docker와 같이 작동하며 배포를 도움
- Docker-compose for multiple machines

### Kubernetes Architecture & Concept

- 컨테이너 = Pod
    - 가장 작은 단위, 구성 파일에서 정의
    - Pod는 컨테이너를 보유, 여러 개일 수 있음
    - Pod는 Worker Node로 자동 분포됨
- Worker Node
    - 머신, 가상 인스턴스과 같음
    - Pod(컨테이너)를 실행
- Proxy
    - Worker Node에 포함
    - Worker Node의 네트워크 트래픽 제어
    - 인터넷 연결 여부, 연결 방법 제어
    - 외부 트래픽이 컨테이너에 도달할 수 있도록 Proxy 구성
- Master Node
    - Control Plane
        - Pod를 Worker Node에 분포
        - Pod 종료 조건 등 설정
        - Worker Node, Pod와 상호작용
    - 큰 배포의 경우 여러 머신에 분할되는 마스터 노드를 가짐
        - 워커 노드는 마스터 노드와 독립됨
        - 워커 노드가 다운되어도 마스터 노드는 정상
- Cluster
    - Master Node, Worker Node로 구성
    - 클라우드 프로바이더 API에 연결
        - 클라우드에 노드 상태를 복제

### Kubernetes가 하는 것 & 해야 할 것

- 해야 하는 것(설정)
    - 클러스터 생성 시 Worker & Master Node 만들고 설정하기
    - 모든 노드에 Kubernetes 설치
    - 클라우드에 따라 필요한 부가 리소스 설정(Load Balancer, File System)
- Kubernetes가 하는 것
    - 생성된 Node(Pod)의 관리
    - Monitor & Restart
    - 설정된 클라우드 리소스 활용하기

### Worker Node

- 워커 노드에서 작동하는 것은 마스터 노드에서 관리
- Pod
    - 하나 이상의 컨테이너 & 종속 리소스 호스트(볼륨 등)
    - 일반적으로 둘 이상의 Pod를 가짐
        - 복사본 → 트래픽 분산
        - 또는 다른 태스크를 수행
- 기본적으로 하나의 local machine으로 생각할 수 있음
- Docker를 설치해야 함
- kubelet → 마스터 - 워커 통신 SW
- kube-proxy → 트래픽 처리, 제어

### Master Node

- API Server → 워커 노드의 kublet과 통신
- Scheduler → Pod 관찰, 생성, 워커 노드에 분배
- Kube-Controller-Mananger
    - 포드의 수가 적절한지
    - 워커 노드 감독
- Cloud-Controller-Manager
    - Kube와 동일하나 클라우드에 따라 다름
    - Kube의 특정 버전
- Kubernetes 설정으로 마스터, 워커 노드를 직접 생성할 필요가 없을 수 있음

### 중요 용어 & 개념

- Cluster
    - Node machine, master & worker node 포함된 집합
- Node
    - Physical or virtual machine
    - Pod 포함 & Cluster와 통신
    - Master Node
        - Cluster Control Plane
        - Worker Node의 Pod 관리
    - Worker Node
        - Pod Hosting
        - run containers
    - Pods
        - app container + resource 포함
    - Container
        - 도커 컨테이너 임
    - Services
        - Logical Sets
        - 독립된 IP를 가진 포드 집합
