## 1. 쿠버네티스 스케줄러란?

- **역할**: 파드를 노드에 배치하는 역할
- **동작 흐름**:
  1. **스케줄링 큐**: 파드가 스케줄링을 위해 대기
  2. **우선순위 정렬**: 파드의 우선순위(PriorityClass)에 따라 큐에서 정렬
  3. **필터링**: 파드의 리소스 요구사항에 맞는 노드만 남김
  4. **스코어링**: 남은 노드들에 점수(score) 부여
  5. **바인딩**: 가장 점수가 높은 노드에 파드 배치

---

## 2. 스케줄링 단계별 플러그인

| 단계          | 주요 플러그인 예시                | 역할 설명                                      |
|---------------|----------------------------------|-----------------------------------------------|
| 큐 정렬       | PrioritySort                     | 파드의 우선순위에 따라 큐에서 정렬             |
| 필터링        | NodeResourcesFit, NodeName       | 리소스 부족, 노드 이름 불일치 등 필터링        |
| 스코어링      | NodeResourcesFit, ImageLocality  | 노드별 점수 부여(리소스, 이미지 미리 존재 등)  |
| 바인딩        | DefaultBinder                    | 파드를 노드에 바인딩                           |

- **플러그인 확장성**: 각 단계(extension point)에 커스텀 플러그인 추가 가능
- **Pre/Post 단계**: 필터링, 스코어링, 바인딩 전후에도 플러그인 추가 가능

---

## 3. 스케줄러 프로파일이란?

- **개념**: 한 개의 스케줄러 바이너리에서 여러 개의 스케줄러처럼 동작할 수 있도록 설정
- **장점**:
  - 여러 개의 스케줄러 바이너리를 실행할 필요 없음
  - 프로파일마다 플러그인 활성화/비활성화 가능
  - 각 파드가 원하는 프로파일 선택 가능(`schedulerName` 지정)
- **구성 방법**: YAML 설정 파일에서 여러 프로파일 정의

---

## 4. 스케줄러 프로파일 예시
```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:

schedulerName: default-scheduler

schedulerName: my-scheduler-2
plugins:
  score:
    disabled:
    - name: TaintToleration
    enabled:
    - name: MyCustomPluginA
    - name: MyCustomPluginB
  
schedulerName: my-scheduler-3
plugins:
  preScore:
    disabled:
    - name: ''
  score:
    disabled:
    - name: ''
```

- **설명**:
  - `default-scheduler`: 기본 플러그인 사용
  - `my-scheduler-2`: TaintToleration 플러그인 비활성화, 커스텀 플러그인 활성화
  - `my-scheduler-3`: preScore, score 플러그인 모두 비활성화

---

## 5. 파드에 스케줄러 프로파일 적용
```yaml
apiVersion: v1
kind: Pod
metadata:
name: my-pod
spec:
  schedulerName: my-scheduler-2 # 원하는 프로파일 지정
  containers:
    - name: nginx
      image: nginx
```

- **설명**: 파드가 `my-scheduler-2` 프로파일로 스케줄링됨

---

## 6. 스케줄러 프로파일의 장점

- **운영 효율**: 여러 스케줄러 바이너리 대신 한 개의 바이너리로 관리
- **경합 방지**: 여러 스케줄러가 동시에 같은 노드에 파드를 배치하는 문제(race condition) 방지
- **워크로드별 맞춤형 스케줄링**: 서비스, 배치 등 워크로드별로 최적의 플러그인 조합 적용

---

## 7. 요약

- **스케줄러 프로파일**: 한 개의 스케줄러에서 여러 개의 스케줄러처럼 동작하도록 설정
- **플러그인**: 스케줄링 단계별로 커스텀 동작 추가 가능
- **파드별 스케줄러 지정**: 파드의 `schedulerName`으로 원하는 프로파일 선택
- **운영 효율, 경합 방지, 워크로드별 맞춤형 스케줄링**이 가능

---

> **실무 팁**  
```bash
kubectl create configmap my-scheduler-config --from-file=/root/my-scheduler-config.yaml --namspace=kube-system
```
