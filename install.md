# Rocky Linux에서 Kubernetes 설치 

> 환경 문제 해결부터 repo 직접 생성까지 실제로 겪은 과정을 정리한 문서

---

## 1. 환경

* OS: **Rocky Linux 9 (Virtual Machine)**
* 목적: Kubernetes 실습 환경 구축
* 방식: `kubeadm` 기반 설치
* 컨테이너 런타임: **containerd**
* 특징:

  * 가상머신 환경
  * 클립보드 복사/붙여넣기 불가
  * 최소 설치 이미지 사용

---

## 2. 문제 상황 요약

### 2.1 containerd 명령어 인식 불가

```bash
containerd config default > /etc/containerd/config.toml
```

* 에러: `명령을 찾을 수 없습니다`
* 원인:

  * `containerd` 패키지가 설치되지 않음
  * Rocky Linux 기본 repo에는 `containerd.io`가 존재하지 않음

---

### 2.2 containerd.io 설치 실패

```bash
dnf install -y containerd.io
```

* 에러: `일치하는 항목을 찾을 수 없습니다`
* 원인:

  * Docker 공식 repo가 시스템에 존재하지 않음

---

### 2.3 dnf config-manager 사용 실패

```bash
dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

* 문제:

  * 명령은 실행되었으나 실제 repo 파일이 생성되지 않음
  * `/etc/yum.repos.d/` 디렉토리에 docker repo 없음

---

## 3. 해결 방법 (정석 & 확실한 방법)

### 3.1 Docker repo 파일 직접 생성

> `dnf config-manager`가 정상 동작하지 않는 경우 가장 확실한 방법

```bash
sudo -i
vi /etc/yum.repos.d/docker-ce.repo
```

#### 입력 내용 (직접 타이핑)

```
[docker-ce-stable]
name=Docker CE Stable
baseurl=https://download.docker.com/linux/centos/9/x86_64/stable
enabled=1
gpgcheck=0
```

* `i` : 입력 모드
* `Esc :wq` : 저장 후 종료

---

### 3.2 repo 등록 확인

```bash
dnf clean all
dnf repolist | grep docker
```

정상 출력:

```
docker-ce-stable    Docker CE Stable
```

---

### 3.3 containerd 설치

```bash
dnf install -y containerd.io
```

설치 확인:

```bash
containerd --version
```

---

## 4. containerd 설정

### 4.1 기본 설정 파일 생성

```bash
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
```

---

### 4.2 cgroup 설정 변경 (필수)

```bash
vi /etc/containerd/config.toml
```

변경 전:

```toml
SystemdCgroup = false
```

변경 후:

```toml
SystemdCgroup = true
```

---

### 4.3 containerd 서비스 실행

```bash
systemctl enable --now containerd
systemctl status containerd
```

상태:

```
active (running)
```

---

## 5. 배운 점

* Rocky Linux 최소 설치 환경에서는 **외부 repo 관리가 가장 큰 허들이었다.**
* `dnf config-manager`는 항상 repo 파일 생성을 보장하지 않음
* 문제 발생 시 `/etc/yum.repos.d/` 직접 관리가 가장 확실함
* `vi` 사용법은 서버 환경에서 필수 스킬
* Kubernetes 설치보다 **환경 구성 문제가 더 어려울 수 있음**

---

## 6. 다음 단계

* kubelet / kubeadm / kubectl 설치
* `kubeadm init` 실행
* CNI 플러그인 (Flannel) 적용
* 단일 노드 클러스터 구성

---

