# Kube Proxy

## 목차
1. [Kube Proxy란 무엇인가?](#Kube-Proxy란-무엇인가)
2. [Kube Proxy의 작동 원리](#Kube-Proxy의-작동-원리)
3. [Kube Proxy 설정](#Kube-Proxy-설정)
4. [Kube Proxy 모드와 라우팅 방식](#Kube-Proxy-모드와-라우팅-방식)
5. [Kube Proxy 관리 및 운영](#Kube-Proxy-관리-및-운영)
6. [Kube Proxy 모니터링 및 로깅](#Kube-Proxy-모니터링-및-로깅)
7. [Kube Proxy 유용한 명령어 모음](#Kube-Proxy-유용한-명령어-모음)

---

## Kube Proxy란 무엇인가?

**Kube Proxy**는 Kubernetes 클러스터의 **서비스 네트워크 트래픽을 관리하고 라우팅**하는 컴포넌트. 클러스터 내부의 파드와 서비스 간의 통신을 가능하게 하며, 각 노드에 하나씩 배포되어 서비스와 파드를 연결하는 네트워크 규칙을 생성하고 유지.

- **기능**: 각 서비스에 대한 가상 IP를 생성하고, 클러스터 내부에서 파드 간의 트래픽을 올바른 엔드포인트로 전달.
- **역할**: iptables, IPVS 또는 유저 스페이스 모드를 통해 네트워크 규칙을 설정하고 트래픽을 관리.

---

## Kube Proxy의 작동 원리

Kube Proxy는 **서비스와 파드 간의 네트워크 요청을 라우팅**하여 Kubernetes의 모든 서비스에 대해 고유한 가상 IP 주소를 생성하고 관리. 이를 통해 클러스터 내에서 동적으로 변경되는 파드와 서비스의 통신을 유지.

1. **가상 IP 생성**: 각 서비스에 고유한 IP 할당.
2. **트래픽 분산**: 다중 파드가 있는 경우 로드 밸런싱을 통해 트래픽 분산.
3. **네트워크 규칙 설정**: iptables, IPVS 등의 규칙을 생성하여 서비스와 파드 간의 트래픽을 올바르게 전달.

---

## Kube Proxy 설정

Kube Proxy는 `/var/lib/kube-proxy/config.conf` 파일에서 **YAML 형식으로 설정 관리**.

1. **기본 설정 플래그**:
   - `--proxy-mode`: Kube Proxy의 작동 모드 설정 (`iptables`, `ipvs`, `userspace` 중 선택).
   - `--kubeconfig`: API 서버와의 통신에 사용할 Kubeconfig 파일 경로.
   - `--cluster-cidr`: 클러스터 CIDR 설정 (옵션).

   ```yaml
   apiVersion: kubeproxy.config.k8s.io/v1alpha1
   kind: KubeProxyConfiguration
   mode: "iptables"
   clusterCIDR: "10.244.0.0/16"
   ```

2. **시작 옵션 플래그**:
   - systemd 서비스 파일 `/etc/systemd/system/kube-proxy.service` 또는 `/var/lib/kube-proxy/config.conf` 파일에서 설정 관리 가능.
   - 예시:
     ```bash
     kube-proxy --config=/var/lib/kube-proxy/config.conf
     ```

3. **Kube Proxy 모드 변경**:
   - Kube Proxy 모드를 `iptables`, `ipvs`, `userspace` 중에서 선택하여 클러스터의 네트워크 트래픽 관리 방식을 변경 가능.

---

## Kube Proxy 모드와 라우팅 방식

1. **iptables 모드**:
   - 기본 모드로, iptables 규칙을 통해 서비스와 파드 간 트래픽을 라우팅.
   - 간단한 규칙 구조를 통해 빠르게 네트워크 규칙을 관리 가능.

2. **IPVS 모드**:
   - 성능이 높은 모드로, IPVS 가상 서버를 사용하여 트래픽을 라우팅.
   - 대규모 트래픽을 처리하는 데 적합하며, 로드 밸런싱에 최적화.

3. **userspace 모드**:
   - 초기 방식으로, 서비스 IP를 사용자 공간에서 라우팅.
   - 성능이 낮아 현재는 거의 사용되지 않음.

각 모드는 Kubernetes 설정 파일에서 `--proxy-mode` 플래그를 사용해 지정 가능.

---

## Kube Proxy 관리 및 운영

1. **네트워크 규칙 확인**:
   - iptables 또는 IPVS 규칙을 직접 확인하여 현재 트래픽 라우팅 상태 점검.
   ```bash
   iptables -t nat -L -n
   ipvsadm -Ln
   ```

2. **Kube Proxy 상태 확인**:
   - 각 노드에서 Kube Proxy가 정상적으로 실행 중인지 확인.
   ```bash
   systemctl status kube-proxy
   ```

3. **Kube Proxy 재시작**:
   - 설정 변경 후 Kube Proxy를 재시작하여 설정 적용.
   ```bash
   sudo systemctl restart kube-proxy
   ```

---

## Kube Proxy 모니터링 및 로깅

1. **로그 확인**:
   - Kube Proxy 로그를 확인하여 네트워크 문제와 관련된 오류 탐색.
   ```bash
   journalctl -u kube-proxy -f
   ```

2. **메트릭 수집**:
   - Prometheus와 같은 도구를 통해 Kube Proxy 메트릭 수집 가능.
   - 메트릭 엔드포인트: `http://localhost:10249/metrics`

3. **로그 레벨 조정**:
   - `--v` 플래그로 로그의 상세 수준을 조정 (`0`이 기본, `10`은 최대 상세).

---

## Kube Proxy 유용한 명령어 모음

- **Kube Proxy 상태 확인**:
  ```bash
  systemctl status kube-proxy
  ```

- **Kube Proxy 로그 확인**:
  ```bash
  journalctl -u kube-proxy -f
  ```

- **iptables 규칙 확인**:
  ```bash
  iptables -t nat -L -n
  ```

- **IPVS 규칙 확인**:
  ```bash
  ipvsadm -Ln
  ```

- **Kube Proxy 재시작**:
  ```bash
  sudo systemctl restart kube-proxy
  ```
