# Manual Scheduling

## 목차
1. [Manual Scheduling이란 무엇인가?](#Manual-Scheduling이란-무엇인가)
2. [Manual Scheduling의 목적](#Manual-Scheduling의-목적)
3. [Manual Scheduling 사용 방법](#Manual-Scheduling-사용-방법)
4. [Manual Scheduling 유용한 명령어 모음](#Manual-Scheduling-유용한-명령어-모음)

---

## Manual Scheduling이란 무엇인가?

**Manual Scheduling**은 Kubernetes에서 기본 스케줄러를 사용하지 않고 **사용자가 특정 노드에 직접 Pod를 할당**하는 방식입니다. 일반적으로는 Kubernetes 스케줄러가 자동으로 Pod를 배치하지만, 특정 조건에서 수동으로 Pod의 배치 위치를 지정해야 할 때 Manual Scheduling을 활용할 수 있습니다. 예를 들어, 테스트 목적이나 특정 리소스 요구사항이 있는 경우 수동 스케줄링이 필요할 수 있습니다.

- **기능**: Kubernetes 스케줄러를 우회하여 사용자가 직접 특정 노드에 Pod를 할당
- **사용 목적**: 테스트 환경 구성, 특정 리소스 활용 최적화 등에서 유용
- **제어 가능성**: 리소스를 최적화하고 필요한 환경에 맞게 제어 가능

---

## Manual Scheduling의 목적

Manual Scheduling은 **스케줄러가 아닌 사용자가 직접 리소스 배치를 제어**할 수 있는 방법입니다. 일반적인 스케줄링 규칙을 넘어서 특정 노드에 리소스를 배치해야 하는 다양한 상황에서 유용합니다.

1. **테스트 환경 구성**: 특정 노드의 성능이나 상태를 점검하기 위해 특정 노드에 리소스를 배치하여 테스트를 수행할 때 사용
2. **리소스 사용 최적화**: 리소스를 효과적으로 집중시키거나 특정 노드에만 리소스를 할당하여 최적화
3. **장애 조치 시나리오 테스트**: 스케줄링 문제를 해결하거나 장애 조치 시나리오를 시뮬레이션할 때 사용

---

## Manual Scheduling 사용 방법

Manual Scheduling을 통해 특정 노드에 Pod를 배치할 때는 **`nodeName` 필드**를 사용해 YAML 파일에서 직접 노드를 지정할 수 있습니다. 이 방식은 Kubernetes 스케줄러를 사용하지 않고 Pod가 지정된 노드에 배치되도록 합니다.

### Pod YAML 파일 예시

아래와 같이 **`nodeName`** 필드를 추가해 수동 스케줄링을 설정할 수 있습니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: manually-scheduled-pod
spec:
  containers:
    - name: nginx
      image: nginx
  nodeName: <target-node-name> # 특정 노드 지정
```

1. **`nodeName` 필드 사용**: 스케줄러 대신 특정 노드에 Pod를 직접 배치
2. **적용 방법**: YAML 파일 작성 후 아래 명령어로 수동 스케줄링 적용
   ```bash
   kubectl apply -f manual-scheduling-pod.yaml
   ```

이렇게 설정된 Pod는 지정된 노드에 배치되며, 다른 노드로는 이동하지 않습니다.

---

## Manual Scheduling 유용한 명령어 모음

- **Pod 생성 및 Manual Scheduling 적용**
  ```bash
  kubectl apply -f manual-scheduling-pod.yaml
  ```

- **Pod 상태 확인**
  ```bash
  kubectl get pods -o wide
  ```

- **Pod 상세 정보 조회**
  ```bash
  kubectl describe pod manually-scheduled-pod
  ```

- **특정 노드에 배치된 Pod 확인**
  ```bash
  kubectl get pods --field-selector spec.nodeName=<target-node-name>
  ```
