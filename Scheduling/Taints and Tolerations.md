# Taints and Tolerations

## 목차
1. [Taints와 Tolerations란 무엇인가?](#Taints와-Tolerations란-무엇인가)
2. [Taints와 Tolerations의 목적](#Taints와-Tolerations의-목적)
3. [Taints 설정 방법](#Taints-설정-방법)
4. [Tolerations 설정 방법](#Tolerations-설정-방법)
5. [Taints와 Tolerations 유용한 명령어 모음](#Taints와-Tolerations-유용한-명령어-모음)

---

## Taints와 Tolerations란 무엇인가?

**Taints**와 **Tolerations**는 Kubernetes에서 특정 노드에 **특정 조건을 적용하여 선택적으로 Pod의 스케줄링을 제한**하는 메커니즘입니다. Taints는 노드에 적용되어, 특정 조건을 충족하는 Pod만 해당 노드에 스케줄링될 수 있도록 합니다. 반대로, Tolerations는 Pod에 설정되어 특정 Taints를 가진 노드에 배치될 수 있도록 허용합니다.

- **Taints**: 노드에 설정하여 특정 Pod의 스케줄링을 제한
- **Tolerations**: Pod에 설정하여 특정 Taints를 가진 노드에 배치될 수 있게 허용

Taints와 Tolerations는 **노드에 선택적으로 Pod를 배치**하여 리소스를 보다 효과적으로 사용할 수 있게 도와줍니다.

---

## Taints와 Tolerations의 목적

Taints와 Tolerations는 다음과 같은 목적을 가지고 사용됩니다.

1. **노드 자원 보호**: 특정 Pod만 지정된 노드에 배치하도록 제한하여 중요 리소스를 보호
2. **특정 작업 전용 노드 구성**: 데이터베이스, 캐시, 백엔드 서버와 같은 특정 용도의 작업만 해당 노드에서 실행되도록 설정
3. **특수 환경 격리**: 테스트, 디버깅 또는 실험적 작업을 격리된 환경에서 실행

---

## Taints 설정 방법

Taints는 노드에 설정되며, 각 Taint는 **키(key), 값(value), 효과(effect)**로 구성됩니다. 다음 세 가지 효과 옵션이 있습니다.

- **NoSchedule**: Taint에 대한 Toleration이 없는 Pod는 스케줄링되지 않음
- **PreferNoSchedule**: Taint에 대한 Toleration이 없는 경우 해당 노드에 스케줄링하지 않도록 권장
- **NoExecute**: 이미 해당 노드에 스케줄링된 Pod를 포함해 Toleration이 없는 Pod를 즉시 종료

### Taint 추가 예시

다음 명령어를 사용하여 노드에 Taint를 추가할 수 있습니다.

```bash
kubectl taint nodes <node-name> key=value:effect
```

예를 들어, `dedicated=database:NoSchedule` Taint를 노드에 추가하는 명령어는 다음과 같습니다.

```bash
kubectl taint nodes node1 dedicated=database:NoSchedule
```

---

## Tolerations 설정 방법

Tolerations는 Pod의 YAML 파일에 **key, operator, value, effect**를 사용하여 설정할 수 있습니다. Tolerations를 사용하여 특정 Taints를 가진 노드에 Pod가 스케줄링될 수 있도록 허용합니다.

### Toleration 예시

아래 예시는 `dedicated=database:NoSchedule` Taint를 허용하는 Toleration을 가진 Pod를 정의한 YAML입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "database"
      effect: "NoSchedule"
  containers:
    - name: nginx
      image: nginx
```

1. **key**: Taint의 키와 일치해야 함
2. **operator**: "Equal" 또는 "Exists"를 사용하여 Taint와 일치하는 조건 지정
3. **value**: Taint의 값과 일치해야 함
4. **effect**: Taint의 효과와 일치해야 함 (NoSchedule, PreferNoSchedule, NoExecute 중 선택)

---

## Taints와 Tolerations 유용한 명령어 모음

- **노드에 Taint 추가**
  ```bash
  kubectl taint nodes <node-name> key=value:effect
  ```

- **노드의 모든 Taint 제거**
  ```bash
  kubectl taint nodes <node-name> key:NoSchedule-
  ```

- **Pod에 적용된 Toleration 확인**
  ```bash
  kubectl describe pod <pod-name> | grep -A 5 "Tolerations"
  ```

- **특정 Taint가 설정된 노드 확인**
  ```bash
  kubectl get nodes -o json | jq '.items[] | select(.spec.taints != null) | .metadata.name,.spec.taints'
  ```

Taints와 Tolerations는 리소스를 보호하고 선택적으로 Pod 배치를 제한하여 Kubernetes 클러스터의 안정성과 효율성을 높이는 데 유용합니다.
