# Multi-Container Pods

## 목차
1. [Multi-Container Pods란 무엇인가?](#Multi-Container-Pods란-무엇인가)
2. [Multi-Container Pods의 목적](#Multi-Container-Pods의-목적)
3. [Multi-Container Pods의 사용 사례](#Multi-Container-Pods의-사용-사례)
4. [Multi-Container Pods 설정 방법](#Multi-Container-Pods-설정-방법)
5. [Multi-Container Pods 관리 및 운영](#Multi-Container-Pods-관리-및-운영)
6. [Multi-Container Pods 유용한 명령어 모음](#Multi-Container-Pods-유용한-명령어-모음)

---

## Multi-Container Pods란 무엇인가?

**Multi-Container Pods**는 Kubernetes에서 **하나의 Pod 안에 여러 개의 컨테이너를 실행**하는 방식입니다. Multi-Container Pods는 **공유 네트워크와 스토리지**를 통해 컨테이너 간에 데이터를 교환하거나, 서로 보완적인 역할을 수행하도록 설계됩니다.

- **특징**:
  - 같은 네트워크 네임스페이스를 공유
  - 같은 볼륨을 공유하여 데이터 교환 가능
- **구성 방식**: 두 개 이상의 컨테이너가 `containers` 배열에 정의되어 하나의 Pod로 관리

---

## Multi-Container Pods의 목적

1. **작업 분리**:
   - 애플리케이션의 주요 기능을 분리하여 각 컨테이너가 특정 역할을 담당
   - 예: 데이터 처리 컨테이너와 로그 수집 컨테이너 분리

2. **보조 작업 수행**:
   - 하나의 컨테이너는 주 애플리케이션을 실행하고, 다른 컨테이너는 이를 지원하는 역할 수행
   - 예: 애플리케이션 컨테이너와 로그 프로세싱 컨테이너

3. **자원 공유**:
   - 같은 네트워크 네임스페이스와 볼륨을 공유하여 효율적인 데이터 교환

---

## Multi-Container Pods의 사용 사례

1. **로그 처리**:
   - 한 컨테이너가 애플리케이션 로그를 생성하고, 다른 컨테이너가 이를 수집 및 전송
   - 예: Nginx와 Fluentd 조합

2. **보안 기능 추가**:
   - 애플리케이션 컨테이너와 보안 작업을 수행하는 컨테이너를 함께 실행
   - 예: 웹 애플리케이션과 모니터링 컨테이너

3. **데이터 처리 파이프라인**:
   - 한 컨테이너가 데이터를 생성하고, 다른 컨테이너가 이를 실시간으로 처리

---

## Multi-Container Pods 설정 방법

Multi-Container Pods는 Pod의 `containers` 배열에 여러 컨테이너를 정의하여 구성합니다.

### Multi-Container Pod YAML 예시

아래는 `nginx`와 `fluentd` 컨테이너를 포함한 Multi-Container Pod 예시입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx
    - name: fluentd-container
      image: fluentd
      volumeMounts:
        - name: shared-logs
          mountPath: /var/log/nginx
  volumes:
    - name: shared-logs
      emptyDir: {}
```

1. **`containers` 배열**: 두 개의 컨테이너를 정의하여 동일한 Pod 안에서 실행
2. **볼륨 공유**:
   - `emptyDir`를 사용하여 두 컨테이너가 `/var/log/nginx` 디렉토리를 공유
3. **네트워크 공유**: 두 컨테이너는 동일한 네트워크 네임스페이스를 사용하여 `localhost`를 통해 통신 가능

---

## Multi-Container Pods 관리 및 운영

1. **Pod 생성**
   ```bash
   kubectl apply -f multi-container-pod.yaml
   ```

2. **Pod 상태 확인**
   - Pod의 두 컨테이너가 정상적으로 실행 중인지 확인
   ```bash
   kubectl get pods
   ```

3. **Pod 상세 정보 확인**
   - Pod 내 각 컨테이너의 상태와 로그 확인
   ```bash
   kubectl describe pod multi-container-pod
   ```

4. **개별 컨테이너의 로그 확인**
   - 컨테이너 이름을 지정하여 특정 컨테이너의 로그를 확인
   ```bash
   kubectl logs multi-container-pod -c nginx-container
   kubectl logs multi-container-pod -c fluentd-container
   ```

5. **Pod 삭제**
   ```bash
   kubectl delete pod multi-container-pod
   ```

---

## Multi-Container Pods 유용한 명령어 모음

- **Pod 생성**
  ```bash
  kubectl apply -f multi-container-pod.yaml
  ```

- **Pod 상태 확인**
  ```bash
  kubectl get pods
  ```

- **Pod의 특정 컨테이너 로그 확인**
  ```bash
  kubectl logs <pod-name> -c <container-name>
  ```

- **Pod 상세 정보 확인**
  ```bash
  kubectl describe pod <pod-name>
  ```

- **Pod 삭제**
  ```bash
  kubectl delete pod <pod-name>
  ```

Multi-Container Pods는 단일 작업을 여러 컨테이너로 나누어 처리하거나, 보조 작업을 지원하는 컨테이너를 추가하여 애플리케이션을 보다 유연하게 구성할 수 있는 강력한 방법을 제공합니다.
