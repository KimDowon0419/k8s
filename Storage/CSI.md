# CSI (Container Storage Interface)

## 목차
1. [CSI란 무엇인가?](#CSI란-무엇인가)
2. [CSI의 주요 목적](#CSI의-주요-목적)
3. [CSI의 구성 요소](#CSI의-구성-요소)
4. [CSI 드라이버의 작동 원리](#CSI-드라이버의-작동-원리)
5. [Kubernetes에서 CSI 드라이버 사용 방법](#Kubernetes에서-CSI-드라이버-사용-방법)
6. [CSI 유용한 명령어 모음](#CSI-유용한-명령어-모음)

---

## CSI란 무엇인가?

**CSI (Container Storage Interface)**는 **컨테이너 오케스트레이션 플랫폼(Kubernetes 등)에서 스토리지 시스템과 통합하기 위한 표준 인터페이스**입니다. CSI를 통해 클러스터 관리자와 스토리지 제공자가 특정 스토리지 기술에 종속되지 않고, 다양한 스토리지 백엔드와 Kubernetes를 통합할 수 있습니다.

- **특징**:
  - 클라우드 네이티브 스토리지 제공
  - 벤더 중립적인 표준화된 인터페이스
  - 확장 가능한 스토리지 플러그인 생태계

---

## CSI의 주요 목적

1. **스토리지 통합 표준화**:
   - 다양한 스토리지 벤더가 Kubernetes와 쉽게 통합할 수 있도록 표준화된 인터페이스 제공

2. **스토리지 유연성 증가**:
   - 관리자와 사용자는 특정 스토리지 기술에 종속되지 않고 다양한 옵션 선택 가능

3. **운영 단순화**:
   - Kubernetes와의 통합 작업 간소화, 스토리지 프로비저닝 및 관리 자동화

4. **확장성 및 커뮤니티 지원**:
   - 새로운 스토리지 기술과 기존 Kubernetes 클러스터 통합 가능

---

## CSI의 구성 요소

1. **CSI 드라이버**:
   - 스토리지 백엔드(예: AWS EBS, Ceph, NFS 등)와 Kubernetes를 연결하는 플러그인
   - Kubernetes 클러스터에 설치하여 동작

2. **Volume Plugin**:
   - CSI 드라이버와 Kubernetes가 통신하도록 지원

3. **External Provisioner**:
   - PersistentVolume(PV) 생성 및 삭제를 처리하는 외부 컨트롤러

4. **External Attacher**:
   - PersistentVolume을 특정 Node에 연결

5. **External Resizer**:
   - PersistentVolume 크기 조정 지원

6. **External Snapshotter**:
   - 볼륨 스냅샷 생성 및 복원 지원

---

## CSI 드라이버의 작동 원리

### CSI의 동작 과정

1. **Volume 요청**:
   - 사용자가 PersistentVolumeClaim(PVC)을 생성하면 Kubernetes는 CSI 드라이버를 호출
2. **Provisioning**:
   - CSI 드라이버는 스토리지 시스템에 요청하여 볼륨 생성
3. **Attachment**:
   - 생성된 볼륨을 특정 Node에 연결
4. **Mount**:
   - 볼륨을 Pod에 마운트하여 사용 가능하게 설정
5. **Volume Lifecycle 관리**:
   - 볼륨 크기 조정, 삭제, 스냅샷 생성 등 수행

---

## Kubernetes에서 CSI 드라이버 사용 방법

### 1. CSI 드라이버 설치

CSI 드라이버는 Helm 차트 또는 YAML 파일로 설치할 수 있습니다. 아래는 예시입니다.

#### Helm 차트를 사용하여 AWS EBS CSI 드라이버 설치
```bash
helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
helm install aws-ebs aws-ebs-csi-driver/aws-ebs-csi-driver
```

---

### 2. StorageClass 생성

스토리지 백엔드와 연결된 `StorageClass`를 생성합니다.

#### StorageClass 예시

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: csi-storage
provisioner: ebs.csi.aws.com
parameters:
  type: gp2
```

- **`provisioner`**: CSI 드라이버 이름 (스토리지 벤더에 따라 다름)
- **`parameters`**: 스토리지 생성 시 사용할 백엔드 설정

---

### 3. PersistentVolumeClaim(PVC) 생성

PVC를 사용하여 필요한 스토리지 요청을 정의합니다.

#### PVC 예시

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: csi-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: csi-storage
```

- **`accessModes`**: 볼륨의 액세스 모드 (예: `ReadWriteOnce`)
- **`resources.requests.storage`**: 요청한 스토리지 크기
- **`storageClassName`**: 사용할 StorageClass 이름

---

### 4. Pod에서 PVC 사용

PVC를 사용하여 볼륨을 Pod에 마운트합니다.

#### Pod 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: csi-app
spec:
  containers:
    - name: app-container
      image: nginx
      volumeMounts:
        - mountPath: "/data"
          name: csi-volume
  volumes:
    - name: csi-volume
      persistentVolumeClaim:
        claimName: csi-pvc
```

- **`volumeMounts`**: 컨테이너 내 볼륨 마운트 경로
- **`volumes.persistentVolumeClaim.claimName`**: PVC 이름 지정

---

## CSI 유용한 명령어 모음

- **설치된 CSI 드라이버 확인**
  ```bash
  kubectl get csidrivers
  ```

- **StorageClass 확인**
  ```bash
  kubectl get storageclass
  ```

- **PersistentVolume 및 PersistentVolumeClaim 상태 확인**
  ```bash
  kubectl get pv,pvc
  ```

- **Pod 상태 및 볼륨 마운트 확인**
  ```bash
  kubectl describe pod <pod-name>
  ```

- **CSI 드라이버 로그 확인**
  ```bash
  kubectl logs <csi-driver-pod-name>
  ```

---

CSI는 Kubernetes에서 스토리지 관리의 표준을 제공하며, 클러스터 내 스토리지의 유연성과 확장성을 크게 향상시킵니다. 다양한 스토리지 백엔드와 통합 가능하며, 클라우드 네이티브 환경에서 필수적인 스토리지 솔루션으로 자리 잡고 있습니다.
