# Service Accounts

## 목차
1. [Service Account란 무엇인가?](#Service-Account란-무엇인가)
2. [Service Account의 목적](#Service-Account의-목적)
3. [Service Account와 User Account의 차이](#Service-Account와-User-Account의-차이)
4. [Service Account 생성 및 관리](#Service-Account-생성-및-관리)
5. [Service Account를 Pod에 연결하기](#Service-Account를-Pod에-연결하기)
6. [Service Account 유용한 명령어 모음](#Service-Account-유용한-명령어-모음)

---

## Service Account란 무엇인가?

**Service Account**는 Kubernetes에서 **Pod와 같은 애플리케이션이 클러스터 내에서 API 서버와 상호작용할 때 사용하는 계정**입니다. 사용자가 아닌 애플리케이션이나 워크로드를 위한 인증 및 권한 부여를 제공하며, Kubernetes API 요청 시 사용됩니다.

- **기능**:
  - 애플리케이션이 Kubernetes API와 통신할 수 있도록 인증 제공
  - RBAC(Role-Based Access Control)과 연계하여 세부적인 권한 설정 가능

---

## Service Account의 목적

1. **애플리케이션의 Kubernetes API 접근 제어**:
   - Pod가 특정 리소스에 접근하거나 작업을 수행할 수 있도록 권한 부여
2. **보안 강화**:
   - 각 Pod에 별도의 Service Account를 부여하여 불필요한 권한 제거
3. **다중 테넌트 환경에서 격리**:
   - 서로 다른 애플리케이션 또는 팀 간의 리소스 접근을 격리하여 보안 유지

---

## Service Account와 User Account의 차이

| **특징**              | **Service Account**                       | **User Account**                           |
|------------------------|-------------------------------------------|--------------------------------------------|
| **사용 목적**         | 애플리케이션 및 Pod를 위한 인증           | 실제 사용자 인증                           |
| **대상**              | Kubernetes API와 상호작용하는 Pod         | kubectl과 같은 클라이언트를 사용하는 사용자 |
| **생성 주체**         | Kubernetes가 관리하거나 수동 생성          | 외부 인증 시스템에서 관리 (예: OAuth)      |
| **연계**              | Pod, Job, Deployment 등                   | CLI 또는 애플리케이션                      |

---

## Service Account 생성 및 관리

Service Account는 기본적으로 각 네임스페이스에 하나씩 자동 생성되며, 필요에 따라 추가로 생성 가능합니다.

### Service Account 생성

#### 1. 기본 Service Account 확인
```bash
kubectl get serviceaccount -n <namespace>
```

#### 2. 새로운 Service Account 생성

```bash
kubectl create serviceaccount <service-account-name> -n <namespace>
```

#### YAML 파일을 사용한 생성
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: custom-service-account
  namespace: default
```

```bash
kubectl apply -f serviceaccount.yaml
```

---

## Service Account를 Pod에 연결하기

Pod가 특정 Service Account를 사용하도록 설정하려면 Pod의 `spec.serviceAccountName` 필드를 설정합니다.

### Service Account를 사용하는 Pod 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod
spec:
  serviceAccountName: custom-service-account
  containers:
    - name: sample-container
      image: nginx
```

1. **`serviceAccountName`**:
   - 해당 Pod에서 사용할 Service Account를 지정
2. **Pod와 Service Account 연결**:
   - Pod는 지정된 Service Account를 사용하여 API 서버와 통신

---

## Service Account 유용한 명령어 모음

- **Service Account 생성**
  ```bash
  kubectl create serviceaccount <service-account-name> -n <namespace>
  ```

- **네임스페이스 내 Service Account 확인**
  ```bash
  kubectl get serviceaccounts -n <namespace>
  ```

- **Service Account 세부 정보 확인**
  ```bash
  kubectl describe serviceaccount <service-account-name> -n <namespace>
  ```

- **Service Account가 사용 중인 Secret 확인**
  ```bash
  kubectl get serviceaccount <service-account-name> -o yaml -n <namespace>
  ```

- **Pod의 Service Account 확인**
  ```bash
  kubectl describe pod <pod-name> | grep ServiceAccount
  ```

---

Service Account는 Kubernetes에서 애플리케이션의 보안성과 API 접근을 관리하는 데 핵심적인 역할을 합니다. 이를 통해 각 애플리케이션에 적절한 권한을 부여하고, 클러스터 보안을 강화할 수 있습니다.
