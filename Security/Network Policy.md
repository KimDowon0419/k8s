# Network Policy

## 목차
1. [Network Policy란 무엇인가?](#Network-Policy란-무엇인가)
2. [Network Policy의 목적](#Network-Policy의-목적)
3. [Network Policy의 구성 요소](#Network-Policy의-구성-요소)
4. [Network Policy 설정 방법](#Network-Policy-설정-방법)
5. [Ingress와 Egress 정책](#Ingress와-Egress-정책)
6. [Network Policy 예제](#Network-Policy-예제)
7. [Network Policy 유용한 명령어 모음](#Network-Policy-유용한-명령어-모음)

---

## Network Policy란 무엇인가?

**Network Policy**는 Kubernetes에서 **Pod 간 트래픽 및 외부 소스와의 네트워크 트래픽을 제어**하는 리소스입니다. 이를 통해 클러스터 내 애플리케이션 간 통신을 보호하고, **불필요한 트래픽을 차단**하여 네트워크 보안을 강화할 수 있습니다.

- **기능**:
  - 특정 Pod로의 트래픽 허용/차단
  - Ingress(들어오는 트래픽) 및 Egress(나가는 트래픽) 제어
  - 레이블 기반 필터링으로 유연한 네트워크 정책 설정

---

## Network Policy의 목적

1. **보안 강화**:
   - 허가되지 않은 Pod 간 통신 및 외부 트래픽 차단
   - 민감한 애플리케이션 데이터를 보호

2. **서비스 격리**:
   - 서비스 간 네트워크 트래픽을 격리하여 다중 테넌트 환경 보장

3. **규정 준수**:
   - 네트워크 접근 제어와 모니터링을 통해 보안 규정을 충족

4. **네트워크 성능 최적화**:
   - 불필요한 트래픽을 차단하여 네트워크 부하 감소

---

## Network Policy의 구성 요소

1. **Pod Selector**:
   - 네트워크 정책이 적용될 대상 Pod를 정의
   - 레이블 기반으로 선택

2. **Policy Types**:
   - **Ingress**: 들어오는 트래픽 제어
   - **Egress**: 나가는 트래픽 제어

3. **Rules**:
   - 트래픽 허용 규칙 (대상, 포트, 프로토콜)
   - 레이블 기반 또는 IP 블록 기반

---

## Network Policy 설정 방법

Network Policy는 YAML 파일로 정의하며, **네트워크 플러그인(CNI)**이 지원해야 적용됩니다. 

### Network Policy 기본 구조

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: <policy-name>
  namespace: <namespace>
spec:
  podSelector:
    matchLabels:
      <key>: <value>
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              <key>: <value>
      ports:
        - protocol: TCP
          port: 80
```

- **`podSelector`**: 정책이 적용될 Pod 선택
- **`policyTypes`**: Ingress와 Egress 제어 여부 설정
- **`ingress/from`**: 트래픽을 허용할 소스 지정
- **`ports`**: 허용할 포트와 프로토콜

---

## Ingress와 Egress 정책

1. **Ingress 정책**:
   - Pod로 들어오는 트래픽을 제어
   - 특정 소스 IP, Pod, 네임스페이스에서의 트래픽 허용

2. **Egress 정책**:
   - Pod에서 나가는 트래픽을 제어
   - 특정 외부 서비스, 네트워크 블록으로의 트래픽 허용

---

## Network Policy 예제

### Ingress 정책: 특정 Pod만 접근 허용

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-specific-pod
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: backend
      ports:
        - protocol: TCP
          port: 80
```

- **설명**:
  - `web` 레이블이 있는 Pod로의 트래픽 중, `backend` 레이블이 있는 Pod에서 오는 TCP 80 포트만 허용

---

### Egress 정책: 외부 네트워크 접근 제한

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-egress
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5432
```

- **설명**:
  - `database` 레이블이 있는 Pod에서 `10.0.0.0/24` 네트워크의 TCP 5432 포트로 나가는 트래픽만 허용

---

## Network Policy 유용한 명령어 모음

- **현재 네임스페이스의 Network Policy 목록 확인**
  ```bash
  kubectl get networkpolicy -n <namespace>
  ```

- **특정 Network Policy의 상세 정보 확인**
  ```bash
  kubectl describe networkpolicy <policy-name> -n <namespace>
  ```

- **Network Policy 적용**
  ```bash
  kubectl apply -f networkpolicy.yaml
  ```

- **Network Policy 삭제**
  ```bash
  kubectl delete networkpolicy <policy-name> -n <namespace>
  ```

---

Network Policy는 Kubernetes에서 네트워크 보안을 강화하는 중요한 도구로, Pod 간 또는 외부 트래픽을 세밀하게 제어할 수 있습니다. 이를 통해 민감한 데이터를 보호하고, 안전하고 효율적인 클러스터 네트워크를 관리할 수 있습니다.
