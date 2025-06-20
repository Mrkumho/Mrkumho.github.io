# 1. kubeconfig란?
쿠버네티스 클러스터 접속 정보를 관리하는 설정 파일로, 다음과 같은 정보를 저장 :
 - 클러스터 주소와 인증 정보
 -   사용자 인증서 및 키
 - 클러스터-사용자 매핑 컨텍스트

💡 매번 kubectl 명령어에 서버 주소/인증서 옵션을 입력하는 번거로움 해소

# 2. kubeconfig 파일 구조 (YAML)
```yaml
apiVersion: v1
kind: Config

# 1. 클러스터 정의
clusters:
- name: my-kube-playground
  cluster:
    server: https://api-server:6443
    certificate-authority: /path/to/ca.crt  # CA 인증서 경로

# 2. 사용자 정의
users:
- name: my-kube-admin
  user:
    client-certificate: /path/to/admin.crt  # 사용자 인증서
    client-key: /path/to/admin.key          # 사용자 개인키

# 3. 컨텍스트 정의
contexts:
- name: my-context
  context:
    cluster: my-kube-playground  # 클러스터 참조
    user: my-kube-admin          # 사용자 참조
    namespace: frontend          # (옵션) 기본 네임스페이스

# 4. 기본 컨텍스트 지정
current-context: my-context
```

# 3. 주요 섹션 설명
## 3.1 Clusters (클러스터 목록)

| 필드                   | 설명                        | 필수 여부 |
|------------------------|-----------------------------|-----------|
| `name`                 | 클러스터 식별명             | ✅         |
| `server`               | API 서버 주소 (https://)    | ✅         |
| `certificate-authority`| CA 인증서 파일 경로         | ✅         |

```bash
clusters:
- name: dev-cluster
  cluster: {server: https://dev-api:6443, ...}
- name: prod-cluster
  cluster: {server: https://prod-api:6443, ...}
```

## 3.2 Users (사용자 목록)
| 필드                  | 설명                        | 대체 옵션                |
|-----------------------|-----------------------------|--------------------------|
| `client-certificate`  | 사용자 인증서 경로          | `client-certificate-data`|
| `client-key`          | 사용자 개인키 경로          | `client-key-data`        |

**Base64 데이터 사용 예시:**
```yaml
users:
- name: encoded-user
  user:
    client-certificate-data: LS0tLS1CR...  # Base64 인코딩
    client-key-data: LS0tLS1CR...
```
## 3.3 Contexts (컨텍스트)


| 필드         | 설명                | 기본값    |
|--------------|---------------------|-----------|
| `cluster`    | 대상 클러스터명     | -         |
| `user`       | 사용할 사용자명     | -         |
| `namespace`  | 기본 네임스페이스   | `default` |

### **예시: 프로덕션 환경 admin 계정**

```yaml
contexts:
- name: prod-admin
  context:
    cluster: prod-cluster
    user: prod-admin
    namespace: production
```
# 4. kubeconfig 관리 명령어
## 4.1 기본 명령어

```bash
# 현재 설정 확인
kubectl config view

# 컨텍스트 목록
kubectl config get-contexts

# 현재 컨텍스트 확인
kubectl config current-context

# 컨텍스트 전환
kubectl config use-context prod-admin

# 네임스페이스 설정
kubectl config set-context --current --namespace=backend
```
## 4.2 파일 위치 관리

### 기본 위치: ~/.kube/config
```bash
ls ~/.kube/config
```

### 커스텀 파일 사용
```bash
kubectl --kubeconfig=/path/to/custom-config get pods
```
# 5. 실무 적용 사례
## 사례 1: 멀티 클러스터 관리
```bash
graph TD
    A[개발자] --> B(dev-cluster)
    A --> C(staging-cluster)
    A --> D(prod-cluster)
    B -->|Context: dev| E[개발 환경]
    C -->|Context: stage| F[스테이징]
    D -->|Context: prod| G[운영 환경]
```
## 사용법:

```bash
# 개발 클러스터 작업
kubectl config use-context dev

# 운영 클러스터로 전환
kubectl config use-context prod
```
## 사례 2: 역할별 접근 제어
```bash
users:
- name: dev-user    # 개발자: 읽기 전용
  user: { ... }
- name: ci-bot      # CI/CD: 배포 권한
  user: { ... }
- name: prod-admin  # 운영자: 전체 권한
  user: { ... }
```
# 6. 인증서 관리 팁
| 방식         | 장점                | 단점                      |
|--------------|---------------------|---------------------------|
| 파일 경로    | 설정 간편           | 파일 이동 시 경로 재설정 필요 |
| Base64 데이터| 파일 의존성 제거    | 설정 파일 크기 증가         |
변환 방법:

```bash
# 인증서 → Base64
cat ca.crt | base64 | tr -d '\n'

# Base64 → 인증서
echo "LS0tLS1CR..." | base64 -d > ca.crt
```
참고: 민감 정보는 Kubernetes Secrets으로 관리하고 kubeconfig에서 참조하는 것이 안전.
