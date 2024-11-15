# Pod

## 목차
1. [Pod란 무엇인가?](#Pod란-무엇인가)
2. [Pod의 작동 원리](#Pod의-작동-원리)
3. [Pod 설정](#Pod-설정)
4. [Multi-Container Pod](#Multi-Container-Pod)
5. [Pod Lifecycle](#Pod-Lifecycle)
6. [Pod 관리 및 운영](#Pod-관리-및-운영)
7. [Pod 주요 플래그](#Pod-주요-플래그)
8. [Pod 유용한 명령어 모음](#Pod-유용한-명령어-모음)

---

## Pod란 무엇인가?

**Pod**는 Kubernetes에서 **컨테이너의 기본 실행 단위**로, 하나 이상의 컨테이너를 그룹으로 묶어 **동일한 네트워크 네임스페이스와 스토리지를 공유**. Pod는 클러스터에서 배포와 스케일링의 최소 단위이며, 일반적으로 단일 애플리케이션 컨테이너를 포함하지만, 여러 개의 컨테이너로 구성될 수도 있음.

- **기능**: 컨테이너를 하나의 유닛으로 묶어 관리, 스케일링, 네트워크 설정 등 수행.
- **네트워크 공유**: 동일한 Pod 내의 모든 컨테이너는 동일한 IP 주소와 포트를 사용.
- **스토리지 공유**: Pod 내 컨테이너는 공유된 볼륨을 통해 파일 시스템을 공유 가능.

---

## Pod의 작동 원리

Pod는 **컨테이너를 하나의 그룹으로 묶어 동일한 네트워크와 스토리지 환경을 공유**. Kubernetes는 파드를 노드에 배치하고, 각 Pod는 고유의 IP 주소를 할당받아 클러스터 내에서 통신 가능. Pod 내 컨테이너들은 서로 **localhost**로 통신하고, 외부로부터 트래픽을 수신하는 경우, 서비스 또는 Ingress를 통해 접근.

1. **Pod IP 할당**: 각 Pod는 고유한 IP 주소를 받아 네트워크 통신 수행.
2. **공유 네트워크**: Pod 내 모든 컨테이너는 동일한 네트워크 네임스페이스 사용.
3. **공유 볼륨**: Pod 내 컨테이너들은 정의된 볼륨을 통해 데이터를 공유.

---

## Pod 설정

Pod는 YAML 파일로 정의되며, **metadata, spec** 섹션에 주요 설정 포함.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
    - name: my-container
      image: nginx:latest
      ports:
        - containerPort: 80
```

1. **metadata**: Pod의 이름, 네임스페이스, 레이블 등 메타데이터를 설정.
2. **spec**: Pod의 실제 사양을 설정하며, 주로 컨테이너의 이미지, 포트, 자원 제한 등을 포함.
3. **containers**: Pod 내에서 실행할 컨테이너 정의. 각 컨테이너의 이름과 이미지를 지정하며, 필요한 경우 포트와 환경 변수를 추가 설정.

---

## Multi-Container Pod

Multi-Container Pod는 **여러 컨테이너가 협력하여 하나의 작업을 수행**할 때 사용. 각 컨테이너는 독립적으로 실행되지만, 동일한 네트워크와 스토리지 자원을 공유하여 상호작용 가능.

### 예시
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
    - name: main-app
      image: nginx:latest
    - name: sidecar
      image: busybox
      command: ["sh", "-c", "echo Hello from the sidecar"]
```

- **주 컨테이너**: 주 애플리케이션 로직을 담당하는 컨테이너.
- **사이드카 컨테이너**: 보조 작업(로깅, 데이터 처리 등)을 수행하는 컨테이너.

---

## Pod Lifecycle

Pod의 라이프사이클은 컨테이너의 상태와 동일하게 여러 단계로 구분.

1. **Pending**: Pod가 생성되었지만, 모든 컨테이너가 아직 실행되지 않은 상태.
2. **Running**: 모든 컨테이너가 성공적으로 시작되어 실행 중.
3. **Succeeded**: 모든 컨테이너가 종료되고 성공적으로 완료된 상태.
4. **Failed**: 하나 이상의 컨테이너가 비정상적으로 종료된 상태.
5. **Unknown**: Pod 상태를 확인할 수 없는 경우.

라이프사이클을 통해 Pod의 현재 상태를 모니터링하고, 필요 시 조치를 취할 수 있음.

---

## Pod 관리 및 운영

1. **Pod 생성**:
   ```bash
   kubectl apply -f pod.yaml
   ```

2. **Pod 조회**:
   ```bash
   kubectl get pods
   ```

3. **Pod 상세 정보 확인**:
   ```bash
   kubectl describe pod <pod-name>
   ```

4. **Pod 로그 조회**:
   - 특정 Pod의 로그를 확인하여 문제를 진단.
   ```bash
   kubectl logs <pod-name> -c <container-name>
   ```

5. **Pod 삭제**:
   ```bash
   kubectl delete pod <pod-name>
   ```

---

## Pod 주요 플래그

1. **--restart-policy**:
   - Pod가 비정상적으로 종료될 경우의 재시작 정책을 설정.
   - 기본값은 `Always`이며, `OnFailure`, `Never` 옵션 선택 가능.

2. **--image-pull-policy**:
   - 컨테이너 이미지의 다운로드 정책 설정.
   - `Always`: 항상 최신 이미지를 다운로드.
   - `IfNotPresent`: 이미지가 없는 경우 다운로드.
   - `Never`: 이미지가 로컬에 있을 때만 사용.

3. **--host-network**:
   - `true`로 설정 시, Pod가 노드의 네트워크 네임스페이스를 사용하여 네트워크 설정을 공유.
   
4. **--service-account**:
   - Pod가 사용할 서비스 계정을 지정하여, API 서버 접근을 제한하거나 특정 권한 부여.

5. **--termination-grace-period-seconds**:
   - Pod가 종료될 때 대기할 시간을 초 단위로 설정하여, 이를 통해 컨테이너가 안전하게 종료될 수 있도록 함.

---

## Pod 유용한 명령어 모음

- **Pod 생성**:
  ```bash
  kubectl apply -f pod.yaml
  ```

- **Pod 상태 확인**:
  ```bash
  kubectl get pods
  ```

- **Pod 상세 정보 조회**:
  ```bash
  kubectl describe pod <pod-name>
  ```

- **Pod 로그 조회**:
  ```bash
  kubectl logs <pod-name> -c <container-name>
  ```

- **Pod 삭제**:
  ```bash
  kubectl delete pod <pod-name>
  ```
