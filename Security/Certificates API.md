# Certificates API

## 목차
1. [Certificates API란 무엇인가?](#Certificates-API란-무엇인가)
2. [Certificates API의 목적](#Certificates-API의-목적)
3. [Certificates API를 사용하는 방법](#Certificates-API를-사용하는-방법)
4. [CSR (Certificate Signing Request) 생성](#CSR-Certificate-Signing-Request-생성)
5. [Certificates API 관리 및 운영](#Certificates-API-관리-및-운영)
6. [Certificates API 유용한 명령어 모음](#Certificates-API-유용한-명령어-모음)

---

## Certificates API란 무엇인가?

**Certificates API**는 Kubernetes에서 **클러스터 내에서 TLS 인증서를 생성 및 관리**하기 위한 API입니다. 이 API는 Kubernetes 리소스인 **CertificateSigningRequest (CSR)**를 통해 **인증서 서명 요청**을 관리합니다.

- **기능**:
  - TLS 인증서 요청 및 발급
  - 인증서 승인 및 거부
  - 클러스터의 보안 통신 설정 간소화
- **주요 리소스**: `CertificateSigningRequest` (CSR)

---

## Certificates API의 목적

Certificates API는 다음과 같은 상황에서 유용합니다.

1. **자동화된 인증서 관리**:
   - 애플리케이션과 클러스터 컴포넌트 간의 보안 통신을 자동화
2. **클러스터 내 보안 통신 강화**:
   - 각 컴포넌트에 고유한 인증서를 발급하여 통신 보안 유지
3. **인증서 요청 워크플로 관리**:
   - 관리자가 승인하거나 거부할 수 있는 요청 기반 인증서 관리

---

## Certificates API를 사용하는 방법

Certificates API는 CSR 리소스를 생성하고, 이를 승인 또는 거부하여 인증서를 관리합니다.

### CSR의 주요 필드

- **`spec.request`**: 인증서 요청(PKCS#10 형식으로 Base64 인코딩된 데이터)
- **`spec.usages`**: 인증서의 사용 용도 (예: `client auth`, `server auth`)
- **`status.certificate`**: 승인된 인증서의 Base64 인코딩 데이터

---

## CSR (Certificate Signing Request) 생성

### 1. 개인 키 및 CSR 생성

```bash
openssl genrsa -out user.key 2048
openssl req -new -key user.key -out user.csr -subj "/CN=my-user/O=my-org"
```

- **`user.key`**: 개인 키
- **`user.csr`**: Certificate Signing Request

### 2. Kubernetes CSR 리소스 생성

아래 YAML 파일을 사용하여 Kubernetes에 CSR 리소스를 생성합니다.

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: my-user-csr
spec:
  request: <BASE64_ENCODED_CSR>
  usages:
    - client auth
```

1. **`request`**: Base64로 인코딩된 CSR 파일의 내용
   ```bash
   cat user.csr | base64 | tr -d '\n'
   ```

2. **`usages`**: 인증서의 사용 목적 (예: `client auth`, `server auth`)

### CSR 리소스 생성 명령어

```bash
kubectl apply -f csr.yaml
```

---

## Certificates API 관리 및 운영

### CSR 요청 승인

CSR 요청을 승인하려면 다음 명령어를 사용합니다.

```bash
kubectl certificate approve my-user-csr
```

### CSR 요청 거부

CSR 요청을 거부하려면 다음 명령어를 사용합니다.

```bash
kubectl certificate deny my-user-csr
```

### 승인된 인증서 확인 및 저장

승인된 CSR로부터 인증서를 저장하려면 다음 명령어를 사용합니다.

```bash
kubectl get csr my-user-csr -o jsonpath='{.status.certificate}' | base64 --decode > user.crt
```

### 클러스터에 인증서 적용

생성된 인증서를 사용하여 애플리케이션이나 클러스터 컴포넌트에 적용합니다.

---

## Certificates API 유용한 명령어 모음

- **CSR 생성**
  ```bash
  openssl genrsa -out user.key 2048
  openssl req -new -key user.key -out user.csr -subj "/CN=my-user/O=my-org"
  cat user.csr | base64 | tr -d '\n'
  ```

- **CSR 리소스 생성**
  ```bash
  kubectl apply -f csr.yaml
  ```

- **CSR 목록 확인**
  ```bash
  kubectl get csr
  ```

- **CSR 승인**
  ```bash
  kubectl certificate approve my-user-csr
  ```

- **CSR 거부**
  ```bash
  kubectl certificate deny my-user-csr
  ```

- **인증서 다운로드**
  ```bash
  kubectl get csr my-user-csr -o jsonpath='{.status.certificate}' | base64 --decode > user.crt
  ```

---

Certificates API는 Kubernetes 클러스터 내에서 TLS 인증서를 효율적으로 관리할 수 있는 강력한 도구입니다. 이를 통해 클러스터 통신 보안을 강화하고, 인증서 관리 작업을 자동화할 수 있습니다.
