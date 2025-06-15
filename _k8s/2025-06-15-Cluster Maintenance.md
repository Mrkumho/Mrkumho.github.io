## 🛠 **Node Down 핵심 명령어 비교표**
| 명령어       | 목적                          | 실행 결과                     | 사용 사례                  |
|--------------|-------------------------------|-------------------------------|---------------------------|
| **`drain`**  | 노드 드레이닝 + 스케줄링 금지 | 파드 종료 및 재생성           | 안전한 유지보수 전 준비   |
| **`cordon`** | 스케줄링 금지                 | 기존 파드 유지, 새 파드 차단  | 부분 점검 시              |
| **`uncordon`**| 스케줄링 허용                 | 노드 정상화                   | 유지보수 완료 후 복구     |

**핵심 메커니즘**  
- **Pod Eviction Timeout**: 5분(default)
컨트롤러 매니저 설정 확인
```bash
kube-controller-manager --pod-eviction-timeout=5m
```
---


## 📝 **명령어 상세 가이드**

### 1. `kubectl drain` 실행 절차
1. 노드 확인
```bash 
kubectl get nodes
```
2. 드레이닝 실행 (DaemonSet 무시)
```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

**동작 원리**  
1. `NoSchedule` 테인트 자동 적용
2. 파드 제거 프로세스:
   - **Deployment/ReplicaSet**: 즉시 재생성
   - **StatefulSet**: 순서대로 재생성
   - **DaemonSet**: 유지 (--ignore-daemonsets 필요)
---

### 2. `kubectl cordon` 사용 가이드
```bash
kubectl cordon <node-name>
```

**주요 특징**  
- 새 파드 스케줄링 차단
- 기존 파드 유지
- **주의**: 노드 장애 시 자동 복구 안 됨

---

### 3. `kubectl uncordon` 복구 프로세스
1. 노드 상태 확인
```bash
kubectl get nodes <node-name>
```
2. 스케줄링 활성화
```bash
kubectl uncordon <node-name>
```

**⚠️ 주의사항**  
- 파드 자동 재배치 안 됨 → 수동 삭제 필요
kubectl delete pod <pod-name> --force --grace-period=0

---

## 🚀 **실전 시나리오: Node Maintenance**
1. 계획적 유지보수
```bash
kubectl drain node-01 --ignore-daemonsets
ssh node-01
apt-get update && apt-get upgrade -y
reboot
kubectl uncordon node-01
```
2. 긴급 장애 대응
```bash
kubectl cordon node-02
kubectl debug node/node-02 -it --image=busybox
```

