## 1. Admission Controller란?

- **역할**: API 서버에 들어오는 요청(예: 파드 생성, 삭제, 수정)을 검증하거나 수정하는 "게이트키퍼" 역할
- **동작 시점**: 인증(Authentication) → 인가(Authorization) → **Admission Controller**(검증/변형) → etcd 저장
- **종류**:
  - **Validating**: 요청을 검증(예: 정책 위반 시 거부)
  - **Mutating**: 요청을 수정(예: 기본값 추가, 라벨/어노테이션 삽입)
  - **Both**: 검증과 변형 모두 수행

---

## 2. 왜 Admission Controller가 필요한가?

- **RBAC(역할 기반 접근 제어) 한계**:  
  - "누가 무엇을 할 수 있는지"만 제한
  - "어떤 값이 들어오는지"는 제어 불가
- **Admission Controller 추가 기능**:
  - 예: 특정 이미지 레지스트리만 허용, latest 태그 금지, root 사용 금지, 라벨 필수 등

---

## 3. Admission Controller 동작 예시

### 3.1. Namespace Exists Admission Controller

- **설명**: 파드 생성 요청이 들어오면, 지정된 네임스페이스가 있는지 확인
- **예시**:
  
```bash
kubectl create pod -n blue
```

- 네임스페이스 `blue`가 없으면 요청 거부
- **동작 흐름**:
1. 인증: 사용자 확인
2. 인가: 권한 확인
3. Admission Controller: 네임스페이스 존재 여부 확인
4. 거부 또는 허용

### 3.2. AlwaysPullImages Admission Controller

- **설명**: 파드 생성 시 이미지 풀 정책을 항상 `Always`로 변경
- **목적**: 노드에 캐시된 이미지를 다른 사용자가 악용하지 못하도록 방지
- **예시**:  
- 파드가 생성될 때마다 이미지를 새로 받아옴

---

## 4. Admission Controller의 종류와 기능

| 이름                                | 타입         | 설명                                                         |
|-------------------------------------|-------------|------------------------------------------------------------|
| NamespaceLifecycle                  | Validating  | 네임스페이스 존재 여부 확인, 기본 네임스페이스 삭제 방지         |
| AlwaysPullImages                    | Mutating    | 이미지 풀 정책을 항상 `Always`로 변경                        |
| DefaultStorageClass                 | Mutating    | PVC에 기본 스토리지 클래스 자동 추가                         |
| EventRateLimit                      | Validating  | API 서버에 들어오는 요청 수 제한                             |
| ResourceQuota                       | Validating  | 네임스페이스별 리소스 할당량 초과 시 거부                     |
| MutatingAdmissionWebhook            | Mutating    | 외부 웹훅을 통해 요청 변형                                   |
| ValidatingAdmissionWebhook          | Validating  | 외부 웹훅을 통해 요청 검증                                   |

---

## 5. Admission Controller 설정 방법

### 5.1. 기본 Admission Controller 활성화/비활성화

- **명령어 예시**:
```bash
kube-apiserver --enable-admission-plugins=NamespaceLifecycle,ResourceQuota
```

- **Kubeadm 환경에서 설정**:
- `/etc/kubernetes/manifests/kube-apiserver.yaml` 파일 수정
- `--enable-admission-plugins` 옵션에 추가

### 5.2. Admission Controller 추가/제거

- **추가**: `--enable-admission-plugins`에 이름 추가
- **제거**: `--disable-admission-plugins`에 이름 추가

---

## 6. 예시: Admission Controller로 정책 강화

### 6.1. 특정 이미지 레지스트리만 허용

- **정책**: `docker.io` 이미지 사용 금지, 내부 레지스트리만 허용
- **방법**: ValidatingAdmissionWebhook 사용

### 6.2. latest 태그 금지

- **정책**: 이미지 태그에 `latest` 사용 금지
- **방법**: ValidatingAdmissionWebhook 또는 OPA Gatekeeper 등 사용

### 6.3. 라벨 필수

- **정책**: 모든 파드에 `team` 라벨 필수
- **방법**: ValidatingAdmissionWebhook 사용

---

## 7. 요약

- **Admission Controller**는 API 요청을 검증하거나 수정하여 클러스터 보안과 정책을 강화[1][3][4]
- **Built-in(내장) 컨트롤러**와 **Dynamic(웹훅 기반) 컨트롤러**로 나뉨
- **정책 예시**: 네임스페이스 존재 확인, 이미지 풀 정책 강제, 리소스 할당량 제한 등
- **설정 방법**: kube-apiserver 옵션 또는 매니페스트 파일 수정

---

**실무 팁**  
```bash
kube-apiserver -h | grep enable-admission-plugins
kubectl exec kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep enable-admission-plugins
kubectl describe pod kube-apiserver -n kube-system | grep "enable-admission-plugins"
kubectl describe pod kube-apiserver-controlplane -n kube-system | grep "disable-admission-plugins"
ps -ef | grep kube-apiserver | grep admission-plugins
```
