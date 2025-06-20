# 쿠버네티스 인증서 관리 & Certificate API

## 1. 기존 인증서 관리 방식 (수동)

### 시나리오
- 클러스터 관리자(admin)가 CA 서버 설정 + 모든 컴포넌트 인증서 생성
- 새 관리자(Alice)가 팀에 합류 → 클러스터 접근 권한 필요

### 과정
```
graph LR
    A[Alice] -->|1. 개인키 생성| B[CSR 생성]
    B -->|2. Admin에게 전달| C[Admin]
    C -->|3. CA 개인키로 서명| D[인증서 생성]
    D -->|4. Alice에게 전달| A
```

### 문제점
- **사용자 증가 시**: 수동 작업 부담 증가
- **인증서 만료 시**: 동일 과정 반복 필요
- **보안 위험**: CA 개인키 관리 위험

---

## 2. Certificate API (자동화)

### 필요성
- 사용자 증가 및 인증서 만료 시 자동 승인/갱신 필요
- `kube-controller-manager` 내장 기능 활용

### 작동 원리
```
graph LR
    A[사용자] -->|1. 개인키 생성 & CSR| B[Admin]
    B -->|2. Base64 인코딩| C[CertificateSigningRequest 생성]
    C -->|3. kubectl get csr| D[관리자 검토]
    D -->|4. kubectl certificate approve| E[Controller Manager]
    E -->|CA 키로 서명| F[인증서 생성]
    F -->|5. Base64 디코딩| A
```

---

## 3. Certificate API 사용 단계

### 사용자(Alice) 작업
1. **개인키 생성**:
```bash
openssl genrsa -out alice.key
```

2. **CSR 생성**:
```bash
openssl req -new -key alice.key -out alice.csr -subj "/CN=alice"
```
text

### 관리자(Admin) 작업
3. **CSR Base64 인코딩**:
```bash
cat alice.csr | base64 | tr -d '\n'
```

4. **CSR 매니페스트 생성** (`alice-csr.yaml`):
```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: alice-csr
spec:
  signerName: kubernetes.io/kube-apiserver-client
  request: <BASE64_ENCODED_CSR> # 인코딩된 CSR
  usages:
- client auth
```

5. **오브젝트 생성**:
```bash
kubectl apply -f alice-csr.yaml
```

6. **CSR 승인**:
```bash
kubectl certificate approve alice-csr
```

7. **인증서 추출**:
```bash
kubectl get csr alice-csr -o yaml | grep "certificate:" | awk '{print $2}' | base64 -d > alice.crt
```

---

## 4. 내부 작동 원리

### 핵심 컴포넌트
- **`kube-controller-manager`**:
- `CSR-Approving`: CSR 승인 관리
- `CSR-Signing`: CA 키로 서명 수행

### 필수 구성
Controller Manager 설정
```bash
--cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
```
```bash
--cluster-signing-key-file=/etc/kubernetes/pki/ca.key
```

### 동작 흐름
1. 승인된 CSR 감지
2. CA 개인키로 서명
3. 인증서 생성 및 저장

---

## 5. 핵심 장점 비교

| **방식**         | **과정**      | **확장성** | **보안**       |
|-------------------|---------------|------------|----------------|
| **수동**          | 5단계         | ❌          | CA 키 노출 위험 |
| **Certificate API** | 2단계 (승인) | ✅          | CA 키 분리 보관 |

---

## 6. 실제 사례

### 사례 1: 사용자 대량 추가
100개 CSR 일괄 승인
```bash
for i in {1..100}; do
kubectl certificate approve user$i-csr
done
```

### 사례 2: 인증서 자동 갱신
kubelet 자동 갱신 활성화
```bash
kubelet --rotate-certificates=true
```

---

## 7. 보안 모범 사례
1. **CA 키 보호**:
```bash
chmod 600 /etc/kubernetes/pki/ca.key
```

2. **인증서 유효기간 단축**:
kube-controller-manager 설정
```bash
--experimental-cluster-signing-duration=2160h # 90일
```

3. **CSR 감사 로깅**:
```bash
kubectl get csr -w -o wide
```

> **결론**: Certificate API는 인증서 생명주기 관리를 자동화해 확장성과 보안성을 동시에 확보 가능.****
