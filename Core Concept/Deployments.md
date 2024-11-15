# Deployments

## 목차
1. [Deployments란 무엇인가?](#Deployments란-무엇인가)
2. [Deployments의 작동 원리](#Deployments의-작동-원리)
3. [Deployments 설정](#Deployments-설정)
4. [Deployments의 주요 기능](#Deployments의-주요-기능)
5. [Deployments 관리 및 운영](#Deployments-관리-및-운영)
6. [Deployments 주요 플래그](#Deployments-주요-플래그)
7. [Deployments 유용한 명령어 모음](#Deployments-유용한-명령어-모음)

---

## Deployments란 무엇인가?

**Deployments**는 Kubernetes에서 **애플리케이션의 배포, 롤아웃, 스케일링, 롤백을 관리**하는 오브젝트입니다. 특정 수의 Pod를 항상 유지하며, ReplicaSets를 통해 각 버전의 애플리케이션 상태를 관리합니다. 여러 버전의 애플리케이션을 점진적으로 배포하거나 롤백할 수 있어 신속하고 안전하게 애플리케이션을 배포할 수 있습니다.

- **기능**: 애플리케이션 버전 관리, 수평 스케일링, 롤백, 자동화된 복구
- **관리 방식**: ReplicaSets를 생성하여 Pod의 복제본 수를 조절
- **업데이트 관리**: 롤링 업데이트와 롤백 기능을 통해 애플리케이션의 점진적 배포 및 복구 수행

---

## Deployments의 작동 원리

Deployments는 **원하는 애플리케이션 상태와 현재 상태를 지속적으로 비교하고, 목표 상태로 맞추기 위해 ReplicaSets를 생성 및 관리**합니다. **롤링 업데이트(Rolling Update)** 방식을 사용해 애플리케이션을 점진적으로 업데이트하며, 문제가 발생하면 **롤백(Rollback)** 기능을 통해 신속하게 이전 상태로 복구 가능합니다.

1. **애플리케이션 상태 관리**: 설정된 Pod 수와 상태를 유지하도록 관리하며, ReplicaSets가 이를 지속적으로 유지
2. **롤링 업데이트**: 새로운 애플리케이션 버전을 점진적으로 배포하여 서비스 중단 없이 업데이트
3. **자동 복구**: 비정상적으로 종료된 Pod를 감지하여 목표 상태와 일치하도록 새로 생성

---

## Deployments 설정

Deployments는 YAML 파일을 통해 설정되며, `metadata`, `spec` 섹션에 주요 설정이 포함됩니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
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

1. **metadata**: Deployment의 이름, 네임스페이스, 레이블 설정
2. **spec.replicas**: Pod 복제본 수 설정
3. **spec.selector**: 관리할 Pod를 선택하는 Label Selector
4. **template**: Pod 템플릿 정의로, 생성할 Pod의 컨테이너 이름, 이미지, 포트 지정

### Label Selector 예시

Label Selector는 레이블을 통해 관리할 Pod를 선택하도록 설정합니다.

```yaml
selector:
  matchLabels:
    app: my-app
```

---

## Deployments의 주요 기능

1. **롤링 업데이트 (Rolling Update)**:
   - 기존 버전을 유지하면서 점진적으로 새로운 버전을 배포하여 다운타임 없이 애플리케이션 업데이트
   - 기본 설정으로 사용되며, 새로운 Pod를 생성하고 기존 Pod를 순차적으로 제거하여 서비스 중단 최소화

2. **롤백 (Rollback)**:
   - 문제가 발생한 경우 애플리케이션을 이전 버전으로 되돌림
   - 여러 이전 버전을 관리하여 특정 버전으로 복구 가능

3. **스케일링 (Scaling)**:
   - 수평 스케일링을 통해 트래픽 증가에 대응하여 Pod의 복제본 수를 동적으로 조정
   - `kubectl scale` 명령어 또는 HPA(Horizontal Pod Autoscaler)를 통해 설정 가능

4. **자동화된 복구**:
   - 비정상적으로 종료된 Pod를 자동으로 재생성하여 목표 상태를 유지하고 안정성 보장

---

## Deployments 관리 및 운영

1. **Deployments 생성**
   ```bash
   kubectl apply -f deployment.yaml
   ```

2. **Deployments 상태 확인**
   ```bash
   kubectl get deployments
   ```

3. **Deployments 업데이트**
   - Deployment를 수정하고 다시 적용하여 애플리케이션을 업데이트
   ```bash
   kubectl apply -f deployment.yaml
   ```

4. **롤백 수행**
   - 이전 버전으로 복원
   ```bash
   kubectl rollout undo deployment <deployment-name>
   ```

5. **Deployments 삭제**
   ```bash
   kubectl delete deployment <deployment-name>
   ```

---

## Deployments 주요 플래그

1. **--replicas**:  
   - 원하는 Pod 복제본 수 설정
   - 기본값은 `1`

2. **--strategy**:  
   - 배포 전략을 설정하며, 기본값은 `RollingUpdate`
   - `RollingUpdate`와 `Recreate` 두 가지 모드 제공
   - **RollingUpdate**: 순차적으로 업데이트 수행
   - **Recreate**: 기존 Pod를 모두 삭제한 후 새롭게 생성

3. **--minReadySeconds**:  
   - 새로 생성된 Pod가 `Ready` 상태를 유지해야 하는 최소 시간(초 단위) 설정
   - 기본값은 `0`

4. **--revision-history-limit**:  
   - 유지할 이전 버전의 수를 설정
   - 기본값은 `10`

5. **--progress-deadline-seconds**:  
   - 배포가 완료되지 않았을 때 실패로 간주하는 시간(초 단위) 설정
   - 기본값은 `600초`

6. **--record**:
   - Deployment 변경 이력을 기록하여 롤아웃 시 업데이트 내용을 기록
   - 변경 사항을 쉽게 파악하여 배포 시각 및 내용을 추적 가능

---

## Deployments 유용한 명령어 모음

- **Deployments 생성**
  ```bash
  kubectl apply -f deployment.yaml
  ```

- **Deployments 상태 확인**
  ```bash
  kubectl get deployments
  ```

- **Deployments 상세 정보 조회**
  ```bash
  kubectl describe deployment <deployment-name>
  ```

- **롤아웃 상태 확인**
  ```bash
  kubectl rollout status deployment <deployment-name>
  ```

- **롤백 수행**
  ```bash
  kubectl rollout undo deployment <deployment-name>
  ```

- **스케일링**
  ```bash
  kubectl scale deployment <deployment-name> --replicas=5
  ```

- **Deployments 업데이트 기록**
  ```bash
  kubectl apply -f deployment.yaml --record
  ```

- **Deployments 삭제**
  ```bash
  kubectl delete deployment <deployment-name>
  ```
