# Namespaces

## 목차
1. [Namespaces란 무엇인가?](#Namespaces란-무엇인가)
2. [Namespaces의 작동 원리](#Namespaces의-작동-원리)
3. [Namespaces 설정](#Namespaces-설정)
4. [Namespaces 관리 및 운영](#Namespaces-관리-및-운영)
5. [Namespaces 주요 플래그](#Namespaces-주요-플래그)
6. [Namespaces 유용한 명령어 모음](#Namespaces-유용한-명령어-모음)

---

## Namespaces란 무엇인가?

**Namespaces**는 Kubernetes 클러스터에서 **리소스들을 논리적으로 그룹화하고 격리**하는 데 사용됩니다. 동일한 클러스터 내에서 여러 팀이 작업할 때 각 팀의 리소스를 독립적으로 관리하고 격리할 수 있도록 하며, 클러스터 리소스(예: Pod, Service 등) 간의 충돌을 방지합니다.

- **기능**: 여러 사용자 또는 팀이 하나의 클러스터를 공유할 수 있도록 리소스를 격리하고 독립적인 공간 제공
- **리소스 구분**: 서로 다른 네임스페이스의 리소스는 이름이 같아도 충돌하지 않음
- **정책 적용**: 리소스 할당, 네트워크 정책 등을 네임스페이스별로 관리 가능

---

## Namespaces의 작동 원리

Namespaces는 **Kubernetes 클러스터 내에서 논리적 구분을 만들어 리소스 그룹을 독립적으로 관리**할 수 있게 합니다. 기본적으로 Kubernetes 클러스터에는 `default`, `kube-system`, `kube-public` 등 몇 가지 네임스페이스가 존재하며, 새로운 네임스페이스를 생성하여 격리된 환경을 추가로 구성할 수 있습니다.

1. **리소스 격리**: 각 네임스페이스는 자체 네임스페이스 범위 내에서만 리소스를 인식하며, 다른 네임스페이스의 리소스와 충돌하지 않음
2. **네트워크 및 정책 관리**: 네트워크 정책, 리소스 할당, 접근 권한 등을 네임스페이스별로 적용 가능
3. **리소스 이름 충돌 방지**: 동일한 이름의 리소스가 네임스페이스별로 존재할 수 있어 중복된 이름으로 인한 충돌 방지

---

## Namespaces 설정

Namespaces는 YAML 파일을 통해 설정되며, `metadata` 섹션에서 네임스페이스의 이름을 지정합니다.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
```

1. **metadata**: 네임스페이스의 이름과 기타 메타데이터 설정
2. **name**: 네임스페이스의 이름을 지정하여 클러스터 내에서 고유하게 식별

네임스페이스를 설정하면 클러스터 내 리소스를 생성할 때 이 네임스페이스 내에서만 리소스를 관리할 수 있습니다.

---

## Namespaces 관리 및 운영

1. **Namespaces 생성**
   ```bash
   kubectl apply -f namespace.yaml
   ```

2. **Namespaces 상태 확인**
   ```bash
   kubectl get namespaces
   ```

3. **Namespace 내 리소스 확인**
   - 특정 네임스페이스 내의 리소스만 필터링하여 확인할 수 있습니다.
   ```bash
   kubectl get pods -n <namespace-name>
   ```

4. **Namespaces 삭제**
   - 네임스페이스를 삭제하면 해당 네임스페이스 내 모든 리소스가 삭제됩니다.
   ```bash
   kubectl delete namespace <namespace-name>
   ```

5. **기본 네임스페이스 설정**
   - `kubectl` 명령어를 사용할 때 `-n` 플래그를 지정하여 명령어가 기본적으로 사용할 네임스페이스를 설정 가능.
   ```bash
   kubectl config set-context --current --namespace=<namespace-name>
   ```

---

## Namespaces 주요 플래그

1. **-n / --namespace**:  
   - 특정 네임스페이스에서 작업을 수행할 때 지정
   - 예: `kubectl get pods -n my-namespace`

2. **--all-namespaces**:  
   - 클러스터의 모든 네임스페이스에서 리소스를 조회할 때 사용
   - 예: `kubectl get pods --all-namespaces`

3. **--context**:  
   - 특정 네임스페이스를 현재 컨텍스트로 설정하여 기본 네임스페이스를 설정할 수 있음
   - 예: `kubectl config set-context --current --namespace=my-namespace`

---

## Namespaces 유용한 명령어 모음

- **Namespaces 생성**
  ```bash
  kubectl apply -f namespace.yaml
  ```

- **Namespaces 상태 확인**
  ```bash
  kubectl get namespaces
  ```

- **Namespace 내 리소스 조회**
  ```bash
  kubectl get pods -n <namespace-name>
  ```

- **모든 네임스페이스의 리소스 조회**
  ```bash
  kubectl get pods --all-namespaces
  ```

- **네임스페이스 삭제**
  ```bash
  kubectl delete namespace <namespace-name>
  ```

- **기본 네임스페이스 설정**
  ```bash
  kubectl config set-context --current --namespace=<namespace-name>
  ```
