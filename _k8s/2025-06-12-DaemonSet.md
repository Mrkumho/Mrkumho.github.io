## 1. DaemonSet이란?

- **DaemonSet**은 쿠버네티스에서 각 노드마다 하나의 파드 인스턴스를 실행시키는 리소스.
- **ReplicaSet/Deployment**와 유사하게 여러 파드를 배포하지만,  
  **각 노드에 정확히 하나의 파드**를 자동으로 배치한다는 점이 다름.
- **새 노드가 추가되면** DaemonSet이 해당 노드에 파드를 자동으로 배치함.
- **노드가 제거되면** 해당 노드의 파드도 자동으로 삭제.
- **즉, DaemonSet은 클러스터의 모든 노드에 항상 하나의 파드를 보장.**

---

## 2. DaemonSet의 주요 사용 예

- **모니터링 에이전트/로그 수집기 배포**
  - 모든 노드에 모니터링이나 로그 수집 파드를 자동으로 배치하여 클러스터를 효율적으로 모니터링할 수 있음.
- **kube-proxy와 같은 필수 컴포넌트 배포**
  - 쿠버네티스 아키텍처에서 각 노드에 반드시 필요한 kube-proxy를 DaemonSet으로 배포할 수 있음.
- **네트워크 에이전트 배포**
  - Flannel, Calico, Vivenet 등 네트워크 솔루션의 에이전트를 모든 노드에 배포할 때 적합.

---

## 3. DaemonSet 정의 방법

- **YAML 구조**
  - `apiVersion: apps/v1`
  - `kind: DaemonSet`
  - `metadata: name: ...`
  - `spec:`
    - `selector:` (파드와 연결)
    - `template:` (파드 스펙)
- **ReplicaSet과 구조가 거의 동일**하며,  
  `kind`만 `DaemonSet`으로 바꿉니다.
- **라벨 매칭**: selector의 라벨과 pod template의 라벨이 일치해야 합니다.

---

## 4. DaemonSet 생성 및 관리

- **생성**
  - `kubectl create -f daemonset.yaml`
- **확인**
  - `kubectl get daemonsets`
- **상세 정보**
  - `kubectl describe daemonset <이름>`

---

## 5. DaemonSet의 스케줄링 동작

- **이전 방식(쿠버네티스 1.12 이전)**
  - 파드 스펙에 `nodeName`을 직접 지정하여 노드에 배치
- **현재 방식(쿠버네티스 1.12 이후)**
  - **기본 스케줄러**와 **node affinity 규칙**을 사용하여 각 노드에 파드를 배치
  - DaemonSet이 자동으로 모든 노드에 파드를 배치하고 관리

---

## 6. 요약

- **DaemonSet은 모든 노드에 하나의 파드를 자동으로 배치 및 관리**
- **모니터링, 로그 수집, 네트워크 에이전트 등에 적합**
- **YAML 구조는 ReplicaSet과 유사하며, kind만 DaemonSet으로 지정**
- **스케줄링은 기본 스케줄러와 node affinity로 처리**
- **생성, 확인, 상세 조회는 kubectl 명령어로 가능**

---

> **핵심 메시지:**  
> DaemonSet은 쿠버네티스 클러스터의 모든 노드에 반드시 필요한 서비스를 자동으로 배포하고 관리할 때 가장 효과적.
