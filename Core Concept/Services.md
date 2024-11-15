# Services

## 목차
1. [Services란 무엇인가?](#Services란-무엇인가)
2. [Services의 작동 원리](#Services의-작동-원리)
3. [Service의 유형](#Service의-유형)
4. [Service 설정](#Service-설정)
5. [Service 관리 및 운영](#Service-관리-및-운영)
6. [Service 주요 플래그](#Service-주요-플래그)
7. [Service 유용한 명령어 모음](#Service-유용한-명령어-모음)

---

## Services란 무엇인가?

**Services**는 Kubernetes에서 **Pod 간의 네트워크 연결을 관리하고, 클러스터 외부의 트래픽을 내부로 라우팅**하는 데 사용됩니다. Pod는 생성과 삭제를 반복하며 IP 주소가 변경되기 때문에, Services는 **고정 IP**와 **DNS 이름**을 통해 Pod를 안정적으로 접근할 수 있게 합니다. 

- **기능**: Pod의 네트워크 접근을 쉽게 관리하며, 클러스터 내부와 외부의 트래픽을 서비스로 라우팅
- **고정 IP 제공**: Services는 Pod 집합에 대한 고정 IP를 제공하여, 특정 Pod가 변경되어도 동일한 IP로 접근 가능
- **로드 밸런싱**: 서비스 내부에서 여러 Pod 간에 로드 밸런싱을 지원

---

## Services의 작동 원리

Services는 **Label Selector**를 사용해 특정 레이블을 가진 Pod를 선택하여 트래픽을 라우팅합니다. 이를 통해 생성과 삭제를 반복하는 Pod 집합에 대해 고정된 네트워크 엔드포인트를 제공합니다. Kubernetes는 iptables나 IPVS를 통해 서비스에 대한 네트워크 규칙을 자동으로 설정하여 트래픽을 라우팅하고, 내부적으로 Endpoints 오브젝트를 생성해 각 Pod의 IP를 저장합니다.

1. **Label Selector**: 특정 레이블을 기반으로 Service가 관리할 Pod를 선택
2. **고정 IP와 DNS**: 서비스마다 고정 IP와 DNS 이름을 부여하여 안정적인 접근 가능
3. **트래픽 라우팅**: Endpoints를 통해 선택된 Pod로 트래픽을 전달하며, iptables/IPVS로 네트워크 규칙 설정
4. **로드 밸런싱**: 여러 Pod에 트래픽을 균등하게 분산

---

## Service의 유형

1. **ClusterIP (기본값)**:
   - 클러스터 내부에서만 접근 가능한 IP를 할당
   - 외부로부터 접근이 불가능하며, 클러스터 내부 애플리케이션 간 통신에 주로 사용

2. **NodePort**:
   - 각 노드에 지정된 포트를 통해 외부에서 접근 가능
   - 노드의 IP 주소와 할당된 포트를 통해 외부 트래픽이 서비스로 라우팅
   - 보통 30000~32767 사이의 포트 번호가 할당

3. **LoadBalancer**:
   - 클라우드 환경에서 LoadBalancer IP를 통해 외부 트래픽을 서비스로 전달
   - 클라우드 제공업체의 로드 밸런서를 사용해 트래픽 분산

4. **ExternalName**:
   - 클러스터 외부의 서비스를 DNS 이름으로 매핑하여 외부로 라우팅
   - CNAME 레코드를 반환하여 외부 DNS로 트래픽 전달

---

## Service 설정

Service는 YAML 파일을 통해 설정되며, `metadata`, `spec` 섹션에 주요 설정 포함.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

### 주요 설정 항목

1. **metadata**: Service의 이름, 네임스페이스, 레이블 등의 메타데이터 설정
2. **spec.selector**: 서비스가 트래픽을 전달할 Pod를 선택하는 Label Selector
3. **spec.ports**: 서비스의 노출 포트와 대상 Pod의 포트 지정
4. **spec.type**: 서비스 유형 설정 (ClusterIP, NodePort, LoadBalancer, ExternalName)

### Port 매핑 예시

`ports` 섹션에서 외부 트래픽이 들어올 서비스의 포트와 내부 Pod가 실제로 사용하는 포트를 지정할 수 있습니다.

```yaml
ports:
  - protocol: TCP
    port: 80       # Service 포트
    targetPort: 8080  # Pod의 컨테이너 포트
```

---

## Service 관리 및 운영

1. **Service 생성**
   ```bash
   kubectl apply -f service.yaml
   ```

2. **Service 상태 확인**
   ```bash
   kubectl get services
   ```

3. **Service의 Endpoints 확인**
   - Service가 연결된 Pod의 IP와 포트를 확인할 수 있습니다.
   ```bash
   kubectl get endpoints <service-name>
   ```

4. **Service 삭제**
   ```bash
   kubectl delete service <service-name>
   ```

---

## Service 주요 플래그

1. **--type**:
   - Service 유형을 설정
   - ClusterIP, NodePort, LoadBalancer, ExternalName 네 가지 옵션 제공

2. **--port**:
   - 외부에서 접근할 Service 포트를 설정
   - 예: `--port=80`

3. **--target-port**:
   - 연결된 Pod에서 실제로 사용 중인 포트를 설정
   - 예: `--target-port=8080`

4. **--protocol**:
   - 트래픽 프로토콜 설정 (TCP 또는 UDP)
   - 기본값은 TCP

5. **--selector**:
   - Label Selector를 통해 Service가 관리할 Pod를 지정
   - 예: `--selector=app=my-app`

---

## Service 유용한 명령어 모음

- **Service 생성**
  ```bash
  kubectl apply -f service.yaml
  ```

- **Service 상태 확인**
  ```bash
  kubectl get services
  ```

- **Service 상세 정보 조회**
  ```bash
  kubectl describe service <service-name>
  ```

- **Service의 Endpoints 확인**
  ```bash
  kubectl get endpoints <service-name>
  ```

- **Service 삭제**
  ```bash
  kubectl delete service <service-name>
  ```

- **Service 포트 전달 변경**
  ```bash
  kubectl edit service <service-name>
  ```
