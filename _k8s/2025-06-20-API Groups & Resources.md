# Kubernetes API Groups & Resources 요약

## 1. API 그룹 구조

- **Core API Group (Legacy Group)**
  - **핵심 리소스:** Pods, Nodes, Namespaces, Services 등
  - **엔드포인트:** `/api/v1`
- **Named API Groups**
  - **기능별 분류:** `apps`, `networking.k8s.io`, `batch` 등
  - **예시:**
    - `apps/v1`: Deployments, StatefulSets
    - `networking.k8s.io/v1`: Ingress, NetworkPolicy
  - **엔드포인트:** `/apis/$GROUP_NAME/$VERSION`

## 2. 리소스(Resources)와 동작(Verbs)

- **리소스 예시:** Pods, Services, Deployments, ConfigMaps 등
- **주요 동작(Verbs):**

| Verb     | 설명           | HTTP Method |
|----------|----------------|-------------|
| get      | 단일 리소스 조회 | GET         |
| list     | 리소스 목록 조회 | GET         |
| create   | 리소스 생성     | POST        |
| update   | 전체 수정       | PUT         |
| patch    | 부분 수정       | PATCH       |
| delete   | 삭제           | DELETE      |

## 3. 네트워킹 컴포넌트

- **Kube-proxy**
  - **역할:** Pod-Service 간 통신 관리
  - **기능:**
    - Load Balancing (트래픽 분산)
    - Service Discovery (서비스 IP ↔ Pod 매핑)
    - Failover (장애 Pod 자동 제외)
  - **동작 모드:** `iptables` (기본), `IPVS`, `userspace`
- **Kube-control proxy**
  - **역할:** Kube API Server에 대한 HTTP 프록시
  - **사용처:** `kubectl` 명령어 내부 통신

## 4. 인증/인가 연계

- **RBAC(Role-Based Access Control):**
  - **설명:** API 그룹/리소스/동작 조합으로 권한 제어
  - **예시 Role:**
    ```
    rules:
    - apiGroups: ["apps"]
      resources: ["deployments"]
      verbs: ["get", "list", "create"]
    ```
