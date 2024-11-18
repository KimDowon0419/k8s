# Service Networking

## 목차
1. [Service Networking이란 무엇인가?](#Service-Networking이란-무엇인가)
2. [Service Networking의 주요 역할](#Service-Networking의-주요-역할)
3. [Kubernetes에서 Service Networking의 동작 원리](#Kubernetes에서-Service-Networking의-동작-원리)
4. [Service Networking 설정 및 유형](#Service-Networking-설정-및-유형)
5. [Service Networking 디버깅 및 문제 해결](#Service-Networking-디버깅-및-문제-해결)
6. [Service Networking 유용한 명령어 모음](#Service-Networking-유용한-명령어-모음)

---

## Service Networking이란 무엇인가?

**Service Networking**은 Kubernetes에서 **Pod와 외부 클라이언트, 다른 Pod 간의 네트워크 통신을 지원하기 위한 구조**입니다. Kubernetes **Service** 리소스는 Pod 집합에 대한 단일 접점을 제공하며, 클러스터 내부와 외부에서 통신을 원활하게 지원합니다.

- **Service 정의**:
  - Kubernetes 리소스로, 네트워크 트래픽을 특정 Pod로 전달하는 역할
  - 고정된 가상 IP(Virtual IP)와 DNS 이름을 통해 Pod 집합에 접근 가능

---

## Service Networking의 주요 역할

1. **Pod 간 통신 지원**:
   - 동일하거나 다른 네임스페이스에 있는 Pod 간 통신을 가능하게 함
2. **외부 클라이언트 접근 허용**:
   - 외부 트래픽이 Kubernetes 클러스터 내부의 Pod로 전달되도록 지원
3. **로드 밸런싱 제공**:
   - 동일한 역할을 하는 여러 Pod 간의 트래픽 분배
4. **고정 IP 제공**:
   - Pod는 재생성될 때마다 새로운 IP를 할당받지만, Service는 고정된 IP로 지속적인 접근 제공

---

## Kubernetes에서 Service Networking의 동작 원리

### 1. 가상 IP와 DNS

- **ClusterIP**:
  - 각 Service는 고유한 클러스터 IP를 가지며, 이를 통해 내부 Pod로 트래픽을 라우팅
  - 클러스터 DNS에 등록되어 이름 기반 접근 가능 (예: `service-name.namespace.svc.cluster.local`)

### 2. `kube-proxy`와 iptables

- **kube-proxy**:
  - Service와 Pod 간 트래픽을 라우팅
  - iptables 또는 IPVS를 사용하여 Service IP와 Pod IP를 매핑

---

## Service Networking 설정 및 유형

### Service 유형

1. **ClusterIP**:
   - 기본 Service 유형
   - 클러스터 내부에서만 접근 가능
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: clusterip-service
   spec:
     selector:
       app: my-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 8080
   ```

2. **NodePort**:
   - 클러스터 외부에서 접근 가능
   - 각 노드의 지정된 포트를 통해 트래픽 전달
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: nodeport-service
   spec:
     type: NodePort
     selector:
       app: my-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 8080
         nodePort: 30080
   ```

3. **LoadBalancer**:
   - 클라우드 제공자의 로드 밸런서를 사용하여 외부 트래픽을 Pod로 전달
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: loadbalancer-service
   spec:
     type: LoadBalancer
     selector:
       app: my-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 8080
   ```

4. **ExternalName**:
   - 클러스터 외부의 DNS 이름을 내부 클라이언트에 노출
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: externalname-service
   spec:
     type: ExternalName
     externalName: example.com
   ```

---

## Service Networking 디버깅 및 문제 해결

1. **Service 상태 확인**
   ```bash
   kubectl describe service <service-name>
   ```

2. **DNS 확인**
   - 클러스터 내 DNS 이름이 제대로 설정되었는지 확인
   ```bash
   kubectl exec -it <pod-name> -- nslookup <service-name>
   ```

3. **Pod와 Service 연결 확인**
   - Service와 연결된 Pod 목록 확인
   ```bash
   kubectl get endpoints <service-name>
   ```

4. **트래픽 라우팅 문제 확인**
   - `kube-proxy` 로그를 통해 라우팅 문제 디버깅
   ```bash
   journalctl -u kube-proxy
   ```

5. **iptables 확인**
   - `iptables`에서 Service IP가 올바르게 설정되었는지 확인
   ```bash
   sudo iptables -L -t nat
   ```

---

## Service Networking 유용한 명령어 모음

- **Service 목록 확인**
  ```bash
  kubectl get services
  ```

- **Service 세부 정보 확인**
  ```bash
  kubectl describe service <service-name>
  ```

- **Service와 연결된 Endpoints 확인**
  ```bash
  kubectl get endpoints
  ```

- **Pod의 Service 연결 테스트**
  ```bash
  kubectl exec -it <pod-name> -- curl <service-name>
  ```

- **Service 삭제**
  ```bash
  kubectl delete service <service-name>
  ```

---

Service Networking은 Kubernetes 클러스터 내외부의 네트워크 트래픽을 효과적으로 관리하며, 다양한 유형의 Service를 통해 네트워크 트래픽을 라우팅하고 로드 밸런싱을 제공합니다. 이를 통해 Kubernetes 환경에서 안정적이고 효율적인 네트워크 구성이 가능합니다.
