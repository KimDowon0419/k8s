# Node Affinity

## 목차
1. [Node Affinity란 무엇인가?](#Node-Affinity란-무엇인가)
2. [Node Affinity의 목적](#Node-Affinity의-목적)
3. [Node Affinity 유형](#Node-Affinity-유형)
4. [Node Affinity 설정 방법](#Node-Affinity-설정-방법)
5. [Node Affinity 유용한 명령어 모음](#Node-Affinity-유용한-명령어-모음)

---

## Node Affinity란 무엇인가?

**Node Affinity**는 Kubernetes에서 **특정 조건을 가진 노드에 Pod를 스케줄링할 수 있도록 제어하는 설정**입니다. Node Affinity는 노드의 레이블을 기반으로 특정 노드에만 Pod를 배치하거나 특정 조건의 노드를 피하는 방식으로 사용됩니다. 이는 **노드의 레이블(Label)**을 사용하여 원하는 노드에 리소스를 집중하거나 특정 노드를 선택적으로 사용해야 할 때 유용합니다.

- **기능**: 노드 레이블을 통해 Pod가 배치될 노드를 제어
- **제어 방식**: 특정 노드에만 배치되거나 특정 조건을 회피하도록 설정 가능
- **사용 목적**: 데이터베이스, 캐시, 전용 애플리케이션 노드와 같은 특정 작업을 위한 노드에 Pod 배치

---

## Node Affinity의 목적

Node Affinity는 특정 조건을 가진 노드에 Pod를 배치해야 하거나 특정 노드를 회피해야 하는 경우에 사용됩니다. 주로 다음과 같은 상황에서 사용됩니다.

1. **특정 노드에 리소스 집중**: 애플리케이션이 높은 성능을 위해 특정 하드웨어 또는 환경을 필요로 할 때
2. **리소스 분리**: 중요한 데이터베이스와 일반 애플리케이션을 별도의 노드에 배치하여 자원 보호
3. **특정 환경 격리**: 개발, 테스트, 운영 환경을 특정 노드로 격리하여 리소스 사용 최적화

---

## Node Affinity 유형

Node Affinity는 크게 두 가지 유형으로 구성됩니다.

1. **requiredDuringSchedulingIgnoredDuringExecution**:
   - 지정된 조건을 만족하는 노드에만 Pod를 스케줄링하며, 필수 조건으로 적용됩니다.
   - 조건을 만족하지 않는 노드에는 스케줄링되지 않음.
   
2. **preferredDuringSchedulingIgnoredDuringExecution**:
   - 조건을 만족하는 노드에 스케줄링하는 것을 **선호**하지만, 반드시 해당 조건을 만족할 필요는 없습니다.
   - 조건을 만족하는 노드가 없으면 조건을 무시하고 다른 노드에 스케줄링될 수 있음.

---

## Node Affinity 설정 방법

Node Affinity는 Pod의 YAML 파일 내에서 `affinity` 섹션을 사용하여 설정합니다. 

### `requiredDuringSchedulingIgnoredDuringExecution` 예시

아래 예시는 `disktype=ssd`라는 레이블을 가진 노드에만 Pod를 배치하는 설정입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd
  containers:
    - name: nginx
      image: nginx
```

### `preferredDuringSchedulingIgnoredDuringExecution` 예시

아래 예시는 `disktype=ssd` 노드에 배치를 선호하되, 조건을 만족하는 노드가 없으면 다른 노드에도 배치될 수 있도록 설정합니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
            matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd
  containers:
    - name: nginx
      image: nginx
```

1. **`matchExpressions`**: 레이블의 키와 값을 지정하여 조건을 설정합니다.
2. **`operator`**: `In`, `NotIn`, `Exists` 등의 연산자를 사용하여 조건을 정의할 수 있습니다.
3. **`weight`**: 여러 선호 조건을 지정할 때, 우선순위를 나타내는 값으로 1~100 사이의 값을 설정하여 특정 조건에 더 높은 우선순위를 부여할 수 있습니다.

---

## Node Affinity 유용한 명령어 모음

- **Node에 레이블 추가**
  ```bash
  kubectl label nodes <node-name> disktype=ssd
  ```

- **Node 레이블 확인**
  ```bash
  kubectl get nodes --show-labels
  ```

- **Node Affinity가 설정된 Pod 생성**
  ```bash
  kubectl apply -f node-affinity-pod.yaml
  ```

- **Pod의 Node Affinity 확인**
  ```bash
  kubectl describe pod <pod-name> | grep -A 5 "Node-Selectors"
  ```
