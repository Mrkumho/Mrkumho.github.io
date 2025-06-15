# Kubernetes 릴리즈 및 버전 관리 개요

---

## 📌 **쿠버네티스 버전 구조**

- **버전 구성:** `major.minor.patch`
  - **major:** 주요 버전 (현재 1)
  - **minor:** 부 버전 (주기적 기능 추가, 몇 달마다 업데이트)
  - **patch:** 패치 버전 (버그 수정, 자주 업데이트)

---

## 🚀 **릴리즈 주기 및 유형**

- **Minor 릴리즈:**  
  - 몇 달마다 새 기능과 개선사항이 포함되어 출시
- **Patch 릴리즈:**  
  - 버그 수정 및 보안 패치가 포함되어 자주 출시
- **Alpha/Beta 릴리즈:**  
  - **Alpha:** 실험적 기능, 기본적으로 비활성화, 버그 많음
  - **Beta:** 충분히 테스트된 기능, 기본적으로 활성화, 안정적
  - **Stable:** 프로덕션 환경에서 사용 권장

---

## 📅 **쿠버네티스 릴리즈 이력 (최신 정보)**

- **최초 Stable 릴리즈:** 1.0 (2015년 7월)
- **최신 Stable 릴리즈 (2024년 기준):**
  - **Kubernetes: v1.30.0** (2024년 4월 기준, 공식 최신은 [Kubernetes Releases](https://github.com/kubernetes/kubernetes/releases) 참고)
- **릴리즈 주기:**  
  - 약 3~4개월마다 Minor 버전 업데이트

---

## 🧩 **컨트롤 플레인 및 외부 컴포넌트 버전**

- **쿠버네티스 컨트롤 플레인:**  
  - 모든 컴포넌트(API 서버, 스케줄러, 컨트롤러 매니저 등)는 동일 버전으로 배포
- **외부 컴포넌트:**  
  - **etcd:**  
    - **최신 버전:** v3.5.13 (2024년 기준, [etcd Releases](https://github.com/etcd-io/etcd/releases) 참고)
  - **CoreDNS:**  
    - **최신 버전:** v1.11.1 (2024년 기준, [CoreDNS Releases](https://github.com/coredns/coredns/releases) 참고)
- **릴리즈 노트 확인:**  
  - 각 릴리즈의 공식 문서에서 지원되는 etcd, CoreDNS 버전 확인 필요

---

## 📚 **릴리즈 다운로드 및 설치**

1. **릴리즈 페이지 방문:**  
   [Kubernetes GitHub Releases](https://github.com/kubernetes/kubernetes/releases)
2. **패키지 다운로드:**  
   - `kubernetes.tar.gz` 파일 다운로드
3. **압축 해제:**  
   - 모든 컨트롤 플레인 컴포넌트(동일 버전) 포함
4. **외부 컴포넌트:**  
   - etcd, CoreDNS는 별도로 설치 및 버전 관리 필요

---

## 🔍 **요약**

- **쿠버네티스 버전:** `major.minor.patch` (예: v1.30.0)
- **릴리즈 주기:** Minor(기능)는 몇 달마다, Patch(버그)는 자주
- **Alpha/Beta/Stable:** 실험 → 테스트 → 안정화 단계
- **컨트롤 플레인:** 모든 컴포넌트 동일 버전
- **외부 컴포넌트:** etcd, CoreDNS는 별도 버전 관리
- **릴리즈 노트:** 지원되는 외부 컴포넌트 버전 확인 필수
