# Kube-API Server

## 목차
1. [Kube-API Server란 무엇인가?](#Kube-API-Server란-무엇인가)
2. [Kube-API Server의 작동 원리](#Kube-API-Server의-작동-원리)
3. [Kube-API Server 설정](#Kube-API-Server-설정)
4. [Kube-API Server 보안 설정](#Kube-API-Server-보안-설정)
5. [Kube-API Server 모니터링 및 로깅](#Kube-API-Server-모니터링-및-로깅)
6. [Kube-API Server 관리 및 트러블슈팅](#Kube-API-Server-관리-및-트러블슈팅)
7. [Kube-API Server 유용한 명령어 모음](#Kube-API-Server-유용한-명령어-모음)

---

## Kube-API Server란 무엇인가?

**Kube-API Server**는 Kubernetes 클러스터의 **중앙 API 엔드포인트**로, 클러스터 구성 요소 및 클라이언트가 이 API 서버를 통해 통신. 모든 요청을 인증 및 승인하며, ETCD에 상태 정보를 저장하고, Kubernetes의 CRUD 연산을 관리.

- **클러스터의 진입점**: 모든 kubectl 명령, 내부 통신 등이 Kube-API Server를 통해 전달.
- **데이터 처리**: API 서버는 모든 요청을 라우팅하고 Kubernetes 상태를 ETCD에 기록하여 클러스터 상태를 유지.
- **확장성**: 수평 확장을 지원하여 여러 API 서버 인스턴스를 동시에 실행 가능.

---

## Kube-API Server의 작동 원리

API 서버는 **클라이언트 요청을 수신하여 검증하고** 승인 절차를 거쳐 해당 작업을 실행. 요청이 승인되면 ETCD에 상태 변경을 기록하고, 필요한 경우 다른 구성 요소에 작업을 전달.

- **인증**: 클러스터에 접근하는 사용자와 서비스 계정을 확인.
- **권한 부여**: 인증된 요청에 대해 RBAC(Role-Based Access Control) 또는 ABAC(Attributes-Based Access Control) 방식으로 권한 검토.
- **어드미션 컨트롤**: 다양한 플러그인을 통해 요청을 검토하고 특정 정책에 따라 수정하거나 거부 가능.

---

## Kube-API Server 설정

Kube-API Server는 일반적으로 `/etc/kubernetes/manifests/kube-apiserver.yaml`에 있는 YAML 파일로 설정 관리.

1. **기본 설정 플래그**:
   - `--advertise-address=<API_SERVER_IP>`: 클러스터 내 API 서버의 주소.
   - `--secure-port=6443`: API 서버가 HTTPS로 요청을 수신하는 포트.
   - `--etcd-servers=<ETCD_SERVER_IP>`: ETCD 서버 주소.
   - `--service-cluster-ip-range=10.96.0.0/12`: 서비스 IP 주소 범위 설정.
   - `--allow-privileged=true`: 관리 권한이 필요한 작업 허용 여부.

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: kube-apiserver
     namespace: kube-system
   spec:
     containers:
     - command:
       - kube-apiserver
       - --advertise-address=10.0.0.1
       - --secure-port=6443
       - --etcd-servers=https://127.0.0.1:2379
       - --client-ca-file=/etc/kubernetes/pki/ca.crt
       - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
       - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
       - --allow-privileged=true
       image: k8s.gcr.io/kube-apiserver:v1.31.0
   ```

2. **인증 관련 플래그**:
   - `--client-ca-file`: 클라이언트 인증서 확인에 사용할 CA 파일 경로.
   - `--kubelet-client-certificate` 및 `--kubelet-client-key`: kubelet과의 인증을 위한 클라이언트 인증서 및 키.
   - `--authorization-mode`: 권한 부여 방식 지정 (Node, RBAC, ABAC 등).

---

## Kube-API Server 보안 설정

1. **TLS 인증서 설정**: TLS 인증서를 사용하여 API 서버와 클라이언트 간의 통신을 암호화.
   - **인증서 생성**: OpenSSL을 사용해 CA, 서버 인증서, 클라이언트 인증서를 생성 후 `/etc/kubernetes/pki/` 경로에 저장.
   - **플래그 설정**:
     ```yaml
     - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
     - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
     - --client-ca-file=/etc/kubernetes/pki/ca.crt
     ```

2. **RBAC 설정**: Kubernetes는 RBAC(Role-Based Access Control)를 통해 사용자 및 서비스 계정에 대한 권한을 제어.
   - ClusterRole과 RoleBinding을 사용해 권한 설정.
   - 예시:
     ```yaml
     apiVersion: rbac.authorization.k8s.io/v1
     kind: ClusterRole
     metadata:
       name: read-only
     rules:
     - apiGroups: [""]
       resources: ["pods"]
       verbs: ["get", "list"]
     ```

3. **API Server 접근 제한**: API 서버의 IP 주소를 제한하여 특정 네트워크 또는 클라이언트만 접근 가능.

---

## Kube-API Server 모니터링 및 로깅

1. **메트릭 수집**:
   - Prometheus와 같은 모니터링 도구를 사용하여 Kube-API Server의 메트릭 수집 및 대시보드 제공.
   - 메트릭 엔드포인트: `https://<API_SERVER_IP>:6443/metrics`

2. **로깅 설정**:
   - Kube-API Server의 로그는 `/var/log/kube-apiserver.log`에 저장.
   - **로그 확인 명령어**:
     ```bash
     tail -f /var/log/kube-apiserver.log
     ```

3. **로그 레벨 조정**:
   - `--v` 플래그를 통해 로그의 상세 수준 설정 (`0`이 기본, `10`은 최대 상세).

---

## Kube-API Server 관리 및 트러블슈팅

1. **API Server 상태 확인**:
   - API 서버와의 연결 상태 확인:
     ```bash
     kubectl get componentstatuses
     ```
   - Kube-API Server 상태를 지속적으로 모니터링하여 오류 탐지.

2. **API 요청 디버깅**:
   - `kubectl`의 `--v=8` 플래그를 사용하여 API 요청의 상세 로그 출력.
   - 예시:
     ```bash
     kubectl get pods --v=8
     ```

3. **API Server 재시작**:
   - `/etc/kubernetes/manifests/kube-apiserver.yaml` 파일을 수정한 후 Kubelet이 변경 사항을 감지하고 자동으로 재시작.

4. **ETCD 연결 오류 해결**:
   - Kube-API Server가 ETCD와 통신하지 못하는 경우 ETCD 서버 주소와 인증서 경로를 재확인.

---

## Kube-API Server 유용한 명령어 모음

- **Kube-API Server 버전 확인**:
  ```bash
  kubectl version --short
  ```

- **클러스터 상태 확인**:
  ```bash
  kubectl get componentstatuses
  ```

- **API 리소스 목록 확인**:
  ```bash
  kubectl api-resources
  ```

- **API 요청 상세 보기**:
  ```bash
  kubectl get pods --v=8
  ```

- **API Server 메트릭 확인**:
  ```bash
  curl -k https://<API_SERVER_IP>:6443/metrics
  ```

이 가이드를 통해 Kube-API Server의 기본 개념, 설정 및 보안 옵션을 익히고 Kubernetes 클러스터에서 안전하고 안정적으로 운영할 수 있음.
