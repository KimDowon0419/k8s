# Kubelet

## 목차
1. [Kubelet란 무엇인가?](#Kubelet란-무엇인가)
2. [Kubelet의 작동 원리](#Kubelet의-작동-원리)
3. [Kubelet 설정](#Kubelet-설정)
4. [Kubelet 관리 및 운영](#Kubelet-관리-및-운영)
5. [Kubelet 모니터링 및 로깅](#Kubelet-모니터링-및-로깅)
6. [Kubelet 보안 설정](#Kubelet-보안-설정)
7. [Kubelet 유용한 명령어 모음](#Kubelet-유용한-명령어-모음)

---

## Kubelet란 무엇인가?

**Kubelet**은 각 노드에서 실행되는 Kubernetes의 에이전트로, **파드가 노드에서 올바르게 실행되고 있는지 지속적으로 모니터링하고 관리**. Kubernetes API 서버와 통신하며, 파드를 노드에서 생성하고 관리하는 역할을 수행.

- **기능**: 각 노드에서 파드가 정상적으로 실행되도록 모니터링하고, 파드 상태를 API 서버에 보고.
- **컨테이너 런타임 인터페이스**: Docker, Containerd 등 다양한 컨테이너 런타임과 호환.
- **자원 관리**: 파드에 할당된 CPU, 메모리와 같은 리소스를 관리하며, 노드 자원에 맞게 파드를 스케줄링.

---

## Kubelet의 작동 원리

Kubelet은 **Kubernetes API 서버로부터 파드 사양을 가져와 노드에서 파드를 생성하고, 이를 관리**. 파드가 종료되거나 오류가 발생한 경우, Kubelet은 이를 감지하여 상태를 API 서버에 보고하며, 필요에 따라 파드를 다시 시작.

1. **파드 사양 수신**: Kubelet은 API 서버로부터 파드 사양을 받음.
2. **컨테이너 실행**: 컨테이너 런타임을 통해 파드 내 컨테이너를 시작.
3. **상태 보고**: 파드 상태를 주기적으로 API 서버에 보고.
4. **헬스 체크 및 재시작**: 파드 상태를 확인하고, 비정상적일 경우 파드를 재시작.

---

## Kubelet 설정

Kubelet의 주요 설정은 **`/var/lib/kubelet/config.yaml`** 또는 **명령어 플래그**를 통해 구성되며, `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` 파일에서도 일부 설정을 수정 가능.

1. **기본 설정 플래그**:
   - `--kubeconfig`: API 서버와 통신하기 위한 Kubeconfig 파일 경로.
   - `--config`: YAML 형식의 설정 파일 경로.
   - `--network-plugin`: 네트워크 플러그인 설정 (예: `cni`).
   - `--cgroup-driver`: Kubelet이 사용할 cgroup 드라이버.

   ```yaml
   kind: KubeletConfiguration
   apiVersion: kubelet.config.k8s.io/v1beta1
   address: "0.0.0.0"
   staticPodPath: "/etc/kubernetes/manifests"
   authentication:
     anonymous:
       enabled: false
     webhook:
       enabled: true
     x509:
       clientCAFile: "/etc/kubernetes/pki/ca.crt"
   ```

2. **시작 옵션 플래그** (예시):
   - `kubelet` 명령어와 함께 플래그 사용하여 설정.
   ```bash
   kubelet --config=/var/lib/kubelet/config.yaml --cgroup-driver=systemd
   ```

3. **systemd로 관리**:
   - Kubelet은 systemd 서비스로 관리되며, 다음과 같이 재시작 가능.
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```

---

## Kubelet 관리 및 운영

1. **파드 및 노드 상태 확인**:
   - Kubelet은 주기적으로 파드 상태를 모니터링하고, 노드 상태를 보고.
   ```bash
   kubectl get nodes
   kubectl describe node <node-name>
   ```

2. **파드 로그 확인**:
   - 특정 파드의 로그를 확인하여 문제를 진단.
   ```bash
   kubectl logs <pod-name> -n <namespace>
   ```

3. **노드 격리 (Drain & Cordon)**:
   - 노드를 격리하고 파드를 다른 노드로 옮길 때 사용.
   ```bash
   kubectl drain <node-name> --ignore-daemonsets
   kubectl cordon <node-name>
   ```

4. **노드 복귀 (Uncordon)**:
   - 격리된 노드를 다시 스케줄링에 포함.
   ```bash
   kubectl uncordon <node-name>
   ```

---

## Kubelet 모니터링 및 로깅

1. **로그 확인**:
   - Kubelet 로그는 `journalctl`을 통해 확인 가능.
   ```bash
   journalctl -u kubelet -f
   ```

2. **메트릭 수집**:
   - Prometheus와 같은 모니터링 도구를 통해 Kubelet 메트릭 수집 가능.
   - 기본 메트릭 엔드포인트: `http://localhost:10255/metrics`

3. **로그 레벨 조정**:
   - `--v` 플래그로 로그의 상세 수준 조정 (`0`이 기본, `10`은 최대 상세).

---

## Kubelet 보안 설정

1. **인증 및 권한 부여**:
   - Kubelet에 대한 인증과 권한 설정을 통해 보안을 강화.
   - 익명 요청 비활성화 및 Webhook 인증 활성화:
     ```yaml
     authentication:
       anonymous:
         enabled: false
       webhook:
         enabled: true
       x509:
         clientCAFile: "/etc/kubernetes/pki/ca.crt"
     ```

2. **TLS 인증서 사용**:
   - Kubelet과 API 서버 간의 통신을 암호화하여 안전성 확보.
   - `kubelet.conf` 파일에서 인증서 경로 설정:
     ```yaml
     serverTLSBootstrap: true
     ```

3. **네트워크 보안**:
   - 외부 접근 제한 및 방화벽 설정으로 Kubelet의 포트 보호.

---

## Kubelet 유용한 명령어 모음

- **Kubelet 상태 확인**:
  ```bash
  systemctl status kubelet
  ```

- **Kubelet 로그 확인**:
  ```bash
  journalctl -u kubelet -f
  ```

- **파드 로그 조회**:
  ```bash
  kubectl logs <pod-name> -n <namespace>
  ```

- **노드 격리**:
  ```bash
  kubectl drain <node-name> --ignore-daemonsets
  kubectl cordon <node-name>
  ```

- **노드 복귀**:
  ```bash
  kubectl uncordon <node-name>
  ```

이 가이드를 통해 Kubelet의 기본 개념, 설정, 모니터링 및 관리 방법을 익히고 Kubernetes 클러스터에서 안정적이고 안전하게 운영할 수 있음.
