# Cluster Roles and Role Binding

## 목차
1. [ClusterRole과 Role의 차이](#ClusterRole과-Role의-차이)
2. [ClusterRole이란 무엇인가?](#ClusterRole이란-무엇인가)
3. [RoleBinding과 ClusterRoleBinding의 차이](#RoleBinding과-ClusterRoleBinding의-차이)
4. [ClusterRole과 ClusterRoleBinding 설정 방법](#ClusterRole과-ClusterRoleBinding-설정-방법)
5. [ClusterRole과 ClusterRoleBinding 유용한 명령어 모음](#ClusterRole과-ClusterRoleBinding-유용한-명령어-모음)

---

## ClusterRole과 Role의 차이

| **특징**             | **Role**                                              | **ClusterRole**                                      |
|-----------------------|-------------------------------------------------------|-----------------------------------------------------|
| **적용 범위**        | 특정 네임스페이스                                      | 클러스터 전체 또는 특정 네임스페이스                |
| **관리 가능 리소스** | 네임스페이스 리소스                                    | 클러스터 리소스와 네임스페이스 리소스 모두 관리 가능 |
| **사용 목적**        | 네임스페이스 단위의 리소스 관리                        | 클러스터 레벨 리소스 관리 또는 다중 네임스페이스 관리 |

---

## ClusterRole이란 무엇인가?

**ClusterRole**은 Kubernetes에서 **클러스터 전체에서 리소스에 대한 권한을 정의**하는 리소스입니다. ClusterRole은 다음과 같은 작업을 수행하는 데 사용됩니다.

1. **클러스터 리소스 관리**:
   - 노드, PersistentVolume, 클러스터 전체에서 사용되는 리소스 관리
2. **네임스페이스 리소스 관리**:
   - 특정 네임스페이스가 아닌 모든 네임스페이스에 대한 리소스 접근 권한 부여
3. **읽기 전용 권한**:
   - 모든 네임스페이스에서 Pod, Service 등을 읽기 전용으로 설정

### ClusterRole 예시

아래는 모든 네임스페이스에서 Pod를 읽을 수 있는 ClusterRole의 정의입니다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

- **`apiGroups`**: 관리할 리소스의 API 그룹 (기본 리소스는 빈 문자열)
- **`resources`**: 관리할 리소스 종류 (예: `pods`, `services`)
- **`verbs`**: 허용된 작업 (예: `get`, `list`, `watch`)

---

## RoleBinding과 ClusterRoleBinding의 차이

| **특징**                     | **RoleBinding**                              | **ClusterRoleBinding**                              |
|-------------------------------|-----------------------------------------------|----------------------------------------------------|
| **연결할 Role 유형**          | Role                                          | ClusterRole                                       |
| **적용 범위**                | 특정 네임스페이스                             | 클러스터 전체 또는 특정 네임스페이스               |
| **사용 목적**                | 네임스페이스 리소스 접근 권한 연결             | 클러스터 또는 다중 네임스페이스 리소스 접근 권한 연결 |

---

## ClusterRole과 ClusterRoleBinding 설정 방법

ClusterRole과 ClusterRoleBinding은 YAML 파일을 통해 정의합니다.

### ClusterRole 설정 예시

아래는 모든 네임스페이스에서 ConfigMap을 읽을 수 있는 ClusterRole의 정의입니다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: configmap-reader
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "list"]
```

### ClusterRoleBinding 설정 예시

아래는 특정 사용자 `jane`에게 `configmap-reader` ClusterRole을 바인딩하는 ClusterRoleBinding 정의입니다.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-configmaps
subjects:
  - kind: User
    name: jane
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: configmap-reader
  apiGroup: rbac.authorization.k8s.io
```

- **`subjects`**: 권한을 부여받을 사용자, 그룹, 또는 서비스 계정
- **`roleRef`**: 연결할 ClusterRole의 이름과 종류

---

## ClusterRole과 ClusterRoleBinding 유용한 명령어 모음

- **ClusterRole 생성**
  ```bash
  kubectl apply -f clusterrole.yaml
  ```

- **ClusterRoleBinding 생성**
  ```bash
  kubectl apply -f clusterrolebinding.yaml
  ```

- **현재 ClusterRole 확인**
  ```bash
  kubectl get clusterrole
  ```

- **현재 ClusterRoleBinding 확인**
  ```bash
  kubectl get clusterrolebinding
  ```

- **사용자의 권한 확인**
  ```bash
  kubectl auth can-i <verb> <resource> --namespace=<namespace>
  ```

---

ClusterRole과 ClusterRoleBinding은 Kubernetes 클러스터 전체에서 리소스 접근 제어를 설정하는 핵심 도구입니다. 이를 통해 네임스페이스 간의 리소스 관리와 클러스터 전반의 보안 강화를 효과적으로 수행할 수 있습니다.
