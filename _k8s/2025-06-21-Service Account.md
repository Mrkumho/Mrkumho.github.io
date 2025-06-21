## 1. 서비스 어카운트란?

- **서비스 어카운트**는 쿠버네티스 클러스터 내에서 애플리케이션이 API에 접근할 수 있도록 인증을 담당 함.
- **사용자 어카운트(User Account)**는 사람(관리자, 개발자 등)이 사용하고, 
  **서비스 어카운트**는 애플리케이션이 사용 가능.
- **예시:**  
  - Prometheus(모니터링 툴) → 쿠버네티스 API에서 메트릭 수집
  - Jenkins(빌드 툴) → 애플리케이션 배포

## 2. 서비스 어카운트 생성 및 사용

- **서비스 어카운트 생성**
```bash
kubectl create serviceaccount dashboard-sa
```

- **서비스 어카운트 목록 조회**
```bash
kubectl get serviceaccount
```

- **서비스 어카운트 생성 시**
- 서비스 어카운트 오브젝트 생성
- 자동으로 토큰 생성 및 시크릿(Secret) 오브젝트에 저장
- 시크릿 오브젝트는 서비스 어카운트에 연결됨
- **토큰 확인**
```bash
kubectl describe secret <시크릿이름>
```

- 토큰은 **Bearer Token**으로 사용 가능

## 3. 서비스 어카운트 토큰 사용

- **토큰을 이용한 API 인증**
- `curl` 등에서 헤더에 `Authorization: Bearer <토큰값>` 추가
- 예시:
  ```bash
  curl -H "Authorization: Bearer <토큰값>" https://<API서버주소>
  ```
- **애플리케이션(예: 대시보드)에 토큰 입력**
- 토큰을 복사해 애플리케이션 설정에 붙여넣으면 인증 완료

## 4. Pod 내 자동 토큰 마운트

- **Pod 생성 시**
- 별도 지정이 없으면 **default 서비스 어카운트**가 자동 할당
- default 서비스 어카운트의 토큰이 Pod에 자동 마운트
- 마운트 위치: `/var/run/secrets/kubernetes.io/serviceaccount`
- Pod 내에서 토큰 파일(`token`)을 읽어 API 접근 가능
- **권한**
- default 서비스 어카운트는 매우 제한적
- 필요시 별도 서비스 어카운트 지정 가능
  ```yaml
  spec:
    serviceAccountName: dashboard-sa
  ```
- Pod의 서비스 어카운트 변경은 삭제/재생성 필요 (Deployment는 자동 롤아웃)

## 5. 서비스 어카운트 토큰 자동 마운트 해제

- **Pod 정의에서 자동 마운트 해제**
```yaml
spec:
automountServiceAccountToken: false
```

## 6. 쿠버네티스 버전별 변화

### 6.1 쿠버네티스 1.22 이전
- **서비스 어카운트 생성 시**
- 토큰이 시크릿 오브젝트로 생성
- 토큰은 만료일 없음, audience(대상) 지정 불가
- Pod에 자동 마운트

### 6.2 쿠버네티스 1.22 이후
- **Token Request API 도입**
- 토큰은 audience(대상), 만료일, 오브젝트 바인딩 가능
- Pod 생성 시 토큰이 **projected volume**으로 자동 마운트
- 토큰은 Pod 수명에 따라 유효 (더 안전)

### 6.3 쿠버네티스 1.24 이후
- **서비스 어카운트 생성 시 시크릿 자동 생성 중단**
- 토큰이 필요하면 명시적으로 생성 필요
  ```bash
  kubectl create token <서비스어카운트이름>
  ```
- 생성된 토큰은 만료일 있음 (기본 1시간)
- 옵션으로 만료기간 조정 가능
- **기존 방식(시크릿 기반 토큰)은 권장하지 않음**
- 보안상 이유로 Token Request API 사용 권장

## 7. 기존 방식(시크릿 기반 토큰) 사용법

- **시크릿 오브젝트 직접 생성**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
  annotations:
    kubernetes.io/service-account.name: <서비스어카운트이름>
type: kubernetes.io/service-account-token  # 여기서 type 지정!
```
- **서비스 어카운트가 먼저 생성되어 있어야 함**
- **보안상 권장하지 않음** (토큰 만료 없음)

## 8. 요약

- **서비스 어카운트**: 애플리케이션 인증용
- **토큰**: API 접근 시 Bearer Token으로 사용
- **Pod 내 자동 마운트**: default 또는 지정 서비스 어카운트 토큰이 자동 마운트
- **쿠버네티스 1.22/1.24 이후**:  
- Token Request API로 더 안전한 토큰 발급
- 시크릿 기반 토큰은 권장하지 않음

---
