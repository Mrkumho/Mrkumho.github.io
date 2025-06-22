# Kubernetes Persistent Volume과 Persistent Volume Claim

## 1. 핵심 개념

- **Persistent Volume(PV):**  
  관리자가 미리 준비하는 실제 스토리지 리소스 (예: NFS, EBS, GPD 등)
- **Persistent Volume Claim(PVC):**  
  사용자가 애플리케이션에서 사용할 스토리지를 요청하는 객체
- **바인딩(Binding):**  
  Kubernetes가 PVC와 PV를 1:1로 연결하는 과정

## 2. PVC 생성 및 바인딩 과정

1. **관리자는 PV를 생성**  
   - 예: 1GB 용량, ReadWriteOnce 액세스 모드
2. **사용자는 PVC를 생성**  
   - 예: 500MB 요청, ReadWriteOnce 액세스 모드
3. **Kubernetes가 조건에 맞는 PV를 찾아 PVC와 바인딩**  
   - 용량, 액세스 모드, 스토리지 클래스 등 조건 일치 확인
   - 조건에 맞는 PV가 여러 개일 경우, 레이블/셀렉터로 특정 PV 지정 가능
   - 조건에 맞는 PV가 없으면 PVC는 Pending 상태로 대기
4. **PV가 새로 생성되면 자동으로 바인딩**

## 3. PVC 삭제 시 PV 처리 방식

- **Retain(기본값):**  
  PV와 데이터를 보존, 수동 삭제 필요  
  → 다른 PVC에서 재사용 불가
- **Delete:**  
  PVC 삭제 시 PV도 자동 삭제  
  → 스토리지 공간 확보
- **Recycle(Deprecated):**  
  데이터 삭제 후 PV 재사용 (Kubernetes 1.15+에서는 권장하지 않음)
  
| 정책    | 동작                                      | 사용 사례           |
|---------|-------------------------------------------|--------------------|
| Retain  | PV 유지, 데이터 보존 (수동 삭제 필요)       | 중요 데이터 백업 용도 |
| Delete  | PV 자동 삭제, 스토리지 공간 확보            | 테스트 환경         |
| Recycle | 데이터 삭제 후 재사용 (K8s 1.15+ Deprecated) | 구형 버전           |

## 4. 실습 요약

### 4.1 PV 생성 예시

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
  labels:
    type: local  # PVC에서 셀렉터로 선택 가능
spec:
  capacity:
    storage: 1Gi  # PVC 요청(500Mi)보다 큰 용량
  accessModes:
    - ReadWriteOnce  # PVC와 동일한 액세스 모드
  persistentVolumeReclaimPolicy: Retain  # PVC 삭제 시 PV 보존
  storageClassName: ""  # 명시적으로 빈 문자열 지정 (기본 클래스)
  hostPath:
    path: /data/pv  # 실제 데이터 저장 경로
```
#### 참고 :
- ReadOnlyMany (ROX): 여러 노드에서 읽기 전용으로 마운트 가능
- ReadWriteMany (RWX): 여러 노드에서 읽기/쓰기로 마운트 가능
- ReadWriteOncePod (RWOP): 쿠버네티스 1.22+에서 지원, 한 파드에서만 읽기/쓰기로 마운트 가능

예시: AWS EBS, GCP PD, hostPath 등은 일반적으로 RWO만 지원.
### 4.2 PVC 생성 예시

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  selector:
    matchLabels:
      type: local
```

### 4.3 Pod 생성 예시
```yaml
# Pod 1 (읽기 전용 마운트)
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: "/data"
      name: data-volume
      readOnly: true
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: shared-pvc
```

#### 참고 : 
PV-PVC 에 Label 및 Selector 가 없어도 용량, 액세스 모드, 스토리지 클래스, 볼륨 모드가 일치하면 
쿠버네티스가 자동으로 PV-PVC를 바인딩 가능. 

### 4.4 특정 PV 지정
```bash
# PV에 레이블 추가
kubectl label pv my-pv type=fast
```

```yaml
# PVC에서 셀렉터 지정
spec:
  selector:
    matchLabels:
      type: fast
```
## 5. 문제 해결 체크리스트

| 문제                      | 확인 사항                                                                 |
|---------------------------|---------------------------------------------------------------------------|
| **PVC가 Pending 상태**    | 1. 충분한 용량의 PV 존재 여부<br>2. 액세스 모드 일치 여부<br>3. 스토리지 클래스 일치 여부 |
| **PVC 삭제 후 PV가 삭제되지 않음** | PV의 `persistentVolumeReclaimPolicy`가 `Delete`로 설정되었는지 확인       |
| **Pod에서 PVC 사용 불가** | 1. PVC 이름 정확성 확인<br>2. 동일 네임스페이스 사용 여부 확인            |

## 6. 주의사항

- **PVC는 항상 하나의 PV에만 바인딩** (1:1 관계)
- **바인딩 후 남은 용량은 다른 PVC에서 사용 불가**
- **PV의 reclaimPolicy에 따라 삭제/보존 여부 결정**
- **조건에 맞는 PV가 없으면 PVC는 Pending 상태**

## 7. 명령어

- PVC 생성
```bash
kubectl apply -f pvc.yaml
```
- PVC 상태 확인
```bash
kubectl get pvc
```
- PV 상태 확인
```bash
kubectl get pv
```
- PVC 삭제
```bash
kubectl delete pvc my-claim
```
