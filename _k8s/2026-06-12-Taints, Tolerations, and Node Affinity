
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
```
