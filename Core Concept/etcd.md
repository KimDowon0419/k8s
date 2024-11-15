# ETCD

## 목차
1. [ETCD란 무엇인가?](#ETCD란-무엇인가)
2. [ETCD의 작동 원리](#ETCD의-작동-원리)
3. [ETCD 설치 및 초기 설정](#ETCD-설치-및-초기-설정)
4. [ETCD 클러스터 설정](#ETCD-클러스터-설정)
5. [ETCD 관리 및 운영](#ETCD-관리-및-운영)
6. [ETCD 백업과 복구](#ETCD-백업과-복구)
7. [ETCD 보안 설정](#ETCD-보안-설정)
8. [ETCD 최적화 및 트러블슈팅](#ETCD-최적화-및-트러블슈팅)
9. [ETCD 유용한 명령어 모음](#ETCD-유용한-명령어-모음)

---

## ETCD란 무엇인가?

**ETCD**는 분산형 키-값 데이터 저장소로, Kubernetes에서 클러스터의 상태 정보를 저장하는 **중앙 데이터 저장소 역할**을 수행. ETCD는 고가용성, 강력한 일관성 및 빠른 복구를 지원하며, 클러스터 내 모든 구성 요소가 이곳에서 상태 정보를 가져오고 저장.

- **용도**: 클러스터 상태 저장 (예: 파드, 서비스, 설정 정보 등)
- **기본 동작**: 데이터는 key-value 형식으로 저장되고, ETCD의 데이터는 Kubernetes API 서버와 통신을 통해 최신 상태로 유지.
- **고가용성**: 3개 이상의 노드로 이루어진 **ETCD 클러스터**를 구성하여 높은 내구성을 보장.

---

## ETCD의 작동 원리

ETCD는 **Raft consensus 알고리즘**을 사용하여 데이터의 일관성과 안정성을 보장. 다중 노드에서 동작하는 클러스터에서는 Leader가 모든 쓰기 요청을 처리하며, **Follower 노드는 Leader의 상태를 지속적으로 복제**하여 최신 상태를 유지.

- **Raft 알고리즘**: 분산 환경에서 강력한 일관성과 데이터 무결성을 보장.
- **Leader Election**: 클러스터 내에서 Leader 노드가 주기적으로 선출되어 쓰기 요청을 담당.

---

## ETCD 설치 및 초기 설정

1. **ETCD 바이너리 설치**
   ```bash
   wget -q --show-progress --https-only --timestamping \
     "https://github.com/etcd-io/etcd/releases/download/v3.5.1/etcd-v3.5.1-linux-amd64.tar.gz"
   tar -xvf etcd-v3.5.1-linux-amd64.tar.gz
   sudo mv etcd etcdctl /usr/local/bin/
   ```

2. **ETCD 서비스 설정**  
   `/etc/systemd/system/etcd.service` 파일을 생성하고, ETCD 서비스 정의.
   ```ini
   [Unit]
   Description=etcd key-value store
   Documentation=https://github.com/coreos
   After=network.target

   [Service]
   ExecStart=/usr/local/bin/etcd \\
     --name <node-name> \\
     --data-dir /var/lib/etcd \\
     --listen-client-urls https://127.0.0.1:2379 \\
     --advertise-client-urls https://127.0.0.1:2379
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```

3. **ETCD 서비스 시작 및 활성화**
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl start etcd
   sudo systemctl enable etcd
   ```

---

## ETCD 클러스터 설정

다중 노드 환경에서 **고가용성 클러스터를 구성**하려면 각 노드를 설정하고 상호 연결해야 함.

1. **클러스터 내 각 노드에 동일한 설정 파일 생성** (`etcd.service`).
2. **노드별 `initial-cluster` 설정**:
   ```ini
   --initial-cluster "node1=https://node1:2380,node2=https://node2:2380,node3=https://node3:2380"
   ```
3. **ETCD 클러스터 부트스트랩**:
   ```bash
   etcd --initial-cluster-state new
   ```

---

## ETCD 관리 및 운영

ETCD는 **`etcdctl`**이라는 CLI를 통해 관리.

- **클러스터 상태 확인**:
  ```bash
  etcdctl endpoint status --write-out=table
  ```

- **키-값 데이터 저장 및 조회**:
  ```bash
  etcdctl put /example/key "Hello ETCD"
  etcdctl get /example/key
  ```

---

## ETCD 백업과 복구

### 백업

ETCD 백업은 **스냅샷** 방식으로 수행되며, 이를 통해 클러스터 데이터를 안전하게 보관.

```bash
ETCDCTL_API=3 etcdctl snapshot save /path/to/backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/etcd-server.crt \
  --key=/etc/etcd/etcd-server.key
```

### 복구

1. **복구 작업 전 기존 데이터 삭제**:
   ```bash
   rm -rf /var/lib/etcd/*
   ```

2. **스냅샷에서 데이터 복구**:
   ```bash
   ETCDCTL_API=3 etcdctl snapshot restore /path/to/backup.db --data-dir /var/lib/etcd
   ```

3. **ETCD 재시작**:
   ```bash
   sudo systemctl start etcd
   ```

---

## ETCD 보안 설정

### TLS 인증서 설정

ETCD는 **TLS 인증서를 사용해 보안을 강화**할 수 있음.

1. **CA 및 인증서 발급**:
   - 인증서 및 키 파일을 생성하여 `/etc/etcd`에 저장.

2. **ETCD 서비스에 인증서 적용**:
   `/etc/systemd/system/etcd.service` 파일의 `ExecStart`에 다음 플래그 추가:
   ```ini
   --cert-file=/etc/etcd/etcd-server.crt \\
   --key-file=/etc/etcd/etcd-server.key \\
   --client-cert-auth \\
   --trusted-ca-file=/etc/etcd/ca.crt
   ```

3. **ETCD 재시작**:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart etcd
   ```

---

## ETCD 최적화 및 트러블슈팅

- **디스크 I/O 최적화**: ETCD는 I/O 집약적인 작업을 수행하므로, 빠른 디스크를 사용하거나 SSD로 성능을 개선.
- **모니터링 및 경고**: Prometheus와 같은 도구로 ETCD 메트릭을 모니터링하고, Latency 및 Disk Space 관련 경고 설정.
- **로그 확인**: ETCD 로그는 `/var/log/etcd.log`에서 확인 가능하며, 오류 발생 시 즉각 조치.

---

## ETCD 유용한 명령어 모음

- **ETCD 버전 확인**:
  ```bash
  etcd --version
  ```

- **특정 키 삭제**:
  ```bash
  etcdctl del /example/key
  ```

- **키 목록 조회**:
  ```bash
  etcdctl get "" --prefix --keys-only
  ```

- **클러스터 구성 확인**:
  ```bash
  etcdctl member list
  ```

이 문서를 통해 ETCD의 기본 개념부터 고급 설정과 관리까지 필요한 내용을 학습하고, Kubernetes에서 안정적이고 안전하게 운영 가능.
