# CNI (Container Network Interface)

## 목차
1. [CNI란 무엇인가?](#CNI란-무엇인가)
2. [CNI의 주요 기능](#CNI의-주요-기능)
3. [Kubernetes에서 CNI의 역할](#Kubernetes에서-CNI의-역할)
4. [CNI 플러그인 종류](#CNI-플러그인-종류)
5. [CNI 설치 및 구성](#CNI-설치-및-구성)
6. [CNI 동작 원리](#CNI-동작-원리)
7. [CNI 유용한 명령어 모음](#CNI-유용한-명령어-모음)

---

## CNI란 무엇인가?

**CNI (Container Network Interface)**는 컨테이너 네트워크 연결을 설정하고 관리하기 위한 표준 인터페이스입니다. Kubernetes와 같은 컨테이너 오케스트레이션 플랫폼에서 **Pod 간 네트워크 통신을 지원**하며, 다양한 네트워크 플러그인을 통해 유연한 네트워크 구성을 가능하게 합니다.

- **목적**:
  - 컨테이너 네트워크의 설정, 업데이트, 삭제 작업 간소화
  - 다양한 네트워크 플러그인(Calico, Flannel, WeaveNet 등)과의 호환성 제공
- **구성 요소**:
  - 네트워크 플러그인(프로비저닝 및 관리)
  - 네트워크 설정 스크립트

---

## CNI의 주요 기능

1. **Pod 간 네트워크 연결**:
   - Pod가 클러스터 내 다른 Pod와 통신할 수 있도록 네트워크 인터페이스 생성
2. **IP 주소 할당**:
   - Pod에 고유한 IP 주소를 동적으로 할당
3. **네트워크 정책 적용**:
   - 트래픽 필터링 및 보안을 위한 정책 관리
4. **확장성 및 유연성**:
   - 다양한 네트워크 플러그인을 지원하여 특정 요구 사항에 맞는 설정 가능

---

## Kubernetes에서 CNI의 역할

1. **Pod 네트워크 생성**:
   - 각 Pod에 고유한 네트워크 인터페이스와 IP 주소 제공
2. **네트워크 트래픽 라우팅**:
   - Pod와 외부 네트워크 또는 다른 Pod 간의 통신 경로 설정
3. **네트워크 정책 적용**:
   - Kubernetes NetworkPolicy 리소스를 통해 트래픽 제어
4. **클러스터 내 통신 지원**:
   - 다중 노드 클러스터에서 노드 간 Pod 통신 보장

---

## CNI 플러그인 종류

Kubernetes에서 사용할 수 있는 주요 CNI 플러그인은 다음과 같습니다.

| **플러그인** | **특징**                                                                 |
|--------------|--------------------------------------------------------------------------|
| **Calico**   | 네트워크 정책과 BGP(Border Gateway Protocol)를 지원하는 고성능 플러그인 |
| **Flannel**  | 간단하고 경량화된 네트워크 오버레이 플러그인                              |
| **WeaveNet** | 다중 클러스터 통신을 지원하며 간편한 설정 가능                            |
| **Cilium**   | eBPF 기반으로 고성능과 세밀한 보안 제공                                  |
| **Canal**    | Calico와 Flannel의 장점을 결합한 하이브리드 플러그인                     |

---

## CNI 설치 및 구성

### 1. CNI 플러그인 설치

Kubernetes 클러스터에서 사용할 CNI 플러그인을 설치합니다.

#### Flannel 설치 예시

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

#### Calico 설치 예시

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

---

### 2. 네트워크 설정 확인

- **Pod CIDR 확인**:
  Kubernetes 클러스터에서 사용하는 Pod CIDR 확인
  ```bash
  kubectl cluster-info dump | grep -m 1 cluster-cidr
  ```

- **CNI 설정 확인**:
  `/etc/cni/net.d` 디렉토리에서 CNI 설정 파일 확인
  ```bash
  ls /etc/cni/net.d
  ```

---

## CNI 동작 원리

1. **Pod 생성 요청**:
   - 사용자가 Pod을 생성하면, kubelet이 CNI 플러그인을 호출
2. **네트워크 설정**:
   - CNI는 Pod의 네트워크 인터페이스를 생성하고, IP 주소를 할당
3. **트래픽 라우팅**:
   - CNI는 Pod의 트래픽을 올바른 경로로 전달하도록 네트워크 설정
4. **정책 적용**:
   - Kubernetes NetworkPolicy 리소스에 따라 트래픽 허용/차단 설정

---

## CNI 유용한 명령어 모음

- **CNI 플러그인 설치 상태 확인**
  ```bash
  kubectl get pods -n kube-system
  ```

- **Pod 네트워크 확인**
  ```bash
  kubectl get pods -o wide
  ```

- **Pod 네트워크 디버깅**
  ```bash
  kubectl exec -it <pod-name> -- ping <target-pod-ip>
  ```

- **CNI 설정 파일 확인**
  ```bash
  cat /etc/cni/net.d/*.conf
  ```

- **네트워크 플러그인 로그 확인**
  ```bash
  journalctl -u kubelet
  ```

---

CNI는 Kubernetes에서 Pod 간 통신 및 네트워크 관리의 핵심 역할을 담당하며, 다양한 플러그인을 통해 유연성과 확장성을 제공합니다. 적절한 CNI 플러그인을 선택하고 구성하면, 클러스터의 네트워크 성능과 보안을 효율적으로 관리할 수 있습니다.
