# KubeConfig

## 목차
1. [KubeConfig란 무엇인가?](#KubeConfig란-무엇인가)
2. [KubeConfig의 구조](#KubeConfig의-구조)
3. [KubeConfig 생성 및 관리](#KubeConfig-생성-및-관리)
4. [KubeConfig 설정 방법](#KubeConfig-설정-방법)
5. [KubeConfig 유용한 명령어 모음](#KubeConfig-유용한-명령어-모음)

---

## KubeConfig란 무엇인가?

**KubeConfig**는 Kubernetes 클라이언트(`kubectl`)가 **클러스터와 통신하기 위한 정보를 저장하는 구성 파일**입니다. KubeConfig 파일에는 **클러스터, 사용자, 컨텍스트**에 대한 설정이 포함되며, Kubernetes API 서버와 통신할 때 인증 및 권한을 설정하는 데 사용됩니다.

- **기능**:
  - Kubernetes 클러스터에 연결
  - 사용자 인증 및 권한 정보 제공
  - 여러 클러스터 및 컨텍스트 관리

기본적으로 KubeConfig 파일은 `$HOME/.kube/config` 경로에 저장됩니다.

---

## KubeConfig의 구조

KubeConfig 파일은 **클러스터, 사용자, 컨텍스트** 정보로 구성됩니다. 아래는 KubeConfig 파일의 기본 구조입니다.

### 기본 구조 예시

```yaml
apiVersion: v1
kind: Config
clusters:
- name: my-cluster
  cluster:
    server: https://my-cluster-api-server:6443
    certificate-authority: /path/to/ca.crt
users:
- name: admin-user
  user:
    client-certificate: /path/to/client.crt
    client-key: /path/to/client.key
contexts:
- name: my-context
  context:
    cluster: my-cluster
    user: admin-user
current-context: my-context
```

1. **clusters**:
   - Kubernetes 클러스터 정보 (API 서버 URL, 인증서 등)
2. **users**:
   - 클러스터와 통신할 사용자 인증 정보 (클라이언트 인증서, 토큰 등)
3. **contexts**:
   - 클러스터와 사용자를 매핑하여 특정 환경 설정
4. **current-context**:
   - 기본적으로 사용할 컨텍스트를 지정

---

## KubeConfig 생성 및 관리

### 1. 인증 정보로 KubeConfig 생성

Kubernetes 클러스터의 인증 정보를 사용하여 KubeConfig 파일을 생성합니다.

#### 예제: `kubectl` 명령으로 KubeConfig 생성
```bash
kubectl config set-cluster my-cluster \
  --server=https://my-cluster-api-server:6443 \
  --certificate-authority=/path/to/ca.crt

kubectl config set-credentials admin-user \
  --client-certificate=/path/to/client.crt \
  --client-key=/path/to/client.key

kubectl config set-context my-context \
  --cluster=my-cluster \
  --user=admin-user

kubectl config use-context my-context
```

1. **클러스터 설정**:
   - `kubectl config set-cluster`
2. **사용자 인증 정보 설정**:
   - `kubectl config set-credentials`
3. **컨텍스트 설정**:
   - `kubectl config set-context`
4. **컨텍스트 전환**:
   - `kubectl config use-context`

---

## KubeConfig 설정 방법

### 1. KubeConfig 경로 지정

`KUBECONFIG` 환경 변수를 사용하여 KubeConfig 파일 경로를 지정할 수 있습니다.

#### 환경 변수 설정
```bash
export KUBECONFIG=/path/to/kubeconfig
```

### 2. 여러 KubeConfig 병합

여러 KubeConfig 파일을 병합하여 관리할 수 있습니다.

#### 병합 예시
```bash
export KUBECONFIG=/path/to/config1:/path/to/config2
kubectl config view --merge --flatten > /path/to/merged-config
```

### 3. 현재 KubeConfig 확인

현재 KubeConfig 파일의 설정을 확인하려면 다음 명령을 사용합니다.

```bash
kubectl config view
```

---

## KubeConfig 유용한 명령어 모음

- **KubeConfig 설정 보기**
  ```bash
  kubectl config view
  ```

- **현재 컨텍스트 확인**
  ```bash
  kubectl config current-context
  ```

- **컨텍스트 전환**
  ```bash
  kubectl config use-context <context-name>
  ```

- **클러스터 정보 추가**
  ```bash
  kubectl config set-cluster <cluster-name> \
    --server=https://<api-server-url>:6443 \
    --certificate-authority=/path/to/ca.crt
  ```

- **사용자 인증 정보 추가**
  ```bash
  kubectl config set-credentials <user-name> \
    --client-certificate=/path/to/client.crt \
    --client-key=/path/to/client.key
  ```

- **컨텍스트 추가**
  ```bash
  kubectl config set-context <context-name> \
    --cluster=<cluster-name> \
    --user=<user-name>
  ```

- **컨텍스트 삭제**
  ```bash
  kubectl config delete-context <context-name>
  ```

KubeConfig는 Kubernetes 클러스터와의 통신 설정을 저장하고 관리하는 핵심 도구로, 여러 클러스터와 컨텍스트를 효과적으로 관리할 수 있도록 돕습니다.
