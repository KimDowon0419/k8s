# Services

Kubernetes(K8s)에서 서비스(Service)는 Pod의 네트워크 접근을 관리하고, 로드 밸런싱을 제공하는 중요한 개념입니다. Kubernetes의 서비스 타입에는 크게 ClusterIP, NodePort, LoadBalancer가 있으며, 각각의 특징과 사용 사례는 다음과 같습니다.

### 1. Services
서비스는 Kubernetes 클러스터 내에서 네트워크 접근을 관리하는 추상화된 리소스입니다. 서비스는 여러 Pod에 걸쳐 로드 밸런싱을 제공하고, Pod의 IP 주소가 변하더라도 클라이언트가 서비스에 접근할 수 있게 해줍니다. 서비스는 일반적으로 다음과 같은 방식으로 정의됩니다:

- **Selector**: 서비스가 트래픽을 라우팅할 대상 Pod를 선택하는 레이블 셀렉터.
- **ClusterIP**: 내부 클러스터 IP 주소를 사용하여 서비스에 접근할 수 있도록 설정.
- **Ports**: 서비스가 노출할 포트 번호와 대상 포트 번호.

### 2. ClusterIP
ClusterIP는 Kubernetes의 기본 서비스 타입으로, 클러스터 내부에서만 접근 가능한 내부 IP 주소를 제공합니다.

- **기능**: 클러스터 내에서만 접근 가능한 내부 IP 주소를 생성.
- **사용 사례**: 클러스터 내부의 마이크로서비스 간 통신. 예를 들어, 백엔드 서비스가 프론트엔드 서비스에 데이터를 제공하는 경우.
- **접근 방법**: 클러스터 내에서 다른 서비스나 Pod가 이 서비스를 접근할 수 있습니다. 외부에서는 접근 불가.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  type: ClusterIP
```

### 3. NodePort
NodePort는 각 노드의 특정 포트를 통해 서비스를 외부에 노출시킵니다. NodePort는 ClusterIP를 기반으로 하며, 외부에서 클러스터의 모든 노드의 IP와 지정된 포트를 통해 서비스에 접근할 수 있습니다.

- **기능**: 클러스터 외부에서 클러스터의 모든 노드의 지정된 포트를 통해 서비스에 접근 가능.
- **사용 사례**: 간단한 외부 접근이 필요한 경우. 예를 들어, 테스트 목적으로 외부에서 서비스를 접근해야 할 때.
- **접근 방법**: 외부 클라이언트는 `<NodeIP>:<NodePort>` 형식으로 서비스에 접근.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
      nodePort: 30007
```

### 4. LoadBalancer
LoadBalancer는 클라우드 제공자(GCP, AWS, Azure 등)의 로드 밸런서를 이용해 외부에 서비스를 노출합니다. NodePort와 ClusterIP를 포함하며, 클라우드 제공자의 로드 밸런서를 통해 외부 트래픽을 받아 NodePort를 통해 서비스로 전달합니다.

- **기능**: 클라우드 제공자의 로드 밸런서를 통해 외부 트래픽을 받아 클러스터 내부의 서비스로 전달.
- **사용 사례**: 클라우드 환경에서 외부 트래픽을 처리할 때. 예를 들어, 웹 애플리케이션의 외부 트래픽을 분산시키고자 할 때.
- **접근 방법**: 클라우드 제공자가 할당한 외부 IP 주소를 통해 서비스에 접근.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

### 요약
- **ClusterIP**: 클러스터 내부에서만 접근 가능한 서비스. 기본 서비스 타입.
- **NodePort**: 클러스터 외부에서 노드의 IP와 포트를 통해 접근 가능한 서비스. 모든 노드에서 동일한 포트로 접근 가능.
- **LoadBalancer**: 클라우드 제공자의 로드 밸런서를 통해 외부 트래픽을 처리하는 서비스. 클라우드 환경에서 외부 접근이 필요한 서비스에 사용.

이러한 서비스 타입을 통해 Kubernetes는 다양한 네트워크 접근 방법을 제공하며, 애플리케이션의 요구 사항에 맞는 네트워크 설정을 할 수 있습니다.
