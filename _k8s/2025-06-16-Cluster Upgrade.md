# 쿠버네티스 클러스터 업그레이드 가이드(kubeadm 기준)

---

## 1. 쿠버네티스 버전 구조와 호환성

- **버전 구성:** `major.minor.patch`
  - **major:** 메이저 버전 (현재 1)
  - **minor:** 마이너 버전 (주기적 기능 추가, 몇 달마다 업데이트)
  - **patch:** 패치 버전 (버그 수정, 자주 업데이트)
- **컴포넌트 버전 호환성**
  - **kube-apiserver:** 모든 컴포넌트 중 가장 높은 버전 유지
  - **controller-manager, scheduler:** kube-apiserver와 동일 또는 1단계 낮은 버전 가능 (예: apiserver 1.10 → 1.9~1.10)
  - **kubelet, kube-proxy:** 2단계 낮은 버전까지 허용 (예: apiserver 1.10 → 1.8~1.10)
  - **kubectl:** apiserver보다 높거나 낮은 버전 모두 사용 가능 (단, 실무에서는 최신 또는 동일 버전 권장)
- **지원 버전:** 최신 3개의 마이너 버전만 지원 (예: 1.30, 1.29, 1.28)

---

## 2. 업그레이드 전략 및 절차

### 2.1. 업그레이드 순서

1. **컨트롤 플레인(마스터 노드) 업그레이드**
   - 마스터 노드의 kube-apiserver, controller-manager, scheduler 등 주요 컴포넌트 업그레이드
   - 업그레이드 중에는 클러스터 관리 기능(예: kubectl, API 접근)이 일시 중단됨
   - 워커 노드와 실행 중인 파드는 계속 서비스 제공 (서비스 영향 없음)
2. **워커 노드 업그레이드**
   - 마스터 업그레이드 후 워커 노드 순차 업그레이드
   - 업그레이드 방식: 한 번에 모두, 한 노드씩, 신규 노드 추가 후 기존 노드 제거 등

### 2.2. 업그레이드 방식

- **한 번에 모두 업그레이드**
  - 모든 워커 노드를 동시에 업그레이드 → 서비스 중단 발생
- **한 노드씩(롤링) 업그레이드**
  - 각 노드를 순차적으로 drain(파드 제거) 및 업그레이드
  - 파드는 다른 노드로 이동, 서비스 영향 최소화
- **신규 노드 추가 후 기존 노드 제거**
  - 신규 버전의 노드를 추가 → 파드 이동 → 기존 노드 제거
  - 클라우드 환경에서 쉽게 적용 가능

---

## 3. kubeadm을 활용한 업그레이드 실습

### 3.1. 업그레이드 준비

- **kubeadm 업그레이드**
```bash
apt-mark unhold kubeadm &&
apt-get update &&
apt-get install -y kubeadm=1.30.0-00 &&
apt-mark hold kubeadm
```

- **업그레이드 가능 버전 확인**
```basj
kubeadm upgrade plan
```

### 3.2. 컨트롤 플레인 업그레이드

- **컨트롤 플레인 업그레이드 실행**
```bash
kubeadm upgrade apply v1.30.0
```

- **컨트롤 플레인 컴포넌트가 최신 버전으로 업그레이드됨**

### 3.3. 워커 노드 업그레이드

1. **노드 drain (파드 제거 및 스케줄링 차단)**
```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

2. **kubelet 업그레이드 및 재시작**
```bash
apt-mark unhold kubelet &&
apt-get update &&
apt-get install -y kubelet=1.30.0-00 &&
apt-mark hold kubelet # kubelet 패키지는 현재 설치된 버전에 고정(보류)
systemctl restart kubelet
```

3. **노드 uncordon (스케줄링 허용)**
```bash
kubectl uncordon <node-name>
```

4. **모든 노드에 위 과정 반복**

---

## 4. 업그레이드 시 주의사항 및 모범 사례

- **업그레이드 계획 수립:** 릴리즈 노트 확인, 스테이징 환경에서 테스트, 롤백 계획 마련[5][6]
- **롤링 업그레이드 권장:** 한 번에 모든 노드를 업그레이드하지 말고, 한 노드씩 또는 배치 단위로 진행[3][5]
- **중요 애플리케이션 확인:** 업그레이드 전후로 애플리케이션 동작 검증
- **kubectl 및 기타 클라이언트 업그레이드:** 클러스터 업그레이드 후 kubectl 등 클라이언트도 최신 버전으로 유지[2][3]
- **ETCD, CoreDNS 등 외부 컴포넌트 버전 확인:** 지원 버전 및 호환성 체크 필요

---

## 5. 업그레이드 후 점검

- **노드 상태 확인**
```bash
kubectl get nodes
```

- 모든 노드가 `Ready` 상태이고, 버전이 일치하는지 확인
- **파드 및 서비스 정상 동작 확인**

---

## 6. 요약

- **업그레이드 순서:** 컨트롤 플레인 → 워커 노드 → 클라이언트(kubectl 등)
- **업그레이드 방식:** 롤링 업그레이드(한 노드씩 또는 신규 노드 추가 후 기존 노드 제거)
- **kubeadm 활용:** `kubeadm upgrade plan` 및 `kubeadm upgrade apply` 명령 사용
- **노드 업그레이드:** drain → 업그레이드 → uncordon 반복
