## 1. ConfigMap이란?
- **정의**: 
  - **ConfigMap**은 쿠버네티스에서 애플리케이션의 환경설정 데이터(키-값 쌍)를 중앙에서 관리하는 리소스.
- **용도**:  
  - 환경변수, 설정 파일 등 다양한 형태로 파드에 주입 가능.
  - 여러 파드에서 동일한 설정을 재사용할 수 있어 관리가 편리.

---

## 2. ConfigMap 생성 방법

### 2.1. 명령형(Imperative) 방식 (kubectl 명령어)

```bash
kubectl create configmap <config-name> --from-literal=<key>=<value>
kubectl create configmap app-config --from-literal=app_color=blue --from-literal=log_level=debug
```

- **설명**:  
  - `--from-literal` 옵션으로 직접 키-값 쌍을 지정
  - 여러 값을 추가하려면 `--from-literal`을 여러 번 사용

```bash
kubectl create configmap <config-name> --from-file=<path-to-file>
kubectl create configmap app-config --from-file=/etc/config/app_color  --from-file=/etc/config/log_level 
```

- **설명**:  
  - `--from-file` 옵션으로 직접 Path to file 을 지정
  - 여러 값을 추가하려면 `--from-file`을 여러 번 사용
---

### 2.2. 선언형(Declarative) 방식 (YAML 파일)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app_color: blue
  log_level: debug
```

- **설명**:  
  - `data` 필드에 키-값 쌍을 지정
  - 여러 ConfigMap을 만들어 각각의 용도에 맞게 관리 가능

---

## 3. ConfigMap 조회 및 확인
```bash
kubectl get configmaps
kubectl describe configmap app-config
```

- **설명**:  
  - ConfigMap 목록 및 상세 정보 확인 가능
  - `data` 섹션에서 설정값 확인

---

## 4. ConfigMap을 파드에 주입하는 방법

### 4.1. 환경변수로 주입
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: webapp
      image: my-webapp
      envFrom:
        - configMapRef:
            name: app-config
```

- **설명**:  
  - `envFrom`을 사용해 ConfigMap의 모든 키-값 쌍을 환경변수로 주입
  - 파드 내부에서 `app_color`, `log_level` 등으로 접근 가능

---

### 4.2. 개별 환경변수로 주입
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: webapp
      image: my-webapp
      env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: app_color
```

- **설명**:  
  - `env`에서 `valueFrom`을 사용해 ConfigMap의 특정 키만 환경변수로 주입

---

### 4.3. 볼륨으로 주입 (설정 파일 형태)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: webapp
      image: my-webapp
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

- **설명**:  
  - ConfigMap의 키-값 쌍이 `/etc/config/app_color`, `/etc/config/log_level` 등 파일로 마운트됨

---

## 5. 요약

| 방법                | 예시/설명                                      | 용도                        |
|---------------------|-----------------------------------------------|-----------------------------|
| 명령형 생성         | `kubectl create configmap ...`                | 빠르게 ConfigMap 생성        |
| 선언형 생성         | YAML 파일 작성                                | 복잡한 설정, 버전 관리에 적합 |
| 환경변수 주입       | `envFrom`, `valueFrom`                        | 환경변수로 설정값 사용        |
| 볼륨 주입           | `volumes`, `volumeMounts`                     | 설정 파일로 사용              |

---

## 6. 실습 팁

- **ConfigMap 이름을 의미 있게 지정**하여 관리 편의성 향상
- **여러 ConfigMap을 만들어 용도별로 분리** 관리
- **설정값이 변경되면 파드를 재시작해야 반영**됨(볼륨 마운트 시 일부는 자동 반영)

---

> **핵심 요약**  
> ConfigMap을 사용하면 환경설정 데이터를 파드 정의 파일과 분리하여 중앙에서 관리할 수 있음.  
> 환경변수, 설정 파일 등 다양한 형태로 파드에 주입할 수 있어 유지보수와 배포가 쉬워짐.
