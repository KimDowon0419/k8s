# Kube Scheduler

## 목차
1. [Kube Scheduler란 무엇인가?](#Kube-Scheduler란-무엇인가)
2. [Kube Scheduler의 작동 원리](#Kube-Scheduler의-작동-원리)
3. [Kube Scheduler 설정](#Kube-Scheduler-설정)
4. [스케줄링 정책과 조건](#스케줄링-정책과-조건)
5. [Kube Scheduler 모니터링 및 로깅](#Kube-Scheduler-모니터링-및-로깅)
6. [Kube Scheduler 관리 및 트러블슈팅](#Kube-Scheduler-관리-및-트러블슈팅)
7. [Kube Scheduler 유용한 명령어 모음](#Kube-Scheduler-유용한-명령어-모음)

---

## Kube Scheduler란 무엇인가?

**Kube Scheduler**는 Kubernetes에서 **새로 생성된 파드를 적합한 노드에 배치**하는 역할을 수행. 스케줄러는 클러스터 상태를 분석하고 파드 요구 사항에 맞는 노드를 찾아 파드를 배치하며, 파드가 어떤 노드에서 실행될지 결정.

- **기능**: 파드가 클러스터 내에서 가장 적합한 노드에 배치되도록 관리.
- **확장성**: 여러 스케줄러를 동시에 실행 가능하며, 특정 스케줄러에 파드를 할당할 수도 있음.
- **스케줄링 전략**: 자원 사용량, 노드 상태, 노드의 `taints` 및 `tolerations` 등을 고려하여 파드 배치.

---

## Kube Scheduler의 작동 원리

Kube Scheduler는 **파드의 스케줄링을 위해 다음 단계**를 수행:

1. **적합한 노드 필터링 (Predicates)**:
   - 파드가 실행 가능한 노드를 필터링하여 후보 노드 선정.
   - `nodeSelector`, `node affinity`, `taints`와 `tolerations` 등 조건을 기반으로 노드를 선택.

2. **노드 점수 부여 (Priorities)**:
   - 후보 노드에 점수를 부여하여 최적의 노드를 선정.
   - CPU와 메모리 사용량, 파드 분산도, 가용성 등을 고려하여 각 노드에 점수 부여.

3. **최적 노드 결정**:
   - 점수가 가장 높은 노드를 선택하여 파드를 할당.

---

## Kube Scheduler 설정

Kube Scheduler는 보통 `/etc/kubernetes/manifests/kube-scheduler.yaml` 파일에 설정을 관리.

1. **기본 설정 플래그**:
   - `--kubeconfig`: API 서버에 접근하기 위한 Kubeconfig 파일 경로.
   - `--leader-elect`: 리더 선출을 활성화하여 하나의 스케줄러만 주요 작업을 담당.
   - `--scheduler-name`: 특정 파드를 스케줄링할 스케줄러의 이름 지정.

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: kube-scheduler
     namespace: kube-system
   spec:
     containers:
     - command:
       - kube-scheduler
       - --kubeconfig=/etc/kubernetes/scheduler.conf
       - --leader-elect=true
       - --scheduler-name=my-custom-scheduler
       image: k8s.gcr.io/kube-scheduler:v1.31.0
   ```

2. **스케줄러 프로파일 설정**:
   - 다양한 스케줄링 정책을 정의한 `scheduler-config.yaml` 파일을 통해 스케줄링 동작을 세밀하게 조정 가능.
   ```yaml
   apiVersion: kubescheduler.config.k8s.io/v1
   kind: KubeSchedulerConfiguration
   profiles:
   - schedulerName: my-scheduler
     plugins:
       score:
         enabled:
           - name: NodeResourcesBalancedAllocation
   ```

---

## 스케줄링 정책과 조건

1. **nodeSelector**: 파드를 특정 노드에 스케줄링하기 위해 사용.
   ```yaml
   spec:
     nodeSelector:
       disktype: ssd
   ```

2. **Node Affinity 및 Anti-Affinity**: `nodeSelector`보다 더 유연한 조건으로, 특정 조건의 노드에 파드를 배치.
   ```yaml
   affinity:
     nodeAffinity:
       requiredDuringSchedulingIgnoredDuringExecution:
         nodeSelectorTerms:
         - matchExpressions:
           - key: disktype
             operator: In
             values:
             - ssd
   ```

3. **Taints와 Tolerations**: 노드에 `taint`를 설정해 특정 파드만 해당 노드에 스케줄링.
   - 노드에 Taint 설정:
     ```bash
     kubectl taint nodes <node-name> key=value:NoSchedule
     ```
   - 파드의 Tolerations 설정:
     ```yaml
     tolerations:
       - key: "key"
         operator: "Equal"
         value: "value"
         effect: "NoSchedule"
     ```

4. **멀티 스케줄러**: 다중 스케줄러를 정의하여 특정 파드가 지정된 스케줄러에서만 스케줄링되도록 설정.
   - 파드 스케줄러 지정:
     ```yaml
     spec:
       schedulerName: my-custom-scheduler
     ```

---

## Kube Scheduler 모니터링 및 로깅

1. **메트릭 수집**:
   - Prometheus 등을 사용하여 스케줄러의 메트릭 수집 및 모니터링 가능.
   - 메트릭 엔드포인트: `https://<API_SERVER_IP>:10251/metrics`

2. **로깅 설정**:
   - Kube Scheduler의 로그는 `/var/log/kube-scheduler.log`에 저장되며, 오류 및 상태를 파악 가능.
   - **로그 확인 명령어**:
     ```bash
     tail -f /var/log/kube-scheduler.log
     ```

3. **로그 레벨 조정**:
   - `--v` 플래그를 통해 로그의 상세 수준 설정 (`0`이 기본, `10`은 최대 상세).

---

## Kube Scheduler 관리 및 트러블슈팅

1. **Scheduler 상태 확인**:
   - Scheduler가 올바르게 실행 중인지 확인.
   ```bash
   kubectl get componentstatuses
   ```

2. **특정 스케줄러 적용**:
   - 다중 스케줄러 환경에서 특정 파드가 지정된 스케줄러에서만 실행되도록 `schedulerName` 설정.

3. **스케줄링 문제 해결**:
   - 파드가 스케줄링되지 않을 때, 자원 부족, 노드 Taints 등 조건 확인.
   - 스케줄링 오류 발생 시 `/var/log/kube-scheduler.log`에서 로그 확인.

4. **Scheduler 재시작**:
   - `/etc/kubernetes/manifests/kube-scheduler.yaml` 파일 수정 후 Kubelet이 변경 사항을 감지하여 자동으로 재시작.

---

## Kube Scheduler 유용한 명령어 모음

- **Scheduler 상태 확인**:
  ```bash
  kubectl get componentstatuses
  ```

- **활성 스케줄러 확인**:
  ```bash
  kubectl describe pod kube-scheduler -n kube-system
  ```

- **스케줄링 메트릭 확인**:
  ```bash
  curl -k https://<API_SERVER_IP>:10251/metrics
  ```

- **파드의 스케줄링 오류 디버깅**:
  ```bash
  kubectl describe pod <pod-name>
  ```

이 가이드를 통해 Kube Scheduler의 기본 개념, 설정 및 스케줄링 정책을 익히고 Kubernetes 클러스터에서 안정적이고 효율적으로 운영할 수 있음.
