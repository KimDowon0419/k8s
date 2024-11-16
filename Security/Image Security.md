# Image Security

## 목차
1. [Image Security란 무엇인가?](#Image-Security란-무엇인가)
2. [왜 Image Security가 중요한가?](#왜-Image-Security가-중요한가)
3. [Image Security를 위한 주요 접근법](#Image-Security를-위한-주요-접근법)
4. [이미지 서명과 검증](#이미지-서명과-검증)
5. [컨테이너 이미지 스캔](#컨테이너-이미지-스캔)
6. [Image Security 유용한 도구](#Image-Security-유용한-도구)
7. [이미지 보안 설정을 위한 Kubernetes 설정](#이미지-보안-설정을-위한-Kubernetes-설정)

---

## Image Security란 무엇인가?

**Image Security**는 **컨테이너 이미지를 안전하게 관리하고 악성 코드나 취약점이 포함되지 않도록 보장**하는 과정입니다. Kubernetes 및 컨테이너 환경에서는 애플리케이션과 그에 필요한 의존성이 **컨테이너 이미지**로 패키징되어 배포되기 때문에, 이미지의 보안이 매우 중요합니다.

- **기능**:
  - 이미지에서 악성 코드, 취약점, 의심스러운 라이브러리 등을 식별
  - 신뢰할 수 있는 이미지만 사용하도록 보장
  - 서명된 이미지를 통해 배포된 이미지를 인증

---

## 왜 Image Security가 중요한가?

1. **악성 코드 방지**:
   - 악성 코드를 포함한 이미지가 배포되면, 전체 클러스터와 시스템이 위험에 처할 수 있습니다. 이를 사전에 차단해야 합니다.
   
2. **취약점 관리**:
   - 오래된 의존성이나 취약점이 포함된 이미지는 보안 리스크를 초래합니다. 지속적으로 취약점을 모니터링하고 해결해야 합니다.
   
3. **리소스 보호**:
   - 이미지의 보안이 강화되면, 전체 클러스터와 애플리케이션에 대한 공격 벡터가 줄어들어 클러스터 리소스를 안전하게 보호할 수 있습니다.

4. **규정 준수**:
   - 법적 요구사항과 산업 표준을 충족하려면 사용되는 모든 이미지에 대해 보안 검사를 수행해야 합니다. 예를 들어, PCI-DSS, HIPAA 등에서 요구하는 보안 기준을 충족해야 합니다.

---

## Image Security를 위한 주요 접근법

1. **신뢰할 수 있는 소스에서만 이미지 가져오기**:
   - 공식 레지스트리 또는 신뢰할 수 있는 서드파티 레지스트리에서 이미지를 가져오는 것이 중요합니다.
   
2. **최소 권한 원칙 적용**:
   - 필요한 라이브러리와 의존성만 포함하여 이미지를 가볍고 안전하게 유지합니다. 불필요한 패키지나 권한을 포함하지 않도록 합니다.

3. **이미지 서명 및 검증**:
   - 이미지가 신뢰할 수 있는 출처에서 왔음을 증명하기 위해 서명을 적용하고 이를 검증합니다.
   
4. **이미지 스캔**:
   - 이미지를 배포하기 전에 취약점 스캐너를 사용하여 보안 취약점을 검토합니다.

---

## 이미지 서명과 검증

이미지 서명은 **이미지가 신뢰할 수 있는 소스에서 왔고, 수정되지 않았음을 증명**하는 중요한 방법입니다. 서명된 이미지는 인증된 출처에서 왔다는 것을 보장하며, 이후 배포나 실행 시 검증할 수 있습니다.

### 이미지 서명 예시

1. **`cosign` 도구 사용하여 이미지 서명**

   ```bash
   cosign sign --key cosign.key my-image:v1
   ```

   - **`cosign.key`**: 서명에 사용할 개인 키
   - **`my-image:v1`**: 서명할 이미지

2. **서명된 이미지 검증**

   ```bash
   cosign verify --key cosign.pub my-image:v1
   ```

   - **`cosign.pub`**: 공개 키를 사용하여 서명된 이미지를 검증

---

## 컨테이너 이미지 스캔

이미지 스캔은 **이미지 내 취약점이나 악성 코드를 검색**하는 과정입니다. 여러 도구를 사용하여 이미지를 스캔하고, 위험 요소를 식별할 수 있습니다.

### 이미지 스캔 도구

1. **Trivy**: Trivy는 **가벼운 이미지 스캐너**로, 보안 취약점, 악성 코드, 라이선스 문제 등을 검사합니다.

   ```bash
   trivy image my-image:v1
   ```

2. **Clair**: Clair는 **컨테이너 이미지의 취약점**을 스캔하고 분석하는 도구입니다. 이미지에 포함된 패키지의 취약점을 점검할 수 있습니다.

3. **Anchore**: Anchore는 **이미지의 취약점 스캐닝과 정책 기반 검사**를 지원하는 도구로, 보안 요구 사항을 자동으로 검증합니다.

---

## Image Security 유용한 도구

1. **Trivy**
   - 이미지 취약점과 악성 코드를 검사하는 도구로, 사용이 간편하고 빠릅니다.
   - 설치:
     ```bash
     brew install trivy
     ```
   - 이미지 스캔:
     ```bash
     trivy image my-image:v1
     ```

2. **Cosign**
   - 컨테이너 이미지를 서명하고 검증하는 도구로, 신뢰할 수 있는 이미지를 보장합니다.
   - 설치:
     ```bash
     brew install cosign
     ```
   - 이미지 서명:
     ```bash
     cosign sign --key cosign.key my-image:v1
     ```

3. **Clair**
   - 취약점 스캐닝 도구로, 이미지 내부의 패키지 취약점을 검사합니다.
   - 설치 및 구성은 비교적 복잡하지만, **CI/CD 파이프라인에서 쉽게 통합할 수 있습니다**.

4. **Anchore**
   - 이미지 취약점 및 정책 스캐닝을 지원하는 도구로, 사용자가 정의한 보안 정책을 바탕으로 이미지를 검사합니다.

---

## 이미지 보안 설정을 위한 Kubernetes 설정

1. **이미지Pull 정책 설정**:
   - Kubernetes에서는 **이미지에 대한 Pull 정책**을 설정하여, 최신 이미지를 강제로 가져오게 할 수 있습니다. `Always` 또는 `IfNotPresent`로 설정하여 이미지 최신 상태 유지가 가능합니다.

   ```yaml
   spec:
     containers:
       - name: my-container
         image: my-image:v1
         imagePullPolicy: Always
   ```

2. **PodSecurityPolicy**:
   - Kubernetes에서 **Pod의 보안을 강화**하기 위해 `PodSecurityPolicy`를 설정하여, 불필요한 권한 상승을 방지합니다.
   
   ```yaml
   apiVersion: policy/v1beta1
   kind: PodSecurityPolicy
   metadata:
     name: restricted-psp
   spec:
     privileged: false
     allowedCapabilities:
       - NET_ADMIN
     volumes:
       - configMap
       - secret
   ```

3. **자체 레지스트리 사용**:
   - 공용 레지스트리 대신 **자체 레지스트리**를 사용하여 인증된 이미지만 배포하도록 설정할 수 있습니다. `ImagePullSecrets`를 설정하여 Kubernetes가 특정 레지스트리에서 이미지를 가져오도록 합니다.

   ```yaml
   spec:
     containers:
       - name: my-container
         image: myregistry.com/my-image:v1
     imagePullSecrets:
       - name: my-registry-secret
   ```

---

이미지 보안은 Kubernetes에서 안전한 애플리케이션 배포를 위한 중요한 요소입니다. 이미지를 서명하고 검증하는 것과 취약점을 스캔하는 것은 보안을 강화하는 데 필수적인 과정입니다. 다양한 도구를 활용하여 이미지를 안전하게 관리하고, Kubernetes 클러스터에 적용할 수 있습니다.
