
## 1. Docker 컨테이너의 보안 기본 개념

- **컨테이너는 가상머신(VM)과 달리 완전히 독립된 환경이 아님**
  - 컨테이너와 호스트는 **동일한 커널**을 공유
  - **네임스페이스(namespaces)**를 통해 프로세스, 네트워크, 파일시스템 등이 격리됨
  - 컨테이너 내에서 프로세스를 조회하면, 해당 컨테이너의 프로세스만 보임
  - 호스트에서는 모든 컨테이너의 프로세스를 볼 수 있음 (PID가 다름)

- **사용자 격리**
  - Docker 호스트에는 root 및 여러 non-root 사용자가 있음
  - **기본적으로 컨테이너의 프로세스는 root로 실행됨**
  - 컨테이너 내에서 root는 호스트의 root와 동일하지 않음 (격리됨)
  - 컨테이너 내 root는 호스트의 root 권한을 모두 갖지 않음

- **사용자 지정**
  - `docker run --user <UID>` 옵션으로 컨테이너 내 사용자 지정 가능
  - Dockerfile에서 `USER` 지시어로 이미지 빌드 시 사용자 지정 가능
    ```
    USER 1000
    ```
  - 컨테이너 실행 시 별도 지정 없으면 Dockerfile의 USER로 실행됨

- **Linux Capabilities**
  - root는 시스템에서 모든 권한을 가짐
  - Docker는 컨테이너의 root에게 **제한된 권한**(capabilities)만 부여
  - 기본적으로 컨테이너의 root는 호스트를 재부팅하거나, 네트워크 포트를 조작하는 등 위험한 작업 불가
  - 필요시 `--cap-add`, `--cap-drop` 옵션으로 권한 추가/제거 가능
    ```
    docker run --cap-add NET_ADMIN ...
    docker run --cap-drop ALL ...
    ```
  - 모든 권한을 주고 싶으면 `--privileged` 플래그 사용 (권장하지 않음)

---

## 2. Kubernetes Security Context

- **컨테이너는 Pod 단위로 관리됨**
  - Pod 내 여러 컨테이너가 존재할 수 있음
  - 보안 설정은 Pod 또는 컨테이너 단위로 지정 가능
    - **Pod 단위 설정**: 모든 컨테이너에 적용
    - **컨테이너 단위 설정**: 해당 컨테이너에만 적용 (Pod 설정을 오버라이드)

- **Security Context 설정 예시**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:  # Pod 단위 설정
    runAsUser: 1000
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]
    securityContext:  # 컨테이너 단위 설정 (Pod 설정 오버라이드)
      runAsUser: 2000
      capabilities:
        add: ["NET_ADMIN"]
```

- **설정 우선순위**
- 컨테이너 단위 설정이 Pod 단위 설정보다 우선함

---

## 3. 핵심 요약

- **Docker 컨테이너는 네임스페이스와 capabilities로 격리 및 권한 제한**
- **기본적으로 컨테이너의 root는 호스트의 root와 다름**
- **사용자 및 capabilities는 Docker run 옵션이나 Dockerfile, Kubernetes Security Context로 지정 가능**
- **Kubernetes에서는 Pod 또는 컨테이너 단위로 Security Context 설정 가능**

---

## 4. 실무 참고
```bash
kubectl exec -it ubuntu-sleeper -- sh
```
```bash
whoami
```
