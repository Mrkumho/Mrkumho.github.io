# 데이터 암호화(Encrypting Secret Data at Rest) 정리
---

## 1. etcd와 저장 방식

- 쿠버네티스는 Secret을 **etcd**라는 저장소에 저장함..
- **기본적으로 Secret은 etcd에 평문(인코딩만 된 상태)으로 저장**됨.
- etcd에 직접 접근하면 Secret 값을 쉽게 볼 수 있음.

---

## 2. 암호화의 필요성

- **etcd가 유출되거나 관리자가 악의적으로 접근하면 Secret이 노출**될 수 있음.
- **암호화(Encryption)를 적용하면, Secret이 etcd에 저장될 때 실제로 암호화되어 저장**되므로, 유출되더라도 내용을 볼 수 없음.

---

## 3. 암호화 설정 방법

### 3.1. 암호화 키 생성
```bash
head -c 32 /dev/urandom | base64
```
- **32바이트 랜덤 키를 생성하고, Base64로 인코딩**.

---

### 3.2. 암호화 설정 파일 작성
>/etc/kubernetes/enc/enc.yaml
```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <YOUR-BASE64-ENCODED-32-BYTE-KEY>
      - identity: {}
```

- **`aescbc`(AES-CBC 암호화) 등 다양한 암호화 알고리즘**을 사용 가능.

---

### 3.3. kube-apiserver에 암호화 설정 적용

- **kube-apiserver 매니페스트 파일을 수정**해,  
  `--encryption-provider-config` 옵션에 위 파일 경로를 지정.★
- **암호화 파일을 kube-apiserver 파드에 볼륨 마운트**로 연결함.☆

>/etc/kubernetes/manifests/kube-apiserver.yaml
```yaml
spec:
  containers:
  - command:
    - kube-apiserver
    ...
    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml  # ★add this line
    volumeMounts:
    ...
    - name: enc                           # ☆add this line
      mountPath: /etc/kubernetes/enc      # ☆add this line
      readOnly: true                      # ☆add this line
    ...
  volumes:
  ...
  - name: enc                             # ☆add this line
    hostPath:                             # ☆add this line
      path: /etc/kubernetes/enc           # ☆add this line
      type: DirectoryOrCreate             # ☆add this line
  ...
```
- **kube-apiserver 재시작**하면 설정이 적용됨.

---

### 3.4. 암호화 적용 후 확인

- **새로 생성한 Secret은 etcd에서 암호화된 상태로 저장**됨.
- **기존 Secret은 암호화가 적용되지 않으므로,  
아래 명령으로 재저장(재암호화)해야 함.**
kubectl get secrets --all-namespaces -o json | kubectl replace -f -


---

## 4. 요약

- **Secret은 기본적으로 etcd에 평문(인코딩만)으로 저장**됨.
- **암호화 설정 파일을 만들고, kube-apiserver에 적용하면 Secret이 etcd에 암호화되어 저장**됨.
- **암호화는 Secret 등 민감한 데이터의 보안을 크게 강화**함.

---

> **핵심 한 줄 요약**  
> **Secret을 etcd에 암호화해서 저장하면, 저장소가 유출되어도 데이터가 안전하게 보호됨.**  
> **암호화 설정 파일과 kube-apiserver 옵션만 바꾸면 쉽게 적용할 수 있음.**

> 참고 : https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
