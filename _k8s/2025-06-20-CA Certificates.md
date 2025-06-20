# Kubernetes 인증서 생성 및 관리 가이드

## 1. 인증서 생성 도구
- **주요 도구**: OpenSSL, Easy-RSA, CFSSL
- **본 가이드에서는 OpenSSL 사용**

## 2. CA(인증기관) 인증서 생성
### 단계별 생성 과정
```bash
# 1. CA 개인키 생성
openssl genrsa -out ca.key

# 2. 인증서 요청서(CSR) 생성
openssl req -new -key ca.key -out ca.csr
# Common Name(CN): Kubernetes-CA

# 3. 자체 서명된 인증서 발급
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

## 3. 클라이언트 인증서 생성
### 3.1 관리자(admin) 인증서
```bash
# 1. 개인키 생성
openssl genrsa -out admin.key

# 2. CSR 생성
openssl req -new -key admin.key -out admin.csr
# CN: kube-admin (임의 지정 가능)

# 3. CA 서명으로 인증서 발급
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```

### 3.2 그룹 정보 추가
- **관리자 권한 부여**: `system:masters` 그룹 지정
- **CSR(Certificate Signing Request) 생성 시 OU(Organizational Unit) 필드에 그룹명 입력**
```bash
openssl req -new -key admin.key -out admin.csr \
  -subj "/CN=kube-admin/OU=system:masters"
```

### 3.3 시스템 컴포넌트 인증서
- **kube-scheduler, controller-manager, kube-proxy**
- **이름 규칙**: `system:` 접두사 사용
  - 예: `system:kube-scheduler`

## 4. 서버 인증서 생성
### 4.1 etcd 서버
```bash
# 1. 개인키 생성
openssl genrsa -out etcd-server.key

# 2. CSR 생성
openssl req -new -key etcd-server.key -out etcd-server.csr
# CN: etcd-server

# 3. 인증서 발급
openssl x509 -req -in etcd-server.csr -CA ca.crt -CAkey ca.key -out etcd-server.crt
```

### 4.2 kube-apiserver 인증서
**특수 요구사항**:  
- 여러 이름(별칭) 지원 필요
  - `kubernetes`
  - `kubernetes.default`
  - `kubernetes.default.svc`
  - `kubernetes.default.svc.cluster.local`
  - 서버 IP 주소

**생성 방법**:
1. OpenSSL 설정 파일(`openssl.cnf`) 생성
```ini
[req]
req_extensions = v3_req

[v3_req]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 
```

2. CSR 생성 시 설정 파일 적용
```bash
openssl req -new -key apiserver.key -out apiserver.csr \
  -config openssl.cnf
```

### 4.3 kubelet 인증서
- **노드별 생성 필요**
- **이름 규칙**: `system:node:`
  - 예: `system:node:node01`
- **그룹 지정**: `system:nodes`

## 5. 인증서 사용 방법
### 5.1 클라이언트 측
- **kubeconfig 파일에 저장**:
  ```yaml
  users:
  - name: admin
    user:
      client-certificate: admin.crt
      client-key: admin.key
  ```
- **REST API 호출 시 직접 사용**:
  ```bash
  curl https:// --cert admin.crt --key admin.key --cacert ca.crt
  ```

### 5.2 서버 측 구성
**kube-apiserver 설정 예시**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
spec:
  containers:
  - command:
    - kube-apiserver
    - --tls-cert-file=apiserver.crt
    - --tls-private-key-file=apiserver.key
    - --client-ca-file=ca.crt
    - --etcd-cafile=ca.crt
    - --etcd-certfile=apiserver-etcd-client.crt
    - --etcd-keyfile=apiserver-etcd-client.key
    - --kubelet-client-certificate=apiserver-kubelet-client.crt
    - --kubelet-client-key=apiserver-kubelet-client.key
```

## 6. 인증서 관리 핵심 원칙
1. **CA 인증서가 루트 신뢰체**:
   - 모든 컴포넌트는 CA 인증서 사본 보유 필요
   - 상호 인증 시 CA 인증서로 검증

2. **개인키 보안**:
   - 절대 외부 유출 금지
   - 파일 권한 설정(`chmod 600 *.key`)

3. **이름/그룹 정보**:
   - RBAC 권한 관리의 기반
   - CN(Common Name): 사용자/컴포넌트 식별
   - OU(Organization Unit): 그룹 지정

4. **서버 인증서의 Alt Names**:
   - kube-apiserver는 모든 접근 이름 포함 필요

## 7. 인증서 생성 요약 표
| 단계 | 명령어 예시 |
|------|-------------|
| **CA 생성** | `openssl genrsa -out ca.key``openssl req -new -key ca.key -out ca.csr``openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt` |
| **클라이언트 인증서** | `openssl genrsa -out user.key``openssl req -new -key user.key -out user.csr``openssl x509 -req -in user.csr -CA ca.crt -CAkey ca.key -out user.crt` |
| **서버 인증서** | 클라이언트와 동일 프로세스별칭 필요 시 `openssl.cnf` 파일 사용 |

## 8. 실무 팁
- **kubeadm 사용 시**: 자동 인증서 생성
- **인증서 갱신**: `openssl ca` 명령어로 갱신
- **인증서 검증**:
  ```bash
  openssl x509 -in  -text -noout
  ```
- **유효기간 관리**: 1년 주기 갱신 권장
