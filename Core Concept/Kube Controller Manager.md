# Kube Controller Manager

## 목차
1. [Kube Controller Manager란 무엇인가?](#Kube-Controller-Manager란-무엇인가)
2. [Kube Controller Manager의 작동 원리](#Kube-Controller-Manager의-작동-원리)
3. [Kube Controller Manager 설정](#Kube-Controller-Manager-설정)
4. [Kube Controller Manager 주요 컨트롤러](#Kube-Controller-Manager-주요-컨트롤러)
5. [Kube Controller Manager 모니터링 및 로깅](#Kube-Controller-Manager-모니터링-및-로깅)
6. [Kube Controller Manager 관리 및 트러블슈팅](#Kube-Controller-Manager-관리-및-트러블슈팅)
7. [Kube Controller Manager 유용한 명령어 모음](#Kube-Controller-Manager-유용한-명령어-모음)

---

## Kube Controller Manager란 무엇인가?

**Kube Controller Manager**는 Kubernetes에서 **컨트롤러를 관리하는 데 사용되는 컴포넌트**로, 클러스터의 상태를 원하는 목표 상태로 유지하는 역할을 수행. 여러 컨트롤러를 포함하여 클러스터의 각 리소스 상태를 지속적으로 모니터링하고 자동으로 조정.

- **기능**: 모든 컨트롤러를 실행하고 관리하여 클러스터 상태를 원하는 대로 유지.
- **확장성**: 필요에 따라 다중 Controller Manager 인스턴스 실행 가능.
- **주요 컨트롤러**: 노드 컨트롤러, 레플리케이션 컨트롤러, 엔드포인트 컨트롤러 등 포함.

---

## Kube Controller Manager의 작동 원리

Controller Manager는 클러스터의 각 리소스를 모니터링하여, **목표 상태와 실제 상태 간의 차이를 조정**. 예를 들어, 레플리카셋의 복제본 수가 설정값과 다를 경우 자동으로 파드를 생성하거나 제거하여 설정된 복제본 수 유지.

- **목표 상태와 실제 상태 비교**: 각 리소스의 설정값과 실제 값을 비교하여 자동으로 수정.
- **비동기 작업**: Kubernetes API 서버와 지속적으로 상호작용하며 리소스 상태 변경.
- **리더 선출**: 여러 인스턴스가 동시에 실행될 수 있으며, 하나의 리더가 주요 작업을 담당.

---

## Kube Controller Manager 설정

Kube Controller Manager는 일반적으로 **YAML 파일**이나 **명령어 플래그**를 통해 설정. 보통 `/etc/kubernetes/manifests/kube-controller-manager.yaml` 파일에 설정.

1. **기본 설정 플래그**:
   - `--kubeconfig`: API 서버에 접근하기 위한 Kubeconfig 파일 경로.
   - `--leader-elect`: 리더 선출 여부, 활성화 시 하나의 컨트롤러만 주요 작업 담당.
   - `--controllers`: 활성화할 컨트롤러 종류 지정 (기본값은 모든 컨트롤러 활성화).

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: kube-controller-manager
     namespace: kube-system
   spec:
     containers:
     - command:
       - kube-controller-manager
       - --kubeconfig=/etc/kubernetes/controller-manager.conf
       - --leader-elect=true
       - --controllers=*,bootstrapsigner,tokencleaner
       image: k8s.gcr.io/kube-controller-manager:v1.31.0
   ```

2. **리소스 할당**:
   - Controller Manager는 여러 작업을 동시에 수행하므로 CPU 및 메모리 리소스를 충분히 할당하여 안정적인 운영 보장.

---

## Kube Controller Manager 주요 컨트롤러

1. **Node Controller**: 노드의 상태를 모니터링하고, 노드 장애 시 해당 노드의 파드 재배치.
2. **Replication Controller**: 복제본 수를 관리하여 파드 수가 설정값에 맞게 유지.
3. **Endpoints Controller**: 각 서비스의 엔드포인트 정보를 유지 관리하여 파드가 올바르게 연결되도록 지원.
4. **Namespace Controller**: 네임스페이스 삭제 시 해당 네임스페이스에 포함된 리소스를 정리.
5. **Service Account Controller**: 새 네임스페이스에 기본 서비스 계정을 자동으로 생성.

---

## Kube Controller Manager 모니터링 및 로깅

1. **메트릭 수집**:
   - Prometheus와 같은 모니터링 도구를 통해 Controller Manager의 메트릭 수집 가능.
   - 메트릭 엔드포인트: `https://<API_SERVER_IP>:10252/metrics`

2. **로깅 설정**:
   - 로그는 `/var/log/kube-controller-manager.log`에 저장되며, 오류 및 상태를 파악 가능.
   - **로그 확인 명령어**:
     ```bash
     tail -f /var/log/kube-controller-manager.log
     ```

3. **로그 레벨 조정**:
   - `--v` 플래그를 통해 로그의 상세 수준 설정 (`0`이 기본, `10`은 최대 상세).

---

## Kube Controller Manager 관리 및 트러블슈팅

1. **Controller Manager 상태 확인**:
   - Controller Manager 상태를 모니터링하여 각 컨트롤러가 올바르게 작동하는지 점검.
   ```bash
   kubectl get componentstatuses
   ```

2. **특정 컨트롤러 비활성화**:
   - 필요에 따라 특정 컨트롤러 비활성화 가능.
   - 예시: `--controllers=-garbagecollector`로 가비지 컬렉터 비활성화.

3. **Controller Manager 재시작**:
   - `/etc/kubernetes/manifests/kube-controller-manager.yaml` 파일 수정 후 Kubelet이 변경 사항을 감지하여 자동 재시작.

4. **API 서버 연결 오류 해결**:
   - Controller Manager가 API 서버와 연결하지 못하는 경우 `--kubeconfig` 경로와 인증서 경로 재확인.

---

## Kube Controller Manager 유용한 명령어 모음

- **Controller Manager 상태 확인**:
  ```bash
  kubectl get componentstatuses
  ```

- **활성화된 컨트롤러 확인**:
  ```bash
  kubectl describe pod kube-controller-manager -n kube-system
  ```

- **메트릭 확인**:
  ```bash
  curl -k https://<API_SERVER_IP>:10252/metrics
  ```

- **특정 컨트롤러 비활성화**:
  ```yaml
  - --controllers=-garbagecollector
  ```

이 가이드를 통해 Kube Controller Manager의 개념과 설정, 관리 방법을 이해하고 Kubernetes 클러스터에서 안정적으로 운영할 수 있음.
