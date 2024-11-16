# TLS in Kubernetes

## 목차
1. [TLS란 무엇인가?](#TLS란-무엇인가)
2. [Kubernetes에서 TLS의 역할](#Kubernetes에서-TLS의-역할)
3. [TLS 인증서 생성 및 관리](#TLS-인증서-생성-및-관리)
4. [TLS를 Kubernetes 리소스에 적용](#TLS를-Kubernetes-리소스에-적용)
5. [TLS 유용한 명령어 모음](#TLS-유용한-명령어-모음)

---

## TLS란 무엇인가?

**TLS (Transport Layer Security)**는 네트워크 통신에서 **데이터 암호화 및 인증**을 제공하는 보안 프로토콜입니다. TLS는 클라이언트와 서버 간의 통신을 보호하고, 데이터가 중간에 가로채지거나 조작되지 않도록 보장합니다.

- **기능**:
  - 데이터 암호화
  - 데이터 무결성 검증
  - 클라이언트 및 서버 인증

---

## Kubernetes에서 TLS의 역할

Kubernetes에서는 **클러스터 내 통신과 외부 통신 모두에서 TLS를 사용하여 보안을 강화**합니다. TLS는 다음과 같은 역할을 합니다.

1. **API 서버 통신 보호**:
   - Kubernetes 클러스터의 API 서버와 클라이언트 간의 통신을 TLS로 암호화
2. **Pod 간 통신 보안**:
   - 애플리케이션 컨테이너 간의 통신을 보호하기 위해 TLS 인증서를 사용
3. **Ingress 리소스에 HTTPS 적용**:
   - 외부에서 클러스터로 들어오는 트래픽에 대해 HTTPS 보안 적용
4. **Secret 데이터 암호화**:
   - Kubernetes Secrets에 TLS 인증서를 저장하고 참조하여 안전하게 관리

---

## TLS 인증서 생성 및 관리

Kubernetes에서 TLS 인증서를 생성하고 관리하는 방법은 다음과 같습니다.

### 인증서 생성

#### 1. **OpenSSL을 사용한 인증서 생성**
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key \
  -out tls.crt \
  -subj "/CN=mydomain.com/O=myorganization"
```

- **`tls.key`**: 개인 키
- **`tls.crt`**: 인증서

#### 2. **Kubernetes Secret으로 TLS 인증서 저장**
```bash
kubectl create secret tls my-tls-secret \
  --cert=tls.crt \
  --key=tls.key
```

- **`my-tls-secret`**: Secret 이름
- Secret은 Pod, Ingress 등에서 TLS 설정에 사용됩니다.

---

## TLS를 Kubernetes 리소스에 적용

### Ingress에 TLS 적용

Ingress 리소스에서 HTTPS를 사용하려면 TLS Secret을 참조하도록 설정합니다.

#### TLS를 사용하는 Ingress 예시

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
    - hosts:
        - mydomain.com
      secretName: my-tls-secret
  rules:
    - host: mydomain.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```

1. **`tls` 필드**:
   - `hosts`: HTTPS를 적용할 도메인
   - `secretName`: TLS 인증서가 저장된 Secret 이름
2. **`rules` 필드**:
   - HTTP 경로 및 백엔드 서비스 설정

### Pod에서 TLS Secret 사용

Pod가 TLS 인증서를 사용하려면 Secret을 볼륨으로 마운트하여 컨테이너가 인증서 파일에 접근할 수 있도록 설정합니다.

#### Pod에서 TLS Secret 사용 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tls-pod
spec:
  containers:
    - name: my-container
      image: nginx
      volumeMounts:
        - name: tls-volume
          mountPath: "/etc/tls"
          readOnly: true
  volumes:
    - name: tls-volume
      secret:
        secretName: my-tls-secret
```

1. **`volumes.secret`**:
   - TLS Secret을 볼륨으로 정의
2. **`volumeMounts`**:
   - 컨테이너 내부에서 인증서 파일 접근 경로 지정

---

## TLS 유용한 명령어 모음

- **TLS Secret 생성**
  ```bash
  kubectl create secret tls my-tls-secret \
    --cert=tls.crt \
    --key=tls.key
  ```

- **TLS Secret 확인**
  ```bash
  kubectl get secret my-tls-secret -o yaml
  ```

- **TLS Secret 디코딩**
  ```bash
  kubectl get secret my-tls-secret -o jsonpath="{.data.tls\.crt}" | base64 --decode
  ```

- **Ingress 설정 적용 및 확인**
  ```bash
  kubectl apply -f tls-ingress.yaml
  kubectl describe ingress tls-ingress
  ```

- **Pod에서 TLS Secret 마운트 상태 확인**
  ```bash
  kubectl describe pod tls-pod
  ```

TLS는 Kubernetes 클러스터의 보안을 강화하는 중요한 역할을 하며, 클러스터 내부 통신 및 외부 트래픽 모두에 안전한 환경을 제공합니다. TLS 인증서를 활용한 보안 설정은 클러스터 관리에서 필수적인 작업입니다.
