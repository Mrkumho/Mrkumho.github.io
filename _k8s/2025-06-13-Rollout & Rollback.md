## 1. Deployment란?

쿠버네티스에서 **Deployment**는 파드(Pod) 집합의 배포, 업데이트, 롤백을 관리하는 리소스.
애플리케이션 버전 업그레이드, 레플리카 수 조정, 라벨 변경 등 다양한 변경을 쉽게 관리할 수 있음.

---

## 2. Deployment YAML 예시
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  labels:
    app: webapp
spec:
  replicas: 5
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
        - name: webapp
          image: webapp:1.0
          ports:
            - containerPort: 80
```

---

## 3. Deployment 생성 및 상태 확인

- **Deployment 생성**
```bash
kubectl apply -f webapp-deployment.yaml
```

- **Deployment 목록 확인**
```bash
kubectl get deployments
```

- **롤아웃 상태 확인**
```bash
kubectl rollout status deployment/webapp
```
- **롤아웃 이력 확인**
```bash
kubectl rollout history deployment/webapp
```

---

## 4. Deployment 업데이트 방법

- **YAML 파일 수정 후 적용**
- `image: webapp:1.0` → `image: webapp:2.0`로 변경
(webapp-deployment.yaml)
```yaml
...
spec:
  containers:
    - name: webapp
    image: webapp:2.0 # 이미지 버전 업데이트
```

- 적용
  ```bash
  kubectl apply -f webapp-deployment.yaml
  ```
  
- **kubectl set image 명령어 사용**
```bash  
kubectl set image deployment/webapp webapp=webapp:2.0
```

- **주의**: 이 방식은 YAML 파일에 반영되지 않으므로, 이후 YAML 파일로 관리 시 주의 필요

---

## 5. Deployment 전략

쿠버네티스 Deployment는 **두 가지 업데이트 전략**을 지원.

### 5.1. Recreate (재생성)

- **설명**:  
- 기존 파드를 모두 삭제한 뒤, 새 파드를 생성.
- 업데이트 중에는 서비스가 중단.
- **YAML 예시**
```yaml  
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 5
  strategy:
    type: Recreate
...
```

### 5.2. RollingUpdate (롤링 업데이트, 기본값)

- **설명**:  
- 기존 파드를 하나씩 삭제하면서 새 파드를 하나씩 생성.
- 서비스 중단 없이 업데이트가 가능.
- **YAML 예시**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
...
```


---

## 6. 롤백 (Rollback)

- **롤백 명령어**
```bash
kubectl rollout undo deployment/webapp
```

- **특정 리비전으로 롤백**
```bash
kubectl rollout undo deployment/webapp --to-revision=2
```

- **롤백 후 레플리카셋 확인**
```bash
kubectl get replicasets
```

- 롤백 전: 새 레플리카셋에 파드가 있음
- 롤백 후: 이전 레플리카셋에 파드가 복구됨

---

## 7. 요약 명령어

| 명령어                                      | 설명                        |
|---------------------------------------------|-----------------------------|
| `kubectl apply -f <yaml>`                   | Deployment 생성/업데이트    |
| `kubectl get deployments`                   | Deployment 목록 확인        |
| `kubectl rollout status deployment/<name>`  | 롤아웃 상태 확인            |
| `kubectl rollout history deployment/<name>` | 롤아웃 이력 확인            |
| `kubectl rollout undo deployment/<name>`    | 롤백 실행                   |
| `kubectl set image deployment/<name>`       | 이미지 직접 업데이트         |

---

## 8. 실무 팁


---
