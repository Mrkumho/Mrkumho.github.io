## 1. Static Pod란?

- **Static Pod**는 쿠버네티스 클러스터의 컨트롤 플레인(API 서버, 스케줄러, etcd 등) 없이도 kubelet이 직접 관리하는 파드.
- **kubelet**은 지정된 디렉토리(예: `/etc/kubernetes/manifests`)에 있는 파드 정의 파일을 주기적으로 읽어서 해당 파드를 생성·관리.
- **프로세스가 죽으면 kubelet이 자동으로 재시작**하며, 파일을 수정하거나 삭제하면 파드도 자동으로 갱신되거나 삭제됨.

## 2. Static Pod의 동작 방식

- **kubelet은 파드 정의 파일을 주기적으로 감시**하며, 파일이 변경되면 파드를 재생성.
- **파일을 삭제하면 파드도 자동으로 삭제.**
- **이 방식으로는 파드만 생성 가능**하며, ReplicaSet, Deployment, Service 등은 생성할 수 없습니다.
- **Static Pod는 kubelet만으로 동작**하므로, API 서버, 스케줄러, etcd 등 다른 쿠버네티스 컴포넌트가 없어도 됨.

## 3. 디렉토리 설정 방법

- **pod-manifest-path 옵션**  
  - kubelet을 실행할 때 `--pod-manifest-path=/etc/kubernetes/manifests`처럼 디렉토리 경로를 지정.
- **config 파일에서 설정**  
  - kubelet의 `--config` 옵션으로 설정 파일을 지정하고, 해당 파일에 `staticPodPath`를 정의할 수도 있음.
- **kubeadm 등으로 클러스터를 구성한 경우**  
  - 보통 `/etc/kubernetes/manifests` 디렉토리를 사용합니다.

## 4. Static Pod와 일반 파드의 차이

- **클러스터에 속하지 않은 노드**  
  - kubelet이 직접 파드 정의 파일을 읽어서 파드를 생성·관리.
  - kubectl 명령은 사용할 수 없으며, 파드 상태는 `docker ps` 등으로 확인.
- **클러스터에 속한 노드**  
  - kubelet은 API 서버의 요청(동적 파드)과 디렉토리의 정의 파일(Static Pod) 모두를 처리할 수 있움.
  - Static Pod는 API 서버에 **읽기 전용 미러 객체**로 등록되어, `kubectl get pods`로 확인할 수 있음.
  - 단, **Static Pod는 API 서버에서 수정/삭제할 수 없고**, 반드시 노드의 정의 파일을 수정해야 함.

## 5. Static Pod의 활용 예시

- **컨트롤 플레인 컴포넌트 배포**  
  - API 서버, 컨트롤러 매니저, etcd 등 쿠버네티스 자체의 컨트롤 플레인을 Static Pod로 배포할 수 있음.
  - kubeadm으로 클러스터를 구성하면 이 방식이 사용됨.
- **고가용성 및 자동 복구**  
  - 컨트롤 플레인 프로세스가 죽어도 kubelet이 자동으로 재시작해줌.

## 6. Static Pod와 DaemonSet의 차이

| 항목          | Static Pod                        | DaemonSet                      |
|---------------|-----------------------------------|--------------------------------|
| 생성 주체     | kubelet                           | DaemonSet 컨트롤러(API 서버)   |
| 관리 방식     | 노드의 정의 파일을 직접 읽음        | API 서버의 오브젝트로 관리      |
| 사용 목적     | 컨트롤 플레인 등 필수 서비스 배포   | 모든 노드에 에이전트 등 배포    |
| 스케줄러 영향 | 없음                              | 없음                           |

## 7. 요약

- **Static Pod는 kubelet이 직접 관리하는 파드**로, API 서버 등 컨트롤 플레인 없이도 동작함.
- **정의 파일을 지정된 디렉토리에 두면 kubelet이 자동으로 생성·관리.**
- **컨트롤 플레인 컴포넌트 배포에 주로 사용**되며, kubeadm 등에서 이 방식을 활용.
- **Static Pod와 DaemonSet은 모두 스케줄러의 영향을 받지 않음.**
- **Static Pod는 API 서버에서 읽기 전용으로만 확인 가능**하며, 반드시 노드의 정의 파일을 수정해야 변경·삭제가 가능.
