## Preemption Policy란?
- **정의**: 높은 우선순위 파드가 리소스 부족 시 **기존 파드를 축출(Evict)할지 대기할지**를 결정하는 정책.
- **두 가지 모드**:
  - **PreemptLowerPriority** (기본값): 낮은 우선순위 파드 축출
  - **Never**: 축출 없이 리소스 확보까지 대기

---

## 1. PreemptLowerPriority 예시

### 시나리오
- **노드 상태**: CPU 1코어, 메모리 2GiB (전체 사용 중)
- **기존 파드**: 우선순위 5인 `low-priority-job` (CPU 1코어 사용)
- **새 파드**: 우선순위 7인 `critical-app` (CPU 1코어 필요)

### 동작
1. `critical-app` 파드 생성 → 노드 리소스 부족.
2. 스케줄러가 `low-priority-job` 파드 축출(Evict).
3. `critical-app` 파드가 노드에 배치.

### YAML 예시
PriorityClass 정의 (축출 허용)
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault : true # Option(생략 가능)
preemptionPolicy: PreemptLowerPriority # 기본값 (생략 가능)
```


---

## 2. Never 예시

### 시나리오
- **노드 상태**: CPU 1코어, 메모리 2GiB (전체 사용 중)
- **기존 파드**: 우선순위 5인 `low-priority-job` (CPU 1코어 사용)
- **새 파드**: 우선순위 7인 `batch-job` (CPU 1코어 필요, `Never` 정책)

### 동작
1. `batch-job` 파드 생성 → 노드 리소스 부족.
2. 스케줄러가 `low-priority-job` 파드를 축출하지 **않음**.
3. `batch-job` 파드는 **대기열에서 우선순위 유지** → 다른 노드에 리소스가 확보되면 실행.

### YAML 예시
PriorityClass 정의 (축출 금지)
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
name: non-preempting
  value: 1000000
preemptionPolicy: Never # 축출 금지 명시적 설정
```

---

## 3. 정책 비교 표

| 정책                  | 축출 동작 | 스케줄링 대기열 우선순위 | 사용 사례                     |
|-----------------------|-----------|--------------------------|------------------------------|
| **PreemptLowerPriority** | O         | 적용 없음                | 즉시 실행이 필요한 중요 워크로드 |
| **Never**                | X         | O (높은 우선순위 유지)   | 백그라운드 작업, 데이터 분석   |

---

## 4. 주의사항
- **기본값**: `PreemptLowerPriority` (명시적 설정 없을 시).
- **축출 영향**: 축출된 파드는 정상 종료 절차(`terminationGracePeriodSeconds`)를 거침.
- **대기열 우선순위**: `Never` 정책 파드도 대기열에선 높은 우선순위를 가짐.
- **시스템 파드**: `system-cluster-critical` 등 시스템 클래스는 항상 축출 권한을 가짐.

---

> **핵심 정리**  
> - **`PreemptLowerPriority`**: 긴급한 워크로드에 사용 (예: 결제 시스템).  
> - **`Never`**: 리소스 확보 시까지 기다려도 되는 작업에 사용 (예: 야간 배치 처리).  
> - 정책 선택은 **워크로드의 중요성과 내구성 요구사항**에 따라 결정.
>

## **※ 유용한 priorityClass 비교 명령어**  

```bash
kubectl get pods -o custom-columns="NAME:.metadata.name,PRIORITY:.spec.priorityClassName"
```

### 1. 기본 동작
```bash
kubectl get pods
```
- 현재 네임스페이스에 있는 파드(Pod) 목록을 조회.

### 2. 커스텀 칼럼 옵션
```bash
-o custom-columns="..."
```
- 출력할 컬럼(필드)을 직접 지정.
- 커스텀 칼럼을 사용하면 기본 출력보다 원하는 정보만 선택적으로 볼 수 있음.

### 3. 커스텀 칼럼 정의
```bash
NAME:.metadata.name
```
- 파드의 이름(.metadata.name)을 NAME 컬럼에 출력.
```bash
PRIORITY:.spec.priorityClassName
```
- 파드에 지정된 우선순위 클래스 이름(.spec.priorityClassName)을 PRIORITY 컬럼에 출력.

### 4. 출력 예시
```text
NAME           PRIORITY
my-pod-1       high-priority
my-pod-2       <none>
```
- 첫 번째 컬럼: 파드 이름
- 두 번째 컬럼: 파드에 할당된 우선순위 클래스 이름 (없으면 <none>)

