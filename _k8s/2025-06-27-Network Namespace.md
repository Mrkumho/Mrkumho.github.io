## 1. 네트워크 네임스페이스란?

- **네임스페이스**는 리눅스에서 컨테이너(예: Docker) 격리의 핵심 기술.
- 네임스페이스를 집에 비유하면, 집(호스트) 안의 각 방(네임스페이스)이 각 컨테이너를 의미.
- 각 방(컨테이너)은 자신만의 공간과 정보만 볼 수 있고, 다른 방이나 집 전체는 볼 수 없음.
- 부모(호스트)는 모든 방을 볼 수 있음.

## 2. 네트워크 네임스페이스의 특징

- 컨테이너는 자신만의 네트워크 인터페이스, 라우팅 테이블, ARP 테이블을 가짐.
- 호스트는 모든 네임스페이스의 정보를 볼 수 있지만, 네임스페이스는 호스트나 다른 네임스페이스의 정보를 볼 수 없음.

## 3. 네트워크 네임스페이스 명령어

### 네임스페이스 생성 및 확인
```bash
ip netns add <네임스페이스명>
ip netns list
```

- 네임스페이스 생성 및 목록 확인

### 네임스페이스 내부에서 명령 실행
```bash
ip netns exec <네임스페이스명> <명령어>

예: ip netns exec red ip link
```

- 특정 네임스페이스 내부에서 명령 실행

### 네임스페이스 내 인터페이스 확인
```bash
ip netns exec <네임스페이스명> ip link
```

- 해당 네임스페이스 내 인터페이스만 표시

## 4. 네임스페이스끼리 연결하기

- **veth(virtual ethernet) 페어**를 사용해 네임스페이스끼리 가상 케이블로 연결
- 각 네임스페이스에 veth의 다른 쪽을 할당.
```bash
ip link add veth-red type veth peer name veth-blue
ip link set veth-red netns red
ip link set veth-blue netns blue
```

- 각 네임스페이스에서 IP 할당 및 인터페이스 활성화
```bash
ip netns exec red ip addr add 192.168.15.1/24 dev veth-red
ip netns exec blue ip addr add 192.168.15.2/24 dev veth-blue
ip netns exec red ip link set veth-red up
ip netns exec blue ip link set veth-blue up
```

- 이제 서로 ping으로 통신 가능

## 5. 여러 네임스페이스를 하나의 네트워크로 묶기

- **Linux Bridge**(가상 스위치)를 사용해 여러 네임스페이스를 하나의 네트워크에 연결
```bash
ip link add vnet0 type bridge
ip link set vnet0 up
```
text
- 각 네임스페이스에 veth 페어 생성, 한쪽은 네임스페이스에, 다른 한쪽은 브릿지에 연결
```bash
ip link add veth-red type veth peer name veth-red-br
ip link set veth-red netns red
ip link set veth-red-br master vnet0
ip link set veth-red-br up
```

- 네임스페이스 내에서 IP 할당 및 인터페이스 활성화(위와 동일)

## 6. 호스트와 네임스페이스 간 통신

- 브릿지 인터페이스(vnet0)에 호스트용 IP 할당
```bash
ip addr add 192.168.15.5/24 dev vnet0
ip link set vnet0 up
```

- 이제 호스트와 네임스페이스 간 ping 가능

## 7. 외부 네트워크(인터넷)와 연결

- 네임스페이스에서 외부 네트워크로 나가려면 **게이트웨이**(호스트의 브릿지 IP)를 라우팅 테이블에 추가
```bash
ip netns exec blue ip route add default via 192.168.15.5
```

- NAT(Masquerade)를 위해 호스트에서 iptables 설정
```bash
iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -o eth0 -j MASQUERADE
```

- 이제 네임스페이스에서 인터넷 접속 가능

## 8. 외부에서 네임스페이스로 접근(포트포워딩)

- 외부에서 네임스페이스 내부 서비스(예: 웹서버)에 접근하려면 호스트에서 포트포워딩 설정 필요
```bash
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.15.2:80
```

---

> **핵심 요약**  
> - 네임스페이스는 격리된 네트워크 환경 제공  
> - veth 페어와 브릿지로 네임스페이스 간/호스트 간/외부 간 네트워크 구성  
> - iptables로 NAT 및 포트포워딩 설정  
