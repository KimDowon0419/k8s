# Configure Secrets in Applications

## 목차
1. [Secrets란 무엇인가?](#Secrets란-무엇인가)
2. [Secrets의 목적](#Secrets의-목적)
3. [Secrets 설정 및 사용 방법](#Secrets-설정-및-사용-방법)
4. [Secrets를 환경 변수로 사용하는 방법](#Secrets를-환경-변수로-사용하는-방법)
5. [Secrets를 파일 볼륨으로 마운트하는 방법](#Secrets를-파일-볼륨으로-마운트하는-방법)
6. [Secrets 유용한 명령어 모음](#Secrets-유용한-명령어-모음)

---

## Secrets란 무엇인가?

**Secrets**는 Kubernetes에서 **비밀번호, 토큰, API 키 등 민감한 정보를 안전하게 저장하고 관리**하기 위해 사용하는 리소스입니다. Secrets는 ConfigMap과 유사하게 애플리케이션에 필요한 데이터를 주입할 수 있지만, Base64로 인코딩되어 저장되며, API 통신 시에도 암호화된 상태로 전송되어 보안성을 높입니다.

- **기능**: 민감한 데이터를 안전하게 저장하고 애플리케이션에 주입
- **보안성**: Base64 인코딩 및 Kubernetes API 암호화로 보호

---

## Secrets의 목적

Secrets는 다음과 같은 목적을 위해 사용됩니다.

1. **민감한 정보 보호**: 비밀번호, 인증 토큰, API 키 등 중요한 정보를 안전하게 관리
2. **환경 맞춤 구성**: 개발, 테스트, 운영 환경별로 각기 다른 보안 설정을 적용하여 애플리케이션을 구성
3. **애플리케이션 보안 강화**: Kubernetes API를 통해 안전하게 정보를 전달하여 보안 사고를 방지

---

## Secrets 설정 및 사용 방법

Secrets는 `kubectl create secret` 명령어나 YAML 파일을 통해 생성할 수 있습니다.

### Secrets 생성 예시

1. **명령어로 Secrets 생성**
   ```bash
   kubectl create secret generic my-secret --from-literal=username=myuser --from-literal=password=mypassword
   ```

2. **YAML 파일로 Secrets 생성**

   ```yaml
   apiVersion: v1
   kind: Secret
   metadata:
     name: my-secret
   type: Opaque
   data:
     username: bXl1c2Vy  # Base64 인코딩된 값
     password: bXlwYXNzd29yZA==
   ```

### Base64 인코딩 예시

Secrets의 데이터를 수동으로 설정할 때는 값을 **Base64로 인코딩**해야 합니다.

```bash
echo -n 'myuser' | base64
# 출력: bXl1c2Vy

echo -n 'mypassword' | base64
# 출력: bXlwYXNzd29yZA==
```

---

## Secrets를 환경 변수로 사용하는 방법

Secrets에 저장된 값을 환경 변수로 주입하여 애플리케이션에서 사용할 수 있습니다.

### 환경 변수로 Secrets 사용 예시

아래는 `my-secret`이라는 Secret을 사용하여 `username`과 `password`를 환경 변수로 주입하는 Pod 설정 예시입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
    - name: my-container
      image: nginx
      env:
        - name: USERNAME
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: username
        - name: PASSWORD
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: password
```

1. **`valueFrom.secretKeyRef`**: Secret에서 특정 키의 값을 가져와 환경 변수로 설정
2. **name과 key**: `name`은 Secret의 이름을, `key`는 Secret에서 가져올 특정 키를 지정

---

## Secrets를 파일 볼륨으로 마운트하는 방법

Secrets는 **파일 형태로 볼륨에 마운트하여** 애플리케이션이 파일로 접근할 수 있도록 설정할 수 있습니다.

### 파일 볼륨으로 Secrets 사용 예시

아래는 `my-secret`이라는 Secret을 파일 형태로 `/etc/secret-volume`에 마운트하는 예시입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-vol-pod
spec:
  containers:
    - name: my-container
      image: nginx
      volumeMounts:
        - name: secret-volume
          mountPath: "/etc/secret-volume"
          readOnly: true
  volumes:
    - name: secret-volume
      secret:
        secretName: my-secret
```

1. **volumeMounts**: Secret을 `/etc/secret-volume` 경로에 마운트
2. **volumes.secret**: `secretName` 필드로 참조할 Secret의 이름을 지정

---

## Secrets 유용한 명령어 모음

- **Secret 생성 (일반 텍스트 사용)**
  ```bash
  kubectl create secret generic my-secret --from-literal=username=myuser --from-literal=password=mypassword
  ```

- **Secret 확인 (디코딩되지 않은 상태)**
  ```bash
  kubectl get secret my-secret -o yaml
  ```

- **Secret의 값 디코딩하여 확인**
  ```bash
  kubectl get secret my-secret -o jsonpath="{.data.username}" | base64 --decode
  ```

- **Secret 삭제**
  ```bash
  kubectl delete secret my-secret
  ```

Secrets는 Kubernetes에서 민감한 정보를 안전하게 관리하고 애플리케이션에 주입하는 필수 도구로, 보안을 유지하면서도 유연한 환경 구성을 가능하게 합니다.
