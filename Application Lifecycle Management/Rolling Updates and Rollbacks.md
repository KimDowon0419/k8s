# Rolling Updates and Rollbacks

## 목차
1. [Rolling Updates와 Rollbacks란 무엇인가?](#Rolling-Updates와-Rollbacks란-무엇인가)
2. [Rolling Updates의 목적과 작동 원리](#Rolling-Updates의-목적과-작동-원리)
3. [Rollbacks의 목적과 작동 원리](#Rollbacks의-목적과-작동-원리)
4. [Rolling Updates 설정 방법](#Rolling-Updates-설정-방법)
5. [Rollback 설정 방법](#Rollback-설정-방법)
6. [Rolling Updates와 Rollbacks 유용한 명령어 모음](#Rolling-Updates와-Rollbacks-유용한-명령어-모음)

---

## Rolling Updates와 Rollbacks란 무엇인가?

**Rolling Updates**와 **Rollbacks**는 Kubernetes에서 **애플리케이션의 점진적 배포와 버전 복구**를 관리하는 기능입니다. Rolling Update는 새 버전의 애플리케이션을 순차적으로 배포하여 **무중단 서비스**를 지원하며, Rollback은 배포 중 문제가 발생했을 때 이전 버전으로 되돌리는 기능입니다.

- **Rolling Updates**: 새로운 버전을 단계적으로 배포하여 서비스 중단 없이 애플리케이션을 업데이트
- **Rollbacks**: 배포 실패 시, 문제 발생 전 상태로 복구하여 서비스 안정성을 유지

---

## Rolling Updates의 목적과 작동 원리

**Rolling Updates**는 애플리케이션의 **새 버전을 점진적으로 배포하여 다운타임을 최소화**하는 방식입니다. 기존 버전을 중단하지 않고 새 버전을 배포하며, 새로운 버전의 Pod가 준비될 때마다 기존 Pod가 순차적으로 교체됩니다.

1. **목적**: 애플리케이션 업데이트 시 무중단 서비스를 보장
2. **작동 원리**:
   - 새로운 버전의 ReplicaSet이 생성되고, 기존 ReplicaSet의 Pod가 순차적으로 대체
   - 새로운 Pod가 준비되면 이전 버전의 Pod가 삭제됨
3. **관리 용이성**: `kubectl apply` 명령으로 YAML 파일 업데이트, 혹은 `kubectl set image`로 이미지 버전 변경 가능

### Rolling Update 예시

Rolling Update는 기본적으로 `kubectl apply` 명령어로 새로운 YAML 파일을 적용하거나, `kubectl set image` 명령으로 컨테이너 이미지를 변경하여 실행할 수 있습니다.

```bash
kubectl set image deployment/my-deployment my-container=my-image:v2
```

---

## Rollbacks의 목적과 작동 원리

**Rollbacks**는 새로운 버전 배포가 실패하거나 예상치 못한 문제가 발생했을 때 **이전 버전으로 빠르게 복구**하는 기능입니다. Rollback은 새로운 버전 배포 시 유지된 기록을 통해 이루어지며, Rollback 실행 시 바로 이전 상태로 복구할 수 있습니다.

1. **목적**: 문제 발생 시 안정적인 이전 버전으로 신속하게 복구
2. **작동 원리**:
   - Kubernetes는 Deployment 버전 기록을 유지하며, `kubectl rollout undo` 명령을 통해 이전 버전으로 되돌림
   - 기본적으로 마지막 배포 기록을 복구하지만, 특정 버전으로도 되돌릴 수 있음
3. **관리 용이성**: Rollback은 `kubectl rollout undo` 명령으로 손쉽게 실행 가능

### Rollback 예시

배포 중 문제가 발생한 경우 다음 명령어로 Rollback을 실행하여 이전 버전으로 되돌립니다.

```bash
kubectl rollout undo deployment/my-deployment
```

---

## Rolling Updates 설정 방법

Rolling Updates는 Deployment의 YAML 파일을 수정하여 `spec.strategy.type`을 `RollingUpdate`로 설정하고, `maxUnavailable` 및 `maxSurge` 파라미터로 업데이트 전략을 세밀하게 조정할 수 있습니다.

### YAML 예시

아래는 Rolling Update 전략을 적용한 Deployment YAML 파일입니다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1   # 업데이트 중 사용 불가한 Pod의 최대 개수
      maxSurge: 1         # 새롭게 추가될 수 있는 Pod의 최대 개수
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-container
          image: my-image:v2
```

---

## Rollback 설정 방법

Rollback은 배포 기록을 통해 특정 시점의 버전으로 되돌릴 수 있으며, 기본적으로 `kubectl rollout undo` 명령어를 사용합니다.

1. **최신 상태로 복구**
   ```bash
   kubectl rollout undo deployment/my-deployment
   ```

2. **특정 버전으로 복구**
   - `--to-revision` 플래그를 사용해 특정 버전으로 되돌립니다.
   ```bash
   kubectl rollout undo deployment/my-deployment --to-revision=2
   ```

---

## Rolling Updates와 Rollbacks 유용한 명령어 모음

- **Rolling Update 시작 (이미지 버전 업데이트)**
  ```bash
  kubectl set image deployment/my-deployment my-container=my-image:v2
  ```

- **Rollout 상태 확인**
  ```bash
  kubectl rollout status deployment/my-deployment
  ```

- **Rollback 실행 (이전 버전으로 복구)**
  ```bash
  kubectl rollout undo deployment/my-deployment
  ```

- **특정 버전으로 Rollback**
  ```bash
  kubectl rollout undo deployment/my-deployment --to-revision=2
  ```

- **Deployment의 버전 기록 확인**
  ```bash
  kubectl rollout history deployment/my-deployment
  ```

Rolling Updates와 Rollbacks는 Kubernetes에서 무중단 배포와 신속한 복구를 가능하게 하는 중요한 기능으로, 애플리케이션의 안정성을 유지하는 데 핵심 역할을 합니다.
