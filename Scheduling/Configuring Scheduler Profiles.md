# Configuring Scheduler Profiles

## 목차
1. [Scheduler Profiles란 무엇인가?](#Scheduler-Profiles란-무엇인가)
2. [Scheduler Profiles의 목적](#Scheduler-Profiles의-목적)
3. [Scheduler Profiles 설정 방법](#Scheduler-Profiles-설정-방법)
4. [Scheduler Profiles의 구성 요소](#Scheduler-Profiles의-구성-요소)
5. [Scheduler Profiles 유용한 명령어 모음](#Scheduler-Profiles-유용한-명령어-모음)

---

## Scheduler Profiles란 무엇인가?

**Scheduler Profiles**는 Kubernetes에서 **다양한 스케줄링 정책을 구현하기 위해 여러 프로파일을 구성**할 수 있는 기능입니다. Scheduler Profiles는 여러 스케줄링 요구사항에 맞춰 **다양한 우선순위 및 필터링 규칙**을 설정할 수 있으며, Kubernetes 스케줄러가 Pod를 배치하는 데 사용하는 다양한 플러그인과 설정을 프로파일별로 지정할 수 있습니다.

- **기능**: 각 프로파일별로 특정 스케줄링 요구 사항에 맞는 정책 구성
- **구성 가능**: 필터, 우선순위 설정 등 다양한 플러그인을 포함해 스케줄링 정책을 세분화 가능
- **유연성**: 클러스터 내 다양한 워크로드에 맞춰 서로 다른 스케줄링 정책을 적용 가능

---

## Scheduler Profiles의 목적

Scheduler Profiles는 특정 워크로드에 맞는 **맞춤형 스케줄링 정책을 구성하여 스케줄링 효율성을 높이기 위해 사용**됩니다. 이를 통해 클러스터 내 다양한 리소스 및 요구사항을 보다 정교하게 관리할 수 있습니다.

1. **워크로드 특성에 따른 스케줄링 최적화**: 고성능 리소스가 필요한 워크로드와 일반 애플리케이션 워크로드에 맞춘 프로파일 설정
2. **리소스 격리와 우선순위 관리**: 프로파일별로 필터 및 우선순위 설정을 적용하여 특정 리소스에 워크로드를 집중 배치하거나 분산
3. **다양한 스케줄링 정책 적용**: 하나의 클러스터에서 여러 스케줄링 정책을 동시에 운영하여 특정 목적에 맞춘 Pod 배치 가능

---

## Scheduler Profiles 설정 방법

Scheduler Profiles는 **kube-scheduler 구성 파일**에서 설정할 수 있습니다. 구성 파일을 통해 스케줄러의 프로파일을 정의하고, 각 프로파일에 대해 고유한 플러그인 및 설정을 지정할 수 있습니다. `profiles` 섹션에 여러 프로파일을 정의하고 각 프로파일마다 필요한 플러그인을 설정합니다.

### Scheduler Profiles 설정 예시

아래 예시는 `default-scheduler`와 `high-priority-scheduler` 두 개의 프로파일을 설정한 예시입니다.

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: default-scheduler
    plugins:
      queueSort:
        enabled:
          - name: PrioritySort
      preFilter:
        enabled:
          - name: NodeResourcesFit
      filter:
        enabled:
          - name: NodeResourcesFit
          - name: NodeAffinity
      score:
        enabled:
          - name: NodeResourcesBalancedAllocation

  - schedulerName: high-priority-scheduler
    plugins:
      queueSort:
        enabled:
          - name: PrioritySort
      preFilter:
        enabled:
          - name: NodeResourcesFit
      filter:
        enabled:
          - name: NodeResourcesFit
          - name: NodeAffinity
      score:
        enabled:
          - name: MostRequested
```

1. **schedulerName**: 스케줄러 이름을 지정하여 Pod의 `schedulerName` 필드에서 특정 프로파일을 지정할 수 있도록 설정합니다.
2. **plugins**: 각 프로파일에 적용할 플러그인 목록을 설정하여, 스케줄링에 필요한 필터, 우선순위, 점수 등을 지정합니다.

---

## Scheduler Profiles의 구성 요소

Scheduler Profiles는 **다양한 플러그인과 설정**으로 구성되며, 이를 통해 스케줄링 프로세스의 세부적인 동작을 제어할 수 있습니다.

1. **queueSort**: 스케줄링 대기열에서 Pod의 순서를 지정하는 플러그인입니다. 예를 들어 `PrioritySort`를 통해 높은 우선순위의 Pod가 먼저 스케줄링됩니다.
   
2. **preFilter**: Pod가 특정 노드에 배치될 수 있는지 미리 검사하는 필터입니다. `NodeResourcesFit`을 통해 노드 리소스와의 적합성을 미리 확인할 수 있습니다.
   
3. **filter**: 스케줄링 가능한 노드를 필터링합니다. `NodeAffinity`, `NodeResourcesFit` 등을 통해 노드 레이블이나 자원 적합성에 따라 필터링을 수행합니다.
   
4. **score**: 필터링 후 남은 노드에 점수를 부여하여 최적의 노드를 선택하는 단계입니다. `NodeResourcesBalancedAllocation` 또는 `MostRequested`와 같은 플러그인을 통해 노드의 리소스 사용 균형을 고려하여 점수를 부여할 수 있습니다.

---

## Scheduler Profiles 유용한 명령어 모음

- **스케줄러 구성 파일 적용**
  - 스케줄러 구성 파일이 변경된 경우 kube-scheduler를 재시작하여 새 구성이 적용되도록 합니다.

- **특정 스케줄러로 Pod 생성**
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-high-priority-pod
  spec:
    schedulerName: high-priority-scheduler
    containers:
      - name: nginx
        image: nginx
  ```
  - Pod의 `schedulerName` 필드에 원하는 스케줄러 이름을 지정합니다.

- **Scheduler 로그 확인**
  ```bash
  kubectl logs -n kube-system <scheduler-pod-name>
  ```
  - 스케줄러 로그를 통해 스케줄링 과정에서 발생한 이벤트 및 스케줄링 결정을 확인할 수 있습니다.

Scheduler Profiles를 사용하면 Kubernetes 스케줄러의 기능을 확장하여 다양한 요구사항에 맞는 스케줄링 정책을 설정할 수 있으며, 클러스터 내에서 특정 작업에 맞춘 리소스 배치 최적화를 달성할 수 있습니다.
