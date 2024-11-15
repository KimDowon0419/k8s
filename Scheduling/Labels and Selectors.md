# Labels and Selectors

## 목차
1. [Labels란 무엇인가?](#Labels란-무엇인가)
2. [Selectors란 무엇인가?](#Selectors란-무엇인가)
3. [Labels와 Selectors의 사용 목적](#Labels와-Selectors의-사용-목적)
4. [Labels 설정 방법](#Labels-설정-방법)
5. [Selectors 사용 방법](#Selectors-사용-방법)
6. [Labels와 Selectors 유용한 명령어 모음](#Labels와-Selectors-유용한-명령어-모음)

---

## Labels란 무엇인가?

**Labels**는 Kubernetes 오브젝트에 **키-값 쌍으로 추가되는 메타데이터**로, 각 리소스를 구분하고 분류하는 데 사용됩니다. Label은 Pod, Service, Deployment 등 모든 리소스에 적용할 수 있으며, 특정 조건의 리소스를 필터링하거나 선택할 때 매우 유용합니다.

- **기능**: 리소스의 상태, 목적, 버전 등의 속성을 표시하여 각 리소스를 관리하기 쉽게 구분
- **구성 방식**: 키와 값의 쌍(`key=value`) 형태로 리소스에 추가
- **다중 레이블**: 리소스에 여러 레이블을 추가해 다양한 기준으로 필터링 가능

### Label 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: web
    environment: production
spec:
  containers:
    - name: nginx
      image: nginx
```

위 예시에서 `app: web`, `environment: production` 레이블이 추가되어 있습니다.

---

## Selectors란 무엇인가?

**Selectors**는 레이블을 기반으로 **특정 조건의 리소스를 선택하는 쿼리 방식**입니다. Label Selector는 Label을 사용하여 지정된 조건과 일치하는 오브젝트를 필터링하거나 집합을 정의하는 데 사용됩니다. Service나 ReplicaSet과 같은 리소스는 Selectors를 통해 특정 레이블을 가진 Pod와 연결됩니다.

- **기능**: 리소스를 선택하고 필터링하여 특정 조건에 맞는 오브젝트만 선택
- **타입**: `matchLabels`와 `matchExpressions` 두 가지 방식으로 조건 설정 가능
  - **`matchLabels`**: 정확히 일치하는 키-값 쌍으로 리소스 선택
  - **`matchExpressions`**: 여러 조건식을 사용해 복잡한 논리로 리소스 필터링

### Selector 예시

```yaml
selector:
  matchLabels:
    app: web
    environment: production
```

위 예시는 `app=web` 및 `environment=production` 레이블을 가진 리소스를 선택하는 Selector입니다.

---

## Labels와 Selectors의 사용 목적

Labels와 Selectors는 Kubernetes에서 **리소스를 그룹화하고 필터링**하여 다양한 관리 작업을 쉽게 수행할 수 있도록 돕습니다. 

1. **리소스 구분과 분류**: 레이블을 통해 환경, 버전, 애플리케이션 유형 등을 구분
2. **자동화된 리소스 선택**: 특정 레이블을 가진 리소스만 선택해 서비스나 ReplicaSet과 연결
3. **유연한 리소스 관리**: Selectors를 통해 리소스의 특정 상태나 속성에 따라 작업 수행

---

## Labels 설정 방법

Labels는 리소스를 생성할 때 또는 생성 후에 추가할 수 있습니다.

### YAML 파일로 Label 설정

YAML 파일을 통해 리소스 생성 시 Label을 함께 설정할 수 있습니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: web
    environment: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
        environment: production
    spec:
      containers:
        - name: nginx
          image: nginx
```

### 명령어로 Label 추가

기존 리소스에 Label을 추가하려면 다음 명령어를 사용합니다.

```bash
kubectl label pod my-pod app=web environment=production
```

---

## Selectors 사용 방법

Selectors는 리소스를 선택하는 데 사용됩니다. `matchLabels`와 `matchExpressions`를 사용해 다양한 조건으로 리소스를 필터링할 수 있습니다.

### `matchLabels` 사용 예시

**`matchLabels`**는 정확히 일치하는 레이블을 가진 리소스를 선택합니다.

```yaml
selector:
  matchLabels:
    app: web
    environment: production
```

### `matchExpressions` 사용 예시

**`matchExpressions`**는 더 복잡한 조건을 사용하여 레이블을 필터링합니다.

```yaml
selector:
  matchExpressions:
    - key: environment
      operator: In
      values:
        - production
        - staging
    - key: app
      operator: NotIn
      values:
        - backend
```

위 예시는 `environment`가 `production` 또는 `staging`인 리소스 중에서 `app`이 `backend`가 아닌 리소스를 선택합니다.

---

## Labels와 Selectors 유용한 명령어 모음

- **Label 추가**
  ```bash
  kubectl label pod my-pod app=web environment=production
  ```

- **Label 삭제**
  ```bash
  kubectl label pod my-pod environment-
  ```

- **특정 Label을 가진 리소스 조회**
  ```bash
  kubectl get pods -l app=web
  ```

- **다중 Label 조건을 만족하는 리소스 조회**
  ```bash
  kubectl get pods -l app=web,environment=production
  ```

- **Selector로 관리되는 리소스 조회**
  ```bash
  kubectl get pods --selector app=web
  ```
