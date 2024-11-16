# View Certificate Details in Kubernetes

## 목차
1. [Certificate Details를 확인하는 이유](#Certificate-Details를-확인하는-이유)
2. [Kubernetes에서 Certificate 확인 방법](#Kubernetes에서-Certificate-확인-방법)
3. [OpenSSL을 사용한 Certificate 확인](#OpenSSL을-사용한-Certificate-확인)
4. [Kubernetes Secret에서 Certificate 확인](#Kubernetes-Secret에서-Certificate-확인)
5. [View Certificate Details 유용한 명령어 모음](#View-Certificate-Details-유용한-명령어-모음)

---

## Certificate Details를 확인하는 이유

Kubernetes에서 **TLS 인증서**는 클러스터 내부 및 외부 통신 보안을 위해 사용됩니다. Certificate Details를 확인하면 인증서가 적절히 구성되었는지, 유효 기간이 충분한지, 사용 중인 암호화 알고리즘이 적절한지 확인할 수 있습니다.

- **Certificate 검증 필요성**:
  - 유효 기간 확인
  - Subject 및 SAN (Subject Alternative Names) 확인
  - 발급 기관 (CA) 확인
  - 암호화 알고리즘 확인

---

## Kubernetes에서 Certificate 확인 방법

Kubernetes는 인증서를 Secret 리소스 형태로 저장하며, Base64로 인코딩된 인증서를 제공하기 때문에 이를 디코딩하여 확인해야 합니다.

### Secret에서 인증서 확인

1. **Secret 확인 명령어**
   ```bash
   kubectl get secret <secret-name> -o yaml
   ```

2. **인증서 디코딩**
   - Secret에서 `tls.crt` 값을 디코딩하여 인증서 내용을 확인합니다.
   ```bash
   kubectl get secret <secret-name> -o jsonpath="{.data.tls\.crt}" | base64 --decode > tls.crt
   ```

---

## OpenSSL을 사용한 Certificate 확인

OpenSSL을 사용하면 디코딩된 인증서 파일의 상세 정보를 확인할 수 있습니다.

### 인증서 상세 정보 확인 명령어

1. **파일에서 인증서 정보 확인**
   ```bash
   openssl x509 -in tls.crt -text -noout
   ```

2. **주요 확인 항목**
   - **Validity**: 인증서의 유효 기간
   - **Subject**: 인증서가 발급된 대상
   - **Issuer**: 인증서를 발급한 CA (Certificate Authority)
   - **SAN (Subject Alternative Names)**: 인증서에 포함된 도메인 이름

---

## Kubernetes Secret에서 Certificate 확인

### Secret에 저장된 인증서 확인 절차

1. **Secret 정보 가져오기**
   ```bash
   kubectl get secret my-tls-secret -o yaml
   ```

2. **인증서 디코딩 및 확인**
   ```bash
   kubectl get secret my-tls-secret -o jsonpath="{.data.tls\.crt}" | base64 --decode > tls.crt
   openssl x509 -in tls.crt -text -noout
   ```

### 출력 예시

```plaintext
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            04:92:71:b1:83:7f:43:34:ef:5a:77:63:1c:8f:2a:4f
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=MyRootCA
        Validity
            Not Before: Oct 1 00:00:00 2023 GMT
            Not After : Oct 1 00:00:00 2024 GMT
        Subject: CN=mydomain.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
    ...
```

---

## View Certificate Details 유용한 명령어 모음

- **Secret에서 인증서 디코딩**
  ```bash
  kubectl get secret <secret-name> -o jsonpath="{.data.tls\.crt}" | base64 --decode > tls.crt
  ```

- **디코딩된 인증서 정보 확인**
  ```bash
  openssl x509 -in tls.crt -text -noout
  ```

- **인증서 유효 기간 확인**
  ```bash
  openssl x509 -in tls.crt -noout -dates
  ```

- **SAN (Subject Alternative Names) 확인**
  ```bash
  openssl x509 -in tls.crt -text -noout | grep -A 1 "Subject Alternative Name"
  ```

---

Kubernetes에서 TLS 인증서를 관리할 때, 인증서의 세부 정보를 확인하는 것은 보안 문제를 예방하고 적절히 구성되었는지 검증하는 중요한 과정입니다. OpenSSL과 Kubernetes CLI를 조합하여 인증서의 유효성, 발급 정보, 암호화 수준 등을 확인할 수 있습니다.
