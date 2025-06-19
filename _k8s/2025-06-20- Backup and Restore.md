## 1. 쿠버네티스 리소스(객체) 백업

### **1.1 선언적(Declarative) 관리**
- **설정 파일(YAML 등) 저장**
  - 애플리케이션을 배포할 때 사용한 Deployment, Service, ConfigMap, Secret 등의 YAML 파일을 저장.
  - 이 파일들은 소스코드 저장소(예: GitHub)에 올려서 팀과 공유하고, 버전 관리를 할 수 있음.
  - 클러스터가 손상되어도 YAML 파일만 있으면 언제든 재배포가 가능.

**예시:**
```bash
kubectl apply -f ./my-app/
```
> `my-app` 폴더 안에 모든 YAML 파일이 있다고 가정

---

### **1.2 명령적(Imperative) 관리**
- **직접 명령어로 생성한 객체**
  - `kubectl create namespace`, `kubectl create secret` 등으로 만든 객체는 YAML 파일로 저장되지 않음.
  - 이런 경우, 클러스터에 존재하는 객체를 YAML로 추출해 백업해야 함.

**예시:**
```bash
kubectl get all --all-namespaces -o yaml > cluster-backup.yaml
```
> 모든 네임스페이스의 주요 리소스(파드, 디플로이먼트, 서비스 등)를 YAML로 백업

---

### **1.3 자동화 도구(Velero 등)**
- **Velero(구 Ark)**
  - 쿠버네티스 리소스와 Persistent Volumes을 자동으로 백업/복원할 수 있음.
  - 스케줄링, 클러스터 간 마이그레이션 등 다양한 기능 제공.

---

## 2. etcd 백업

etcd는 쿠버네티스 클러스터의 모든 상태 정보를 저장하는 키-값 저장소.

---

### **2.1 etcd 스냅샷 백업**

**예시:**
```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/server.crt \
  --key=/etc/etcd/server.key
```
> 스냅샷 파일(`snapshot.db`) 생성

**스냅샷 상태 확인:**
```bash
etcdctl snapshot status snapshot.db
```

---

### **2.2 etcd 복원**

1. **kube-apiserver 중지**
   ```bash
   systemctl stop kube-apiserver
   ```
2. **스냅샷 복원**
   ```bash
   etcdctl snapshot restore snapshot.db \
     --data-dir /var/lib/etcd-from-backup
   ```
3. **etcd 설정 파일 수정**
   - 데이터 디렉토리를 `/var/lib/etcd-from-backup`로 변경
4. **서비스 재시작**
   ```bash
   systemctl daemon-reload
   systemctl restart etcd
   systemctl restart kube-apiserver
   ```

---

## 3. 백업 방법 비교

| **방법**            | **장점**                         | **단점**                        | **적용 환경**         |
|---------------------|----------------------------------|----------------------------------|-----------------------|
| **리소스 백업**     | 관리형 쿠버네티스에서 사용 가능   | 일부 설정 누락 가능성            | 모든 환경             |
| **etcd 백업**       | 전체 클러스터 상태 복원 가능      | 노드 접근 필요, 복잡함           | 온프레미스/셀프 관리  |

---

## 4. 요약 및 추천

- **YAML 파일을 반드시 저장**하고, 소스코드 저장소에도 백업 필요.
- **Velero** 같은 도구를 활용하면 백업/복원이 더 쉽고 안전 함.
- **etcd 백업**은 클러스터 전체를 복원할 때 필요하지만, 관리형 쿠버네티스에서는 접근이 제한될 수 있음.
- **두 방법을 조합**해서 사용하는 것이 가장 안전 함.

---

> **참고:**  
> (예: `kubectl get all --all-namespaces -o yaml > backup.yaml`로 백업해보기)
