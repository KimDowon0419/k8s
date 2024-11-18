# DNS and CoreDNS in Kubernetes

## 목차
1. [Kubernetes에서 DNS란 무엇인가?](#Kubernetes에서-DNS란-무엇인가)
2. [CoreDNS란 무엇인가?](#CoreDNS란-무엇인가)
3. [DNS의 역할](#DNS의-역할)
4. [CoreDNS의 구성 요소](#CoreDNS의-구성-요소)
5. [CoreDNS 설정](#CoreDNS-설정)
6. [DNS와 CoreDNS 디버깅 및 문제 해결](#DNS와-CoreDNS-디버깅-및-문제-해결)
7. [DNS와 CoreDNS 유용한 명령어 모음](#DNS와-CoreDNS-유용한-명령어-모음)

---

## Kubernetes에서 DNS란 무엇인가?

**DNS (Domain Name System)**는 Kubernetes 클러스터 내에서 **서비스와 Pod 간 네트워크 통신을 위한 이름 해석**을 제공합니다. Kubernetes DNS는 Pod와 Service의 **고유 이름**을 사용해 IP 주소로 변환하며, 클러스터 내의 서비스 검색을 간소화합니다.

- **기능**:
  - 서비스 이름을 IP 주소로 해석
  - 네임스페이스를 기반으로 격리된 이름 공간 관리
  - 외부 DNS 이름 해석 지원

---

## CoreDNS란 무엇인가?

**CoreDNS**는 Kubernetes의 **DNS 서버 역할을 하는 플러그인 기반 DNS 솔루션**입니다. Kubernetes 클러스터에서 기본적으로 사용되며, DNS 요청을 처리하고, 클러스터 내외부의 네트워크 리소스에 대한 이름 해석을 제공합니다.

- **특징**:
  - 경량화된 DNS 서버
  - 플러그인 아키텍처를 통해 확장 가능
  - Kubernetes 클러스터에 최적화

---

## DNS의 역할

1. **서비스 디스커버리**:
   - 클러스터 내 서비스 이름을 통해 Pod와 통신 가능
   - 예: `my-service.default.svc.cluster.local`

2. **네임스페이스 기반 격리**:
   - 네임스페이스별로 격리된 DNS 네임스페이스 제공

3. **외부 이름 해석**:
   - 클러스터 외부의 DNS 요청 처리
   - 예: `example.com`

4. **Pod 이름 해석**:
   - Pod의 IP 주소를 이름으로 해석
   - 예: `<pod-ip-address>.pod.cluster.local`

---

## CoreDNS의 구성 요소

1. **CoreDNS ConfigMap**:
   - CoreDNS의 설정을 관리하는 Kubernetes ConfigMap
   - 기본 위치: `kube-system` 네임스페이스

2. **CoreDNS 플러그인**:
   - DNS 요청을 처리하는 다양한 기능 제공
   - 주요 플러그인:
     - **`kubernetes`**: Kubernetes 서비스와 Pod의 이름 해석
     - **`forward`**: 외부 DNS 서버로 요청 전달
     - **`cache`**: 반복적인 요청에 대한 응답 캐싱

3. **CoreDNS Pod**:
   - DNS 서버 역할을 하는 Kubernetes Pod

---

## CoreDNS 설정

### CoreDNS ConfigMap 기본 예시

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
            pods insecure
            fallthrough in-addr.arpa ip6.arpa
        }
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
```

- **`kubernetes` 플러그인**:
  - 클러스터 내부 DNS 이름 해석
  - `cluster.local`은 기본 도메인
- **`forward` 플러그인**:
  - 외부 DNS 서버로 요청 전달
- **`cache` 플러그인**:
  - DNS 응답을 캐싱하여 성능 향상

---

## DNS와 CoreDNS 디버깅 및 문제 해결

1. **DNS 이름 해석 확인**
   ```bash
   kubectl exec -it <pod-name> -- nslookup <service-name>
   ```

2. **DNS 구성 확인**
   - ConfigMap에 설정된 CoreDNS 구성을 확인
   ```bash
   kubectl -n kube-system get configmap coredns -o yaml
   ```

3. **CoreDNS Pod 상태 확인**
   ```bash
   kubectl get pods -n kube-system -l k8s-app=kube-dns
   ```

4. **CoreDNS Pod 로그 확인**
   - CoreDNS Pod의 로그를 확인하여 DNS 문제 해결
   ```bash
   kubectl logs -n kube-system <coredns-pod-name>
   ```

5. **서비스 엔드포인트 확인**
   - Service에 연결된 Pod가 올바르게 설정되었는지 확인
   ```bash
   kubectl get endpoints <service-name>
   ```

---

## DNS와 CoreDNS 유용한 명령어 모음

- **CoreDNS ConfigMap 확인**
  ```bash
  kubectl -n kube-system get configmap coredns -o yaml
  ```

- **DNS 이름 해석 테스트**
  ```bash
  kubectl exec -it <pod-name> -- nslookup <service-name>
  ```

- **CoreDNS 상태 확인**
  ```bash
  kubectl get pods -n kube-system -l k8s-app=kube-dns
  ```

- **CoreDNS Pod 로그 확인**
  ```bash
  kubectl logs -n kube-system <coredns-pod-name>
  ```

- **클러스터 외부 이름 해석 테스트**
  ```bash
  kubectl exec -it <pod-name> -- nslookup example.com
  ```

---

DNS와 CoreDNS는 Kubernetes 클러스터에서 안정적인 이름 해석과 서비스 검색을 가능하게 합니다. CoreDNS는 플러그인 아키텍처를 통해 유연성과 확장성을 제공하며, 클러스터 내부 통신뿐 아니라 외부 통신에서도 중요한 역할을 수행합니다. 적절한 설정과 디버깅을 통해 네트워크 문제를 효율적으로 해결할 수 있습니다.
