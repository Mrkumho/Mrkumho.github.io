
# Kubernetes Taints, Tolerations, and Node Affinity

## 문제 상황

- **3개의 노드**: 각각 blue, red, green 색상
- **3개의 파드**: 각각 blue, red, green 색상
- **목표**: blue 파드는 blue 노드에, red 파드는 red 노드에, green 파드는 green 노드에 배치
- **공유 환경**: 다른 팀의 파드와 노드도 클러스터에 존재

## 제약 조건

- **우리의 파드가 다른 노드에 배치되지 않는 것**
- **다른 팀의 파드가 우리 노드에 배치되지 않는 것**

---

## 1. Taints와 Tolerations로 시도

- **노드에 taint 적용**: 각 노드에 색상(blue, red, green) taint를 부여
- **파드에 toleration 설정**: 각 파드에 해당 색상의 toleration 부여
- **결과**:  
  - 색상에 맞는 파드만 해당 노드에 배치됨  
  - 하지만, taints와 tolerations만으로는 파드가 반드시 해당 노드에만 배치된다는 보장이 없음  
  - 예: toleration이 없는 다른 파드가 노드에 배치될 수 있음

---

## 2. Node Affinity로 시도

- **노드에 라벨 부여**: 각 노드에 색상(blue, red, green) 라벨 부여
- **파드에 nodeSelector 또는 nodeAffinity 설정**: 각 파드가 해당 색상 노드에만 배치되도록 설정
- **결과**:  
  - 파드가 원하는 노드에 배치됨  
  - 하지만, 다른 팀의 파드가 우리 노드에 배치될 수 있음

---

## 3. Taints, Tolerations, Node Affinity 조합

- **taints와 tolerations 사용**: 다른 팀의 파드가 우리 노드에 배치되지 않도록 방지
- **node affinity 사용**: 우리 파드가 다른 노드에 배치되지 않도록 방지
- **최종 결과**:  
  - 노드가 특정 파드에 완전히 종속됨

---

## 요약

- **taints와 tolerations**: 노드에 특정 파드만 허용, 다른 파드 배치 방지
- **node affinity**: 파드가 특정 노드에만 배치되도록 보장
- **두 기능을 조합**하면, 노드를 특정 파드에 완전히 종속 시킬 수 있음

> **결론:**  
> Taints & Tolerations와 Node Affinity를 함께 사용하여 노드와 파드의 배치를 완벽하게 제어할 수 있음.

## Operator 옵션 정리

쿠버네티스에서 Taint와 Toleration을 사용할 때,  
`operator`는 **어떤 조건에서 toleration이 적용될지**를 결정합니다.

---

### operator: Exists

- **의미:**  
  - 해당 키(key)가 존재하는 모든 Taint에 대해 toleration이 적용.
  - 값(value)은 무시.
- **예시:**
```yaml
tolerations:
  key: "node.kubernetes.io/not-ready"
  operator: "Exists"
  effect: "NoExecute"
```
- **node.kubernetes.io/not-ready**라는 키가 있는 모든 Taint에 대해 toleration 적용

### operator: Equal

- **의미:**  
- Taint의 키와 값이 모두 일치해야 toleration이 적용.
- `value` 필드를 반드시 지정해야 함.
- **예시:**
```yaml
tolerations:

key: "example.com/special"
operator: "Equal"
value: "true"
effect: "NoSchedule"
```
- **example.com/special=true**인 Taint에만 toleration 적용

---

| operator   | 의미                                            | value 필요? | 예시 적용 대상                  |
|------------|------------------------------------------------|-------------|---------------------------------|
| Exists     | 해당 키가 있는 모든 Taint에 적용                | No          | node.kubernetes.io/not-ready    |
| Equal      | 키와 값이 모두 일치하는 Taint에만 적용          | Yes         | example.com/special=true        |

---

## 참고

- **effect:**  
- `NoSchedule`, `NoExecute`, `PreferNoSchedule` 등 Taint의 효과(Effect)도 함께 지정해야 함.
- **실제 사용 예시:**  
- **Exists**는 노드 상태(ready, not-ready, unreachable 등)와 같이 값이 없는 Taint에 주로 사용
- **Equal**은 사용자 정의 Taint(예: 특정 노드에만 배포)에 주로 사용

---

## 정리

- **operator: Exists** → 키가 있으면 toleration 적용
- **operator: Equal** → 키와 값이 모두 일치해야 toleration 적용
