# Pod Networking

## 목차
1. [Pod Networking이란 무엇인가?](#Pod-Networking이란-무엇인가)
2. [Pod Networking의 기본 원리](#Pod-Networking의-기본-원리)
3. [Kubernetes에서 Pod 간 네트워크 통신](#Kubernetes에서-Pod-간-네트워크-통신)
4. [Pod Networking 설정](#Pod-Networking-설정)
5. [Pod Networking 디버깅 및 문제 해결](#Pod-Networking-디버깅-및-문제-해결)
6. [Pod Networking 유용한 명령어 모음](#Pod-Networking-유용한-명령어-모음)

---

## Pod Networking이란 무엇인가?

**Pod Networking**은 Kubernetes에서 **Pod 간, Pod과 외부 네트워크 간의 통신**을 제공하는 네트워크 환경을 의미합니다. Kubernetes의 설계 원칙에 따라 모든 Pod는 **고유한 IP 주소**를 가지며, 클러스터 내 다른 Pod와 직접 통신할 수 있습니다.

- **특징**:
  - Pod 간의 평등한 네트워크 모델 제공
  - Pod는 클러스터 내부의 모든 다른 Pod와 직접 통신 가능
  - 각 Pod는 자신만의 네트워크 네임스페이스와 IP를 가짐

---

## Pod Networking의 기본 원리

### Kubernetes 네트워크 모델의 주요 원칙

1. **Pod 간 고유한 IP 주소**:
   - 각 Pod는 고유한 IP를 가지며, 같은 노드 내의 다른 Pod와 동일한 네트워크를 공유하지 않음

2. **Pod 간 직접 통신 가능**:
   - 네트워크의 NAT(Network Address Translation) 없이 Pod 간 직접 통신 가능

3. **플러그인 기반 네트워크 관리**:
   - 다양한 CNI(Container Network Interface) 플러그인을 사용해 네트워크 설정 관리

---

## Kubernetes에서 Pod 간 네트워크 통신

1. **Pod-to-Pod 통신**:
   - 동일한 네임스페이스에 속하는 Pod 간 통신은 별도의 설정 없이 가능
   - 다른 네임스페이스에 있는 Pod와 통신하려면 FQDN(예: `<service-name>.<namespace>.svc.cluster.local`) 사용 가능

2. **Pod-to-Service 통신**:
   - Kubernetes Service를 사용하여 Pod와 연결된 애플리케이션 간 통신을 관리
   - 클러스터 내부 DNS를 통해 Service 이름으로 접근 가능

3. **Pod-to-External 통신**:
   - 외부 네트워크와 통신하려면 `Egress` 네트워크 정책 또는 외부 IP 설정 필요

---

## Pod Networking 설정

Pod Networking은 네트워크 플러그인(CNI)과 함께 설정되며, 플러그인에 따라 구성 방식이 달라집니다.

### 네트워크 플러그인 설정

#### Flannel 네트워크 플러그인 설정
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

#### Calico 네트워크 플러그인 설정
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

---

### Pod 생성 시 네트워크 확인

Pod를 생성하면 Kubernetes는 해당 Pod에 IP 주소를 자동으로 할당합니다. IP 주소는 `Pod CIDR`에 따라 결정됩니다.

#### Pod YAML 예시
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod
  labels:
    app: test
spec:
  containers:
    - name: test-container
      image: nginx
```

Pod 생성 후 IP를 확인합니다.
```bash
kubectl get pods -o wide
```

---

## Pod Networking 디버깅 및 문제 해결

1. **Pod 간 통신 확인**
   - `ping` 명령으로 Pod 간 네트워크 연결 확인
   ```bash
   kubectl exec -it <source-pod-name> -- ping <target-pod-ip>
   ```

2. **Service 연결 확인**
   - `curl` 명령으로 Service와 연결된 Pod 확인
   ```bash
   kubectl exec -it <source-pod-name> -- curl <service-name>
   ```

3. **네트워크 정책(Network Policy) 문제 해결**
   - Pod가 통신을 허용하지 않는 경우 NetworkPolicy 확인
   ```bash
   kubectl get networkpolicy -n <namespace>
   ```

4. **Pod 로그 확인**
   - Pod에서 네트워크 관련 오류가 발생하면 로그 확인
   ```bash
   kubectl logs <pod-name>
   ```

5. **CNI 플러그인 로그 확인**
   - 네트워크 플러그인 로그를 통해 문제 진단
   ```bash
   journalctl -u kubelet
   ```

---

## Pod Networking 유용한 명령어 모음

- **Pod 목록 확인 및 IP 주소 확인**
  ```bash
  kubectl get pods -o wide
  ```

- **Pod 간 네트워크 연결 테스트**
  ```bash
  kubectl exec -it <pod-name> -- ping <target-ip>
  ```

- **네임스페이스별 Pod IP 확인**
  ```bash
  kubectl get pods -n <namespace> -o wide
  ```

- **NetworkPolicy 확인**
  ```bash
  kubectl get networkpolicy -n <namespace>
  ```

- **CNI 설정 파일 확인**
  ```bash
  cat /etc/cni/net.d/*.conf
  ```

---

Pod Networking은 Kubernetes 클러스터에서 Pod 간 통신을 지원하는 중요한 기능입니다. Kubernetes 네트워크 모델의 원칙을 이해하고 CNI 플러그인을 적절히 설정하면, 안정적이고 유연한 네트워크 환경을 제공할 수 있습니다.
