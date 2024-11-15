# Init Containers

## 목차
1. [Init Containers란 무엇인가?](#Init-Containers란-무엇인가)
2. [Init Containers의 목적](#Init-Containers의-목적)
3. [Init Containers의 사용 사례](#Init-Containers의-사용-사례)
4. [Init Containers 설정 방법](#Init-Containers-설정-방법)
5. [Init Containers 관리 및 운영](#Init-Containers-관리-및-운영)
6. [Init Containers 유용한 명령어 모음](#Init-Containers-유용한-명령어-모음)

---

## Init Containers란 무엇인가?

**Init Containers**는 Kubernetes Pod 내에서 **애플리케이션 컨테이너가 실행되기 전에 초기 작업을 수행하는 컨테이너**입니다. Init Container는 **순차적으로 실행되며, 성공적으로 완료되어야만 다음 Init Container 또는 메인 애플리케이션 컨테이너가 시작**됩니다. Init Containers는 애플리케이션 컨테이너와 별도로 구성할 수 있으며, 주로 **환경 설정, 준비 작업, 외부 의존성 검사** 등의 작업을 수행합니다.

- **특징**: 각 Init Container는 순차적으로 실행되고, 모두 성공적으로 완료되어야 애플리케이션 컨테이너가 시작
- **독립적 구성**: 애플리케이션 컨테이너와 별도로 정의되어 각기 다른 이미지와 설정 사용 가능

---

## Init Containers의 목적

Init Containers는 다음과 같은 목적을 위해 사용됩니다.

1. **환경 설정**: 애플리케이션 실행 전에 필요한 환경이나 의존성을 설정
2. **데이터 초기화 및 준비**: 애플리케이션이 시작되기 전에 필요한 데이터나 파일을 생성
3. **외부 서비스 의존성 확인**: 애플리케이션이 의존하는 데이터베이스나 외부 서비스가 준비되었는지 확인

---

## Init Containers의 사용 사례

1. **외부 서비스 대기**:
   - 메인 컨테이너가 시작되기 전에 데이터베이스가 준비되었는지 확인하는 작업 수행
2. **데이터 복사 및 설정 파일 생성**:
   - ConfigMap이나 Secret에서 가져온 데이터를 메인 컨테이너가 사용하는 위치로 복사
3. **권한 설정**:
   - 애플리케이션 컨테이너가 접근해야 하는 디렉토리에 권한 설정

---

## Init Containers 설정 방법

Init Containers는 Pod의 `initContainers` 배열에 정의합니다. `containers`와 유사하지만, Init Containers는 주 애플리케이션 컨테이너보다 먼저 실행되며 순차적으로 실행됩니다.

### Init Container 예시

아래 예시는 `init-myservice`라는 Init Container가 실행되어 애플리케이션 컨테이너가 시작되기 전 `/work-dir`에 초기 파일을 생성하는 설정입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-container-pod
spec:
  initContainers:
    - name: init-myservice
      image: busybox
      command: ["sh", "-c", "echo Initializing... > /work-dir/init-status.txt"]
      volumeMounts:
        - name: workdir
          mountPath: "/work-dir"
  containers:
    - name: myapp-container
      image: nginx
      volumeMounts:
        - name: workdir
          mountPath: "/usr/share/nginx/html"
  volumes:
    - name: workdir
      emptyDir: {}
```

1. **initContainers**: Pod의 초기 작업을 담당하는 Init Containers 정의
2. **volumeMounts**: Init Container와 애플리케이션 컨테이너 간에 데이터 공유
3. **순차적 실행**: 모든 Init Containers가 성공적으로 완료되어야 애플리케이션 컨테이너가 시작

---

## Init Containers 관리 및 운영

1. **Pod 생성**
   ```bash
   kubectl apply -f init-container-pod.yaml
   ```

2. **Pod 상태 확인**
   - Init Containers의 실행 상태를 포함하여 Pod의 상태를 확인합니다.
   ```bash
   kubectl get pods
   ```

3. **Pod 상세 정보 확인**
   - Init Containers의 실행 상태와 로그를 확인할 수 있습니다.
   ```bash
   kubectl describe pod init-container-pod
   ```

4. **Init Container 로그 확인**
   - Init Container의 작업 로그를 확인하여 초기화 작업이 정상적으로 완료되었는지 확인할 수 있습니다.
   ```bash
   kubectl logs init-container-pod -c init-myservice
   ```

5. **Pod 삭제**
   ```bash
   kubectl delete pod init-container-pod
   ```

---

## Init Containers 유용한 명령어 모음

- **Pod 생성**
  ```bash
  kubectl apply -f init-container-pod.yaml
  ```

- **Pod 상태 확인**
  ```bash
  kubectl get pods
  ```

- **Pod 상세 정보 확인 (Init Container 포함)**
  ```bash
  kubectl describe pod <pod-name>
  ```

- **Init Container의 로그 확인**
  ```bash
  kubectl logs <pod-name> -c <init-container-name>
  ```

- **Pod 삭제**
  ```bash
  kubectl delete pod <pod-name>
  ```

Init Containers는 애플리케이션 실행 전 필요한 초기 설정을 수행하는 데 유용하며, 환경 설정, 데이터 초기화, 의존성 확인 등 다양한 준비 작업을 수행할 수 있습니다.
