# ReplicaSets

## 목차
1. [ReplicaSets란 무엇인가?](#ReplicaSets란-무엇인가)
2. [ReplicaSets의 작동 원리](#ReplicaSets의-작동-원리)
3. [ReplicaSets 설정](#ReplicaSets-설정)
4. [ReplicaSets와 Deployments의 차이점](#ReplicaSets와-Deployments의-차이점)
5. [ReplicaSets 관리 및 운영](#ReplicaSets-관리-및-운영)
6. [ReplicaSets 주요 플래그](#ReplicaSets-주요-플래그)
7. [ReplicaSets 유용한 명령어 모음](#ReplicaSets-유용한-명령어-모음)

---

## ReplicaSets란 무엇인가?

**ReplicaSets**는 Kubernetes에서 **Pod의 특정 수를 항상 유지**하도록 하는 오브젝트입니다. 특정 수의 Pod가 실행 중인지 지속적으로 확인하고 관리하며, Pod가 종료되거나 실패한 경우 새로운 Pod를 생성하여 목표 수를 유지합니다. ReplicaSets는 주로 **Deployments**에서 사용하지만, 단독으로 생성하여 사용할 수도 있습니다.

- **기능**: Pod의 복제본(Replicas) 수를 관리하여 애플리케이션의 가용성과 안정성 보장
- **목표 상태 유지**: ReplicaSets는 설정된 목표 Pod 수를 항상 유지하며, 이를 위해 필요 시 Pod 생성 또는 삭제 수행
- **자동화된 스케일링**: 설정된 복제본 수에 따라 Pod를 생성하거나 삭제하여 트래픽 증가에 대응

---

## ReplicaSets의 작동 원리

ReplicaSets는 **Label Selector**를 사용해 관리할 Pod를 지정하고, 이를 통해 특정 레이블이 부여된 Pod 수를 설정된 복제본 수와 비교하여 생성 또는 삭제 작업을 수행합니다. Pod가 종료되거나 비정상적으로 동작하면 이를 감지하여 새로운 Pod를 생성하고, 설정된 목표 Pod 수와 일치하지 않으면 목표에 도달할 때까지 조정 작업을 수행합니다.

1. **Label Selector**: 특정 레이블을 기반으로 관리할 Pod를 선택
   - `matchLabels`: 키-값 쌍으로 정확히 일치하는 레이블을 가진 Pod를 선택
   - `matchExpressions`: 여러 조건식을 통해 보다 유연하게 레이블이 일치하는 Pod를 선택
2. **목표 상태와 실제 상태 비교**: 현재 실행 중인 Pod 수와 설정된 복제본 수를 비교하여 조정 작업 수행
3. **자동 복구**: 관리 중인 Pod가 종료되거나 실패한 경우 자동으로 새로운 Pod 생성

---

## ReplicaSets 설정

ReplicaSets는 YAML 파일을 사용해 정의하며, `metadata`, `spec` 섹션에 주요 설정이 포함됩니다. 

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-container
          image: nginx:latest
          ports:
            - containerPort: 80
```

### 주요 설정 항목

1. **metadata**: ReplicaSets의 이름, 네임스페이스, 레이블 등 메타데이터 설정
2. **spec.replicas**: 생성하고자 하는 Pod의 복제본 수 설정
3. **spec.selector**: 관리할 Pod를 선택하는 Label Selector 정의
   - **matchLabels**: 특정 레이블과 정확히 일치하는 Pod를 선택
   - **matchExpressions**: 조건을 조합해 보다 유연한 레이블 매칭 가능
4. **template**: 생성할 Pod의 템플릿을 정의하며, 각 컨테이너의 이름, 이미지, 포트를 지정

### Label Selector 예시

Label Selector를 통해 다양한 조건으로 관리할 Pod를 선택할 수 있습니다.

```yaml
selector:
  matchLabels:
    app: my-app
```

또는 복잡한 조건을 지정할 수 있습니다.

```yaml
selector:
  matchExpressions:
    - key: tier
      operator: In
      values:
        - frontend
        - backend
    - key: environment
      operator: NotIn
      values:
        - dev
```

위 예시는 `tier` 레이블이 `frontend` 또는 `backend`인 Pod를 선택하며, `environment` 레이블이 `dev`가 아닌 Pod만 선택합니다.

---

## ReplicaSets와 Deployments의 차이점

1. **ReplicaSets**:
   - Pod의 수만을 관리하고, 새로운 애플리케이션 버전을 롤아웃하는 기능은 없음
   - 수동으로 생성해 사용할 수 있지만, 일반적으로 **Deployments**에서 사용
   - 각 ReplicaSets는 동일한 Pod 사양을 유지하며 버전 업데이트 기능은 없음

2. **Deployments**:
   - 애플리케이션의 버전 관리와 롤아웃/롤백 기능 제공
   - 새로운 버전 배포 시 새로운 ReplicaSets를 생성하여 기존 ReplicaSets를 교체
   - 애플리케이션의 점진적 업데이트와 롤백 작업 가능

---

## ReplicaSets 관리 및 운영

1. **ReplicaSets 생성**
   ```bash
   kubectl apply -f replicaset.yaml
   ```

2. **ReplicaSets 상태 확인**
   ```bash
   kubectl get replicasets
   ```

3. **ReplicaSets 스케일링**
   - 스케일링을 통해 레플리카 수를 조정하여 Pod 수 변경 가능
   ```bash
   kubectl scale replicaset <replicaset-name> --replicas=5
   ```

4. **ReplicaSets 삭제**
   - ReplicaSets를 삭제해도 생성된 Pod는 유지
   ```bash
   kubectl delete replicaset <replicaset-name>
   ```

5. **ReplicaSets에 의해 관리되는 Pod 삭제**
   - ReplicaSets는 목표 수를 유지하기 위해 삭제된 Pod를 자동으로 다시 생성
   ```bash
   kubectl delete pod <pod-name>
   ```

---

## ReplicaSets 주요 플래그

1. **--replicas**:  
   - 원하는 Pod의 수 설정
   - 기본값은 `1`

2. **--selector**:  
   - 관리할 Pod를 레이블 셀렉터로 지정
   - `matchLabels`와 `matchExpressions` 두 가지 옵션으로 정의 가능

3. **--min-ready-seconds**:  
   - 새로운 Pod가 Ready 상태로 유지되어야 하는 최소 시간(초 단위) 설정
   - 기본값은 `0`

4. **--template**:  
   - 생성할 Pod의 템플릿 정의
   - 컨테이너 이미지, 포트, 자원 요청 등을 설정 가능

---

## ReplicaSets 유용한 명령어 모음

- **ReplicaSets 생성**
  ```bash
  kubectl apply -f replicaset.yaml
  ```

- **ReplicaSets 상태 확인**
  ```bash
  kubectl get replicasets
  ```

- **ReplicaSets 상세 정보 조회**
  ```bash
  kubectl describe replicaset <replicaset-name>
  ```

- **ReplicaSets 스케일링**
  ```bash
  kubectl scale replicaset <replicaset-name> --replicas=5
  ```

- **ReplicaSets 삭제**
  ```bash
  kubectl delete replicaset <replicaset-name>
  ```
