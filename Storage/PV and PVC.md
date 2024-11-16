# Persistent Volumes (PV) and Persistent Volume Claims (PVC)

## 목차
1. [Persistent Volume (PV)란 무엇인가?](#Persistent-Volume-PV란-무엇인가)
2. [Persistent Volume Claim (PVC)란 무엇인가?](#Persistent-Volume-Claim-PVC란-무엇인가)
3. [PV와 PVC의 관계](#PV와-PVC의-관계)
4. [PV와 PVC 설정 방법](#PV와-PVC-설정-방법)
5. [PV와 PVC의 동적 프로비저닝](#PV와-PVC의-동적-프로비저닝)
6. [PV와 PVC 유용한 명령어 모음](#PV와-PVC-유용한-명령어-모음)

---

## Persistent Volume (PV)란 무엇인가?

**Persistent Volume (PV)**는 Kubernetes에서 관리되는 **클러스터 내외부의 스토리지 리소스를 추상화한 개념**입니다. PV는 네트워크 스토리지, 클라우드 디스크, 로컬 디스크 등 다양한 스토리지 백엔드를 지원하며, Kubernetes 클러스터에서 독립적으로 프로비저닝되고 관리됩니다.

- **특징**:
  - 클러스터 관리자가 사전 정의
  - Pod의 수명 주기와 독립적
  - 지속적인 데이터 저장소로 사용

---

## Persistent Volume Claim (PVC)란 무엇인가?

**Persistent Volume Claim (PVC)**는 **사용자가 PV를 요청하기 위해 생성하는 리소스**입니다. PVC는 필요한 스토리지 크기, 액세스 모드 등을 정의하며, Kubernetes는 적합한 PV를 PVC에 바인딩합니다.

- **특징**:
  - 사용자 요청에 따라 PV와 바인딩
  - Pod에 마운트하여 데이터 저장소로 사용
  - 필요한 스토리지 크기와 액세스 모드 정의

---

## PV와 PVC의 관계

PV와 PVC는 서로 연관되어 있으며, Kubernetes는 PVC 요청에 따라 적합한 PV를 자동으로 바인딩합니다.

1. **PV**:
   - 관리자가 생성
   - 클러스터 내외의 스토리지 리소스를 정의
2. **PVC**:
   - 사용자가 요청
   - PV와 매칭되어 Pod에서 사용 가능

PVC가 삭제되면 관련된 PV는 **Retain**, **Recycle**, **Delete** 정책에 따라 처리됩니다.

---

## PV와 PVC 설정 방법

### 1. Persistent Volume (PV) 정의

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/mnt/data"
```

- **`capacity.storage`**: PV의 스토리지 크기
- **`accessModes`**:
  - `ReadWriteOnce`: 하나의 Node에서 읽기/쓰기가 가능
  - `ReadOnlyMany`: 여러 Node에서 읽기 가능
  - `ReadWriteMany`: 여러 Node에서 읽기/쓰기 가능
- **`persistentVolumeReclaimPolicy`**:
  - `Retain`: 데이터 유지
  - `Recycle`: 기본 데이터 삭제
  - `Delete`: PV 삭제
- **`hostPath`**: 로컬 디스크 경로

---

### 2. Persistent Volume Claim (PVC) 정의

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

- **`accessModes`**: PV에서 지원하는 액세스 모드와 일치해야 함
- **`resources.requests.storage`**: 요청한 스토리지 크기

---

### 3. Pod에서 PVC 사용

PVC를 사용하여 Pod에서 스토리지 마운트를 설정합니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-using-pvc
spec:
  containers:
    - name: app-container
      image: nginx
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: storage-volume
  volumes:
    - name: storage-volume
      persistentVolumeClaim:
        claimName: pvc-example
```

- **`volumeMounts.mountPath`**: 컨테이너 내부 경로
- **`volumes.persistentVolumeClaim.claimName`**: PVC 이름

---

## PV와 PVC의 동적 프로비저닝

### StorageClass를 사용한 동적 프로비저닝

StorageClass를 사용하면 PV를 수동으로 생성하지 않고 PVC 요청 시 자동으로 PV가 생성됩니다.

#### StorageClass 정의

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: dynamic-storage
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
```

- **`provisioner`**: 스토리지 프로비저너 이름 (클라우드 또는 스토리지 벤더에 따라 다름)
- **`parameters`**: 스토리지 유형 설정

#### PVC 정의

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: dynamic-storage
```

PVC가 생성되면 StorageClass의 설정에 따라 PV가 동적으로 프로비저닝됩니다.

---

## PV와 PVC 유용한 명령어 모음

- **현재 PV 목록 확인**
  ```bash
  kubectl get pv
  ```

- **현재 PVC 목록 확인**
  ```bash
  kubectl get pvc
  ```

- **PV와 PVC 상태 확인**
  ```bash
  kubectl describe pv <pv-name>
  kubectl describe pvc <pvc-name>
  ```

- **PV 및 PVC 삭제**
  ```bash
  kubectl delete pv <pv-name>
  kubectl delete pvc <pvc-name>
  ```

- **Pod에서 PVC 연결 확인**
  ```bash
  kubectl describe pod <pod-name>
  ```

---

PV와 PVC는 Kubernetes에서 애플리케이션의 데이터 저장소를 제공하고 관리하는 핵심 리소스입니다. 동적 프로비저닝을 활용하면 스토리지 관리가 더욱 간소화되며, 다양한 스토리지 백엔드를 지원하여 유연한 데이터 관리가 가능합니다.
