## 쿠버네티스 Node Selector

쿠버네티스에서 **Node Selector(노드 선택기)**를 사용해 특정 노드에 파드를 배치하는 방법을 실생활 예시와 함께 정리.

---

### 1. 기본 상황

- **클러스터 구성**
  - 작은 노드 2대(`node2`, `node3`)
  - 큰 노드 1대(`node1`)
- **목표**
  - 고사양 작업(예: 데이터 처리)을 반드시 `node1`에만 배치하고 싶음

---

### 2. 노드에 라벨 추가하기

노드에 라벨을 붙여 분류.  
예를 들어, `node1`에 `size=large` 라벨을 추가.

```bash
kubectl label nodes node1 size=large
```

> `node2`, `node3`에는 `size=small` 라벨을 붙일 수 있음.

---

### 3. 파드에 Node Selector 지정

파드 정의 파일에서 `nodeSelector`를 사용해 라벨을 지정.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: data-processor
spec:
  containers:
    - name: data-processor
      image: data-processing-image
  nodeSelector:
    size: large  # "size=large" 라벨이 있는 노드에만 배치
```

---

### 4. 결과

- 파드는 `node1`에만 배치.
- 실제 배치 결과는 다음 명령어로 확인할 수 있음.

```bash
kubectl get pods -o wide
```

---

### 5. Node Selector의 한계

- **단순 조건만 가능**
  - 예: "large 노드에만 배치"는 가능
  - 예: "large 또는 medium 노드" 또는 "small 노드 제외" 같은 복잡한 조건은 불가
- **해결책**
  - 더 복잡한 조건이 필요하다면 **Node Affinity/Anti-Affinity** 기능을 사용해야 함

---

### 예시로 이해하는 한계

- **원하는 조건**: "large 또는 medium 노드에 배치"
- **Node Selector로는 불가능** → Node Affinity 사용 필요

---

### 정리

- Node Selector는 간단한 라벨 기반 스케줄링에 적합.
- 복잡한 조건에는 Node Affinity를 사용해야 함.
- 실제 운영 환경에서는 라벨링과 셀렉터를 적절히 조합해 효율적으로 리소스를 관리할 수 있음.

---
