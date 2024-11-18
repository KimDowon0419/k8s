# Ingress in Kubernetes

## 목차
1. [Ingress란 무엇인가?](#Ingress란-무엇인가)
2. [Ingress의 주요 역할](#Ingress의-주요-역할)
3. [Ingress 구성 요소](#Ingress-구성-요소)
4. [Ingress Controller와 동작 원리](#Ingress-Controller와-동작-원리)
5. [Ingress 설정 예제](#Ingress-설정-예제)
6. [Ingress 디버깅 및 문제 해결](#Ingress-디버깅-및-문제-해결)
7. [Ingress 유용한 명령어 모음](#Ingress-유용한-명령어-모음)

---

## Ingress란 무엇인가?

**Ingress**는 Kubernetes에서 **클러스터 외부의 HTTP(S) 트래픽을 내부 서비스로 라우팅**하기 위한 API 리소스입니다. Service와 달리 Ingress는 도메인 기반 라우팅, HTTPS, 로드 밸런싱 등 고급 트래픽 관리 기능을 제공합니다.

- **기능**:
  - HTTP/HTTPS 라우팅
  - 도메인 기반의 트래픽 분산
  - TLS 인증서 관리

---

## Ingress의 주요 역할

1. **외부 트래픽 관리**:
   - 클러스터 외부에서 들어오는 HTTP/HTTPS 요청을 내부의 적절한 서비스로 전달

2. **도메인 기반 라우팅**:
   - 요청의 도메인 또는 경로를 기반으로 트래픽 라우팅

3. **로드 밸런싱**:
   - 동일한 역할을 하는 여러 서비스 또는 Pod 간 트래픽 분산

4. **TLS 관리**:
   - HTTPS 요청을 처리하고 TLS 인증서를 관리

---

## Ingress 구성 요소

1. **Ingress Resource**:
   - HTTP 및 HTTPS 트래픽 라우팅 규칙을 정의하는 Kubernetes 리소스

2. **Ingress Controller**:
   - Ingress 리소스를 처리하고 실제 트래픽을 라우팅하는 컨트롤러
   - 예: NGINX Ingress Controller, Traefik, HAProxy

3. **TLS/SSL 인증서**:
   - HTTPS 요청을 처리하기 위한 인증서

---

## Ingress Controller와 동작 원리

### Ingress Controller란?
**Ingress Controller**는 Ingress 리소스의 정의에 따라 트래픽을 적절히 라우팅하는 역할을 합니다. Kubernetes는 기본적으로 Ingress Controller를 제공하지 않으며, 사용자는 별도로 설치해야 합니다.

### 동작 원리
1. **Ingress 리소스 생성**:
   - 사용자가 Ingress 리소스를 생성하면 API 서버에 저장
2. **Ingress Controller 동작**:
   - Ingress Controller가 API 서버에서 Ingress 리소스를 감지
   - 정의된 규칙에 따라 로드 밸런서 또는 프록시 설정
3. **트래픽 라우팅**:
   - 외부 트래픽이 Ingress Controller를 통해 클러스터 내부의 서비스로 전달

---

## Ingress 설정 예제

### 1. 기본 Ingress 설정

아래 예제는 `example.com` 도메인의 요청을 `my-service` 서비스로 라우팅합니다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: example.com
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

- **`host`**: 특정 도메인 이름 요청 처리
- **`path`**: 요청 경로 지정
- **`backend`**: 트래픽이 전달될 서비스와 포트 지정

---

### 2. TLS 인증서를 포함한 Ingress 설정

아래 예제는 HTTPS를 처리하기 위해 TLS 인증서를 추가합니다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  tls:
    - hosts:
        - example.com
      secretName: tls-secret
  rules:
    - host: example.com
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

- **`tls`**:
  - `hosts`: TLS가 적용될 도메인
  - `secretName`: 인증서를 저장한 Secret 이름

---

## Ingress 디버깅 및 문제 해결

1. **Ingress 리소스 상태 확인**
   ```bash
   kubectl describe ingress <ingress-name>
   ```

2. **Ingress Controller Pod 상태 확인**
   ```bash
   kubectl get pods -n ingress-nginx
   ```

3. **Ingress Controller 로그 확인**
   ```bash
   kubectl logs <ingress-controller-pod-name> -n ingress-nginx
   ```

4. **DNS 이름 확인**
   - 도메인이 올바르게 설정되었는지 확인
   ```bash
   nslookup example.com
   ```

5. **트래픽 테스트**
   - Ingress를 통해 서비스가 응답하는지 테스트
   ```bash
   curl -H "Host: example.com" http://<external-ip>
   ```

---

## Ingress 유용한 명령어 모음

- **Ingress 목록 확인**
  ```bash
  kubectl get ingress
  ```

- **Ingress 세부 정보 확인**
  ```bash
  kubectl describe ingress <ingress-name>
  ```

- **Ingress Controller 로그 확인**
  ```bash
  kubectl logs <ingress-controller-pod-name> -n ingress-nginx
  ```

- **Ingress 삭제**
  ```bash
  kubectl delete ingress <ingress-name>
  ```

---

Ingress는 Kubernetes에서 외부 트래픽을 관리하고 내부 서비스로 라우팅하는 중요한 역할을 합니다. 도메인 기반 라우팅, TLS 지원, 로드 밸런싱 등 다양한 기능을 활용하여 클러스터의 네트워크 관리와 보안을 강화할 수 있습니다.
