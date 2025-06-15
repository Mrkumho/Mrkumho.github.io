## 1. Sidecar & Co-located Containers

쿠버네티스에서 **파드(Pod)**는 가장 작은 배포 단위로,  
여러 컨테이너가 **같은 네트워크 네임스페이스와 볼륨(저장소)**을 공유하며 함께 실행될 수 있음.
이 구조에서 자주 사용되는 두 가지 패턴이 있음:  
**사이드카(Sidecar) 컨테이너**와 **코로케이티드(Co-located) 컨테이너**

---

## 2. 사이드카 컨테이너

### 2.1 정의

- **사이드카 컨테이너**는 메인(주) 애플리케이션 컨테이너와 같은 파드에서 함께 실행되는 보조 컨테이너.
- 메인 컨테이너의 기능을 확장하거나 보조 역할(예: 로깅, 모니터링, 프록시, 데이터 동기화 등)을 담당.
- 메인 컨테이너의 코드를 수정하지 않고도 기능을 추가할 수 있음.

### 2.2 쿠버네티스 v1.29 이후

- 사이드카 컨테이너는 **initContainers** 필드에 `restartPolicy: Always`로 정의할 수 있음.
- 이렇게 하면 사이드카 컨테이너가 메인 컨테이너보다 먼저 시작되고, 파드가 삭제될 때까지 계속 실행됨.
- 예기치 않게 종료되면 독립적으로 재시작할 수도 있음.

### 2.3 예시: 로깅 사이드카

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-logging-sidecar
spec:
  volumes:
    - name: shared-logs
      emptyDir: {}
  initContainers:
    - name: log-shipper
      image: alpine:latest
      command: ['sh', '-c', 'tail -F /opt/logs.txt']
      volumeMounts:
        - name: shared-logs
          mountPath: /opt
      restartPolicy: Always
  containers:
    - name: main-app
      image: alpine:latest
      command: ['sh', '-c', 'while true; do echo "logging" >> /opt/logs.txt; sleep 1; done']
      volumeMounts:
        - name: shared-logs
          mountPath: /opt
```

- **설명**:
  - 메인 앱이 로그를 파일에 기록하면,
  - 사이드카 컨테이너가 그 로그 파일을 실시간으로 읽어서 처리.
  - 두 컨테이너는 **shared-logs** 볼륨을 통해 데이터를 공유 함.

---

## 3. 코로케이티드(Co-located) 컨테이너

### 3.1 정의

- **코로케이티드 컨테이너**는 한 파드 안에 여러 컨테이너가 함께 실행되며,
  **동등한 역할**을 하거나 서로 협력하여 하나의 목적을 달성하는 구조입니다.
- 기존에 사이드카라고 불렸던 컨테이너도,
  쿠버네티스 v1.29 이후부터는 **코로케이티드**라는 용어로 구분합니다.
- 각 컨테이너는 독립적으로 시작/종료할 수 있고,
  **정해진 시작 순서가 없습니다.**

### 3.2 예시: 웹 서버와 콘텐츠 생성기
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server-with-helper
spec:
  volumes:
    - name: shared-content
      emptyDir: {}
  containers:
    - name: content-generator
      image: busybox
      command: ['sh', '-c', 'echo "Hello from Helper!" > /output/index.html && sleep 3600']
      volumeMounts:
        - name: shared-content
          mountPath: /output
    - name: web-server
      image: nginx
      volumeMounts:
        - name: shared-content
          mountPath: /usr/share/nginx/html
```

- **설명**:
  - content-generator 컨테이너가 HTML 파일을 생성하고,
  - web-server 컨테이너가 그 파일을 웹 서버로 제공 합.
  - 두 컨테이너가 **shared-content** 볼륨을 통해 데이터를 공유 함.

---

## 4. 사이드카 vs 코로케이티드 컨테이너 비교

| 특징              | 사이드카 컨테이너                | 코로케이티드 컨테이너            |
|-------------------|----------------------------------|----------------------------------|
| 주요 역할         | 메인 앱 기능 확장/보조           | 동등하게 협력하여 목적 달성      |
| 시작 순서         | 메인 컨테이너보다 먼저 시작(최신) | 시작 순서 없음                   |
| 생명주기          | 메인 컨테이너와 함께/독립 재시작  | 독립적 생명주기                  |
| 대표적 사용 예시  | 로깅, 모니터링, 프록시, 동기화    | 멀티 프로세스 앱, 헬퍼 컨테이너  |
| 구현 방식(최신)   | initContainers + restartPolicy   | containers 필드                  |

---

## 5. 결론

- **사이드카 컨테이너**는 메인 앱의 기능을 확장하거나 보조 역할을 담당.
- **코로케이티드 컨테이너**는 여러 컨테이너가 동등하게 협력하여 하나의 목적을 달성.
- **쿠버네티스 v1.29**부터는 사이드카 컨테이너를 initContainers로 정의할 수 있어,
  시작 순서와 생명주기 관리가 더 쉬워짐.
- **어떤 패턴을 사용할지는 애플리케이션의 구조와 요구사항에 따라 결정**하면 됨.

---

> **핵심 요약**  
> 사이드카 컨테이너는 메인 앱을 보조하고,  
> 코로케이티드 컨테이너는 여러 컨테이너가 동등하게 협력함.  
> 쿠버네티스 v1.29 이후에는 사이드카 컨테이너를 initContainers로 정의해  
> 시작 순서와 생명주기 관리가 더 간결해짐.
