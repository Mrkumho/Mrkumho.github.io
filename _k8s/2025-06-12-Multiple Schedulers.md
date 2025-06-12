
# 쿠버네티스에서 여러 스케줄러 사용하기

## 1. 기본 개념

- **쿠버네티스 기본 스케줄러**  
  - 파드를 노드에 자동으로 배치하는 역할  
  - 리소스, 테인트 등 다양한 조건을 고려하여 스케줄링
- **커스텀 스케줄러 필요성**  
  - 특정 애플리케이션에 맞는 추가 조건이나 로직이 필요할 때  
  - 예: GPU 노드만 선택, 특정 라벨이 있는 노드만 선택 등

## 2. 여러 스케줄러 배포 방법

### 2.1. 스케줄러 구성 파일 만들기

각 스케줄러는 고유한 이름과 설정 파일을 가져야 함.

```yaml
# 예: my-scheduler-config.yaml (ConfigMap)
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
    leaderElection:
      leaderElect: false
```
- **schedulerName**: 스케줄러의 고유 이름 (중복 불가)
- **leaderElection**: 리더 선언(고가용성) 여부 (단일 스케줄러면 false)

### 2.2. ConfigMap으로 설정 적용

```bash
kubectl create configmap my-scheduler-config -n kube-system --from-file=my-scheduler-config.yaml
```

### 2.3. 스케줄러 Deployment 배포

```yaml
# 예: my-scheduler-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-scheduler
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      name: my-scheduler
  template:
    metadata:
      labels:
        name: my-scheduler
    spec:
      serviceAccountName: my-scheduler
      containers:
      - command:
        - /usr/local/bin/kube-scheduler
        - --leader-elect=false
        - --config=/etc/kubernetes/scheduler-config.yaml
        image: k8s.gcr.io/kube-scheduler:v1.25.12
        name: my-scheduler
        volumeMounts:
        - mountPath: /etc/kubernetes/scheduler-config.yaml
          name: my-scheduler-config
          subPath: my-scheduler-config.yaml
      volumes:
      - name: my-scheduler-config
        configMap:
          name: my-scheduler-config
```

- **serviceAccountName**: API 접근 권한을 위한 서비스 계정
- **volumes**: 설정 파일을 컨테이너에 마운트

### 2.4. 스케줄러 배포

```bash
kubectl apply -f my-scheduler-deployment.yaml
```

## 3. 파드에 스케줄러 지정하기

파드나 디플로이먼트에서 `schedulerName` 필드로 사용할 스케줄러를 지정 가능.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  schedulerName: my-scheduler   # 커스텀 스케줄러 지정
  containers:
  - name: nginx
    image: nginx
```

## 4. 관련 예시

### 예시 1: GPU 전용 스케줄러

- **스케줄러 이름**: my-gpu-scheduler
- **파드에 지정**: `schedulerName: my-gpu-scheduler`
- **설정**: GPU 노드만 선택하도록 조건 추가

```yaml
# gpu-scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-gpu-scheduler
    plugins:
      - name: NodeResourcesFit
        args:
          ignoredResources: ["nvidia.com/gpu"]
```

### 예시 2: 기본 스케줄러와 병행 사용

- **기본 스케줄러**: kube-scheduler
- **커스텀 스케줄러**: my-scheduler
- **파드마다 원하는 스케줄러 지정 가능**

## 5. 확인 및 트러블슈팅

- **스케줄러 상태 확인**
  ```bash
  kubectl get pods -n kube-system
  ```
- **파드 스케줄링 이벤트 확인**
  ```bash
  kubectl get events -o wide
  ```
  - `Source` 컬럼에서 스케줄러 이름 확인 가능

## 6. 요약

- **여러 스케줄러를 동시에 운영**할 수 있음
- **각 스케줄러는 고유한 이름과 설정 파일**을 가져야 함
- **파드/디플로이먼트에서 `schedulerName`으로 원하는 스케줄러 지정**
- **특정 워크로드에 맞는 커스텀 스케줄링 로직 구현 가능**

> **실무 팁**  
> - GPU, 고성능 스토리지 등 특수 하드웨어가 필요한 워크로드에 커스텀 스케줄러를 활용.
> - 스케줄러 배포 시 권한(serviceAccount, ClusterRoleBinding) 설정 확인 필수.
```
