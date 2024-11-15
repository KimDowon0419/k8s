# DaemonSets

## 목차
1. [DaemonSets란 무엇인가?](#DaemonSets란-무엇인가)
2. [DaemonSets의 목적](#DaemonSets의-목적)
3. [DaemonSets 사용 사례](#DaemonSets-사용-사례)
4. [DaemonSets 설정 방법](#DaemonSets-설정-방법)
5. [DaemonSets 관리 및 운영](#DaemonSets-관리-및-운영)
6. [DaemonSets 유용한 명령어 모음](#DaemonSets-유용한-명령어-모음)

---

## DaemonSets란 무엇인가?

**DaemonSets**는 Kubernetes에서 **클러스터의 모든 노드에 특정 Pod를 배포하는 컨트롤러**입니다. DaemonSet은 노드가 추가되거나 제거될 때마다 자동으로 Pod를 배포하거나 삭제하여, 클러스터 내 모든 노드에 항상 동일한 Pod가 유지되도록 합니다. 이를 통해 로깅, 모니터링, 시스템 관리 등의 기능을 모든 노드에 동일하게 적용할 수 있습니다.

- **기능**: 클러스터 내 모든 노드에 지정된 Pod 배포
- **자동 업데이트**: 새로운 노드가 추가되면 자동으로 해당 노드에 Pod 생성
- **관리 편의성**: 특정 애플리케이션을 모든 노드에 적용하기 위해 별도의 배포 작업이 필요 없음

---

## DaemonSets의 목적

DaemonSets는 클러스터 내 모든 노드에서 동일한 애플리케이션을 실행해야 하는 상황에서 사용됩니다. 다음과 같은 목적으로 사용됩니다.

1. **시스템 로깅**: 모든 노드에서 로그를 수집하기 위해 로깅 에이전트를 배포할 때
2. **모니터링**: 모든 노드의 리소스 상태를 모니터링하기 위해 모니터링 에이전트를 배포할 때
3. **노드 관리 작업**: 각 노드의 설정이나 보안 작업을 자동으로 적용할 때

---

## DaemonSets 사용 사례

DaemonSets는 주로 다음과 같은 작업을 위해 사용됩니다.

1. **로그 수집**: 각 노드에서 로그 데이터를 수집하기 위한 Fluentd, Filebeat 등의 로깅 에이전트 배포
2. **모니터링 에이전트**: Prometheus Node Exporter와 같은 노드 모니터링 에이전트를 배포하여 각 노드의 리소스 상태 모니터링
3. **보안 에이전트**: 보안 에이전트를 배포하여 각 노드에서 실시간으로 보안 상태를 감시
4. **스토리지 관리**: 각 노드의 디스크 관리나 스토리지 설정을 위한 스토리지 관리 에이전트 배포

---

## DaemonSets 설정 방법

DaemonSets는 YAML 파일을 통해 설정할 수 있으며, `metadata`, `spec` 섹션에 주요 설정이 포함됩니다.

### DaemonSet 예시

아래 예시는 모든 노드에서 `fluentd` 로깅 에이전트를 배포하는 DaemonSet의 설정 예시입니다.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
        - name: fluentd
          image: fluentd:latest
          resources:
            limits:
              memory: "200Mi"
              cpu: "100m"
```

1. **metadata**: DaemonSet의 이름과 레이블을 설정합니다.
2. **spec.selector**: DaemonSet이 관리할 Pod를 선택하는 Label Selector를 설정합니다.
3. **spec.template**: 생성할 Pod의 템플릿을 정의하여, 컨테이너 이름, 이미지, 자원 제한 등을 설정합니다.

---

## DaemonSets 관리 및 운영

1. **DaemonSets 생성**
   ```bash
   kubectl apply -f daemonset.yaml
   ```

2. **DaemonSets 상태 확인**
   ```bash
   kubectl get daemonsets
   ```

3. **DaemonSets의 Pod 상태 확인**
   - 모든 노드에 DaemonSet이 배포된 상태를 확인할 수 있습니다.
   ```bash
   kubectl get pods -o wide
   ```

4. **DaemonSets 업데이트**
   - DaemonSet을 업데이트할 때는 새로운 이미지나 설정을 적용한 후 다시 배포해야 합니다.
   ```bash
   kubectl apply -f daemonset.yaml
   ```

5. **DaemonSets 삭제**
   ```bash
   kubectl delete daemonset <daemonset-name>
   ```

---

## DaemonSets 유용한 명령어 모음

- **DaemonSets 생성**
  ```bash
  kubectl apply -f daemonset.yaml
  ```

- **DaemonSets 상태 확인**
  ```bash
  kubectl get daemonsets
  ```

- **DaemonSet이 생성한 Pod 상태 확인**
  ```bash
  kubectl get pods -l app=fluentd -o wide
  ```

- **DaemonSets 업데이트**
  ```bash
  kubectl apply -f daemonset.yaml
  ```

- **DaemonSets 삭제**
  ```bash
  kubectl delete daemonset <daemonset-name>
  ```

DaemonSets는 모든 노드에서 일관된 작업을 수행해야 할 때 매우 유용하며, 특히 로깅, 모니터링, 보안과 같은 인프라 작업에서 중요한 역할을 합니다.
