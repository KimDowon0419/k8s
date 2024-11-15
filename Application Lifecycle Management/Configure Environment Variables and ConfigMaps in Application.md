# Configure Environment Variables and ConfigMaps in Applications

## 목차
1. [Environment Variables란 무엇인가?](#Environment-Variables란-무엇인가)
2. [ConfigMaps란 무엇인가?](#ConfigMaps란-무엇인가)
3. [Environment Variables와 ConfigMaps의 목적](#Environment-Variables와-ConfigMaps의-목적)
4. [Environment Variables 설정 방법](#Environment-Variables-설정-방법)
5. [ConfigMaps 설정 및 사용 방법](#ConfigMaps-설정-및-사용-방법)
6. [Environment Variables와 ConfigMaps 유용한 명령어 모음](#Environment-Variables와-ConfigMaps-유용한-명령어-모음)

---

## Environment Variables란 무엇인가?

**Environment Variables**는 Kubernetes에서 **컨테이너 내 애플리케이션의 환경 구성을 설정하는 데 사용되는 변수**입니다. 애플리케이션이 실행될 때 설정값을 동적으로 주입하여 특정 환경에 맞게 설정할 수 있습니다. 이를 통해 컨테이너 이미지를 수정하지 않고도 다양한 환경에 대응할 수 있습니다.

- **기능**: 실행 시 설정 정보를 주입하여 애플리케이션의 환경 구성
- **사용 용도**: 데이터베이스 URL, API 키, 서비스 포트 등의 동적 설정

---

## ConfigMaps란 무엇인가?

**ConfigMaps**는 Kubernetes에서 **비밀 정보가 아닌 구성 데이터를 관리하기 위해 사용하는 리소스**입니다. ConfigMaps는 다양한 설정값을 텍스트 형태로 저장하여 여러 컨테이너에 적용할 수 있습니다. ConfigMaps에 저장된 데이터는 환경 변수, 명령어 인자, 파일 볼륨 형태로 컨테이너에 주입할 수 있습니다.

- **기능**: 설정 파일이나 문자열 데이터를 관리하고, 여러 애플리케이션에 일관된 설정 제공
- **사용 용도**: 애플리케이션 구성 파일, 환경 변수, 설정 값 저장

---

## Environment Variables와 ConfigMaps의 목적

1. **환경 맞춤 설정**: 개발, 테스트, 운영 등 각기 다른 환경에서 동일한 이미지를 사용하면서도 환경별 설정을 적용할 수 있음
2. **구성 관리 용이성**: 설정 값을 애플리케이션 코드와 분리하여 유지보수성을 높임
3. **보안성 유지**: 중요한 정보는 ConfigMaps 대신 Secrets에 저장하며, ConfigMaps는 비밀이 아닌 설정값 관리에 활용

---

## Environment Variables 설정 방법

Environment Variables는 Pod의 YAML 파일에서 **`env` 필드**를 통해 설정할 수 있습니다.

### Static Environment Variables 설정 예시

아래 예시는 환경 변수를 직접 Pod YAML에 정의하여 설정하는 방식입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-var-pod
spec:
  containers:
    - name: my-container
      image: nginx
      env:
        - name: ENVIRONMENT
          value: "production"
        - name: LOG_LEVEL
          value: "debug"
```

1. **`env` 필드**: 컨테이너 내부에 설정할 환경 변수를 지정합니다.
2. **name/value 쌍**: `name`은 환경 변수 이름, `value`는 해당 변수에 주입할 값을 정의합니다.

### ConfigMap을 통한 Environment Variables 설정 예시

ConfigMap에 저장된 값을 환경 변수로 사용하는 예시입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-from-configmap-pod
spec:
  containers:
    - name: my-container
      image: nginx
      envFrom:
        - configMapRef:
            name: my-configmap
```

- **envFrom**: ConfigMap의 모든 키-값 쌍을 환경 변수로 로드합니다.
- **configMapRef**: `name` 필드를 통해 참조할 ConfigMap을 지정합니다.

---

## ConfigMaps 설정 및 사용 방법

ConfigMap은 `kubectl create configmap` 명령어나 YAML 파일로 생성할 수 있습니다.

### ConfigMap 생성 예시

1. **명령어로 ConfigMap 생성**
   ```bash
   kubectl create configmap my-configmap --from-literal=APP_COLOR=blue --from-literal=APP_MODE=production
   ```

2. **YAML 파일로 ConfigMap 생성**

   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: my-configmap
   data:
     APP_COLOR: blue
     APP_MODE: production
   ```

### ConfigMap을 Pod에 주입하기

ConfigMap을 환경 변수로 주입하여 Pod에서 사용할 수 있습니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: my-container
      image: nginx
      env:
        - name: APP_COLOR
          valueFrom:
            configMapKeyRef:
              name: my-configmap
              key: APP_COLOR
```

1. **`valueFrom` 필드**: 환경 변수의 값을 ConfigMap에서 가져옵니다.
2. **configMapKeyRef**: ConfigMap의 특정 키 값을 가져와 환경 변수로 설정할 수 있습니다.

---

## Environment Variables와 ConfigMaps 유용한 명령어 모음

- **ConfigMap 생성**
  ```bash
  kubectl create configmap my-configmap --from-literal=APP_COLOR=blue --from-literal=APP_MODE=production
  ```

- **ConfigMap 확인**
  ```bash
  kubectl get configmap my-configmap -o yaml
  ```

- **Pod에 설정된 환경 변수 확인**
  ```bash
  kubectl exec <pod-name> -- printenv
  ```

Environment Variables와 ConfigMaps는 Kubernetes에서 애플리케이션의 동적 구성을 지원하여 다양한 환경에서 유연한 애플리케이션 구성을 가능하게 합니다.
