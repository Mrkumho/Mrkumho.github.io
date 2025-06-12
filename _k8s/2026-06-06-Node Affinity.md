## 쿠버네티스 Node Affinity

쿠버네티스 **Node Affinity** 기능을 사용해 파드를 특정 노드에 배치하는 고급 설정 방법 정리.  
기본적인 Node Selector의 한계를 넘어 복잡한 조건을 지원.

---

### 1. Node Affinity vs Node Selector

| 기능          | Node Selector                          | Node Affinity                          |
|---------------|----------------------------------------|----------------------------------------|
| **조건**      | 단일 라벨(key=value)만 가능            | `AND/OR/NOT` 등의 복잡한 조건 지원    |
| **유연성**    | 제한적                                 | 높음 (연산자 조합 가능)                |
| **사용 사례** | 단순 배치                              | 고급 배치 전략 필요 시                 |

---

### 2. Node Affinity 유형

#### 1) **`requiredDuringSchedulingIgnoredDuringExecution`**
- **스케줄링 시**: 반드시 조건을 만족하는 노드에 배치
- **실행 중**: 노드 라벨 변경 시 파드 영향 없음 (무시됨)
- **예시**: "무조건 large 노드에 배치"

#### 2) **`preferredDuringSchedulingIgnoredDuringExecution`**
- **스케줄링 시**: 조건을 만족하는 노드 우선 배치 (불가 시 다른 노드 허용)
- **실행 중**: 노드 라벨 변경 시 파드 영향 없음
- **예시**: "가능하면 large 노드에 배치, 아니면 다른 노드"

---

### 3. 주요 연산자(Operators)

| 연산자   | 설명                                      | 예시                          |
|----------|-------------------------------------------|-------------------------------|
| `In`     | 라벨 값이 목록 중 하나와 일치             | `size In (large, medium)`     |
| `NotIn`  | 라벨 값이 목록에 없음                     | `size NotIn (small)`          |
| `Exists` | 라벨 키가 존재함 (값 무관)                | `size Exists`                 |

---

### 4. YAML 예시

#### 기본 예시 (requiredDuringScheduling)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: data-processor
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: In
                values: [large]
  containers:
    - name: data-processor
      image: data-processing-image
```

#### 복합 조건 예시 (OR)
```yaml
nodeSelectorTerms:
  - matchExpressions:
      - key: size
        operator: In
        values: [large, medium]  # large 또는 medium 노드
```

---

### 5. 라이프사이클 동작

1. **스케줄링 단계**  
   - 조건에 맞는 노드 탐색 → 없으면 파드가 **Pending** 상태 유지 (required 타입)
   - `preferred` 타입은 조건 무시하고 배치

2. **실행 중 라벨 변경**  
   - 기존 파드는 영향 받지 않음 (`IgnoredDuringExecution`)
   - **향후 계획**: `requiredDuringExecution` 타입 추가 예정  
     (라벨 변경 시 파드 강제 퇴출)

---

### 6. 실전 활용 팁

- **라벨 확인**: `kubectl get nodes --show-labels`
- **라벨 추가**:  
  ```bash
  kubectl label nodes node1 size=large
  ```
- **배치 확인**:  
  ```bash
  kubectl get pods -o wide
  ```

---

### 7. Node Affinity vs Taints/Tolerations

| 기능                | Node Affinity                          | Taints/Tolerations                |
|---------------------|----------------------------------------|------------------------------------|
| **목적**            | 파드가 특정 노드에 배치되도록 유도     | 노드가 특정 파드를 차단하도록 설정 |
| **주체**            | 파드 → 노드 선택                       | 노드 → 파드 차단/허용             |
| **조건**            | 라벨 기반                              | Taint/Toleration 매칭             |

---

### 8. 쿠버네티스의 nodeAffinity에서 operator 필드

- **In**: 지정한 값들 중 하나와 라벨 값이 일치해야 함.
- **NotIn**: 지정한 값들 중 어느 것과도 라벨 값이 일치하지 않아야 함.
- **Exists**: 해당 라벨 키가 존재해야 함.(값은 상관 없음).
- **DoesNotExist**: 해당 라벨 키가 존재하지 않아야 함.
- **Gt**: 라벨 값이 지정한 값보다 크아야 함.(숫자 값만 가능).
- **Lt**: 라벨 값이 지정한 값보다 작아야 함.(숫자 값만 가능).
