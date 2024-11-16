# Storage Class

## 목차
1. [Storage Class란 무엇인가?](#Storage-Class란-무엇인가)
2. [Storage Class의 주요 목적](#Storage-Class의-주요-목적)
3. [Storage Class의 구성 요소](#Storage-Class의-구성-요소)
4. [Storage Class 설정 방법](#Storage-Class-설정-방법)
5. [Storage Class와 PVC의 동작 원리](#Storage-Class와-PVC의-동작-원리)
6. [Storage Class 유용한 명령어 모음](#Storage-Class-유용한-명령어-모음)

---

## Storage Class란 무엇인가?

**Storage Class**는 Kubernetes에서 **스토리지 프로비저닝을 동적으로 관리**하기 위한 리소스입니다. 사용자는 Storage Class를 통해 스토리지 종류, 속성, 동작 방식을 정의하고, PVC(Persistent Volume Claim)가 생성될 때 이를 기반으로 PV(Persistent Volume)를 자동으로 생성할 수 있습니다.

- **기능**:
  - 동적 스토리지 프로비저닝 지원
  - 스토리지 종류와 성능 요구 사항 지정
  - 클러스터 관리자의 스토리지 관리 간소화

---

## Storage Class의 주요 목적

1. **동적 스토리지 프로비저닝**:
   - 사용자가 PVC를 생성할 때 필요한 PV를 자동으로 프로비저닝

2. **스토리지 옵션 표준화**:
   - 스토리지 유형(예: SSD, HDD)과 매개변수를 클러스터에 정의

3. **스토리지 관리 효율화**:
   - 다양한 워크로드 요구 사항을 충족하면서 관리 간소화

4. **플랫폼 간 유연성**:
   - 클라우드 제공업체 또는 온프레미스 스토리지와 통합 가능

---

## Storage Class의 구성 요소

1. **provisioner**:
   - 스토리지를 프로비저닝할 드라이버를 지정
   - 예: `kubernetes.io/aws-ebs`, `kubernetes.io/gce-pd`, `csi.ceph.com`

2. **parameters**:
   - 스토리지 유형이나 성능 옵션을 지정
   - 예: AWS EBS의 `type=gp2`, Ceph의 `pool=rbd`

3. **reclaimPolicy**:
   - PV가 사용되지 않을 때의 동작 정의
   - `Retain`, `Recycle`, `Delete`

4. **volumeBindingMode**:
   - PVC가 PV와 바인딩되는 시점 정의
   - `Immediate`: PVC 생성 시 즉시 바인딩
   - `WaitForFirstConsumer`: Pod가 PVC를 사용하는 시점에 바인딩

---

## Storage Class 설정 방법

### Storage Class YAML 예제

아래는 AWS EBS를 사용한 Storage Class 설정 예제입니다.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
```

- **`provisioner`**: AWS EBS CSI 드라이버 사용
- **`parameters`**:
  - `type`: 스토리지 유형 지정 (예: `gp2`는 SSD)
  - `fsType`: 파일 시스템 타입 (예: `ext4`)
- **`reclaimPolicy`**:
  - `Retain`: PV 삭제 시 데이터를 유지
- **`volumeBindingMode`**:
  - `WaitForFirstConsumer`: Pod가 생성될 때 PV 바인딩

---

### PVC와 연계한 동적 프로비저닝 예제

Storage Class를 사용하여 PVC 생성 시 PV가 동적으로 프로비저닝됩니다.

#### PVC 정의 예시

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-fast
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: fast-storage
```

- **`storageClassName`**: 사용할 Storage Class 이름 (`fast-storage`)
- **`resources.requests.storage`**: 요청된 스토리지 크기

PVC가 생성되면 `fast-storage` 설정을 기반으로 PV가 자동으로 생성됩니다.

---

## Storage Class와 PVC의 동작 원리

1. **사용자가 PVC 생성**:
   - PVC는 특정 Storage Class를 참조하여 스토리지 요청
2. **Storage Class 프로비저닝**:
   - Storage Class에 정의된 `provisioner`와 `parameters`를 기반으로 PV 생성
3. **PVC와 PV 바인딩**:
   - PVC가 요청한 리소스와 일치하는 PV와 자동으로 바인딩
4. **Pod에서 PVC 사용**:
   - 생성된 PV를 Pod에서 사용

---

## Storage Class 유용한 명령어 모음

- **현재 Storage Class 확인**
  ```bash
  kubectl get storageclass
  ```

- **Storage Class 세부 정보 확인**
  ```bash
  kubectl describe storageclass <storage-class-name>
  ```

- **PVC 상태 확인**
  ```bash
  kubectl get pvc
  ```

- **PV 상태 확인**
  ```bash
  kubectl get pv
  ```

- **Storage Class 삭제**
  ```bash
  kubectl delete storageclass <storage-class-name>
  ```

---

Storage Class는 Kubernetes에서 동적 스토리지 프로비저닝을 가능하게 하여, 관리자의 수작업을 줄이고 스토리지 리소스를 효율적으로 사용할 수 있도록 돕는 핵심 리소스입니다. 이를 통해 다양한 스토리지 백엔드와 요구 사항을 유연하게 관리할 수 있습니다.
