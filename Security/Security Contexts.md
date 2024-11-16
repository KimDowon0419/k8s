# Security Contexts

## 목차
1. [Security Context란 무엇인가?](#Security-Context란-무엇인가)
2. [Security Context의 목적](#Security-Context의-목적)
3. [Security Context의 주요 설정](#Security-Context의-주요-설정)
4. [Pod에 Security Context 적용하기](#Pod에-Security-Context-적용하기)
5. [컨테이너에 Security Context 적용하기](#컨테이너에-Security-Context-적용하기)
6. [Security Context 유용한 명령어 모음](#Security-Context-유용한-명령어-모음)

---

## Security Context란 무엇인가?

**Security Context**는 Kubernetes에서 **Pod 또는 컨테이너가 실행되는 환경의 보안 설정**을 정의하는 필드입니다. 이를 통해 실행 권한, 네트워크 설정, 파일 시스템 권한 등을 세부적으로 제어할 수 있습니다.

- **기능**:
  - Pod 및 컨테이너에 보안 설정 적용
  - 최소 권한 원칙(Principle of Least Privilege) 구현
  - 네트워크 및 파일 시스템 격리 강화

---

## Security Context의 목적

1. **권한 제어**:
   - 특정 사용자 ID(User ID, UID) 또는 그룹 ID(Group ID, GID)로 컨테이너 실행
   - 컨테이너 내에서 루트 권한 사용 방지

2. **네트워크 격리**:
   - 네트워크 기능 제한 및 불필요한 네트워크 권한 제거

3. **파일 시스템 보안**:
   - 읽기 전용 파일 시스템 적용
   - 특정 볼륨에 대한 파일 권한 설정

4. **보안 표준 준수**:
   - 조직의 보안 정책 및 규정을 준수하도록 환경 설정

---

## Security Context의 주요 설정

### 주요 필드 설명

| **필드**                | **설명**                                                                                   |
|-------------------------|--------------------------------------------------------------------------------------------|
| `runAsUser`            | 컨테이너를 실행할 사용자 ID 지정                                                             |
| `runAsGroup`           | 컨테이너를 실행할 그룹 ID 지정                                                              |
| `fsGroup`              | 컨테이너가 액세스하는 볼륨의 기본 그룹 ID 설정                                               |
| `privileged`           | 컨테이너를 특권 모드로 실행 (루트 액세스 제공)                                                |
| `allowPrivilegeEscalation` | 컨테이너가 권한 상승을 허용할지 여부 설정                                                  |
| `readOnlyRootFilesystem` | 컨테이너의 루트 파일 시스템을 읽기 전용으로 설정                                            |
| `capabilities`         | 컨테이너에 허용되거나 제거할 리눅스 기능(캡빌리티) 설정                                      |

---

## Pod에 Security Context 적용하기

Pod 수준에서 Security Context를 설정하면 해당 Pod의 모든 컨테이너에 동일한 보안 설정이 적용됩니다.

### Pod Security Context 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
    - name: nginx
      image: nginx
```

1. **`runAsUser`**: Pod 내 모든 컨테이너를 사용자 ID 1000으로 실행
2. **`runAsGroup`**: 그룹 ID 3000으로 실행
3. **`fsGroup`**: 볼륨의 파일 권한 그룹을 2000으로 설정

---

## 컨테이너에 Security Context 적용하기

컨테이너 수준에서 Security Context를 설정하면 해당 컨테이너에만 보안 설정이 적용됩니다.

### 컨테이너 Security Context 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: container-security-context-pod
spec:
  containers:
    - name: nginx
      image: nginx
      securityContext:
        allowPrivilegeEscalation: false
        capabilities:
          add: ["NET_ADMIN", "SYS_TIME"]
          drop: ["ALL"]
        runAsUser: 1000
        readOnlyRootFilesystem: true
```

1. **`allowPrivilegeEscalation`**: 권한 상승 방지
2. **`capabilities`**:
   - `add`: 컨테이너에 추가할 리눅스 기능
   - `drop`: 제거할 리눅스 기능 (기본적으로 모든 기능 제거)
3. **`readOnlyRootFilesystem`**: 루트 파일 시스템을 읽기 전용으로 설정
4. **`runAsUser`**: 사용자 ID 1000으로 컨테이너 실행

---

## Security Context 유용한 명령어 모음

- **Pod의 Security Context 확인**
  ```bash
  kubectl describe pod <pod-name>
  ```

- **Security Context가 적용된 Pod 생성**
  ```bash
  kubectl apply -f security-context-pod.yaml
  ```

- **Pod의 컨테이너 로그 확인**
  ```bash
  kubectl logs <pod-name> -c <container-name>
  ```

- **Pod 삭제**
  ```bash
  kubectl delete pod <pod-name>
  ```

---

Security Context는 Kubernetes에서 애플리케이션의 실행 환경을 안전하게 제어하고, 보안 사고를 예방하는 데 중요한 역할을 합니다. 이를 통해 컨테이너 권한 제어, 네트워크 제한, 파일 시스템 보안을 효과적으로 강화할 수 있습니다.
