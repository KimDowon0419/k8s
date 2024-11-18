# IP Address Management in Kubernetes

## 목차
1. [IP Address Management란 무엇인가?](#IP-Address-Management란-무엇인가)
2. [Kubernetes에서 IP 주소 관리의 중요성](#Kubernetes에서-IP-주소-관리의-중요성)
3. [IP 할당 방식](#IP-할당-방식)
4. [Kubernetes 네트워크와 CIDR 설정](#Kubernetes-네트워크와-CIDR-설정)
5. [IP 주소와 CNI 플러그인](#IP-주소와-CNI-플러그인)
6. [IP 주소 관리 및 확인 명령어](#IP-주소-관리-및-확인-명령어)
7. [IP Address Management 유용한 팁](#IP-Address-Management-유용한-팁)

---

## IP Address Management란 무엇인가?

**IP Address Management (IPAM)**는 Kubernetes 클러스터 내에서 **Pod, Service, 노드 간 통신을 위한 IP 주소 할당과 관리**를 의미합니다. Kubernetes는 클러스터 네트워크 모델을 통해 각 Pod와 Service에 고유한 IP 주소를 부여하며, 이를 통해 통신을 지원합니다.

---

## Kubernetes에서 IP 주소 관리의 중요성

1. **고유한 네트워크 식별**:
   - 각 Pod는 고유한 IP 주소를 가지며, 이를 통해 클러스터 내부에서 통신 가능
2. **대규모 클러스터 지원**:
   - 여러 노드와 수천 개의 Pod를 포함한 클러스터에서 효율적인 IP 주소 관리 필요
3. **네트워크 충돌 방지**:
   - 올바른 CIDR 설정으로 IP 주소 충돌 방지
4. **외부 네트워크 통합**:
   - 클러스터 외부와의 통신을 원활히 관리

---

## IP 할당 방식

1. **Pod IP**:
   - 각 Pod는 클러스터의 **Pod CIDR** 범위 내에서 고유한 IP 주소를 할당받음
   - Pod 간 통신은 NAT 없이 가능

2. **Service IP**:
   - Service는 클러스터의 **Service CIDR** 범위 내에서 고유한 가상 IP(Virtual IP)를 할당받음
   - Service IP는 kube-proxy에 의해 Pod로 라우팅

3. **Node IP**:
   - 각 노드는 클러스터 네트워크 외부와 통신하기 위한 고유한 IP 주소를 가짐

---

## Kubernetes 네트워크와 CIDR 설정

### 1. Pod CIDR

Pod가 할당받는 IP 주소 범위는 `--pod-network-cidr` 플래그를 통해 설정됩니다.

#### Pod CIDR 확인
```bash
kubectl cluster-info dump | grep -m 1 cluster-cidr
```

---

### 2. Service CIDR

Service의 가상 IP 주소 범위는 `--service-cluster-ip-range` 플래그를 통해 설정됩니다.

#### Service CIDR 확인
```bash
kubectl cluster-info dump | grep -m 1 service-cluster-ip-range
```

---

### 3. 노드의 CIDR 확인

CNI 플러그인 설정 파일에서 CIDR 범위를 확인합니다.

#### 설정 파일 확인
```bash
cat /etc/cni/net.d/*.conf | grep cidr
```

---

## IP 주소와 CNI 플러그인

CNI 플러그인은 Kubernetes 클러스터의 IP 주소 관리를 담당합니다. 주요 CNI 플러그인의 IPAM 관리 방식:

1. **Flannel**:
   - 단순한 오버레이 네트워크
   - Pod 네트워크를 자동으로 관리하고 IP 주소를 할당
2. **Calico**:
   - 네트워크 정책과 IPAM 관리 기능 제공
   - IP 풀을 커스터마이징 가능
3. **WeaveNet**:
   - Pod 네트워크를 간단히 구성하고 IP를 동적으로 할당
4. **Cilium**:
   - eBPF 기반으로 IPAM과 네트워크 정책을 통합 관리

---

## IP 주소 관리 및 확인 명령어

- **Pod IP 확인**
  ```bash
  kubectl get pods -o wide
  ```

- **Service IP 확인**
  ```bash
  kubectl get services -o wide
  ```

- **노드 IP 확인**
  ```bash
  kubectl get nodes -o wide
  ```

- **CNI 플러그인 설정 확인**
  ```bash
  cat /etc/cni/net.d/*.conf
  ```

- **현재 클러스터의 CIDR 범위 확인**
  ```bash
  kubectl cluster-info dump | grep -E "cluster-cidr|service-cluster-ip-range"
  ```

---

## IP Address Management 유용한 팁

1. **Pod 및 Service CIDR 범위 계획**:
   - CIDR 범위는 클러스터 규모와 미래 확장성을 고려하여 설정
2. **IP 주소 충돌 방지**:
   - 클러스터 외부 네트워크와 CIDR 범위가 겹치지 않도록 주의
3. **CNI 플러그인 설정 주기적 검토**:
   - CNI 플러그인의 IP 풀 및 설정을 주기적으로 검토하여 관리
4. **네트워크 디버깅 도구 활용**:
   - `ping`, `curl`, `traceroute` 등을 사용해 네트워크 문제를 빠르게 파악

---

IP Address Management는 Kubernetes 클러스터에서 안정적인 네트워크 통신을 유지하는 핵심 요소입니다. Pod, Service, Node에 고유한 IP 주소를 올바르게 할당하고, CNI 플러그인을 활용해 IP 풀을 관리함으로써 클러스터의 확장성과 보안을 강화할 수 있습니다.
