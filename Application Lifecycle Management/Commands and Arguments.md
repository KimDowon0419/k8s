# Commands and Arguments

## 목차
1. [Commands와 Arguments란 무엇인가?](#Commands와-Arguments란-무엇인가)
2. [Commands와 Arguments 설정 목적](#Commands와-Arguments-설정-목적)
3. [Commands와 Arguments 설정 방법](#Commands와-Arguments-설정-방법)
4. [Commands와 Arguments 예시](#Commands와-Arguments-예시)
5. [Commands와 Arguments 유용한 명령어 모음](#Commands와-Arguments-유용한-명령어-모음)

---

## Commands와 Arguments란 무엇인가?

**Commands**와 **Arguments**는 Kubernetes에서 **컨테이너가 시작될 때 실행할 명령어와 인자를 지정**하는 방법입니다. 기본적으로 컨테이너는 도커 이미지에 정의된 `ENTRYPOINT`와 `CMD`를 사용해 시작되지만, Kubernetes에서는 이를 `command`와 `args`로 재정의할 수 있습니다.

- **Commands**: 컨테이너가 시작될 때 실행할 기본 명령어를 지정 (Docker의 `ENTRYPOINT`에 해당)
- **Arguments**: 명령어에 추가적으로 전달할 인자를 지정 (Docker의 `CMD`에 해당)

이를 통해 컨테이너의 실행 방식을 유연하게 제어할 수 있습니다.

---

## Commands와 Arguments 설정 목적

Commands와 Arguments는 다음과 같은 목적을 위해 사용됩니다.

1. **기본 실행 동작 재정의**: 이미지의 기본 설정을 덮어쓰고, 컨테이너가 특정 명령어로 시작되도록 설정
2. **환경 맞춤 구성**: 동일한 이미지로 다양한 실행 옵션을 지원하여, 애플리케이션의 동작을 환경에 맞게 조정
3. **작업 자동화 및 스크립트 실행**: 컨테이너 시작 시 특정 스크립트 또는 작업을 자동으로 수행하도록 설정

---

## Commands와 Arguments 설정 방법

Commands와 Arguments는 Pod의 YAML 파일에 `command`와 `args` 필드를 추가하여 설정합니다.

1. **command**: 컨테이너의 실행 명령어를 지정합니다.
2. **args**: 명령어에 전달할 인자를 지정합니다.

컨테이너가 시작되면 `command`에 지정된 명령어가 실행되고, `args`에 지정된 인자가 해당 명령어에 전달됩니다.

---

## Commands와 Arguments 예시

아래는 Nginx 컨테이너에 `command`와 `args`를 사용하여 실행 시 `sleep` 명령어로 대체하는 예시입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: nginx
      image: nginx
      command: ["sleep"]
      args: ["3600"]
```

위 예시는 다음과 같이 작동합니다:

1. **command**: `["sleep"]`는 `nginx` 대신 `sleep` 명령어를 실행하도록 설정합니다.
2. **args**: `["3600"]`는 `sleep` 명령어에 `3600`(초)을 전달하여, 컨테이너가 1시간 동안 대기하도록 만듭니다.

### 복잡한 명령어 예시

아래는 `curl` 명령어를 사용하여 `http://example.com`에 요청을 보내는 컨테이너 예시입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: curl-pod
spec:
  containers:
    - name: curl-container
      image: curlimages/curl
      command: ["curl"]
      args: ["-s", "http://example.com"]
```

1. **command**: `["curl"]`은 `curl` 명령어를 실행하도록 설정합니다.
2. **args**: `["-s", "http://example.com"]`은 `curl`에 인자로 `-s`와 URL을 전달하여 요청을 실행합니다.

---

## Commands와 Arguments 유용한 명령어 모음

- **컨테이너의 명령어 및 인자 확인**
  ```bash
  kubectl describe pod <pod-name> | grep -A 5 "Containers"
  ```

- **Pod 로그 확인 (명령어 실행 결과 확인)**
  ```bash
  kubectl logs <pod-name>
  ```

- **Pod 생성 및 명령어 실행 확인**
  - YAML 파일로 Pod 생성 후, 설정한 명령어가 올바르게 실행되었는지 확인합니다.
  ```bash
  kubectl apply -f command-args-pod.yaml
  ```

Commands와 Arguments는 Kubernetes에서 컨테이너의 실행 동작을 제어할 때 중요한 도구로, 다양한 작업 자동화와 실행 환경 맞춤 설정에 유용하게 사용됩니다.
