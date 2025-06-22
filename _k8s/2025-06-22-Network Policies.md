# Kubernetes Network Policies 가이드

## 1. Network Policies 기본 개념

- **목적:**  
  특정 Pod 간 통신을 제어하여 클러스터 보안을 강화.
- **기본 동작:**  
  - 기본적으로 네트워크 정책이 없는 경우 클러스터 내의**모든 Pod 간 통신이 허용** 됨.
  - Network Policies 가 적용되면 **명시적으로 허용된 트래픽만 통과** 함.

## 2. 핵심 구성 요소
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:  # 정책 적용 대상 Pod 선택
    matchLabels:
      role: db
  policyTypes:  # 정책 유형 (Ingress/Egress)
  - Ingress
  ingress:  # 수신 규칙
  - from:    # 허용 소스
      - podSelector:
          matchLabels:
            role: api
    ports:    # 허용 포트
    - protocol: TCP
      port: 3306
```

## 3. 주요 선택자(Selector) 유형

### 3.1 대상 지정

| **선택자**           | **용도**                                 | **예시**                     |
|----------------------|------------------------------------------|------------------------------|
| `podSelector`        | 동일 네임스페이스 내 Pod 지정            | `matchLabels: {app: api}`    |
| `namespaceSelector`  | 특정 네임스페이스의 모든 Pod 허용        | `matchLabels: {env: prod}`   |
| `ipBlock`            | 클러스터 외부 IP 대역 지정               | `cidr: 192.168.5.0/24`       |

### 3.2 규칙 조합 방식 
```yaml
ingress:
- from:
  - podSelector: {matchLabels: role: api}      # AND
    namespaceSelector: {matchLabels: env: prod}
  - ipBlock: {cidr: 192.168.5.0/24}           # OR
```

## 4. 실전 적용 사례: 데이터베이스 보호

### 4.1 요구사항

- DB Pod(`role: db`)는 API Pod(`role: api`)의 3306 포트 접근만 허용
- 다른 모든 트래픽 차단

### 4.2 정책 정의

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress  # 수신 트래픽만 제어
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: api
    ports:
    - protocol: TCP
      port: 3306
```

### 4.3 동작 원리
```bash
graph LR
    A[API Pod] -->|허용| B[(DB Pod)]
    C[Web Pod] -->|차단| B
    D[외부 서버] -->|차단| B
```

## 5. 고급 시나리오

### 5.1 프로덕션 네임스페이스만 허용

```yaml
ingress:
- from:
  - podSelector: {matchLabels: role: api}
    namespaceSelector: {matchLabels: env: prod}  # prod 네임스페이스만
```

### 5.2 백업 서버 접근 허용
```yaml
ingress:
- from:
  - ipBlock: {cidr: 192.168.5.10/32}  # 특정 IP 허용
```

### 5.3 아웃바운드(egress) 제어

```yaml
egress:
- to:
  - ipBlock: {cidr: 10.0.0.0/24}
  ports:
  - port: 80
    protocol: TCP
```

## 6. 기본 정책 설정

### 6.1 모든 수신 트래픽 차단

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}  # 모든 Pod 선택
  policyTypes:
  - Ingress
```

### 6.2 모든 수신 트래픽 허용

```yaml
ingress:
- {}  # 빈 값 = 모든 소스 허용
```

## 7. 주의사항

1. **네트워크 플러그인 필수:**  
   Calico, Cilium 등 Network Policy 지원 플러그인 필요
2. **정책 우선순위:**  
   동일 Pod에 여러 정책 적용 시 **OR 조건**으로 결합
3. **IP Block 한계:**  
   Pod IP는 임시적 → 지속성 보장 불가  
   외부 IP 제어용으로만 권장
