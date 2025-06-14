# 쿠버네티스 파드에서 환경 변수(Environment Variable) 사용법

---

## 1. 환경 변수란?

- **정의**:  
  - **환경 변수**는 프로그램이 실행되는 환경(컨테이너, 서버 등)에서 사용할 수 있는 변수.
  - 프로그램은 이 변수를 읽어서 동작을 변경하거나 설정값을 가져올 수 있음.
- **예시**:  
  - `DB_HOST=db.example.com`
  - `LOG_LEVEL=debug`
  - `APP_PORT=8080`
- **용도**:  
  - 데이터베이스 연결 정보, 로그 레벨, 포트 번호 등 **설정값을 코드 외부에서 관리**할 때 사용.
  - **코드를 수정하지 않고도 동작을 바꿀 수 있어** 유지보수와 배포가 쉬움.

---

## 2. Docker에서 환경 변수 설정

- **명령어 예시**:
```bash
docker run -e "DB_HOST=db.example.com" -e "LOG_LEVEL=debug" my-app
```

- **설명**:  
- `-e` 옵션으로 환경 변수를 직접 지정할 수 있습니다.
- 컨테이너 내부에서 `DB_HOST`, `LOG_LEVEL` 변수를 사용할 수 있습니다.

---

## 3. 쿠버네티스 파드에서 환경 변수 설정

- **YAML 예시**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: my-app
      image: my-app
      env:
        - name: DB_HOST
          value: "db.example.com"
        - name: LOG_LEVEL
          value: "debug"
```

- **설명**:  
- `env`는 배열로, 각 환경 변수는 `name`과 `value`로 정의.
- 파드가 생성되면, 컨테이너 내부에서 `DB_HOST`, `LOG_LEVEL` 변수를 사용할 수 있음.
- **Docker와의 차이**:  
- Docker는 `-e` 옵션으로 명령어에서 지정
- 쿠버네티스는 YAML 파일의 `env` 필드로 지정

---

## 4. 환경 변수의 활용 예시

- **프로그램 코드에서 환경 변수 읽기**
- **Python 예시**:
  ```
  import os
  db_host = os.getenv("DB_HOST")
  log_level = os.getenv("LOG_LEVEL")
  ```
- **Node.js 예시**:
  ```
  const dbHost = process.env.DB_HOST;
  const logLevel = process.env.LOG_LEVEL;
  ```

---

## 5. 요약

- **환경 변수**는 프로그램이 실행되는 환경에서 사용하는 설정값.
- **Docker**에서는 `-e` 옵션으로, **쿠버네티스**에서는 `env` 필드로 지정.
- **코드를 수정하지 않고도 동작을 바꿀 수 있어** 유지보수와 배포가 쉬움.

---

> **핵심 요약**  
> 환경 변수는 애플리케이션의 설정값을 코드 외부에서 관리하는 표준 방법.  
> Docker의 `-e` 옵션과 쿠버네티스의 `env` 필드는 같은 역할을 함.  
> 환경 변수를 활용하면, 코드를 수정하지 않고도 애플리케이션의 동작을 쉽게 바꿀 수 있음.
