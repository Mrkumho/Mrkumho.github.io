# Kubernetes에서 프라이빗 도커 레지스트리 이미지 사용 가이드

## 1. 도커 레지스트리 인증 정보 시크릿 생성

### 1.1 kubectl 명령어로 시크릿 생성
```bash
kubectl create secret docker-registry regcred \
--docker-server=<레지스트리주소> \
--docker-username=<사용자명> \
--docker-password=<비밀번호> \
--docker-email=<이메일>
```

### 1.2 YAML 파일로 시크릿 생성
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: regcred
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <Base64_인코딩_자격증명>
```

> **Base64 인코딩 방법 예시:**  
> ```bash
> echo -n '{"auths":{"<레지스트리주소>":{"username":"<사용자명>","password":"<비밀번호>","email":"<이메일>","auth":"<Base64_사용자명:비밀번호>"}}}' | base64
> ```

---

## 2. 파드에서 시크릿 사용 방법
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-app
spec:
  containers:
  - name: app
    image: <프라이빗-이미지-주소>
  imagePullSecrets:
  - name: regcred  # 생성한 시크릿 이름
```

---

## 3. 동작 원리

1. **워커 노드의 kubelet**이 파드를 생성할 때 `imagePullSecrets`를 확인.
2. **regcred 시크릿**에서 도커 자격증명을 추출.
3. **프라이빗 레지스트리**에 인증 정보와 함께 이미지 요청.
4. 인증에 성공하면 이미지를 받아와 컨테이너를 생성.

---

## 4. 주요 개념 정리

| 용어                  | 설명                                                |
|-----------------------|-----------------------------------------------------|
| docker-registry 시크릿 | 도커 레지스트리 인증을 위한 특수 시크릿 타입         |
| imagePullSecrets      | 파드에서 프라이빗 이미지 풀을 위한 시크릿 지정 필드  |
| Kubelet               | 워커 노드에서 파드 생성을 담당하는 에이전트          |
| 프라이빗 레지스트리    | 인증이 필요한 도커 이미지 저장소                    |

---

## 5. 실제 예시: GitHub Container Registry 사용

### 5.1 GitHub 개인 접근 토큰 생성

- `read:packages` 권한이 있는 토큰을 생성합니다.

### 5.2 시크릿 생성
```bash
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=<GitHub사용자명> \
  --docker-password=<토큰값> \
  --docker-email=<GitHub이메일>
```

### 5.3 파드에서 사용
```yaml
spec:
  containers:
  - image: ghcr.io/<사용자명>/<이미지명>:latest
  imagePullSecrets:
  - name: ghcr-secret
```

---

## 6. 문제 해결

1. **시크릿 이름 확인**  
   `imagePullSecrets.name`과 실제 시크릿 이름이 일치해야 함.
2. **레지스트리 주소 확인**  
   `docker-server`에는 프로토콜(`https://`)을 포함하지 않음.
3. **시크릿 타입 확인**  
   `type: kubernetes.io/dockerconfigjson`이 지정되어야 함.
4. **네임스페이스 확인**  
   시크릿과 파드가 같은 네임스페이스에 있어야 함.

> **명령어:**  
> `kubectl describe pod <파드명>`으로 이벤트 확인  
> **에러 예시:**  
> `Failed to pull image...: unauthorized`

---

## 7. 보안 모범 사례

1. **최소 권한 원칙**  
   레지스트리 토큰에 `read-only` 권한만 부여.
2. **네임스페이스 분리**  
   시크릿을 필요한 네임스페이스에만 생성.
3. **정기적 토큰 갱신**  
   장기 유효 토큰 대신 단기 토큰을 사용.
4. **RBAC 제한**  
   시크릿 접근 권한이 있는 사용자를 제한.

### RBAC 예시: dev-team만 시크릿 접근 허용
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: secret-access
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["regcred"]
  verbs: ["get"]
```
