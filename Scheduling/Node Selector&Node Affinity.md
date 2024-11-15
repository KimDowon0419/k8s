# Node Selector & Node Affinity

## 목차
1. [Node Selector와 Node Affinity란 무엇인가?](#Node-Selector와-Node-Affinity란-무엇인가)
2. [Node Selector와 Node Affinity의 목적](#Node-Selector와-Node-Affinity의-목적)
3. [Node Selector 사용 방법](#Node-Selector-사용-방법)
4. [Node Affinity 사용 방법](#Node-Affinity-사용-방법)
5. [Node Selector & Node Affinity 유용한 명령어 모음](#Node-Selector--Node-Affinity-유용한-명령어-모음)

---

## Node Selector와 Node Affinity란 무엇인가?

**Node Selector**와 **Node Affinity**는 Kubernetes에서 **Pod를 특정 조건의 노드에만 배치**할 수 있도록 설정하는 메커니즘입니다. 

- **Node Selector**는 노드의 레이블(Label)을 기반으로 특정 조건을 만족하는 노드에만 Pod를 배치하는 가장 단순한 방식입니다.
- **Node Affinity**는 Node Selector와 유사하지만 더 복잡한 조건과 제어를 제공하는 방식입니다. Node Affinity는 여러 조건을 결합하거나 선호 조건을 설정할 수 있어 좀 더 유연하게 노드를 선택할 수 있습니다.

---

## Node Selector와 Node Affinity의 목적

Node Selector와 Node Affinity는 다음과 같은 목적을 가지고 사용됩니다.

1. **리소스 최적화**: 특정 조건을 만족하는 노드에만 리소스를 집중하여 성능을 최적화
2. **리소스 격리**: 데이터베이스, 캐시 등 민감한 애플리케이션을 특정 노드에만 배치하여 자원을 보호
3. **특수 환경 지원**: GPU 노드, 고성능 SSD 노드와 같은 특정 하드웨어 요구 사항이 있는 애플리케이션을 해당 노드에만 배치

---

## Node Selector 사용 방법

Node Selector는 **단순한 키-값 쌍 조건**을 사용해 특정 노드에만 Pod를 배치할 때 사용됩니다. Node Selector는 Pod의 YAML 파일에서 `nodeSelector` 필드로 설정할 수 있습니다.

### Node Selector 예시

아래 예시는 `disktype=ssd`라는 레이블을 가진 노드에만 Pod를 배치하는 설정입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  nodeSelector:
    disktype: ssd
  containers:
    - name: nginx
      image: nginx
```

Node Selector는 조건이 매우 단순하여 복잡한 조건을 설정할 수 없지만, 빠르고 쉽게 특정 노드에 Pod를 배치할 수 있습니다.

---

## Node Affinity 사용 방법

Node Affinity는 Node Selector보다 더 **복잡한 조건**을 설정할 수 있는 기능을 제공합니다. Node Affinity는 Pod의 YAML 파일 내에서 `affinity` 섹션에 설정할 수 있으며, `requiredDuringSchedulingIgnoredDuringExecution`과 `preferredDuringSchedulingIgnoredDuringExecution` 두 가지 방식이 있습니다.

### Node Affinity 유형

1. **requiredDuringSchedulingIgnoredDuringExecution**:
   - 필수 조건으로, 지정된 조건을 만족하는 노드에만 Pod를 스케줄링합니다.
   - 조건을 만족하지 않는 노드에는 스케줄링되지 않음.
   
2. **preferredDuringSchedulingIgnoredDuringExecution**:
   - 조건을 만족하는 노드에 스케줄링을 **선호**하지만, 조건을 만족하는 노드가 없는 경우 다른 노드에 스케줄링될 수도 있음.

### Node Affinity 설정 예시

#### `requiredDuringSchedulingIgnoredDuringExecution` 예시

아래 예시는 `disktype=ssd`라는 레이블을 가진 노드에만 Pod를 필수적으로 배치하는 설정입니다.

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

#### `preferredDuringSchedulingIgnoredDuringExecution` 예시

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

- **`matchExpressions`**: 레이블의 키와 값을 설정하여 조건을 지정합니다.
- **`operator`**: `In`, `NotIn`, `Exists` 등 연산자를 사용해 조건을 설정할 수 있습니다.
- **`weight`**: 여러 선호 조건이 있을 때 우선순위를 지정하여 특정 조건에 높은 우선순위를 부여할 수 있습니다.

---

## Node Selector & Node Affinity 유용한 명령어 모음

- **Node에 레이블 추가**
  ```bash
  kubectl label nodes <node-name> disktype=ssd
  ```

- **Node 레이블 확인**
  ```bash
  kubectl get nodes --show-labels
  ```

- **Node Selector 또는 Node Affinity가 설정된 Pod 생성**
  ```bash
  kubectl apply -f node-affinity-pod.yaml
  ```

- **Pod의 Node Affinity 확인**
  ```bash
  kubectl describe pod <pod-name> | grep -A 5 "Node-Selectors"
  ```

Node Selector와 Node Affinity는 Kubernetes 클러스터 내 특정 노드에 리소스를 배치하거나 특정 조건의 노드를 회피하는 데 유용합니다.
