# Container Resource Optimizer

성능 목표(SLI)를 만족하는 가장 비용 효율적인 컨테이너 리소스를 자동으로 찾아주는 최적화 시스템

## 프로젝트 개요

> 개발 기간: 2025.03 ~ 2025.07

### 팀 멤버

- 김두현: ([iamdudumon](https://github.com/iamdudumon))
- 조재영: ([jojaeyoung](https://github.com/joejaeyoung))

### 🛠️ 기술 스택

- Backend: Node.js, Express.js, TypeScript, PostgreSQL
- Optimizer: Node.js, TypeScript
- Infra-Runner: Node.js, TypeScript, Docker, Docker Compose
  - Load Testing: k6
  - Monitoring: Prometheus, cAdvisor, Grafana
- Communication: gRPC, REST API

</br>

### 🎯 1. 문제 제기 (Motivation)

클라우드 네이티브 환경에서 애플리케이션을 컨테이너로 배포할 때, 얼마만큼의 CPU와 메모리를 할당해야 할까요?

>- 너무 적게 할당하면? ➡️ OOM(Out of Memory) 등으로 서비스가 중단되어 안정성에 문제가 생깁니다.
>- 너무 많이 할당하면? ➡️ 사용하지 않는 자원에도 비용이 발생하여 심각한 비용 낭비로 이어집니다.

실제로 여러 보고서에 따르면 클라우드 지출의 약 27%가 불필요하게 낭비되고 있으며, 이는 많은 기업이 리소스 관리의 어려움을 겪고 있음을 보여줍니다.  개발자의 감이나 경험에 의존하는 수동적인 리소스 할당 방식은 더 이상 효율적이지 않습니다.

이 프로젝트는 이러한 비용과 성능 사이의 딜레마를 자동화된 방식으로 해결하기 위해 시작되었습니다.

### 📌 2. 솔루션 요약

이 시스템은 **자동화된 부하 테스트와 효율적인 탐색 알고리즘**을 결합하여 비용과 성능의 딜레마를 해결합니다.

1. 성능 목표(SLI) 기반 검증: 먼저 "요청의 95%는 200ms 안에 처리되어야 한다"와 같이 명확하게 측정 가능한 성능 목표(SLI)를 설정합니다. 
2. 부하 테스트 자동화: k6와 Docker를 활용해, 특정 CPU와 메모리가 할당된 컨테이너에 가상의 부하를 발생시켜 위에서 설정한 SLI를 만족하는지 자동으로 검증합니다. 
3. 이진 탐색을 통한 효율적 탐색: 가능한 모든 리소스 조합을 무식하게 테스트하는 대신, 이진 탐색(Binary Search) 알고리즘을 도입했습니다. 
4. 비용 함수(Cost Function) 기반 최종 선택: SLI를 만족하는 여러 조합 중에서, 실제 클라우드 요금 정책을 반영한 최적 리소스 조합을 최종적으로 추천합니다. 

### ✨ 3. 핵심 기능 (Features)

- 🤖 이진 탐색 기반 자동 리소스 탐색: 모든 조합을 테스트하는 대신, 이진 탐색 알고리즘을 통해 SLI를 만족하는 최소 리소스 조합을 효율적으로 탐색합니다. 
- 💰 비용-성능 최적화: 실제 클라우드 요금 정책을 반영한 비용 함수(Cost Function)를 통해, 단순히 성능을 만족하는 것을 넘어 가장 비용이 저렴한 최적의 조합을 추천합니다. 
- 🚀 End-to-End 테스트 자동화: 사용자가 테스트 설정을 입력하면, 원격지에 컨테이너 환경을 동적으로 구성하고, 부하 테스트를 실행하며, 결과를 분석하는 전 과정을 자동화합니다.

### 🌟 4. 주요 기능 및 기대효과

- 비용 절감: SLA를 만족하는 최소한의 리소스를 찾아 클라우드 비용을 직접적으로 절감합니다.
- 생산성 향상: 리소스 최적화 과정을 완전 자동화하여 개발자의 수작업을 없애고 의사결정을 돕습니다.
- 안정성 확보: 부하 테스트 기반의 검증을 통해 성능 병목 현상을 사전에 예방하고, 일관된 사용자 경험을 제공합니다.

</br>

## 🏗️ 프로젝트 아키텍쳐

> 현재 버전은 총 3개의 레포지토리로 구성
> 
> 각 서비스가 gRPC와 HTTP 통신을 통해 유기적으로 동작

### 1. Optimizer

- 전체 최적화 프로세스를 지휘하는 역할입니다.
- 이진 탐색 알고리즘을 통해 다음 테스트할 리소스 조합을 결정하고 `Backend`에 테스트를 요청
- [Github](https://github.com/ContainerResourceOptimzer/BinarySearch-Optimzer)

### 2. Backend

- `Optimizer`와 `Infra-Runner`를 중개하는 중앙 API 서버입니다.
- 모든 실험 과정과 결과를 PostgreSQL 데이터베이스에 기록하고, gRPC를 통해 `Infra-Runner`에게 실제 테스트 실행을 명령합니다.
- [Github](https://github.com/ContainerResourceOptimzer/Node-Backend)

### 3. Infra-Runner

- 원격 서버에서 실제 부하 테스트 인프라를 생성하고 관리하는 행동대장입니다.
- `docker-compose`를 사용하여 리소스가 제한된 테스트 앱과 `k6` 부하 테스트 컨테이너를 실행하고, `Prometheus`에서 테스트 결과를 수집하여 `Backend`로 보고합니다.
- [Github](https://github.com/ContainerResourceOptimzer/Infra-Runner)

<img width="4398" height="3292" alt="image" src="https://github.com/user-attachments/assets/c04eedbf-e2dc-4dbb-9d7b-ded27af37b4f" />

### 시스템 동작 방식

#### Sequence Diagram

```진행 중 ...```

#### 실험 시나리오 가정
“test-api-server 이미지를 대상으로, p(95) 응답 시간 3000ms 이내, 총 요청 777건 처리라는 SLI 목표를 만족하는 최적 리소스를 찾자”

1. 초기화
<img width="2136" height="602" alt="image" src="https://github.com/user-attachments/assets/56d03293-94df-4feb-9ab5-1d76b07247b8" />

- 실험과 관련된 `Configs` 값을 서버로 전달
- 실험 레코드 생성 및 실험 id를 옵티마이저로 전달
- 옵티마이저는 자원 탐색 시작

2. 자원 탐색 (Loop)

<img width="2982" height="1135" alt="image" src="https://github.com/user-attachments/assets/97c41683-2794-4d3d-8983-372eb902e3d6" />

```아래의 동작을 최적의 자원 조합을 찾을 때까지 반복 ```
> 1. 테스트 작업 요청 (Optimizer -> Backend)
> 2. 컨테이너 생성 및 테스트 지시 (Backend -> Remote Agent)
> 3. 부하 테스트 실행 (k6 -> Application)
> 4. 테스트 결과 전달 (k6 -> Prometheus -> Agent)
> 5. 테스트 결과 반환 (Agent -> Backend)
> 6. 부하 테스트 피드백 (Backend -> Optimizer)

3.결과

<img width="1354" height="559" alt="image" src="https://github.com/user-attachments/assets/05d249d5-c732-4345-a322-177779ee3dd2" />

- `(cpu: 1.50 vCPU, mem: 756MB)`와 같이 최적화된 조합 반환
- Backend는 experimentsummaries table에 최종 결과 저장

</br>

## 🚀 시작하기 (Getting Started)

사전 요구사항

- Docker & Docker Compose
- Node.js (v18 이상)
- PostgreSQL

`각 레포지토리 README 참조`
