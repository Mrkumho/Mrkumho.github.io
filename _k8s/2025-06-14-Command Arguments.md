
## 1. Command와 Arguments란?

쿠버네티스 파드에서 컨테이너가 실행될 때 **실행 명령(command)**과 **인자(arguments)**를 지정 가능.
이 설정은 Docker 이미지의 `ENTRYPOINT`와 `CMD`를 오버라이드(덮어쓰기)하는 역할을 함.

---

## 2. Docker 이미지와 쿠버네티스 파드의 관계

- **Docker 이미지**는 `ENTRYPOINT`(실행 명령)와 `CMD`(기본 인자)를 정의.
- **쿠버네티스 파드**에서는 이 두 가지를 각각 오버라이드할 수 있음.
  - `command`: Docker의 `ENTRYPOINT`를 오버라이드
  - `args`: Docker의 `CMD`를 오버라이드

---

## 3. 예시: ubuntu-sleeper 이미지

### 3.1. 기본 동작
```bash
docker run ubuntu-sleeper
```

- **설명**: 컨테이너가 5초간 대기 후 종료 (Dockerfile의 `CMD`에 기본값 `["5"]`가 설정되어 있음)

### 3.2. 인자 오버라이드
```bash
docker run ubuntu-sleeper 10
```


- **설명**: 컨테이너가 10초간 대기
- **쿠버네티스 파드**에서는 `args`에 `["10"]`을 지정하면 동일하게 동작

---

## 4. 쿠버네티스 파드 정의 예시

### 4.1. 인자(args) 오버라이드
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sleeper
spec:
  containers:
    - name: sleeper
    image: ubuntu-sleeper
    args: ["10"] # CMD를 오버라이드, 10초 대기
```

### 4.2. 명령(command) 오버라이드

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sleeper
spec:
  containers:
    - name: sleeper
    image: ubuntu-sleeper
    command: ["sleep2.0"] # ENTRYPOINT를 오버라이드
    args: ["20"] # 인자로 20초 지정
```

---

## 5. 정리

| Dockerfile      | 쿠버네티스 파드 | 설명                        |
|-----------------|----------------|-----------------------------|
| ENTRYPOINT      | command        | 실행 명령 오버라이드         |
| CMD             | args           | 기본 인자 오버라이드         |

---

## 6. 실습 및 팁

- **파드 정의 파일에서 `args`를 사용해 기본 인자를 오버라이드**
- **`command`를 사용해 실행 명령 자체를 오버라이드**
- **둘 다 배열 형태로 지정**
- **이를 통해 다양한 커맨드와 인자를 파드에 적용 가능**

---

> **핵심 요약**  
> 쿠버네티스 파드에서 `command`와 `args`를 사용하면 Docker 이미지의 실행 명령과 인자를 자유롭게 오버라이드할 수 있음.
